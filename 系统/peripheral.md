# 外设系统

本节主要介绍 NutShell 所使用的外设系统. 由于实现的原因, 我们在仿真和流片版本使用了不同的外设系统, 其中仿真外设示意图如下：

![](mmio.svg)



流片外设示意图如下：

![](peripheral-real.svg)



### NutShell SoC

NutShell SoC 主要由 NutCore 处理器核, 各个外设 IP 核以及 SDRAM 控制器组成. NutCore 提供了对外的 AXI-MEM 接口 (AXI4 协议), 与 SDRAM 控制器连接. 而 MMIO 地址空间访问则是通过 AXI-MMIO (AXI-Lite协议) 接口, 首先信号会被转换为 APB 协议总线信号. APB 信号选择器会根据访问的地址空间将 NutShell 的访问信号 (master) 传递给不同的外设控制器 (slave), 达到对 MMIO 空间进行访问的目的.   

各个外设 IP 会将 APB 信号转换为内部控制器的信号, 来进行对外设 IP 内部寄存器的读写. 而 ETHMAC 以及 SDC 两种外设还需要实现对主存进行读写, 因此这两个外设的信号也同时能作为 APB master, 转换为 AXI-frontend (AXI4协议), 接入到 NutCore. 这个信号在 NutCore 内部会转换为一个对 Data Cache 的 SimpleBus 访存请求. 



### 外设

我们的外设 IP 主要来源于以 OpenCore 为代表的开源网站. 以下是对各个外设的详细介绍: 

* UART (Universal Asynchronous Receiver/Transmitter), 用于NutCore对串口的输入输出. 
* GPIO (General-Purpose Input/Output), 我们使用 4 个 GPIO 输入端口作为我们的中断输入端口, 连接到各个外设 IP 的中断信号. 当外设产生中断时, 会使能 GPIO 中断并向 NutCore 处理器发送中断. 
* SPI_Flash, 我们使用 SPI (Serial Peripheral Interface) 信号端口的 Flash 来作为我们的 ROM, NutCore 会从 SPI_Flash 中读取第一条指令. 
* ETHMAC, 以太网控制器, SoC 通过 Tx 端发送以太网帧. 而当 Rx 接收到数据时, 会作为 APB master 向 NutCore 发送 AXI-frontend 写请求写入 DMA 接收地址. 
* SDC (Secure Digital Card), 已经流片的板子上我们实现了 SDHC (Secure Digital High Capacity) 以及 SPI 两种接口的 SDC controller
