# CSR 单元

CSR 单元主要负责特权级指令的执行以及例外/中断的处理. 在 NutShell 中, CSR 单元是流水线中的一个功能部件. 

> 在 NutShell 中, "CSR 寄存器" 和 "CSR 单元" 是不同的概念. "CSR 寄存器" 是指 RISC-V 手册中规定的控制与状态寄存器. "CSR 单元" 则是指执行级流水线中负责处理特权指令、中断、例外的功能单元.

RISC-V 特权级是比较复杂的, 涉及到很多寄存器的读写与控制, 为了简化设计难度、提高代码的可读性, 我们构造了一个特殊的 MaskedRegMap 结构来优化特权级寄存器的读写, 并围绕这一设计给出相应的特权级控制逻辑. 针对每一个 CSR 寄存器构造 MaskedRegMap, 包括了以下这几个经过凝练的通用读写控制域:

* CSR 寄存器索引
* CSR 寄存器存储位置
* 读、写掩码
* 写副作用 (side effect) 函数

每当接受到读写请求时均会按照设定做出相应的修饰再最终读写物理寄存器. 具体实现细节请参考源码中的 RegMap.scala

在 CSR 单元中, 我们大量地使用了 BoringUtils 这一 Chisel 内置类来进行飞线, 这是因为很多其他的功能部件需要从 CSR 中获取到当前系统状态来实现相应功能 (比如 LSU, TLB 等).

NutShell 默认支持以下的 CSR 寄存器. 要调整可使用的 CSR 以及设置 CSR 的初始值, 参见源码中的 CSR.scala:

```
# U 模式
1. Ustatus    2. Uie        3. Utvec     4. Uscratch
5. Uepc       6. Ucause     7. Utval     8. Uip
# S 模式
1. Sstatus    2. Sedeleg    3. Sideleg   4. Sie
5. Stvec      6. Scounteren 7. Sscratch  8. Sepc
9. Scause     10. Stval     11. Sip      12. Satp
# M 模式
1. Mvendorid  2. Marchid    3. Mimpid    4. Mhartid
5. Mstatus    6. Mstatus    7. Misa      8. Medeleg 
9. Mideleg    10. Mie       11. Mtvec    12. Mcounteren
13. Mscratch  14. Mepc      15. Mcause   16. Mtval 
29. Mip
```

除访存例外之外的所有例外/中断均会在译码阶段被附加在指令上, 这样的指令随后会被发送给 CSR 单元执行.

访存例外的处理相对特殊. 在一条访存指令实际执行前, 我们无法判断这条访存指令是否会导致例外. 因此, 在访存指令发生例外时, 访存单元会将产生例外的指令以及例外信息转发到 CSR 单元, 由 CSR 单元控制 CSR 寄存器的变化以及指令的写回.