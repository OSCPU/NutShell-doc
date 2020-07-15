# NutShell 文档

目前该 repo 已经和 gitbook.io 绑定，域名为 https://oscpu.gitbook.io/nutshell/ ，大家做出修改后能立马在网站上看到变化



## 文档编写规范

没啥特别的规范，先写就完事了，反正这个 repo 是 private 的，后续随时编辑，随便列几条吧

1. 文档格式是 Markdown，使用中文编写，可夹杂英文；在使用英文的缩写前，保证本页面或相近页面出现过对应的全称解释，比如：

   ```
   我们的分支预测器使用了分支目标缓冲器(Branch Target Buffer，BTB)这一结构, ......
   ......
   目前 BTB 中包含了 ......
   ```

   （或者统一全都不写最后搞一个 Terminology? I don't know）

2. 说明结构的时候尽量配图，如果嫌麻烦的话可以手画一张草图传上去然后 cue 我一下，我来使用软件画出来然后做润色等（工具人就是我了 = =

3. 写的时候要记住文档给面向外面的人读的，所以多写一些总览框架，不必拘泥于类似信号的含义这样的细节

先写这么多，后面想到再补充。



## TODO List

`流水线：`

总览

IFU

IDU

ISU (包含 Register Files and Bypass Network)

EXU

WBU

`功能部件：`

LSU

CSR

Cache

TLB

BPU

`外围部件`

内存系统

MMIO外设

`其他`

上手教程

Debugging

配套测试集





