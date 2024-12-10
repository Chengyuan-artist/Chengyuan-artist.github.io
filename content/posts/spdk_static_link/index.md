---
title: "SPDK应用静态链接"
date: 2024-12-09T16:48:38+08:00
lastmod: 2024-12-09T19:07:51+08:00
draft: false
description: "如何完全静态链接一个SPDK应用"

tags: []
categories: []

image: 
toc: true
math:
license:
---

<!--more-->

在编译SPDK应用程序时，有时需要使用静态链接SPDK库的版本。

有时甚至需要**完全**静态链接（包括静态链接系统库，libc等）的版本。

> 用于特定TEE环境，无动态库甚至无完整操作系统支持

去网上查询，没有直接SPDK静态链接的解决办法。探索了一番后，把我的做法总结上来。

**注意**：本文的解决方案在于**完全静态链接**SPDK程序（包括**libc**）, 如果你寻找的是仅静态链接SPDK库的方法，请参考[Linking SPDK applications with pkg-config](https://spdk.io/doc/pkgconfig.html)。



## TL;DR

以SPDK自带的`spdk_nvme_perf`应用为例。

首先按照正常流程编译DPDK和SPDK。

> 对于交叉编译（以RISC-V为例），先交叉[编译DPDK](https://doc.dpdk.org/guides/linux_gsg/cross_build_dpdk_for_riscv.html)并安装到路径`/path/to/dpdk/bin`，随后如下编译SPDK:
>
> 进入SPDK根目录：
>
> ```shell
> export CC=riscv64-linux-gnu-gcc
> export CXX=riscv64-linux-gnu-g++
> ./configure --target-arch=rv64gc --cross-prefix=riscv64-linux-gnu --with-dpdk=/path/to/dpdk/bin
> make -j($nproc)
> ```

进入`SPDK_ROOT/app/spdk_nvme_perf`

编辑`Makefile` 在尾部添加

```makefile
PKG_CONFIG_PATH = $(SPDK_ROOT_DIR)/build/lib/pkgconfig
SPDK_LIB := $(shell PKG_CONFIG_PATH="$(PKG_CONFIG_PATH)" pkg-config --libs spdk_nvme spdk_env_dpdk spdk_vmd)
SYS_LIB := $(shell PKG_CONFIG_PATH="$(PKG_CONFIG_PATH)" pkg-config --libs --static spdk_syslibs)

static:
	$(CC) -o $(SPDK_ROOT_DIR)/build/bin/perf-static perf.o \
	-static -static-libgcc \
	-Wl,--whole-archive $(SPDK_LIB) \
	-Wl,--no-whole-archive $(SYS_LIB)
```

执行

```bash
make static
```

 即可获得位于`$(SPDK_ROOT_DIR)/build/bin/perf-static`的静态链接可执行文件。



## 解析

### `pkg-config`

`pkg-config`是一个编译辅助工具，用于确定第三方库的位置以获取正确的链接位置和编译命令。需要配合环境变量`PKG_CONFIG_PATH`使用以指向第三方库提供的`*.pc`文件。

`PKG_CONFIG_PATH = $(SPDK_ROOT_DIR)/build/lib/pkgconfig`即SPDK库的pakage config所在位置。

`pkg-config --libs spdk_nvme spdk_env_dpdk spdk_vmd`命令给出了`spdk_nvme`,`spdk_env_dpdk`和`spdk_vmd`三个库的链接命令，这些库的源文件位于`$(SPDK_ROOT_DIR)/lib`。

> 命令`PKG_CONFIG_PATH = $(SPDK_ROOT_DIR)/build/lib/pkgconfig pkg-config --libs spdk_nvme`示例输出:
>
> ```
> -L/path/to/build/lib -lspdk_nvme -lspdk_sock -lspdk_sock_posix -lspdk_trace -lspdk_rpc -lspdk_jsonrpc -lspdk_json -lspdk_util -lspdk_vfio_user -lspdk_log
> ```

` pkg-config --libs --static spdk_syslibs`命令给出了SPDK库所需链接的系统库。

### 编译选项

`-static` 静态链接全局选项，所有库会进行静态链接。如果只想对部分库进行静态链接，可以使用`-Wl,-Bstatic`和`-Wl,-Bdynamic` 包裹要进行静态链接的库链接命令。

`-static-libgcc` 静态链接libgcc

`-Wl, --whole-archive` 给链接器传递的指令，强制将后面指定的静态库的**所有对象文件**都包含到最终的可执行文件中（包括未被引用的符号）

`-Wl,--no-whole-archive` 禁用 `--whole-archive` 的行为



我尝试了许多不同命令组合，最终上文那种是最小的有效命令。



**碎碎念**

本文是今年7月份帮师兄做实验的时候就打算写的一篇小trick, 拖到现在才写，坏处就是回顾了半天才想起来当时的做法。下次还是要更勤快些，现解决，现分享 :p

## 参考

1. [Linking SPDK applications with pkg-config](https://spdk.io/doc/pkgconfig.html)
2. [SPDK Libraries: #SPDK Static Objects](https://spdk.io/doc/libraries.html)
3. [compile statically linked dpdk apps](https://gist.github.com/krsna1729/10900eaabeb648ddbf4642eb2c053624)

