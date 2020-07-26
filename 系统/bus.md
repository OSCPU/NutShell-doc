# 总线

我们使用 SimpleBus 作为 NutCore 的访存总线, 它的设计借鉴了 AMBA, TileLink 等总线的思想, 根据我们的需求加入了一定的功能, 是专门为本项目设计的一套最小满足功能需求的总线.

目前, SimpleBus 有两个实现层级, 分别是 SimpleBusUC 和 SimpleBusC.

## SimpleBusUC

SimpleBusUC 是 SimpleBus 的最基本实现, 用于非 Cache 的访存通路中, 它包含了 req 和 resp 两个通路, 使用 Decoupled 方式握手, 信号细节如下：

| req 信号名称 | 位宽       | 注释                                                  |
| :----------- | ---------- | ----------------------------------------------------- |
| req.addr     | AddrBits   | 访存地址（位宽与体系结构实现相关）                    |
| req.size     | 3          | 访存大小（访存Byte = 2^(req.size)）                   |
| req.cmd      | 4          | 访存指令, 详见 SimpleBus.scala 中 SimpleBusCmd 的实现 |
| req.wdata    | DataBits   | 内存写数据（位宽与体系结构实现相关）                  |
| req.wmask    | DataBits/8 | 内存写掩码                                            |
| req.user     | UserBits   | 用户自定义数据, 在访存过程中不被修改                  |
| req.id       | IdBits     | 标识访存请求的来源, 在访存过程中不被修改               |



| resp 信号名称 | 位宽     | 注释                                 |
| ------------- | -------- | ------------------------------------ |
| resp.cmd      | 4        | 访存状态回复                         |
| resp.rdata    | DataBits | 访存读数据                           |
| resp.user     | UserBits | 用户自定义数据, 在访存过程中不被修改 |
| resp.id       | IdBits   | 标识访存请求的来源, 在访存过程中不被修改 |

SimpleBus的id通道在NutShell顺序核的设计中并没有被使用.

我们也内置了基于 SimpleBusUC 的各类 CrossBar, 经过了一定的验证, 方便开发者进行复用.



## SimpleBusC

SimpleBusC 在 SimpleBusUC 的基础上增加了与一致性相关的功能, 用于 Cache 的访存通路中, 本质上是由两个 SimpleBusUC 组合而成的：

```
class SimpleBusC(val userBits: Int = 0) extends SimpleBusBundle {
  val mem = new SimpleBusUC(userBits)
  val coh = Flipped(new SimpleBusUC(userBits))
}
```

其中 mem 是访存通道, coh 是一致性维护通道.



## 与 AXI4 的转换

考虑到在 FPGA 验证和实际流片过程中, 相关 IP 接口通常是标准化的总线协议（比如 AMBA 系列）, 我们无法直接让 SimpleBus 总线接入外设和内存, 因此我们加入了 SimpleBus 到 AXI4 的转换部件, 最终以 AXI4 协议的形式暴露给 SoC, 详见[访存系统](./mem.md)和[外设系统](./peripheral.md)章节.