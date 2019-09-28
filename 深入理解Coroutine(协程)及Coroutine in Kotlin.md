最近Coroutine(协程)开始流行起来，在python，Go语言得到应用，在Kotlin也大放异彩，大火的Flutter的也用到了Coroutine，本篇文章以Coroutine在Flutter中的应用作为Motivation Example，并随之深入理解Coroutine的知识点。

本篇的内容组织如下：

1. **Coroutine in Flutter**
2. **Coroutine的定义以及其基本概念**
4. **Coroutine核心功能**
5. **Coroutine实现方式**
6. **Coroutine in Kotlin使用，原理和实践**
7. **Kotlin Coroutine in Android**

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

## 协程核心功能

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

维基百科上对迭代定义为：**迭代**是重复反馈过程的活动，其目的通常是为了接近并到达所需的目标或结果。每一次对过程的重复被称为一次"迭代"，而每一次迭代得到的结果会被用来作为下一次迭代的初始值。

####可迭代对象(Iterable)与迭代器(iterator)

所有能够接受for…in…操作的对象都是可迭代对象，如列表、字符串、文件等。从Java语言来看Iterable类，注释所说实现了Iterable接口类都能够接受"for-each loop" statement。

```java
/**
 * Implementing this interface allows an object to be the target of
 * the "for-each loop" statement. 
 */
public interface Iterable<T> {
    /**
     * Returns an iterator over elements of type {@code T}.
     * @return an Iterator.
     */
    Iterator<T> iterator();
    /**
     * Performs the given action for each element of the {@code Iterable}
     * until all elements have been processed or the action throws an
     * exception.  Unless otherwise specified by the implementing class,
     * actions are performed in the order of iteration 
     */
    default void forEach(Consumer<? super T> action) {
        Objects.requireNonNull(action);
        for (T t : this) {
            action.accept(t);
        }
    }
    default Spliterator<T> spliterator() {
        return Spliterators.spliteratorUnknownSize(iterator(), 0);
    }
}
```

Iterable接口提供2个常见的方法:

- Iterator<T> iterator();
- default void forEach(Consumer<? super T> action)

可以看到iterator方法返回Iterator对象，在Java中Iterator同样是一个接口：

```java
/**
 * An iterator over a collection.  {@code Iterator} takes the place of
 * {@link Enumeration} in the Java Collections Framework.  
 */
public interface Iterator<E> {
    /**
     * @return {@code true} if the iteration has more elements
     */
    boolean hasNext();
    /**
     * Returns the next element in the iteration.
     */
    E next();

    /**
     * Removes from the underlying collection the last element returned
     * by this iterator (optional operation).  This method can be called
     * only once per call to {@link #next}.  
     */
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}
```

Iterator提供了如下常见方法：

- boolean hasNext();
- E next();
- default void remove();

正是 next() 使得iterator能在每次被调用时，返回一个单一的值，即iterator是消耗型的，即每一个值都被使用过后，就消失了。

在Java里日常使用的就是ArrayList类了:

```java

    /**
     * An optimized version of AbstractList.Itr
     */
    private class Itr implements Iterator<E> {
        protected int limit = ArrayList.this.size;

        int cursor;       // index of next element to return
        int lastRet = -1; // index of last element returned; -1 if no such
        int expectedModCount = modCount;

        public boolean hasNext() {
            return cursor < limit;
        }

        @SuppressWarnings("unchecked")
        public E next() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            int i = cursor;
            if (i >= limit)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }

        public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
                limit--;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
    }
```

可以看到ArrayList用一个变量 int cursor;来保存每次next()后的上下文。

对一个iterable用for…in…进行迭代时，通常是通过调用iterator()方法得到一个Iterator,然后循环的调用Iterator的next()方法取得每一次对值，知道Iterator为空。如下所图：

![WX20190928-173115@2x](/Users/sunquan/Desktop/WX20190928-173115@2x.jpg)

#### 生成器Generator

定义如下：

```markdown
In computer science, a generator is a special routine that can be used to **control the iteration behaviour of a loop**. 
In fact, all generators are iterators.[1] A generator is very similar to a function that returns an array, in that a generator has parameters, can be called, and generates a sequence of values. 
However, instead of building an array containing all the values and returning them all at once, **a generator yields the values one at a time**, which requires less memory and allows the caller to get started processing the first few values immediately. In short, a generator looks like a function but behaves like an iterator.
```

生成器是这样一个函数，它记住**上一次返回时在函数体中对位置。**

对生成器函数的第二次（或第n次）调用跳转至该函数中间，而**上次调用的所有局部变量都保持不变。**

生成器不仅 **“记住” 了它的数据状态**；生成器还 **“记住” 了它的流控制构造**中的位置。 

#### Yield关键字

在定义中所说：a generator yields the values one at a time。

