#ThreadPoolExecutor
```
java.lang.Object
	java.util.concurrent.AbstractExecutorService
		java.util.concurrent.ThreadPoolExecutor

All Implemented Interfaces:
	Executor, ExecutorService
Direct Known Subclasses:
	ScheduledThreadPoolExecutor
```
ThreadPoolExecutor是一个 ExecutorService，它使用可能的几个池线程之一执行每个提交的任务，通常使用 Executors 工厂方法配置。

线程池可以解决两个不同问题：由于减少了每个任务调用的开销，它们通常可以在执行大量异步任务时提供增强的性能，并且还可以提供绑定和管理资源（包括执行任务集时使用的线程）的方法。每个 ThreadPoolExecutor 还维护着一些基本的统计数据，如完成的任务数。

为了便于跨大量上下文使用，此类提供了很多可调整的参数和扩展钩子 (hook)。但是，强烈建议程序员使用较为方便的 Executors 工厂方法 Executors.newCachedThreadPool()（无界线程池，可以进行自动线程回收）、Executors.newFixedThreadPool(int)（固定大小线程池）和 Executors.newSingleThreadExecutor()（单个后台线程），它们均为大多数使用场景预定义了设置。

##Executor interface
Executor是用来执行提交的Runnable任务的对象，并以接口的形式定义，提供一种提交任务(submission task）与执行任务(run task)之间的解耦方式，还包含有线程使用与周期调度的详细细节等。Executor常常用来代替早期的线程创建方式，如new Thread(new(RunnableTask())).start(),在实际中可以用如下的方式来提交任务到线程池里，Executor会自动执行 你的任务：
```
Executor executor = anExecutor;
executor.execute(new RunnableTask1());
```
Executor接口中定义的方法如下：

> void execute(Runnable command);

在接下来的某个时刻执行提交的command任务。由于Executor不同的实现，执行的时候可能在一个新线程中或由一个线程池里的线程执行，还可以是由调用者线程执行 

##ExecutorService interface
ExecutorService接口扩展了Executor,提供管理线程池终止的一组方法，还提供了产生Future的方法，Future是用于追踪一个或多个异步任务的对象，并能返回异步任务的计算结果。

ExecutorService关闭后将不再接收新的任务，ExecutorService提供了两种不同类型的关闭方法，shutdown方法允许执行完之前提交的任务才终止，而shutdownNow将不再执行等待的任务，并试图终止当前执行的任务。ExecutorService终止后，内部已没有活动的任务，没有等待的任务，也不能再提交新任务，没有使用ExecutorService需要回收相应的资源。

submit方法是基于Executor.execute()方法之上的，通过创建并返回一个Future对象就可以实现取消执行或者等待执行完成。invokeAny和invokeAll通常是用于批量执行，可以提交一个task集合并等待task的逐个完成。

ExecutorService接口中定义的方法如下：
> void shutdown();

不再接收新的task,执行完之前提交的task后，开始有序的终止线程。

> List<Runnable> shutdownNow();

试图终止所有活动的执行任务，停止对等待任务的处理，并返回待执行的Runnable列表。
但是，不能保证能够终止掉所有的正在执行的任务。比如，在典型的实现中会调用Thread.interupt来作取消，但是一些不能响应中断的task将永远不会被终止。

> boolean isShutdown();

如果Executor调用shutdown或者shutdownNow将返回true。

> boolean isTerminated();

所有的任务都关闭后，线程池才会关闭成功。届时返回true

> boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;

发起关闭请求后，将一直阻塞等待关闭，直到所有的task已执行完成。
如果超时返回，或者当前线程中断时， 则返回false。

> < T > Future< T > submit(Callable< T > task);
> < T > Future< T > submit(Runnable task, T result);

提交一个等待返回值的task,返回的Future表示task执行后的待定结果。执行成功后，Future的get方法将返回实际的结果。

> Future<?> submit(Runnable task);

提交一个Runnable任务并返回一个Future。不过Future.get方法将返回null。

> <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>>tasks)throws InterruptedException;

执行提交的task集合。当执行完成后，返回task各自的Future。对应返回的Future集合，Future.isDone方法将返回true。

> <T> T invokeAny(Collection<? extends Callable<T>> tasks)throws InterruptedException, ExecutionException;

执行提交的task集合，返回一个task成功执行后的结果,而其他没有执行完成的task将被取消。


