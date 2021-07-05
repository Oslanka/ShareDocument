## 解析-协程coroutine

### 什么是协程

协程coroutine：又称微线程，纤程，可以并行执行、交换执行权的线程（或函数），就称为协程。

协程？

- 轻量级线程，或者叫用户态线程
- 用同步的方式解决异步的问题

我们理解的协程定义：

https://www.liaoxuefeng.com/wiki/897692888725344/923057403198272

- 被挂起/被恢复
- 更灵活
- 轻量，比线程更小
- 同线程内的协程，执行是顺序执行的

协程起源于一种汇编语言方法，但有一些高级编程语言支持它（摘自维基百科）：

![image-20210622161821444](/Users/caining/Documents/文稿/Q_share/多语言解析-协程coroutine.assets/image-20210622161821444.png)

### 语法糖

- `yield`：（python/php等）yield是生成器中的特有用法，而生成器是一种可以封闭整个运行状态、可以随时暂停继续的模型；（协程概念）

- `suspend`
- `async/await`

### 一、 Kotlin 协程

- 定义：协程是一种并发设计模式，您可以在 Android 平台上使用它来简化异步执行的代码，是在版本 1.3 中添加到 Kotlin 的，它基于来自其他语言的既定概念。
- 特点
  - **轻量**：您可以在单个线程上运行多个协程，因为协程支持[挂起](https://kotlinlang.org/docs/reference/coroutines/basics.html)，不会使正在运行协程的线程阻塞。挂起比阻塞节省内存，且支持多个并行操作。
  - **内存泄漏更少**：使用[结构化并发](https://kotlinlang.org/docs/reference/coroutines/basics.html#structured-concurrency)机制在一个作用域内执行多项操作。
  - **内置取消支持**：[取消](https://kotlinlang.org/docs/reference/coroutines/cancellation-and-timeouts.html)操作会自动在运行中的整个协程层次结构内传播。
  - **Jetpack 集成**：许多 Jetpack 库都包含提供全面协程支持的[扩展](https://developer.android.com/kotlin/ktx)。某些库还提供自己的[协程作用域](https://developer.android.com/topic/libraries/architecture/coroutines)，可供您用于结构化并发。

- 引入依赖

  ```groovy
  dependencies {
      implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.9'
  }
  ```

- kotlin 协程-hello world

  - `CoroutineScope`管理一个或多个相关的协程
  - `launch` 是一个函数，用于创建协程并将其函数主体的执行分派给相应的调度程序。
  - `Dispatchers.IO` 指示此协程应在为 I/O 操作预留的线程上执行。

```kotlin
fun main() = runBlocking { // this: CoroutineScope
    launch { // launch a new coroutine and continue
        delay(1000L) // non-blocking delay for 1 second (default time unit is ms)
        println("World!") // print after delay
    }
    println("Hello") // main coroutine continues while a previous one is delayed
}
```



### 二、Python 协程

### 三、Flutter-Dart 协程

Python 协程之 生产消费例子



Flutter之Dart中的协程使用：

https://blog.csdn.net/github_33420275/article/details/88063322

​	Dart关键字`async`、`await`本质上就是协程的一种语法糖。

四、JavaScript 协程

