# 配套生态

在 OSCPU 项目组中, 我们还提供了一系列测试程序与操作系统, 包括：

* nexus-am
* FreeRTOS
* RT-Thread
* xv6
* Linux Kernel

下面将简要介绍一下如何在 NutShell 上测试与运行这些配套的生态软件, 部分内容在 `快速上手教程` 中已经提过, 这里不再赘述.

### nexus-am

AM的相关内容在 `快速上手教程` 里有所介绍. 我们在 AM 中提供了一个新的 riscv64-nutshell 抽象机器用于对 NutShell 处理器进行测试. 配置好 AM 的运行环境后, 在需要运行的 APP 或 Test 中执行 `make ARCH=riscv64-nutshell run` 即可.

### FreeRTOS

在 Demo/riscv64/ 目录下执行 `make nutshell`

### RT-Thread

参考工程 README

### xv6

在根目录下执行 `make nutshell`

### Linux Kernel

克隆 riscv-rootfs 项目, 根据应用的需求编译好 rootfsimg.

克隆 riscv-linux 项目, 配置使用 riscv 架构下的 `emu_defconfig`, 手动额外指定一下 rootfs 的位置为之前编译好的 initramfs, 然后进行编译

克隆 riscv-pk 项目, 根据 `快速上手教程` 里的步骤操作, 最后执行 `make nutshell`

