# 本机环境记录

本文件记录项目维护者当前用于构建和启动 xv6 的环境。工具安装在用户目录，不依赖管理员权限，也不要求改动系统 `/usr/local`。

## 已验证环境

- 主机：Intel `x86_64`，macOS `15.7.7`
- RISC-V GCC/binutils：xPack GNU RISC-V Embedded GCC `15.2.0-1`
- QEMU：xPack QEMU RISC-V `9.2.4`
- GNU Make：`3.81`
- Python：`3.9.6`
- Perl：系统 `/usr/bin/perl`
- `bc`：系统 `/usr/bin/bc`

安装目录：

```text
~/.local/toolchains/xpack-riscv-none-elf-gcc-15.2.0-1/
~/.local/toolchains/xpack-qemu-riscv-9.2.4-1/
```

`~/.local/bin` 已由现有 `~/.zshrc` 配置加入 `PATH`。其中有一层用户级兼容包装：

- `riscv64-none-elf-gcc` 固定补充 `-mabi=lp64`；xPack 编译器默认 ABI 是 `ilp32`。
- `riscv64-none-elf-ld` 固定选择 `elf64lriscv`；否则 xPack `ld` 可能默认选择 32 位仿真目标。
- `qemu-system-riscv64` 在 `--version` 时输出上游 Makefile 可解析的标准版本格式，其余参数原样转发。

这些包装只影响本机命令解析，不改变 xv6 源码或编译参数。

## 常用验证

在仓库根目录执行：

```sh
command -v riscv64-none-elf-gcc
command -v qemu-system-riscv64
make clean
make
make fs.img
python3 test-xv6.py -q usertests
```

交互启动：

```sh
make qemu
```

在 xv6 shell 中按 `Ctrl-a`，再按 `x` 退出 QEMU。

## 重新下载来源

- GCC：<https://github.com/xpack-dev-tools/riscv-none-elf-gcc-xpack/releases/tag/v15.2.0-1>
- QEMU：<https://github.com/xpack-dev-tools/qemu-riscv-xpack/releases/tag/v9.2.4-1>

下载时应核对对应 release asset 的 SHA-256；不要从聊天记录或未验证的镜像复制可执行文件。
