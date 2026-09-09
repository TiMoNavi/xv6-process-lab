# 协作约定

## 分支

- `main` 只接受已经评审并通过基线测试的提交。
- 每项工作使用 `feature/<短名>` 分支；不要直接在 `main` 上开发。
- 一个提交只解决一个可说明的问题，提交信息使用 `area: action` 格式，例如 `kernel: add process runtime counters`。

## Pull request 门槛

提交 PR 前必须：

1. 说明修改了哪些文件、为什么这样改，以及未解决的限制。
2. 运行 `make qemu` 的启动检查和 `make grade`（如果本地课程测试脚本可用）。
3. 运行与改动相关的用户态回归测试；调度器改动还要附上至少一组相同负载下的对比数据。
4. 标明是否改变系统调用、用户 ABI、锁顺序或磁盘镜像格式。

## 合并顺序

先合入测试和观测工具，再合入最小内核改动，最后合入策略切换和性能实验。遇到冲突时，以 `main` 的最新代码为准重新 rebase，并重新运行测试。

## 代码约束

- 保持 xv6 现有 C 风格和锁粒度；不为了重构而重写无关代码。
- 新增系统调用时同步修改 `kernel/syscall.h`、`kernel/syscall.c`、`kernel/sysproc.c`、`user/user.h`、`user/usys.pl` 及测试。
- 新增用户程序时同步更新 `Makefile` 的 `UPROGS`。
- 文档中的测量结果必须包含 QEMU 配置、负载、运行次数和原始输出位置。
