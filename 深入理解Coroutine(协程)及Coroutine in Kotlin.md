最近Coroutine(协程)开始流行起来，在python，Go语言得到应用，在Kotlin也大放异彩，大火的Flutter的也用到了Coroutine，本篇文章以Coroutine在Flutter中的应用作为Motivation Example，并随之深入理解Coroutine的知识点。

本篇的内容组织如下：

1. **Coroutine in Flutter**
2. **Coroutine的定义以及其基本概念**
4. **Coroutine核心功能**
5. **Coroutine实现方式**
6. **Coroutine in Kotlin使用，原理和实践**
7. **Coroutine for Room and Coroutine for retrofit**
8. **Coroutine in Android**

## Coroutine in Flutter

#### Flutter 线程模型

18年底**Flutter1.0**发布以来，开始爆发式增长，大前端概念流行起来。在学习的同时，谨慎观望，毕竟GitHub上还有7200多个open issues。

Flutter使用Dart语言，Dart的线程模型，有以下几个概念：

1. **Dart是单线程执行；**

2. **Dart有一个事件循环；**

3. **Isolate: 字面翻译是隔离，它是一种让代码运行在其他线程的方式；**

4. **异步编程。**

当不产生一个Isolate时，代码默认运行在**main UI Thread**。可对比Android主线程，同样event loop 也可类比Android's main Looper。通过下面的图可进行简单的理解。

![image-20190812124946274](/Users/sunquan/Library/Application Support/typora-user-images/image-20190812124946274.png)

- Event Loop 通过读取队列来控制代码的执行
- Microtask Queue用来存储一些非常短时的 asynchronous internal actions

#### async、await

Dart提供异步编程模型，核心是**：async、await、Future；**Async、await本质上是Dart对异步操作封装的一个语法糖，在JS，Kotlin中也类似存在，使用如下：

```dart
loadData() async {
  String dataURL = "https://jsonplaceholder.typicode.com/posts";
  http.Response response = await http.get(dataURL);
  setState(() {
    widgets = json.decode(response.body);
  });
}
```

async发起异步操作，await等待执行结果，于此同时在单线程执行模型中也不会阻塞线程。而要想了解async、await的原理，就要先了解**协程Coroutine**的概念，这里终于引出了**协程Coroutine**

## 协程的定义以及其基本概念

####定义：

wikipedia里定义如下

> Coroutines are  **computer program components** that generalize subroutines for **non-preemptive multitasking**, by allowing execution to be **suspended and resumed**. 

协程是一个**计算机组件**，用于**非抢占式的多任务执行**，于此同时允许执行的程序可以被**挂起和重新恢复**，具体什么概念呢，后面将围绕这个定义慢慢道来。

#### 背景

协程概念早在**1958年**就由**Melvin Conway**提出，并最初用于将"语法分析"和"词法分析"分离的一种方案。协程在很多语言中都有其实现方案，如感兴趣，可自行查找相关资料。

![image-20190812190018930](/Users/sunquan/Library/Application Support/typora-user-images/image-20190812190018930.png)

#### 进程，线程和协程

**进程**：**一段程序的执行过程**，**资源分配和调度的基本单位**，有其独立地址空间，互相之间不发生干扰
**线程**：**轻量级进程**，资源调度的最小单位，共享父进程地址空间和资源，其调度和进程一样要**切换到内核态**
**并行**：同时发生，在多核CPU中，多个任务可同时在不同CPU上面同一时间执行
**并发**：**宏观并行，微观串行**，操作系统根据相关算法，分配时间片来调度，从而达到一种宏观上并行的方式
**上下文**：**程序执行的状态**，通常用调用栈记录程序执行的当前状态以及其相关的环境信息

- 早期，CPU是单核，无法真正并行，为了产生共享CPU的假象，提出了时间片概念，**将时间分割成连续的时间片段，多个程序交替获得CPU使用权限。**而管理时间片分配调度的调度器则成为操作系统的核心组件。
- 程序能交替执行，但上下文切换必然会引起程序相关变量混乱，因此在物理地址基础上提出虚拟地址概念
  - CPU增加内存管理单元，进行虚拟地址和物理地址的转换
  - 操作系统加入内存管理模块，管理物理内存和虚拟内存
  - **进程出现**
