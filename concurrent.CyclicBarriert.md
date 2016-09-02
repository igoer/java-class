#CyclicBarrier
```
java.lang.Object
	java.util.concurrent.CyclicBarrier
```
CyclicBarrier的字面意思为回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。

##构造方法
```java
public CyclicBarrier(int parties, Runnable barrierAction) {
}

public CyclicBarrier(int parties) {
}
```
参数parties指让多少个线程或者任务等待至barrier状态；
参数barrierAction为当这些线程都达到barrier状态时会执行的内容。

##API
> await()

```
public int await() throws
	InterruptedException, BrokenBarrierException
```
挂起当前线程，直至所有线程都到达barrier状态再同时执行后续任务。

>await(long timeout, TimeUnit unit)

```
public int await(long timeout, TimeUnit unit) throws
	InterruptedException,BrokenBarrierException,TimeoutException
```
挂起当前线程，让这些线程等待至一定的时间，如果还有线程没有到达barrier状态就直接让到达barrier的线程执行后续任务，并抛出BrokenBarrierException异常。

> getNumberWaiting()

```
public int getNumberWaiting()
```
返回当前正在等待的线程数量。

> getParties()

```
public int getParties()
```
返回要求启动此 barrier 的参与者数目。

> isBroken()

```
public boolean isBroken()
```
查询此屏障是否处于损坏状态。

> reset()

```
public void reset()
```
重置为其初始状态的屏障。如果任何一方目前的障碍等，他们将返回一个brokenbarrierexception异常。复位后其他原因发生破损记录都可以进行复杂的；线程需要在一些其他的方式重新同步，并选择一个进行复位。它可能是最好的，而不是创建一个新的障碍，为后续使用。

##如何使用
**1、 await() 的使用方法**
```java
public static void main(String[] args) throws InterruptedException {
    CyclicBarrier cb = new CyclicBarrier(5);
    for (int i = 0; i < 5; i++) {
        new T(cb).start();
    }
    System.out.println(Thread.currentThread().getName() + " 任务分配完了, 先下班了");
}

static class T extends Thread {

    CyclicBarrier cb;

    public T (CyclicBarrier cb) {
        this.cb = cb;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " 开始工作");
        try {
            long time = (long)(Math.random() * 1000);
            Thread.sleep(time);
            System.out.println(Thread.currentThread().getName() + " 工作结束, 等待其他线程，工作了：" + time);
            cb.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 所有线程工作完毕");
    }
}
```
输出结果：
```
Thread-0 开始工作
Thread-2 开始工作
Thread-1 开始工作
main 任务分配完了, 先下班了
Thread-3 开始工作
Thread-4 开始工作
Thread-0 工作结束, 等待其他线程，工作了：428
Thread-2 工作结束, 等待其他线程，工作了：461
Thread-3 工作结束, 等待其他线程，工作了：535
Thread-4 工作结束, 等待其他线程，工作了：543
Thread-1 工作结束, 等待其他线程，工作了：941
Thread-0 所有线程工作完毕
Thread-1 所有线程工作完毕
Thread-2 所有线程工作完毕
Thread-3 所有线程工作完毕
Thread-4 所有线程工作完毕
```
从上面输出结果可以看出，每个工作线程执行完数据操作之后，就在等待其他线程操作完毕，当所有线程线程工作完毕之后，所有线程就继续进行后续的操作了。

**2、 CyclicBarrier(int parties, Runnable barrierAction) 的使用方法**
```java
public static void main(String[] args) throws InterruptedException {
    CyclicBarrier cb = new CyclicBarrier(5, new Runnable() {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " 来执行Runnable工作了");
        }
    });

    for (int i = 0; i < 5; i++) {
        new T(cb).start();
    }
    System.out.println(Thread.currentThread().getName() + " 任务分配完了, 先下班了");
}

static class T extends Thread {

    CyclicBarrier cb;

    public T (CyclicBarrier cb) {
        this.cb = cb;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " 开始工作");
        try {
            long time = (long)(Math.random() * 1000);
            Thread.sleep(time);
            System.out.println(Thread.currentThread().getName() + " 工作结束, 等待其他线程，工作了：" + time);
            cb.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 所有线程工作完毕");
    }
}
```
输出结果：
```
Thread-0 开始工作
Thread-2 开始工作
Thread-1 开始工作
main 任务分配完了, 先下班了
Thread-3 开始工作
Thread-4 开始工作
Thread-0 工作结束, 等待其他线程，工作了：428
Thread-2 工作结束, 等待其他线程，工作了：461
Thread-3 工作结束, 等待其他线程，工作了：535
Thread-4 工作结束, 等待其他线程，工作了：543
Thread-1 工作结束, 等待其他线程，工作了：941
Thread-1 来执行Runnable工作了
Thread-0 所有线程工作完毕
Thread-1 所有线程工作完毕
Thread-2 所有线程工作完毕
Thread-3 所有线程工作完毕
Thread-4 所有线程工作完毕
```
如果说想在所有线程写入操作完之后，进行额外的其他操作可以为CyclicBarrier提供Runnable参数。
从结果可以看出，当四个线程都到达barrier状态后，会从四个线程中选择一个线程去执行Runnable。