##构造方法：
```
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
	TimeUnit unit, BlockingQueue<Runnable> workQueue)
```
```
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
	TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler)
```
```
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
	TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory)
```
```
ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime,
	TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory,
    RejectedExecutionHandler handler)
```
- corePoolSize： 线程池维护线程的最少数量

- maximumPoolSize：线程池维护线程的最大数量

- keepAliveTime： 线程池维护线程所允许的空闲时间

- unit： 线程池维护线程所允许的空闲时间的单位

- workQueue： 线程池所使用的缓冲队列

- handler： 线程池对拒绝任务的处理策略

##线程创建策略
一个任务通过 execute(Runnable)方法被添加到线程池，任务就是一个 Runnable类型的对象，任务的执行方法就是 Runnable类型对象的run()方法。

当一个任务通过execute(Runnable)方法欲添加到线程池时，线程池中的线程创建判断条件如下：

1、如果此时线程池中的数量小于corePoolSize，即使线程池中的线程都处于空闲状态，也要创建新的线程来处理被添加的任务。

2、如果此时线程池中的数量等于 corePoolSize，但是缓冲队列 workQueue未满，那么任务被放入缓冲队列。

3、如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量小于maximumPoolSize，建新的线程来处理被添加的任务。

4、如果此时线程池中的数量大于corePoolSize，缓冲队列workQueue满，并且线程池中的数量等于maximumPoolSize，那么通过 handler 所指定的策略来处理此任务。

##workQueue 缓冲队列
workQueue为一个BlockingQueue阻塞队列接口，该接口主要有以下实现：

1、java.util.concurrent.ArrayBlockingQueue
基于数组的阻塞队列实现，也是最为常用的实现

2、java.util.concurrent.LinkedBlockingQueue
基于链表的阻塞队列，同ArrayListBlockingQueue类似

3、java.util.concurrent.DelayQueue
DelayQueue中的元素只有当其指定的延迟时间到了，才能够从队列中获取到该元素

4、java.util.concurrent.PriorityBlockingQueue
基于优先级的阻塞队列（优先级的判断通过构造函数传入的Compator对象来决定）

5、java.util.concurrent.SynchronousQueue
一种无缓冲的等待队列，类似于无中介的直接交易

##ThreadFactory 线程工厂
线程池最主要的一项工作，就是在满足某些条件的情况下创建线程。而在ThreadPoolExecutor线程池中，创建线程的工作交给ThreadFactory来完成。要使用线程池，就必须要指定ThreadFactory。

类似于上文中，如果我们使用的构造函数时并没有指定使用的ThreadFactory，这个时候ThreadPoolExecutor会使用一个默认的ThreadFactory：DefaultThreadFactory。（这个类在Executors工具类中）

当然，在某些特殊业务场景下，还可以使用一个自定义的ThreadFactory线程工厂，如下代码片段：


##hander 拒绝策略
handler有四个选择：

1、ThreadPoolExecutor.AbortPolicy
抛出java.util.concurrent.RejectedExecutionException异常

2、ThreadPoolExecutor.CallerRunsPolicy
重试添加当前的任务，他会自动重复调用execute()方法

3、ThreadPoolExecutor.DiscardOldestPolicy
直接丢弃后来的任务

4、ThreadPoolExecutor.DiscardOldestPolicy
丢弃在队列中队首的任务，抛弃当前的任务

##使用Executors创建线程池
java.util.concurrent.Executors工具类提供了基本的线程池创建方法，可以使用该工具类进行线程池的创建。当然你也可以通过构造函数创建符合自己要求的线程池。

> newFixedThreadPool()

创建线程数固定大小的线程池，由于使用了LinkedBlockingQueue所以maximumPoolSize没用，当corePoolSize满了之后就加入到LinkedBlockingQueue队列中。每当某个线程执行完成之后就从LinkedBlockingQueue队列中取一个。所以这个是创建固定大小的线程池。
```
public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
        	0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<Runnable>());
}
```

> newSingleThreadPool()

创建线程数为1的线程池，由于使用了LinkedBlockingQueue所以maximumPoolSize没用，corePoolSize为1表示线程数大小为1,满了就放入队列中，执行完了就从队列取一个。
```
public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
}
```

> newCachedThreadPool()

创建可缓冲的线程池，没有大小限制。由于corePoolSize为0所以任务会放入SynchronousQueue队列中，SynchronousQueue只能存放大小为1，所以会立刻新起线程，由于maxumumPoolSize为Integer.MAX_VALUE所以可以认为大小为2147483647。受内存大小限制。
```
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
}
```
