---
title: FTrace 源码学习
date: 2025-12-12 00:46:50 +0800
categories: [内核]
tags: [内核, Kernel, Trace]
---

## 0 ftrace_init

```c
// kernel/trace/ftrace.c
void __init ftrace_init(void)
{
	extern unsigned long __start_mcount_loc[]; // 某个 section 的起始地址
	extern unsigned long __stop_mcount_loc[]; // 该 section 的结束地址
	unsigned long count, flags;
	int ret;

	local_irq_save(flags);
	ret = ftrace_dyn_arch_init(); // 1 架构相关初始化
	local_irq_restore(flags);
	if (ret)
		goto failed;

	count = __stop_mcount_loc - __start_mcount_loc;
	if (!count) {
		pr_info("ftrace: No functions to be traced?\n");
		goto failed;
	}

	ret = ftrace_process_locs(NULL, // 2 处理所有 mcount 位置
				  __start_mcount_loc,
				  __stop_mcount_loc);
	if (ret) {
		pr_warn("ftrace: failed to allocate entries for functions\n");
		goto failed;
	}

	pr_info("ftrace: allocated %ld pages with %ld groups\n", // 统计信息输出：页数和分组数
		ftrace_number_of_pages, ftrace_number_of_groups);

	last_ftrace_enabled = ftrace_enabled = 1; // 启用 ftrace

	set_ftrace_early_filters(); // 初始化早期过滤，决定哪些函数入口需要被激活

	return;
 failed:
	ftrace_disabled = 1;
}
```

## 1 ftrace_dyn_arch_init

```c
// arch\powerpc\kernel\trace\ftrace.c
int __init ftrace_dyn_arch_init(void)
{
	unsigned int *tramp[] = { ftrace_tramp_text, ftrace_tramp_init };
	unsigned long addr = FTRACE_REGS_ADDR;
	long reladdr;
	int i;
	u32 stub_insns[] = { // trampoline 模板（未填写立即数的指令序列）
#ifdef CONFIG_PPC_KERNEL_PCREL
		/* pla r12,addr */
		PPC_PREFIX_MLS | __PPC_PRFX_R(1),
		PPC_INST_PADDI | ___PPC_RT(_R12),
		PPC_RAW_MTCTR(_R12),
		PPC_RAW_BCTR()
#elif defined(CONFIG_PPC64)
...
#else
...
#endif
	};

	if (IS_ENABLED(CONFIG_PPC_KERNEL_PCREL)) { // 补立即数
		for (i = 0; i < 2; i++) {
			reladdr = addr - (unsigned long)tramp[i];

			if (reladdr >= (long)SZ_8G || reladdr < -(long)SZ_8G) {
				pr_err("Address of %ps out of range of pcrel address.\n",
					(void *)addr);
				return -1;
			}

			memcpy(tramp[i], stub_insns, sizeof(stub_insns));
			tramp[i][0] |= IMM_H18(reladdr);
			tramp[i][1] |= IMM_L(reladdr);
			add_ftrace_tramp((unsigned long)tramp[i]); // 把该 trampoline 的地址注册到 ftrace 的 tramp 列表里，供后续 patch、管理
		}
	} else if (IS_ENABLED(CONFIG_PPC64)) {
...
	} else {
...
	}

	return 0;
}
```

## 2 ftrace_process_locs

1. 从 `[__start_mcount_loc, __stop_mcount_loc)` 遍历每个函数入口地址；
2. 为每个入口创建一个内部结构（如 `struct ftrace_page` / `struct dyn_ftrace`）；
3. 组织成链表或分组（按内存地址排序分 page/group）；

