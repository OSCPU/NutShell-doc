# NutShell 文档

该 repo 和 github.io 绑定, 域名为 https://oscpu.github.io/NutShell-doc/, 欢迎访问！

备用部署域名：https://oscpu.gitbook.io/nutshell

Gitbook 构建工作目前已移交给 Github-CI 完成; 文档改写完成后, 本地可运行 `gitbook serve ./ ./building` 查看的生成网页, 但请不要把 ./building/ push 到主线上.



## 文档编写规范

1. 文档格式是 Markdown, 使用中文编写, 可夹杂英文；在使用英文的缩写前, 保证本页面或相近页面出现过对应的全称解释, 比如：

   ```
   我们的分支预测器使用了分支目标缓冲器(Branch Target Buffer, BTB)这一结构, ......
   ......
   目前 BTB 中包含了 ......
   ```

2. 说明结构的时候尽量配图, 如果嫌麻烦的话可以手画一张草图传上去然后 后续使用软件画出来做润色

3. 写的时候要记住文档给面向一般读者, 所以多写一些总览框架, 不必拘泥于类似信号的含义这样的细节