- yield类似return关键字，不同在于函数返回的是一个生成器
- yield可以暂停一个函数并返回中间结果。使用yield的函数将保存执行环境，即函数的参数。在下一次调用时，所有参数都会恢复，然后从先前暂停的地方开始执行，直到遇到下一个yield再次暂停。

如下斐波那契数列：

```python
>>> def fib():
	a,b = 0,1
	while True:
		a,b = b,a+b
		yield b

		
>>> myFib = fib()
>>> next(myFib)
1
>>> next(myFib)
2
>>> next(myFib)
3
>>> next(myFib)
5
>>> 
```

#Coroutine实现方式

协程有一下几种实现方式：

- 利用glibc的ucontext组件
- 使用汇编代码来切换上下文
- 利用C语言语法switch-case来实现
- 利用了C语言的setjmp和longjmp

这里分析下第一种的思想：

实现思路：每一线程中，有一个调度器，就是一个循环，不断的从可运行的协程队列中取出协程，并利用swapcontext恢复写成的上下文。当一个协程放弃CPU时，通过swapcontext恢复调度器上下文从而将控制权归还给调度器。每个协程通过getcontext和makecontext。

先介绍下几个API

```c
//该函数初始化ucp所指向的结构体ucontext_t
int getcontext(ucontext_t *ucp)
  
//函数恢复用户上下文为ucp所指向的上下文。
int setcontext(const ucontext_t *ucp)
  
//函数修改ucp所指向的上下文，程序的执行会切换到func的调用，通过makecontext()调用的argc传递func的参数。
void makecontext(ucontext_t *ucp, void (*func)(void), int argc, ...)

//函数保存当前的上下文到oucp所指向的数据结构，并且设置到ucp所指向的上下文。
int swapcontext(ucontext_t *restrict oucp, const ucontext_t *restrict ucp)

```



用来表示协程的Task类

```c
struct Task        
{ 
  Context context;// 当前协程上下文
  Task  *next; //通过这两个指针将task串起来
  Task  *prev;
  Task  *allnext;
  Task  *allprev;
  void  (*startfn)(void*);//当前协程的执行入口函数
  void  *startarg;//参数
  void  *udata;
  uchar *stk; // 当前协程可以使用的堆栈，初始化为栈顶地址
  uint  stksize;// 当前协程可以使用的堆栈大小
  int ready;
	...
};
```

新增一个协程

```c
static Task* taskalloc(void (*fn)(void*), void *arg, uint stack)
{                                                                                                                                                                    
  Task *t;
  sigset_t zero;
  uint x, y;
  ulong z;

  /* allocate the task and stack together */
  t = malloc(sizeof *t+stack);    
  if(t == nil){
    fprint(2, "taskalloc malloc: %r\n");
    abort();
  }
  memset(t, 0, sizeof *t);
  //初始化各种参数
  t->stk = (uchar*)(t+1);
  t->stksize = stack;
  t->startfn = fn;                // 协程入口函数
  t->startarg = arg;              // 协程入口函数参数
	//...
  //这里调用了getcontext初始化当前协程上下文
  if(getcontext(&t->context.uc) < 0){              
    fprint(2, "getcontext: %r\n");
    abort();
  }
	//...
  t->context.uc.uc_stack.ss_size = t->stksize-64;   //ss_size成员为栈大小
  t->context.uc.uc_stack.ss_sp = 
    (char*)t->context.uc.uc_stack.ss_sp
    +t->context.uc.uc_stack.ss_size;
  //这里调用makecontext入口函数为taskstart参数为y,x
  makecontext(&t->context.uc, (void(*)())taskstart, 2, y, x);   
  //...
  return t;
}
```

然后调用taskready将这个协程放入可运行队列中：

```c
void taskready(Task *t)
{
  t->ready = 1; //
  addtask(&taskrunqueue, t);   //将协程放入到可运行队列中
}
```

调度器

```c
static void taskscheduler(void)
{
  int i;
  Task *t;
  for(;;){                          //无限循环
    if(taskcount == 0)
      exit(taskexitval);
    t = taskrunqueue.head;          //从可运行队列头部取出下一个运行的协程
   	//...
    deltask(&taskrunqueue, t);      //从可运行队列中将它删除
    t->ready = 0;
    taskrunning = t;   //将t设置为当前正在运行的协程，taskrunning是一个全局变量
    tasknswitch++;     //统计值，协程一共执行了多少次
    // 通过swapcontext切换到目标协程，并且将调度器上下文保存在全局变量taskschedcontext中
    swapcontext(&taskschedcontext, &t->context);    
    //...
}
```

协程的yield方法

```c
int taskyield(void)         
{
  int n;
  n = tasknswitch;
  taskready(taskrunning); // 将自己设置为ready重新放回可运行队列
  taskstate("yield");
  taskswitch();           //将控制权还给调度器
  return tasknswitch - n - 1;
}
```