```c
// kernel\trace\ftrace.c
static int ftrace_process_locs(struct module *mod,
			       unsigned long *start,
			       unsigned long *end)
{
	struct ftrace_page *pg_unuse = NULL; // struct ftrace_page 是内部管理 struct dyn_ftrace 的页容器，可串链
	struct ftrace_page *start_pg;
	struct ftrace_page *pg;
	struct dyn_ftrace *rec;
	unsigned long skipped = 0;
	unsigned long count;
	unsigned long *p;
	unsigned long addr;
	unsigned long flags = 0; /* Shut up gcc */
	unsigned long pages;
	int ret = -ENOMEM;

	count = end - start;

	if (!count)
		return 0;

	pages = DIV_ROUND_UP(count, ENTRIES_PER_PAGE);

	/*
	 * Sorting mcount in vmlinux at build time depend on
	 * CONFIG_BUILDTIME_MCOUNT_SORT, while mcount loc in
	 * modules can not be sorted at build time.
	 */
	if (!IS_ENABLED(CONFIG_BUILDTIME_MCOUNT_SORT) || mod) {  // 保证地址升序排列
		sort(start, count, sizeof(*start),
		     ftrace_cmp_ips, NULL);
	} else {
		test_is_sorted(start, count);
	}

	start_pg = ftrace_allocate_pages(count); // 根据记录条目数给 ftrace 分配足够数量的 struct ftrace_page，每页里预留若干 dyn_ftrace
	if (!start_pg) // start_pg 是本次处理（内核/模块）所使用的 page 链表头
		return -ENOMEM;

	mutex_lock(&ftrace_lock);

	/*
	 * Core and each module needs their own pages, as
	 * modules will free them when they are removed.
	 * Force a new page to be allocated for modules.
	 */
	if (!mod) {
		WARN_ON(ftrace_pages || ftrace_pages_start);
		/* First initialization */
		ftrace_pages = ftrace_pages_start = start_pg;
	} else {
		if (!ftrace_pages)
			goto out;

		if (WARN_ON(ftrace_pages->next)) {
			/* Hmm, we have free pages? */
			while (ftrace_pages->next)
				ftrace_pages = ftrace_pages->next;
		}

		ftrace_pages->next = start_pg;
	}

	p = start;
	pg = start_pg;
	while (p < end) {
		unsigned long end_offset;

		addr = *p++;

		/*
		 * Some architecture linkers will pad between
		 * the different mcount_loc sections of different
		 * object files to satisfy alignments.
		 * Skip any NULL pointers.
		 */
		if (!addr) {
			skipped++;
			continue;
		}

		/*
		 * If this is core kernel, make sure the address is in core
		 * or inittext, as weak functions get zeroed and KASLR can
		 * move them to something other than zero. It just will not
		 * move it to an area where kernel text is.
		 */
		if (!mod && !(is_kernel_text(addr) || is_kernel_inittext(addr))) {
			skipped++;
			continue;
		}

		addr = ftrace_call_adjust(addr); // 将 raw mcount 地址转换为 ftrace 要保存的标准 IP

		end_offset = (pg->index+1) * sizeof(pg->records[0]);
		if (end_offset > PAGE_SIZE << pg->order) {
			/* We should have allocated enough */
			if (WARN_ON(!pg->next))
				break;
			pg = pg->next;
		}

		rec = &pg->records[pg->index++]; 
		rec->ip = addr; // 填充
	}

	if (pg->next) { // 截取多余 page
		pg_unuse = pg->next;
		pg->next = NULL;
	}

	/* Assign the last page to ftrace_pages */
	ftrace_pages = pg;

	/*
	 * We only need to disable interrupts on start up
	 * because we are modifying code that an interrupt
	 * may execute, and the modification is not atomic.
	 * But for modules, nothing runs the code we modify
	 * until we are finished with it, and there's no
	 * reason to cause large interrupt latencies while we do it.
	 */
	if (!mod)
		local_irq_save(flags);
	ftrace_update_code(mod, start_pg); // 对所有刚填充好的 dyn_ftrace 记录对应的地址进行代码更新（patch）
    // 比如把 call mcount 替换为 nop 或 jmp ftrace_caller；或插入/更新 ftrace trampoline 跳转
	if (!mod) 
		local_irq_restore(flags);
	ret = 0;
 out:
	mutex_unlock(&ftrace_lock);

	/* We should have used all pages unless we skipped some */
	if (pg_unuse) { // 处理未使用的多余 pages
		unsigned long pg_remaining, remaining = 0;
		unsigned long skip;

		/* Count the number of entries unused and compare it to skipped. */
		pg_remaining = (ENTRIES_PER_PAGE << pg->order) - pg->index;

		if (!WARN(skipped < pg_remaining, "Extra allocated pages for ftrace")) {

			skip = skipped - pg_remaining;

			for (pg = pg_unuse; pg; pg = pg->next)
				remaining += 1 << pg->order;

			pages -= remaining;

			skip = DIV_ROUND_UP(skip, ENTRIES_PER_PAGE);

			/*
			 * Check to see if the number of pages remaining would
			 * just fit the number of entries skipped.
			 */
			WARN(skip != remaining, "Extra allocated pages for ftrace: %lu with %lu skipped",
			     remaining, skipped);
		}
		/* Need to synchronize with ftrace_location_range() */
		synchronize_rcu();
		ftrace_free_pages(pg_unuse);
	}

	if (!mod) {
		count -= skipped;
		pr_info("ftrace: allocating %ld entries in %ld pages\n",
			count, pages);
	}

	return ret;
}
```

