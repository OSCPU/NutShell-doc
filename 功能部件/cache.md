# Cache

## Cache模块
NutShell的数据Cache和指令Cache以及L2Cache均采用一个可定制的Cache模块，这个Cache模块的结构图如下所示。
![](cache-module.svg)  

----

### 1. Cache模块的结构
Cache模块可以分为存储部分(Cache Array)和控制逻辑部分(control logic)。其中存储部分又可以分为元数据(Meta Array)和数据(Data Array)。控制逻辑部分被划分分为三级流水线，对外会提供对Memory Mapped Input/Output(MMIO)空间、Memory空间的访问接口。

----

### 2. 存储部分
指令Cache和数据Cache的存储的数据大小均为为32KB。L2Cache则为128KB。映射方式采用的是四路组相联映射，Cache的替换策略为随机替换，并采取写回的策略。每个Cache行的大小为64B。
顺序核Cache目前采用实地址作为标签(tag)，实地址作为索引(index)。因此访问Cache的地址为经过TLB(Translation Lookaside Buffer)进行虚实地址转化后的32位实地址。地址划分如下图所示：  
![](address-division.svg)  
元数据项包含了每个Cache组(Cache Set)的标签(tag)、有效位(valid)、脏位(dirty)以及路掩码(waymask)。
<!-- Meta Array以及Data Array均采用了支持优先级多端口读写的存储模块。 -->

----

### 3. 控制逻辑
<p style="text-indent:2em">
Cache的控制逻辑是一个三级流水线的结构。阶段1(Stage 1)接收来自NutCore或者其它Cache的访存请求，这个访存请求的信息通过SimpleBus总线传递。同时进行地址划分，截取访存地址的索引并向Cache Array的读请求，读取这个索引对应的Cache组。在下一拍返回数据。阶段2(Stage 2)得到上一级流水线传递过来的访存请求信息，以及Cache Array返回的数据。在这一阶段进行命中检查：截取访存地址的标签，并与该Cache组四路的tag进行比较。命中则会生成Cache hit信号传递给阶段3。否则发送Cache miss信号，并随机指定该组中的受害者Cache行(victim cacheline)以做Cache替换用。阶段3(Stage 3)根据阶段2的结果进行不同的处理：  
 
* Cache hit:读指令(load)直接返回数据，写指令(store)发送对Cache Array的写请求(hit write)。写的内容包括写入的数据(通过写掩码选择的写数据和命中的原始数据进行拼接)，以及将该Cache行的脏位置1表示该Cache行的数据被改写过。  

* Cache miss:当缓存中不存在访存请求所需要的数据时，就会触发Cache miss。此时Cache需要通过Memory访问接口向下一级存储设备发Burst访存请求(SimpleBus总线也会支持Burst请求)，读取该miss地址的一个Cacheline的数据填入受害者Cache行。而如果受害者Cache行的数据被修改过，则需要在读取数据之前将该Cache行的数据写入下一级存储。重填时我们采用了关键字优先(critical word first)技术，即首先发送的Burst请求地址是miss的访存请求需要操作的字地址，这样做的好处是，当第一个字从下一级存储返回的时候，就可以根据访存请求对其进行读写操作，然后提交给处理器。而不需要等待整个Cache行都返回才进行操作和提交。在重填阶段，阶段3会向Cache Array发送写请求(refill write)，将返回的Cache行以每次写一个字(8 Byte)的形式分8次写入受害者Cache行。由于我们顺序核采用的是blocking Cache，因此当发生Cache miss的时候会阻塞整个控制逻辑流水线，停止处理新的访存请求。  

* MMIO 请求：MMIO请求来自于处理器NutCore对MMIO地址空间的访问，主要用于访问MMIO外设的寄存器、RAM。这部分地址空间由于不是主存中的数据，因此是不会在Cache中命中的。阶段3对于MMIO请求和miss一样会阻塞流水线，但是会通过MMIO访问接口对这部分地址进行读写，在得到response后提交该MMIO请求。不同于使用Memory访问接口，MMIO的访问不是Burst请求，意味着它一次请求只会对应一个字。
</p>
----

### 4. Cache一致性

TODO

### 5. L2Cache预取器

TODO