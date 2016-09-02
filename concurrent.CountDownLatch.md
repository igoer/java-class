#CountDownLatch
```
java.lang.Object
	java.util.concurrent.CountDownLatch
```

CountDownLatch是在java1.5被引入的，跟它一起被引入的并发工具类还有CyclicBarrier、Semaphore、ConcurrentHashMap和BlockingQueue，它们都存在于java.util.concurrent包下。CountDownLatch这个类能够使一个线程等待其他线程完成各自的工作后再执行。例如，应用程序的主线程希望在负责启动框架服务的线程已经启动所有的框架服务之后再执行。

CountDownLatch是通过一个计数器来实现的，计数器的初始值为线程的数量。每当一个线程完成了自己的任务后，计数器的值就会减1。当计数器值到达0时，它表示所有的线程已经完成了任务，然后在闭锁上等待的线程就可以恢复执行任务。

有时候会有这样的需求，多个线程同时工作，然后其中几个可以随意并发执行，但有一个线程需要等其他线程工作结束后，才能开始。举个例子，开启多个线程分块下载一个大文件，每个线程只下载固定的一截，最后由另外一个线程来拼接所有的分段，那么这时候我们可以考虑使用CountDownLatch来控制并发。

##API
> public CountDownLatch(int count)

唯一的构造函数，构造器中的计数值（count）实际上就是闭锁需要等待的线程数量。这个值只能被设置一次，而且CountDownLatch没有提供任何机制去重新设置这个计数值。

> await()

```java
public void await() throws InterruptedException
```
阻塞当前线程，直到 count 计数器的值为 0。

> await(long timeout, TimeUnit unit)

```java
public boolean await(long timeout, TimeUnit unit) throws InterruptedException
```
阻塞当前线程，直到 count 计数器的值为 0，或者超过了 timeout 所设定的时间。
timeout 为时间数，unit 为时间单位。

> countDown()

```java
public void countDown()
```
计数器减1，建议将该方法调用放在 finally 块中，如果发生异常避免 await 一直等待。

> getCount()

```java
public long getCount()
```
获取计数器当前值。

##使用示例
```java
// 创建计数器为10
public static CountDownLatch cdl = new CountDownLatch(5);

public static void main(String[] args) throws InterruptedException {
    for (int i = 0; i < 5; i++) {
        new T().start();
    }

    cdl.await(); // 等待全部任务完成
    System.out.println("OK, 任务全部完成了");
}

static class T extends Thread {

    @Override
    public void run() {
        try {
            Thread.sleep(1000);
            System.out.println(Thread.currentThread().getName() + " 完成了任务");
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            cdl.countDown();
        }
    }
}
```
输出结果：
```
Thread-0 完成了任务
Thread-1 完成了任务
Thread-2 完成了任务
Thread-4 完成了任务
Thread-3 完成了任务
OK, 任务全部完成了
```
在 main 线程中调用 await 方法则阻塞主线程直到 CountDownLatch 的计数器为0。