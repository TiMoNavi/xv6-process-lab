# 调度路径与锁顺序（M1）

这份笔记对应闫保捷负责的 M1 内核梳理。当前实现仍是 xv6 原有的按进程表扫描策略，新增计数只观察边界，不改变调度选择结果。

## 调用链

```text
时钟中断
  -> kernelvec.S / uservec
  -> kerneltrap() 或 usertrap()
  -> devintr()
  -> clockintr()                 记录当前 RUNNING 进程的 run_ticks
  -> proc_account_preemption()   记录 timer-driven preemption
  -> yield()
  -> proc_set_runnable()         写入 runnable_since，状态 RUNNABLE
  -> sched()
  -> swtch(&p->context, &cpu->context)
  -> scheduler()
  -> 选中 RUNNABLE 进程
  -> 结算 wait_ticks，增加 dispatches，状态 RUNNING
  -> swtch(&cpu->context, &p->context)
```

阻塞路径为：

```text
sleep_prepare() -> sleep() -> proc_set_sleeping() -> SLEEPING -> sched() -> scheduler()
wakeup()/kkill() -> proc_set_runnable() -> 下一次 scheduler() 结算 wait_ticks
```

## 锁顺序

- 调度器选择进程时只持有该进程的 `p->lock`；结算计数不获取 `tickslock`，而是原子读取 `ticks`。
- `clockintr()` 先在 `p->lock` 下更新当前进程计数并释放它，再由 CPU 0 获取 `tickslock` 并执行 `wakeup(&ticks)`。
- `wakeup()` 遍历进程表并逐个获取 `p->lock`，因此不能在持有 `p->lock` 时等待 `tickslock`，否则会形成反向锁依赖。
- `wait_lock` 的既有规则保持不变：获取 `wait_lock` 前不能持有任何 `p->lock`；需要同时操作父子关系时按 xv6 既有顺序处理。

## 计数语义

- `run_ticks`：时钟中断到达且进程状态仍为 `RUNNING` 时加一。
- `wait_ticks`：进程从 `RUNNABLE` 被调度选中时，用全局 `ticks` 快照结算排队时长。
- `sleep_ticks`：进程从 `SLEEPING` 被唤醒或被 kill 时，用全局 `ticks` 快照结算阻塞时长。
- `dispatches`：调度器把进程从 `RUNNABLE` 置为 `RUNNING` 的次数。
- `preemptions`：定时器导致 `yield()` 前增加一次；当前 xv6 没有用户态主动让出 CPU 的接口。

计数不暴露内核指针，也不改变系统调用 ABI。暂时可通过控制台 `Ctrl-P` 查看；成员 A 后续可在此语义基础上设计稳定的用户态快照接口。

## 策略开关

`kernel/param.h` 定义 `SCHED_POLICY_RR` 和预留的 `SCHED_POLICY_MLFQ`；`Makefile` 的 `SCHED_POLICY` 变量默认是 `0`，因此可以用 `make SCHED_POLICY=0` 明确构建默认策略。`scheduler_candidate()` 是策略选择的单一入口；当前用 `make SCHED_POLICY=1` 尝试启用尚未实现的 MLFQ 时会明确编译失败。成员 B 在 M3 评审 MLFQ 参数后可替换该分支，同时保留 RR 作为回归对照。