## Coroutine in Kotlin使用，原理和实践

Kotlin官网上详细的介绍了协程的基本使用，基本使用方式如下：

```kotlin
import kotlinx.coroutines.*

fun main() {
    GlobalScope.launch { // 在后台启动一个新的协程并继续
        delay(1000L) // 非阻塞的等待 1 秒钟（默认时间单位是毫秒）
        println("World!") // 在延迟后打印输出
    }
    println("Hello,") // 协程已在等待时主线程还在继续
    Thread.sleep(2000L) // 阻塞主线程 2 秒钟来保证 JVM 存活
}
```

Kotlin Coroutine是Kotlin为了实现更好的异步和并发程序所提供的一个特性，从1.1版本开始引入不同于其他的编程语言，Kotlin将其Coroutine特性的大部分内容作为了一个扩展库：Kolinx.coroutines。

Kotlin 1.2中，Kotlin Coroutine还只是一个实验特性。Kotlin Coroutine相关类的包名包含了experimental的字样。Kotline1.3则正是包含Coroutine的特性，并趋于稳定。

用一句话概括Kotlin Couroutine的特点即是**"以同步之名，行异步之实"**。同时认清以下几个概念：

**suspending**

- 功能：使程序执行过程暂停，又不挂起线程
- 使用限制：
  - 在另一个suspending方法中
  - 在Coroutine Builder中被调用
- 一些方法：
  - delay
  - await
  - awaitSingle

**Coroutine Builder**

- 定义：
  - 创建Coroutine, 通过一些方法，接受suspending lambda作为参数，将其放入Coroutine中执行
- 使用限制
  - 在另一个suspending方法中
  - 在Coroutine Builder中被调用
- 常见方法
  - runBlocking
  - launch
  - async

**CoroutineScope 和CoroutineContext**

协程总是运行在一些以 CoroutineContext 类型为代表的上下文中，它们被定义在了 Kotlin 的标准库里。

GlobalScope.runBlocking(launch, async)

**CoroutineDispatcher**

 协程调度器，它确定了哪些线程或与线程相对应的协程执行。协程调度器可以将协程限制在一个特定的线程执行，或将它分派到一个线程池，亦或是让它不受限地运行。如下：

```kotlin
/**
 * Groups various implementations of [CoroutineDispatcher].
 */
public actual object Dispatchers {
    @JvmStatic
    public actual val Default: CoroutineDispatcher = createDefaultDispatcher()

    @JvmStatic
    public actual val Main: MainCoroutineDispatcher get() = MainDispatcherLoader.dispatcher

    @JvmStatic
    @ExperimentalCoroutinesApi
    public actual val Unconfined: CoroutineDispatcher = kotlinx.coroutines.Unconfined

    @JvmStatic
    public val IO: CoroutineDispatcher = DefaultScheduler.IO
}

```



### Kotlin Coroutine原理解析

我们从一段代码开始：

```kotlin
fun postItem(item: Item): PostResult {
  val token = requestToken()
  val post = createPost(token, item)
  val postResult = processPost(post)
  return postResult
}
```

优点：Direct Style，清晰可见，

缺点：同步执行，执行线程会被阻塞导致效率不高。并发量高的场景会产生性能问题影响整个系统。

因此可以转为callBack方式

```kotlin
fun postItem(item: Item) {
  requestToken { token ->
    createPost(token, item) { post ->
      processPost(post) { postResult ->
        handleResult(postResult)
      }
    }
  }
}
```

优点：提高了执行效率

缺点：可读性差，debug起来难

自己曾经写过这样的业务代码，一旦出现一些问题，真的很难调试。

```kotlin
fun launch(context: Activity) {
  /**优先级561324
        1.新手礼包弹框
        2.视频断了重拍弹框
        3.push消息开启弹框
        4.游戏断线重连弹框
        5.强制升级弹框
        6.赛季结束开始弹框
         **/
  UpdateManager.getInstance().checkUpdateSilent(context, object : ActionUtil.Action1<Boolean> {
    override fun call(isUpdate: Boolean) {
      if (!isUpdate) {
        showNewSeasonNotifyDialog(context, object : ActionUtil.Action0 {
          override fun call() {
            showNewUserPropsDialog(context, object : ActionUtil.Action0 {
              override fun call() {
                showPushDialog(context, object : ActionUtil.Action0 {
                  override fun call() {
                    showGameReconnDialog(context)
                  }
                })
              }
            })
          }
        })
      }
    }
  })
}
```

而使用Coroutine则只需要加入suspend关键字就可以了

```kotlin
suspend fun postItem(item: Item): PostResult {
  val token = requestToken()
  val post = createPost(token, item)
  val postResult = processPost(post)
  return postResult
}
```