- 进程是一个实体，包括**程序代码以及其相关资源**(内存，I/O，文件等)，可被操作系统调度。但想一边操作I/O进行输入输出，一边想进行加减计算，就得两个进程，这样写代码，内存就爆表了。于是又想着能否有一轻量级进程呢，**只执行程序，不需要独立的内存，I/O等资源，而是共享已有资源，于是产生了线程。**

![1](/Users/sunquan/Desktop/1.jpg)

- 一个进程可以跑很多个线程处理并发，但是**线程进行切换的时候，操作系统会产生中断，线程会切换到相应的内核态，并进行上下文的保存**，这个过程不受上层控制，是操作系统进行管理。然而内核态线程会产生性能消耗，因此线程过多，并不一定提升程序执行的效率。正是由于**1.线程的调度不能精确控制**；**2.线程的切换会产生性能消耗**。**协程出现了。**

####协程

> 1.协程是一种**轻量级**的**用户态线程**
> 2.开发者**自行控制程序切换时机**，而不是像进程和线程那样把控制权交给操作系统
> 3.协程**没有线程、进程切换的时间和资源开销**
> 4.协程是**非抢占式调度**，当前协程切换到其他协程是由自己控制；线程则是时间片用完抢占时间片调度

####优缺点

> **协程**
>
> **优点**：1.用户态，语言级别；2.无切换性能消耗；3.非抢占式；4.同步代码思维；5.减少同步锁
> **缺点**：1.注意全局变量；2.阻塞操作会导致整个线程被阻塞

协程属于**语言级别的调度算法**实现，一个线程里可发起多个协程，不需要对**协程共享变量施加同步锁**，当然**不同线程**的**全局性变量**还是要**谨慎使用**。同步代码思维可看如下搜索用户接口，

1. GlobalScope.launch发起了一个协程，并在IO线程上执行，
2. 在协程里，去调用接口获取结果。
3. 拿到结果，使用withContext(Dispatchers.Main)切换到主线程并更新界面

是不是很简洁，这就是同步思维写异步代码，不需要传入回调，也不需要Observe或轮询查询请求结果。

```kotlin
@GET("/user/search")
public fun searchUser(@Query("query") query: String): Deferred<BaseResponse<LSearchResult>>

GlobalScope.launch(Dispatchers.IO) {
  var result = L.client().commonApi.searchUser(param).await()
  withContext(Dispatchers.Main) {
    //do something
  }
}
```

## 协程核心

协程的**核心功能**是：**控制流的主动恢复和让出**

协程需要提供**两个重要操作**

- **Yield：让出CPU，放弃调度控制权，回到上一次Resume的地方**

- **Resume：获取调度控制权，继续执行程序，到上一次Yield的地方**

以上就是协程最核心的概念，各种语言协程的实现都跳不出该核心概念。以下用2个例子帮助理解。

### 线程和协程的工作方式

**线程：A，B是线程** （A是工作线程，B是网络I/O线程）

1. A发送一个包给B，然后阻塞
2. B接受到响应包回掉A传递过来的回调函数并发送数据
3. A接收到包继续干活

**协程：A，B是协程**

1. A要发送一个包，将包Push到A，B之间的一个Channel；放弃CPU给其他协程
2. B从Channel pop包，发送，接受到响应包后，放到A能拿到的地方，将A置为Ready
3. A下次被调度，即可拿到响应包继续做事情

### 生产者消费者模型

![image-20190816122754924](/Users/sunquan/Library/Application Support/typora-user-images/image-20190816122754924.png)



2个协程，一个生产者，一个消费者，当生产者放入一个item的时候，当即放弃控制权。接着消费者拿到控制权去消费产品。

![image-20190816123339543](/Users/sunquan/Library/Application Support/typora-user-images/image-20190816123339543.png)

可以看到核心还是**Yield**和**Resume**方法。

### 迭代器，生成器和Yield

