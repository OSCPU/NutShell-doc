# 总线

我们使用 SimpleBus 作为 NutCore 的访存总线，这是专门为本项目设计的一套简单总线.



## SimpleBusUC

SimpleBusUC 是 SimpleBus 的最基本实现，用于非 Cache 的访存通路中，它包含了 req 和 resp 两个通路，信号细节如下：



我们也内置了基于 SimpleBusUC 的各类 CrossBar，经过了一定的验证，方便开发者进行复用.



## SimpleBusC

SimpleBusC 在 SimpleBusUC 的基础上增加了与一致性相关的功能，用于 Cache 的访存通路中，本质上是由两个 SimpleBusUC 组合而成的：



## 与 AXI4 的转换

考虑到在 FPGA 验证和实际流片过程中，相关 IP 接口通常是标准化的总线协议（比如 AMBA 系列），我们无法直接让 SimpleBus 总线接入外设和内存，因此我们加入了 SimpleBus 到 AXI4 的转换部件，最终以 AXI4 协议的形式暴露给 SoC，详见 ”访存系统” 和 ”外设系统” 章节.