## 3  ftrace_make_call / ftrace_make_nop

```c
// arch\powerpc\kernel\trace\ftrace.c
// ftrace patch
// 把某个函数入口“接入”到 ftrace（从 nop → branch/call）
int ftrace_make_call(struct dyn_ftrace *rec, unsigned long addr)
{
	ppc_inst_t old, new;
	unsigned long ip = rec->ip;
	int ret = 0;

	/* This can only ever be called during module load */
	if (WARN_ON(!IS_ENABLED(CONFIG_MODULES) || core_kernel_text(ip)))
		return -EINVAL;

	old = ppc_inst(PPC_RAW_NOP());
	if (IS_ENABLED(CONFIG_PPC_FTRACE_OUT_OF_LINE)) {
		ip = ftrace_get_ool_stub(rec) + MCOUNT_INSN_SIZE; /* second instruction in stub */
		ret = ftrace_get_call_inst(rec, (unsigned long)ftrace_caller, &old);
	}

	ret |= ftrace_get_call_inst(rec, addr, &new);

	if (!ret)
		ret = ftrace_modify_code(ip, old, new); 

	ret = ftrace_rec_update_ops(rec);
	if (ret)
		return ret;

	if (!ret && IS_ENABLED(CONFIG_PPC_FTRACE_OUT_OF_LINE))
		ret = ftrace_modify_code(rec->ip, ppc_inst(PPC_RAW_NOP()), // 从 nop → 跳到 stub
			 ppc_inst(PPC_RAW_BRANCH((long)ftrace_get_ool_stub(rec) - (long)rec->ip)));

	return ret;
}

int ftrace_make_nop(struct module *mod, struct dyn_ftrace *rec, unsigned long addr)
{
	/*
	 * This should never be called since we override ftrace_replace_code(),
	 * as well as ftrace_init_nop()
	 */
	WARN_ON(1);
	return -EINVAL;
}
```

##  4 ftrace_caller

`ftrace_caller` 是 所有被跟踪函数的统一汇编入口，负责把“某个函数被调用”这件事转交给已注册的所有 `ftrace_ops`（每个 tracer 代表一个 ops）。

1）入口如何跳到 `ftrace_caller`

- 编译时在每个函数入口插桩（`mcount` / `_mcount`）。
- 启动阶段通过 `ftrace_process_locs()` 把这些位置组织成 `dyn_ftrace` 记录，并在 `ftrace_update_code()` 中对对应指令做 patch：
  - 要启用时：把原来的 `nop` / `mcount` 改成“跳转到 ftrace trampoline / ftrace_caller”。
  - 要禁用时：反向 patch 回 `nop`。
- 对于有 out-of-line stub 的架构，入口先跳到每条记录的 trampoline，再由 trampoline 跳到 `ftrace_caller`。

2）`ftrace_caller` 做的事

在汇编里大致是：

- 保存当前寄存器现场（为各个 tracer 回调做准备）；
- 从汇编 stub 里取出：
  - 当前函数的 `ip`（被跟踪函数地址）；
  - `parent_ip`（调用者地址）；
  - 指向 `struct dyn_ftrace` 或 trampoline 的指针；
- 切到 C 代码/包装逻辑，进入真正的分发环节。

3）分发给匹配的 `ftrace_ops`

C 侧的逻辑大致是：

- 通过 RCU 遍历全局的 `ftrace_ops_list`；

- 对每个 `ops`，用它的 `func_hash`（也就是你在第 5 节会详细讲的 filter/notrace 过滤结构）判断当前 `ip` 是否应该被此 `ops` 处理；

- 若匹配，则调用该 `ops` 的 `ops_func`：

  ```
  ops->ops_func(ip, parent_ip, ops, regs);
  ```

  `ops_func` 是为这个 `ops` 合成的“包装函数”，内部再调用真正的 `ops->func`，并根据 `ops->flags` / `ops->private` 等信息决定是否递归过滤、是否跳过某些场景等。