增加`suspend`关键字，就能达到同Callback风格相同的效率，那么这是怎么实现的呢？

我们来看一下编译后的样子：

```kotlin
fun postItem(item: Item, cont: Continuation): Any? {
  val sm = cont as? ThisSM ?: object : ThisSM {
    fun resume(…) {
      postItem(null, this)
    }
  }
 
  switch (sm.label) {
	case 0:
      sm.item = item
      sm.label = 1
      return requestToken(sm)
    case 1:
      val item = sm.item
      val token = sm.result as Token
      sm.label = 2 
      return createPost(token, item, sm)
    case 2:
      val post = sm.result as Post
      sm.label = 3
      return processPost(post, sm)
    case 3:
      return sm.result as PostResult
}
```

可以看出多了一个类`Continuation`类，那么Kotlin编译器做了哪些处理？如下：

- 增加了`Continuation`类型入参，返回值变为`Object`
- 生成 `Continuation` 类型的匿名内部类
- 对 suspending 方法的调用变为 switch 形式的状态机

 `Continuation` 类定义如下：

```kotlin
public interface Continuation<in T> {
  public val context: CoroutineContext
  public fun resume(value: T)
  public fun resumeWithException(exception: Throwable)
}
```

- `resume`方法，用于恢复暂停的Coroutine的执行，

- `CoroutineContext`属性用于保存Coroutine的上下文

上文已知：协程的核心功能是控制流的恢复和让出， 以及其2个核心方法：yield和resume

**那么什么时候暂停？什么时候恢复呢？**

**1.暂停：**

```kotlin
switch (sm.label) {
  case 0:
    sm.item = item
    sm.label = 1
    return requestToken(sm)
  case 1:
    val item = sm.item
    val token = sm.result as Token
    sm.label = 2 
    return createPost(token, item, sm)
  case 2:
    val post = sm.result as Post
    sm.label = 3
    return processPost(post, sm)
  case 3:
    return sm.result as PostResult
```

从上文代码可以看到suspending 方法编译之后，会将原来的方法体变为一个由 switch 语句构成的状态机

Kotlin Coroutine 的运行依赖于各种 Callback 机制。一个 suspending 方法调用到最后，就是注册一个回调，方法执行结果通过这个回调来处理。当回调注册完毕之后，线程就无需等待，从而返回，结束调用。

**2.恢复**

```kotlin
val sm = cont as? ThisSM ?: object : ThisSM {
    fun resume(…) {
      postItem(null, this)
    }
  }
```

函数开头定义了sm，在sm中实现了resume方法，在resume方法中会调用自己，并把自己作为参数传入。

在switch的每个case中回去更新sm.label字段，从而达到记录上下文的作用，从而在恢复的时候找到对应的入口。

了解了Kotlin Coroutine的状态机原理，完全可以根据其思想自己封装一个小框架，从而解决某些特定业务，从而提高开发效率。

##Kotlin Coroutine in Android

以往Android project通常使用RxJava作为线程切换，但是RxJava的库很大，当然可以自己抽离起线程切换实现类和一些操作符，或者可以自己根据RxJava自己写一个简单的线程切换，但是其丰富的操作符则不是一时半会能自己写出来的。 

因此在新项目中，自己则使用了Kotlin Coroutine来作为线程使用场景，具体在Android项目里使用到的业务场景如下：

**1.和Room结合管理数据库**

- ```groovy
  //Kotlin Extensions and Coroutines support for Room
  androidx.room:room-ktx:$room_version
  ```

androidx.room:room-ktx已经提供了Coroutines support for Room只要在Dao中更改如下：

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    suspend fun getAll(): List<User>

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    suspend fun loadAllByIds(userIds: IntArray): List<User>

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND " +
            "last_name LIKE :last LIMIT 1")
    suspend fun findByName(first: String, last: String): User

    @Insert
    suspend fun insertAll(vararg users: User)

    @Delete
    suspend fun delete(user: User)
}
```

注意suspend方法要在Coroutine Builder里面调用哦

**2.和Retrofit作为请求数据所用**

- Retrofit也已经提供了支持库

- ```groovy
  LIB_RETRIFIT_COROUTINE=com.jakewharton.retrofit:retrofit2-kotlin-coroutines-adapter:0.9.2
  ```

只要**传入对应的Adapter就可以了CoroutineCallAdapterFactory.create()**，至于如何实现的，可参考Retrofit的源码

```kotlin
retrofit = new Retrofit.Builder()
                .baseUrl(Constants.API_HOST_PREFIX)
          .addConverterFactory(GsonConverterFactory.create(Json.getGson()))
               .addCallAdapterFactory(CoroutineCallAdapterFactory.create())
                .client(apiOkHttpClient)
                .build();
```

**3.封装一个协程类作为线程使用场景**