**3、 await(long timeout, TimeUnit unit) 的使用方法 **
```java
public static void main(String[] args) throws InterruptedException {
    CyclicBarrier cb = new CyclicBarrier(5, new Runnable() {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " 来执行Runnable工作了");
        }
    });

    for (int i = 0; i < 5; i++) {
        new T(cb).start();
    }
    System.out.println(Thread.currentThread().getName() + " 任务分配完了, 先下班了");

}

static class T extends Thread {

    CyclicBarrier cb;

    public T (CyclicBarrier cb) {
        this.cb = cb;
    }

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " 开始工作");
        try {
            long time = (long)(Math.random() * 10000);
            Thread.sleep(time);
            System.out.println(Thread.currentThread().getName() + " 工作结束, 等待其他线程，工作了：" + time);
            cb.await(2000, TimeUnit.MICROSECONDS);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (BrokenBarrierException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + " 所有线程工作完毕");
    }
}
```
输出结果：
```
Thread-0 开始工作
Thread-1 开始工作
Thread-2 开始工作
Thread-3 开始工作
main 任务分配完了, 先下班了
Thread-4 开始工作
Thread-4 工作结束, 等待其他线程，工作了：42
java.util.concurrent.TimeoutException
Thread-4 所有线程工作完毕
	at java.util.concurrent.CyclicBarrier.dowait(Unknown Source)
	at java.util.concurrent.CyclicBarrier.await(Unknown Source)
	at com.hp.CountDownLatchTest$T.run(CountDownLatchTest.java:43)
Thread-2 工作结束, 等待其他线程，工作了：3510
java.util.concurrent.BrokenBarrierException
	at java.util.concurrent.CyclicBarrier.dowait(Unknown Source)
	at java.util.concurrent.CyclicBarrier.await(Unknown Source)
	at com.hp.CountDownLatchTest$T.run(CountDownLatchTest.java:43)
Thread-2 所有线程工作完毕
Thread-1 工作结束, 等待其他线程，工作了：6301
java.util.concurrent.BrokenBarrierException
	at java.util.concurrent.CyclicBarrier.dowait(Unknown Source)
	at java.util.concurrent.CyclicBarrier.await(Unknown Source)
	at com.hp.CountDownLatchTest$T.run(CountDownLatchTest.java:43)
Thread-1 所有线程工作完毕
Thread-3 工作结束, 等待其他线程，工作了：6986
java.util.concurrent.BrokenBarrierException
	at java.util.concurrent.CyclicBarrier.dowait(Unknown Source)
	at java.util.concurrent.CyclicBarrier.await(Unknown Source)
	at com.hp.CountDownLatchTest$T.run(CountDownLatchTest.java:43)
Thread-3 所有线程工作完毕
Thread-0 工作结束, 等待其他线程，工作了：8568
java.util.concurrent.BrokenBarrierException
	at java.util.concurrent.CyclicBarrier.dowait(Unknown Source)
Thread-0 所有线程工作完毕
	at java.util.concurrent.CyclicBarrier.await(Unknown Source)
	at com.hp.CountDownLatchTest$T.run(CountDownLatchTest.java:43)
```
使用await(long timeout, TimeUnit unit)来让线程等待一定时间后抛出java.util.concurrent.BrokenBarrierException异常并继续运行后续的代码。

**4、 CyclicBarrier 重用 **
```java
public class Test {
    public static void main(String[] args) {
        int N = 4;
        CyclicBarrier barrier  = new CyclicBarrier(N);

        for(int i=0;i<N;i++) {
            new Writer(barrier).start();
        }

        try {
            Thread.sleep(25000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("CyclicBarrier重用");

        for(int i=0;i<N;i++) {
            new Writer(barrier).start();
        }
    }
    static class Writer extends Thread{
        private CyclicBarrier cyclicBarrier;
        public Writer(CyclicBarrier cyclicBarrier) {
            this.cyclicBarrier = cyclicBarrier;
        }

        @Override
        public void run() {
            System.out.println("线程"+Thread.currentThread().getName()+"正在写入数据...");
            try {
                Thread.sleep(5000);      //以睡眠来模拟写入数据操作
                System.out.println("线程"+Thread.currentThread().getName()+"写入数据完毕，等待其他线程写入完毕");
                cyclicBarrier.await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }catch(BrokenBarrierException e){
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName()+"所有线程写入完毕，继续处理其他任务...");
        }
    }
}
```
输出结果：
```
线程Thread-0正在写入数据...
线程Thread-1正在写入数据...
线程Thread-3正在写入数据...
线程Thread-2正在写入数据...
线程Thread-1写入数据完毕，等待其他线程写入完毕
线程Thread-3写入数据完毕，等待其他线程写入完毕
线程Thread-2写入数据完毕，等待其他线程写入完毕
线程Thread-0写入数据完毕，等待其他线程写入完毕
Thread-0所有线程写入完毕，继续处理其他任务...
Thread-3所有线程写入完毕，继续处理其他任务...
Thread-1所有线程写入完毕，继续处理其他任务...
Thread-2所有线程写入完毕，继续处理其他任务...
CyclicBarrier重用
线程Thread-4正在写入数据...
线程Thread-5正在写入数据...
线程Thread-6正在写入数据...
线程Thread-7正在写入数据...
线程Thread-7写入数据完毕，等待其他线程写入完毕
线程Thread-5写入数据完毕，等待其他线程写入完毕
线程Thread-6写入数据完毕，等待其他线程写入完毕
线程Thread-4写入数据完毕，等待其他线程写入完毕
Thread-4所有线程写入完毕，继续处理其他任务...
Thread-5所有线程写入完毕，继续处理其他任务...
Thread-6所有线程写入完毕，继续处理其他任务...
Thread-7所有线程写入完毕，继续处理其他任务...
```
从执行结果可以看出，在初次的4个线程越过barrier状态后，又可以用来进行新一轮的使用。而CountDownLatch无法进行重复使用。