4）回到原函数继续执行

所有匹配的 `ftrace_ops` 处理完之后，`ftrace_caller` 恢复寄存器现场，并跳回被跟踪函数（或其下一条指令）继续执行。
 对于 direct call 或 BPF trampoline 这类变体，整体控制流类似，只是“从入口到 tracer”走的是各自的特制 trampoline，但最终都是在这个阶段完成“回调 → 恢复 → 继续执行”。

## 5 ftrace_ops 与注册 / 调度流程 

`struct ftrace_ops` 可以看成是 ftrace 框架的“插件描述符”：每一个 tracer、每一种 hook，都通过一个 `ftrace_ops` 接到 ftrace 核心上，然后由上一节的 `ftrace_caller` 统一分发。

1） `struct ftrace_ops` 结构与核心字段

```c
// include\linux\ftrace.h
struct ftrace_ops {
	ftrace_func_t			func; // 主回调，所有 tracer 的逻辑入口，在 ftrace_caller 中被调用
	struct ftrace_ops __rcu		*next;
	unsigned long			flags; // 控制该 ops 的行为
	void				*private;
	ftrace_func_t			saved_func; // 保存原始 func，用于临时替换 / 禁用时恢复
#ifdef CONFIG_DYNAMIC_FTRACE
	struct ftrace_ops_hash		local_hash; // ftrace_ops_hash 包含 filter_hash / notrace_hash
	struct ftrace_ops_hash		*func_hash;
	struct ftrace_ops_hash		old_hash;
	unsigned long			trampoline; // 专门为这个 ops 分配的一小段代码段地址
	unsigned long			trampoline_size;
	struct list_head		list;
	struct list_head		subop_list;
	ftrace_ops_func_t		ops_func; // 真正被 ftrace_caller 调用的函数指针。裸回调；包装回调
	struct ftrace_ops		*managed;
#ifdef CONFIG_DYNAMIC_FTRACE_WITH_DIRECT_CALLS
	unsigned long			direct_call; // 直接通过目标函数地址调用，而不是统一走 ftrace_caller 分发
#endif
#endif
};
```

2）注册：从 tracer 到 ftrace 核心

一个 tracer 要接入 ftrace 的典型步骤：

1. 定义并初始化一个（或多个）`static struct ftrace_ops`：
   - 设置自己的回调 `func`；
   - 配好 `flags`（是否需要 regs、是否允许递归等）；
   - 按需初始化 `local_hash` 或通过接口设置 filter/notrace。
2. 调用 `register_ftrace_function(&ops)`：
   - 把 `ops` 加入全局 `ftrace_ops_list`（用 `next` + RCU 指针维护）；
   - 初始化 `local_hash`，并让 `func_hash` 指向它；
   - 将 `ops` 挂到内部管理链表（`list` / `subop_list` / `managed`），方便统一启停和卸载；
   - 如需要，为此 `ops` 分配独立 trampoline，生成对应的 `ops_func`（包装函数），有的情况下也会填充 `direct_call`；
   - 必要时调用 `ftrace_startup()`，触发对所有相关 `dyn_ftrace` 的 patch（NOP → branch）。

整体上，注册阶段只是在“数据结构这边”把 ops 挂好，并根据 ops 的需要决定 patch 策略。

3）调度：`ftrace_caller` 如何用 `ftrace_ops`

当某个函数入口跳进 `ftrace_caller` 后，调度流程可以简化成：

```c
// 伪代码示意，只保留关键逻辑
ftrace_caller(ip, parent_ip, regs):
    rcu_read_lock();
    for (ops = ftrace_ops_list; ops; ops = rcu_dereference(ops->next)) {

        // 通过 func_hash 判断当前 ip 是否在该 ops 的 filter/notrace 范围内
        if (!match_ip_in_ops_hash(ops->func_hash, ip))
            continue;

        // 调用为该 ops 合成的包装回调
        ops->ops_func(ip, parent_ip, ops, regs);
    }
    rcu_read_unlock();
```

- `func_hash` 决定“这个 ops 对哪些函数入口感兴趣”，由 tracer 通过 tracefs 的 filter/notrace 接口或直接调用 API 来维护。
- `ops_func` 再内部调用 `ops->func`，并处理：递归保护、flags 判断、是否需要传入完整 `pt_regs`、是否需要对返回路径做额外处理（如 function_graph）等。