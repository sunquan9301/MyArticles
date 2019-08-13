最近Coroutine(协程)开始流行起来，在python，Go语言得到应用，在Kotlin也大放异彩，大火的Flutter的也用到了Coroutine，本篇文章以Coroutine在Flutter中的应用作为Motivation Example，并随之深入理解Coroutine的知识点。

本篇的内容组织如下：

1. **Coroutine in Flutter**
2. **Coroutine的定义以及其基本概念**
3. **Process，Thread and Coroutine**
4. **Coroutine核心功能**
5. **Coroutine实现方式**
6. **Coroutine in Kotlin使用，原理和实践**
7. **Coroutine for Room and Coroutine for retrofit**
8. **Coroutine in Android**

## Coroutine in Flutter

#### Flutter 线程模型

18年底Flutter1.0正式发布以来，国内出现了爆发式增长，大前端的概念流行起来，在学习的同时，还是保持谨慎的态度观望，毕竟GitHub上还有7200多个open issues。

Flutter使用Dart语言，简单理解Dart的线程模型，要知道以下几个概念：**1.Dart是单线程执行；2.Dart有一个事件循环；3.Isolate: 字面翻译是隔离，它是一种让代码运行在其他线程的方式；4.异步编程。**当你不产生一个Isolate的时候，代码是默认运行在main UI Thread。可对比Android中的主线程，同样event loop 也可类比到Android's main Looper。通过下面的图可进行简单的理解。

![image-20190812124946274](/Users/sunquan/Library/Application Support/typora-user-images/image-20190812124946274.png)

- Event Loop 通过读取队列来控制代码的执行
- Microtask Queue用来存储一些非常短时的 asynchronous internal actions

#### async、await

Dart同时提供异步编程模型，异步调用中有几个关键词：async、await、Future；Async、await本质上就是Dart对异步操作封装的一个语法糖，在别的语言如JS，Kotlin中也类似存在，使用如下：

```dart
loadData() async {
  String dataURL = "https://jsonplaceholder.typicode.com/posts";
  http.Response response = await http.get(dataURL);
  setState(() {
    widgets = json.decode(response.body);
  });
}
```

通过async获取发起异步操作，通过await等待执行结果，于此同时在单线程执行模型中也不会阻塞线程。而要想了解async、await的原理，就要先了解**协程Coroutine**的概念，这里终于引出了**协程Coroutine**

## 协程的定义以及其基本概念

####定义：

那么协程到底是什么呢？wikipedia里定义如下

```markdown
Coroutines are **computer program components** that generalize subroutines for **non-preemptive multitasking**, by allowing execution to be **suspended and resumed**. 
```

关键词有：

- **computer program components**

- **non-preemptive multitasking**

- **suspended and resumed**

协程是一个计算机组件，用于非抢占式的多任务执行，于此同时允许执行的程序可以被挂起和重新恢复。

#### 背景

那么协程是怎么来的呢，其实协程这个概念早在**1958年**就由**Melvin Conway**提出，并最初用于将"语法分析"和"词法分析"分离的一种方案。协程在很多语言中都有其实现方案，如感兴趣，可自行查找相关资料。

![image-20190812190018930](/Users/sunquan/Library/Application Support/typora-user-images/image-20190812190018930.png)



#### 基本概念

```markdown
**进程**：一段程序的执行过程，系统进行资源分配和调度的基本单位，每个进程有独立地址空间，互相之间不发生干扰
**线程**：轻量级进程，资源调度的最小单位，共享父进程的地址空间和资源，线程的调度和进程一样，都要切换到内核态
**并行**：同时发生，在多核CPU中，多个任务可同时在不同CPU上面同一时间执行
**并发**：宏观上并行，微观上串行，操作系统通过管理调度根据相关算法，分配时间片来达到一种宏观上并行的方式
**上下文**：程序执行的状态，通常用调用栈记录程序执行的当前状态以及其相关的环境信息
```

- 早期，CPU都是单核，无法真正并行，为了产生共享CPU的假象，提出了时间片概念，**将时间分割成连续的时间片段，多个程序交替获得CPU使用权限。**而管理时间片分配调度的调度器则成为操作系统 的核心组件。
- 程序能交替执行了，但上下文切换必然会引起程序相关变量混乱，因此在物理地址的基础上提出了虚拟地址的概念
  - CPU增加内存管理单元，进行虚拟地址和物理地址的转换
  - 操作系统加入内存管理模块，管理物理内存和虚拟内存
  - **进程出现**
- 进程是一个实体，包括程序代码以及其相关资源(内存，I/O，文件等)，可被操作系统调度。但是如果想一遍操作I/O进行输入输出，一遍想进行加减计算，就得两个进程，这样随便写写代码，内存就爆表了。于是又想着能不能有一轻量级的进程呢，**只执行程序，不需要独立的内存，I/O等资源，而是共享已有资源，于是产生了线程。**

![1](/Users/sunquan/Desktop/1.jpg)

- 一个进程可以跑很多个线程处理并发，但是**线程进行切换的时候，操作系统会产生中断，线程会切换到相应的内核态，并进行上下文的保存**，这个过程不受上层控制，是操作系统进行管理。然而内核态线程会产生性能消耗，因此线程过多，并不一定提升程序执行的效率。正是由于**1.线程的调度不能精确控制**；**2.线程的切换会产生性能消耗**。**协程出现了。**

