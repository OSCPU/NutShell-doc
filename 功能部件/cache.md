# Cache

NutShell 的数据 Cache 和指令 Cache 以及 L2 Cache 均采用一个可定制的 Cache 模块, 这个 Cache 模块的结构图如下所示. 
![](cache-module.svg)  

### Cache模块的结构
Cache 模块可以分为存储部分 (Cache Array) 和控制逻辑部分 (control logic). 其中存储部分又可以分为元数据 (Meta Array) 和数据 (Data Array). 控制逻辑部分被划分分为三级流水线, 对外会提供对 Memory Mapped Input/Output (MMIO) 空间、Memory 空间的访问接口. 



### 存储部分

指令 Cache 和数据 Cache 的存储的数据大小均为为 32KB. L2 Cache 则为 128KB. 映射方式采用的是四路组相联映射, Cache的替换策略为随机替换, 并采取写回的策略. 每个 Cache 行的大小为 64B. 
顺序核 Cache 目前采用实地址作为标签 (tag), 实地址作为索引 (index). 因此访问 Cache 的地址为经过 TLB (Translation Lookaside Buffer) 进行虚实地址转化后的 32 位实地址. 地址划分如下图所示：  
![](address-division.svg)  
元数据项包含了每个 Cache 组 (Cache Set) 的标签 (tag)、有效位 (valid)、脏位 (dirty) 以及路掩码 (waymask). 
<!-- Meta Array以及Data Array均采用了支持优先级多端口读写的存储模块.  -->



### 控制逻辑

Cache 的控制逻辑是一个三级流水线的结构. 阶段 1 (Stage 1) 接收来自 NutCore 或者其它 Cache 的访存请求, 这个访存请求的信息通过 SimpleBus 总线传递. 同时进行地址划分, 截取访存地址的索引并向 Cache Array 发送读请求, 读取这个索引对应的 Cache 组. 在下一拍返回数据. 阶段2(Stage 2) 得到上一级流水线传递过来的访存请求信息, 以及 Cache Array 返回的数据. 在这一阶段进行命中检查: 截取访存地址的标签, 并与该 Cache 组四路的 tag 进行比较. 命中则会生成 Cache hit 信号传递给阶段 3. 否则发送 Cache miss 信号, 并随机指定该组中的受害者 Cache 行 (Victim CacheLine) 以做 Cache 替换用. 阶段 3 (Stage 3) 根据阶段 2 的结果进行不同的处理：

* Cache hit: 读指令 (load) 直接返回数据, 写指令 (store) 发送对 Cache Array 的写请求 (hit write). 写的内容包括写入的数据 (通过写掩码选择的写数据和命中的原始数据进行拼接), 以及将该 Cache 行的脏位置 1 表示该Cache 行的数据被改写过.   

* Cache miss: 当缓存中不存在访存请求所需要的数据时, 就会触发 Cache miss. 此时 Cache 需要通过 Memory 访问接口向下一级存储设备发 Burst 访存请求 (SimpleBus 总线支持 Burst 请求), 读取该 miss 地址的一个 Cacheline 的数据填入受害者 Cache 行. 而如果受害者 Cache 行的数据被修改过, 则需要在读取数据之前将该 Cache 行的数据写入下一级存储. 重填时我们采用了关键字优先 (critical word first) 技术, 即首先发送的 Burst 请求地址是 miss 的访存请求需要操作的字地址. 这样做的好处是当第一个字从下一级存储返回的时候就可以根据访存请求对其进行读写操作, 然后提交给处理器, 而不需要等待整个 Cache 行都返回才进行操作和提交. 在重填阶段, 阶段3会向 Cache Array 发送写请求 (refill write), 将返回的 Cache 行以每次写一个字 (8 Byte) 的形式分 8 次写入受害者 Cache 行. 由于我们顺序核采用的是 blocking Cache, 因此当发生 Cache miss 的时候会阻塞整个控制逻辑流水线, 停止处理新的访存请求.   

* MMIO 请求: MMIO 请求来自于处理器 NutCore 对 MMIO 地址空间的访问, 主要用于访问 MMIO 外设的寄存器和 RAM. 这部分地址空间由于不是主存中的数据, 因此是不会在 Cache 中命中的. 阶段 3 收到 MMIO 请求后，会类似 Cache miss 一样阻塞流水线, 但是通过 MMIO 访问接口对这部分地址进行读写, 在得到 response 后提交该 MMIO 请求. 不同于使用 Memory 访问接口, MMIO 的访问不是 Burst 请求, 意味着它一次请求只会对应一个字. 



### L2 Cache预取器

NutCore 处理器在 L2 Cache 中加入了 Next-line 预取器. 当触发 L1 Cache miss 时, L1 Cache 会向 L2 Cache 发送 Burst 请求, L2 Cache 预取器每收到一个 Burst 请求都会在下一拍向 L2 Cache 发送 Prefetch 请求. Prefetch 请求和读请求在 Cache 中的控制逻辑基本相同，只是最后不会响应或返回数据.