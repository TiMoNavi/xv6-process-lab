# xv6 Process Lab

这是一个以 MIT PDOS 官方 `xv6-riscv` 为基线的五人协作工程。项目主题是：

> **为 xv6 增加可观测的进程调度实验平台，并实现可比较的多级反馈队列（MLFQ）调度器。**

第一版仓库只保留上游 xv6 代码和立项资料，确保所有后续功能都从可复现的原始系统开始。

## 先读这些

- [项目立项、选题、分工和路线图](docs/PROJECT_PLAN.md)
- [协作与提交约定](CONTRIBUTING.md)
- [上游 xv6 说明](README)
- [MIT xv6 许可证](LICENSE)

## 基线

- 上游项目：<https://github.com/mit-pdos/xv6-riscv>
- 分支：`riscv`
- 基线提交：`9e3161a9abf5f51ea402562d1874caf6c4926597`
- 获取基线：

  ```sh
  git clone --branch riscv --single-branch https://github.com/mit-pdos/xv6-riscv.git
  ```

## 当前状态

当前提交是工程起点：尚未改变内核行为。后续实现必须先补测试或实验脚本，再修改内核，并在 pull request 中记录测量结果和已知限制。

## 目标结果

完成后，用户可以在 QEMU 中创建不同 CPU/IO 行为的进程，观察进程状态、运行时间、等待时间、抢占次数和队列变化，并在保持默认调度器可用的前提下切换到 MLFQ 进行可重复对比。
