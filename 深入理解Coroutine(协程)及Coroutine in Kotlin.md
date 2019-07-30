最近Coroutine(协程)的概念流行起来，Coroutine不光在python，Go语言得到应用，在Kotlin也大放异彩，于此同时Flutter的异步框架也用到了Coroutine，本篇文章就来详细的说一说Coroutine。

本篇的内容组织如下：

1. 首先会介绍Coroutine的定义以及其相关的基本概念
2. Process，Thread，Coroutine进行一定的比较
3. Motivation Example帮助理解Coroutine
4. 选择Coroutine的一种实现方式进行具体分析
5. Coroutine in Kotlin的相关用法
6. Coroutine in Kotlin的实现原理
7. Coroutine in Kotlin practice
8. Coroutine for Room and Coroutine for retrofit
9. Kotine practice for Android App

