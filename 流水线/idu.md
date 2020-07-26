# 译码级

译码 (Decode) 阶段接收指令对齐缓冲给出的指令进行译码. 译码后的结果暂存到译码结果缓冲区中, 等待进入处理器后端执行部分.

译码部分主要使用了模式匹配方法, 支持 I、M、A、C、Zicsr、Zifencei 等扩展, 最终获得规则化的译码信息如下：

```
class CtrlSignalIO extends NutCoreBundle {
  val src1Type = Output(SrcType())
  val src2Type = Output(SrcType())
  val fuType = Output(FuType())
  val fuOpType = Output(FuOpType())
  val rfSrc1 = Output(UInt(5.W))
  val rfSrc2 = Output(UInt(5.W))
  val rfWen = Output(Bool())
  val rfDest = Output(UInt(5.W))
  val isNutCoreTrap = Output(Bool())
  val isSrc1Forward = Output(Bool())
  val isSrc2Forward = Output(Bool())
  val noSpecExec = Output(Bool())  // This inst can not be speculated
  val isBlocked = Output(Bool())   // This inst requires pipeline to be blocked
}

class DataSrcIO extends NutCoreBundle {
  val src1 = Output(UInt(XLEN.W))
  val src2 = Output(UInt(XLEN.W))
  val imm  = Output(UInt(XLEN.W))
}
```

目前译码部分没有使用压缩指令展开逻辑将压缩指令展开成普通指令. 压缩指令与普通指令一样被直接译码.