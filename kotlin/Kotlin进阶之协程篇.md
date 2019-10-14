##### 1. 协程定义：轻量级线程,简单的说就是代码块或者函数调用，一个线程可以包含多个协程

```
Android 官方已经慢慢集成到自己的框架的 viewmodel中
Rtrofit2.6.0 以上已经支持协程，可能后续会变成主流的操作
```

##### 2. 协程、线程、进程比较


 区别 | 协程 | 线程 | 进程
---|---|---|---
定义 |轻量线程 | 运算调度最小单位 | 一段程序的执行过程
占用资源 |单线程，子程序切换，程序自身可控 |多线程消耗资源大，线程执行不可控 | 
数据共享 |单线程，不需要要加锁效率高 |需要加锁，效率低下 | 
可读性 |同步代码相似，可读性高 |线程间切换需要嵌套，可读性低 | 
Thread代码比较 |GlobalScope.launch { …… }  |thread { …… } | 
Thread.sleep代码比较 |delay(……)  |Thread.sleep(……) | 

##### 3、开始使用
```
Android中使用协程 kotlin版本必须大于1.3.+
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.2'
```

##### 4、名词解释

```
作用域： 执行代码的线程
上下文： 是一组各种元素的集合。 主要元素是协程的Job及其调度器
挂起：   暂时不执行
调度器： 可以切换线程
协程体： 执行代码的地方
```


##### 5、生命周期

```
isActive：是否活动
isCompleted：是否完成
isCancelled：是否取消
```


##### 6、协程使用方法


名称 | 作用
---|---
runBlocking | 创建一个会阻塞当前线程的Coroutine
launch | 创建一个不会阻塞当前线程、没有返回结果的 Coroutine,但会返回一个 Job 对象，可以用于控制这个 Coroutine 的执行和取消
async | 创建一个不会阻塞当前线程、有返回结果的 Coroutine,但会返回一个 Deferred 对象，可以用于控制这个 Coroutine 的执行和取消，Deferred.await()可以获取结果
flow | 创建异步流，可以执行多个事件


##### 7、参数解析

```
CoroutineScope.launch(
    context: CoroutineContext = EmptyCoroutineContext,
    start: CoroutineStart = CoroutineStart.DEFAULT,
    block: suspend CoroutineScope.() -> Unit
)
```

```
context: CoroutineContext: 即上下文，是一组各种元素的集合。 主要元素是协程的Job及其调度器
```

```
启动模式 CoroutineStart
```
模式 | 功能
---|---
DEFAULT | 立即执行协程体
ATOMIC | 立即执行协程体，但在开始运行之前无法取消
UNDISPATCHED | 立即在当前线程执行协程体，直到第一个 suspend 调用
LAZY | 需要时候调用start运行
```
协程体 block: suspend CoroutineScope.() 即我们执行代码地方
```

##### 8、协程启动


```
fun main() = runBlocking{
    
}

fun main() = launch{
    
}

fun main() = async{
    
}

suspend 修饰的方法可以在协程中执行，即挂起函数
```

##### 9、协程取消


```
1. 取消协程执行
val job = launch{
    
}

job.cancel()

2、取消计算代码
val job = launch{
    while(isActive){
        
    }
}
job.cancelAndJoin()

3、超时
withTimeout() {
//抛出TimeoutCancellationException代表取消协程
}

withTimeoutOrNull(){
//返回null 代表取消协程
}
```

##### 10、协程上下文与调度器
模式 | 功能
---|---
Dispatchers.Default  | 运行父协程的上下文
Dispatchers.Unconfined | 事件线程池, 即有执行顺序的
Dispatchers.IO  | 使用IO线程池
newSingleThreadContext | 创建新线程


```
launch(Dispatchers.IO) {//运行在IO线程
    withContext(Dispatchers.Main){
    //回到Main线程，可以进行修改UI
    }
}
```

##### 11、异步流


```
异步流的主要功能就是执行一串事件，例如for循环，while循环
```

```
流的构建两种方式
1、flow { // 流构建器
    for (i in 1..3) {
        delay(100) // 假装我们在这里做了一些有用的事情
        emit(i) // 发送下一个值
    }.collect { value -> println(value) //获取emit发送的参数 } 
}

2、(1..3).asFlow().collect { value -> println(value) }
```

```
流的操作符(操作符比较多，就挑选几个日常常用的，使用方式与list的差不多，可以查看kotlin官方)

filter：过滤操作符
transform：转换操作符
take：限长操作符
```


##### 12、通道


```
val channel = Channel<Int>()
launch {
    // 这里可能是消耗大量 CPU 运算的异步逻辑，我们将仅仅做 5 次整数的平方并发送
    for (x in 1..5) channel.send(x * x)
}
// 这里我们打印了 5 次被接收的整数：
repeat(5) { println(channel.receive()) }
```

