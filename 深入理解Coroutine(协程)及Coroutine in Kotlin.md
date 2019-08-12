最近Coroutine(协程)开始流行起来，在python，Go语言得到应用，在Kotlin也大放异彩，大火的Flutter的也用到了Coroutine，本篇文章以Coroutine在Flutter中的应用作为Motivation Example，并随之深入理解Coroutine的知识点。

本篇的内容组织如下：

1. Coroutine in Flutter
2. Coroutine的定义以及其基本概念
3. Process，Thread and Coroutine
4. Coroutine核心功能
5. Coroutine实现方式
6. Coroutine in Kotlin使用，原理和实践
7. Coroutine for Room and Coroutine for retrofit
8. Coroutine in Android

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
Coroutines are computer program components that generalize subroutines for non-preemptive multitasking, by allowing execution to be suspended and resumed. 
```

关键词有：

- **computer program components**

- **non-preemptive multitasking**

- **suspended and resumed**

协程是一个计算机组件，用于非抢占式的多任务执行，于此同时允许执行的程序可以被挂起和重新恢复。

#### 起源

那么协程是怎么来的呢，其实协程这个概念早在**1963**就由**Melvin Conway**提出，并最初用于将"语法分析"和"词法分析"分离的一种方案。

