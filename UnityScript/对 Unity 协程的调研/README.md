## 1. 什么是协程 # #

A coroutine is a function that is executed partially and, presuming suitable conditions are met, will be resumed at some point in the future until its work is done. → 协程是一个部分执行, 遇到条件 (`yield return`) 会挂起, 直到条件满足才会被唤醒继续执行后面代码的一种函数.

## 2. 为什么要有协程 # #

协程的作用一共有两点 :

```html
1. 延时等待一段时间执行代码
2. 等某个操作完成之后再执行后面的代码
```

总而言之, 协程控制了代码在特定的时机执行, 是对单线程控制权的交替.

## 3. 协程的原理 # #

Coroutines 不是多线程, 不是异步技术, 协程都在 MainThread 中执行, 而且每个时刻只有一个 Coroutine 在执行. Coroutine 是一个 function, 可以部分执行, 当条件满足时, 未来会被再次执行直到整个函数执行完毕.

协程能够把一个计算或操作，分解成若干步，并且可以在任何一步停下来，并在需要的时候继续执行剩下的步骤。这样的模型给予了更细粒度的控制一个操作或是功能, 比如, 一个非常耗时间的操作, 被分步执行可以更好的控制程序响应. 比如, 一个操作需要依赖各种条件, 可以更好的处理条件不满足的时候的情况. 也能够更好的把操作或是计算过程中的状态变化, 与其他的状态变化交互, 然而, 程序运行的过程就是抽象数据结构和结构不断变化的过程, 协程能够优雅自然的进行这个变化过程的需求.

Unity 在每一帧都会去处理 GameObject 里带有的 Coroutine Function, 直到 Coroutine Function 被执行完毕. 当一个 Coroutine 开始启动时, 它会执行到遇到 yield 为止, 遇到 yield 的时候 Coroutine 会暂停执行, 直到满足 yield 语句的条件, 会开始执行 yield 语句后面的内容, 直到遇到下一个 yield 为止 ... 如此循环直到整个函数结束, **这就是可以将一个函数分割到多个帧里去执行的思想**.

也就是说, Coroutines 最棒的就是函数的执行可以不在一次 Frame 里完成, 可以在多个 Frame 中完成. 比如, 我们希望看到物体透明度的改变, 如果让 color.a 的变化在一帧内完成, 那么我们是看不出来这其中的变化得, 因为太快, 所以让 color.a 的变化必须在多个帧内完成, 我们才有可能用肉眼看到这一变化的过程.

Unity 的协程系统基于 C# 的接口 : IEnumerator, 允许为自己的集合类型编写枚举类.

![Unity 函数执行图](http://img.blog.csdn.net/20140814131547421?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMDE1MzcwMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 4. 协程与线程的区别 # #

1. 历史上先有的协程, 是操作系统用来模拟多任务并发, 协程是非抢占式的, 多任务时间片不能公平分享, 线程是抢占式的;

2. 线程能利用多核达到真正的并行计算, 如果任务设计的好, 线程能几乎成倍的提高你的计算能力, 但是线程的缺点也很明显, 那就是没有设计好导致大量的锁 | 切换 | 等待, 这些很多都是应用层的问题. 而协程因为是非抢占式, 所以需要用户自己释放使用权来切换到其他协程, 因此同一时间其实只有一个协程拥有运行权, 相当于单线程的能力.

3. 协程相对线程的最大优点就是 : 让原来要使用异步 + 回调方式写的非人类代码, 可以用看似同步的方式写出来.

## 5. Unity 中协程相关的类 # #

![](http://images2015.cnblogs.com/blog/1098699/201703/1098699-20170309162925438-1396246789.png)

1. yield return null; → 等待下一帧中的 Update 函数执行完之后再继续执行;
2. yield return new WaitForSeconds(10); → 延迟 10 秒后再继续执行;
3. yield return new WaitForFixedUpdate(); → 等待所有脚本中的 FixedUpdate 函数结束之后再继续执行;
4. yield return new WaitForEndOfFrame(); → 等待该帧中所有 Camera 和 GUI 对象渲染完毕, 在帧被显示到屏幕之前恢复执行前面的代码;
5. yield return new WWW(url); → 等待 url 下载完后再继续执行;
6. yield return StartCoroutine(MyFunc()); → 等待协程结束后再执行;

## 6. 需要注意的几点 # #

```html
- 协程是 C# 线程的替代品, 是 Unity 不使用线程的解决方案. 但是, 协程不是线程, 不能进行异步执行, 协程和 MonoBehaviour 的 Update 函数一样也是在 MainThread 中执行的.

- 使用协程不用考虑同步和锁的问题.

- 在程序中调用 StopCoroutine() 方法只能终止以字符串形式启动（开始）的协程；

- 多个协程可以同时运行，它们会根据各自的启动顺序来更新；

- 协程可以嵌套任意多层；

- 如果你想让多个脚本访问一个协程，那么你可以定义静态的协程；

- 协程不是多线程（尽管它们看上去是这样的），它们运行在同一线程中，跟普通的脚本一样；

- 如果你的程序需要进行大量的计算，那么可以考虑在一个随着时间进行的协程中处理它们；

- IEnumerator 类型的方法不能带 ref 或者 out 型的参数，但可以带被传递的引用；

- 目前在 Unity 中没有简便的方法来检测作用于对象的协程数量以及具体是哪些协程作用在对象上。

- 协程可以减少 callback 的使用, 但是不能完全替换 callback. 基于事件驱动的编程里面反而不能发挥协程的作用, 而用 callback 更适合. 想象一下用协程来写 GUI 的事件处理你怎么写, 计算密集型的异步代码里面也只能用 callback. 而 NodeJs 那种 io 瓶颈单任务流程用协程的确很适合, 但是也需要 callback 作补充.

- 状态机用协程其实也有问题, 比如状态里面嵌套子状态, 再由子状态切换到其他状态的子状态, 开销和代码都会变差, 反而不如经典的状态机简单明了高效.
```

## 7. 几条准则 # #

1. 协程的返回值必须是 IEnumerator;
2. 协程的参数不能加关键字 ref 或 out;
3. 在 C# 脚本中, 必须通过 StartCoroutine 来启动协程;
4. yield 语句要用 yield return 来代替;
5. 在函数 Update 和 FixedUpdate 中不能使用 yield 语句, 但是可以启动协程;
6. yield return 语句不能位于 try-catch 语句块中, 但可以位于 try-finally 的 try 语句块中;
7. yield return 语句不能放在匿名方法中;
8. yield return 语句不能放在 unsafe 语句块中;

## 8. 本文参考来源 # #

Book `<<Unity Case Study Manual>>`
http://blog.csdn.net/u010153703/article/details/38557237
http://blog.csdn.net/huang9012/article/details/38492937
https://www.zhihu.com/question/20511233?rf=23290260
https://www.zhihu.com/question/20511233

End.
