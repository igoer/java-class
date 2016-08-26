#Thread

```
java.lang.Object
	java.lang.Thread

All Implemented Interfaces:
	Runnable
Direct Known Subclasses:
	ForkJoinWorkerThread
```
##构造方法
```
Thread()

Thread(Runnable target)

Thread(Runnable target, String name)

Thread(String name)

Thread(ThreadGroup group, Runnable target)

Thread(ThreadGroup group, Runnable target, String name)

Thread(ThreadGroup group, Runnable target, String name, long stackSize)

Thread(ThreadGroup group, String name)
```
####Runnable target
实现了Runnable接口的类的实例。要注意的是Thread类也实现了Runnable接口，因此，从Thread类继承的类的实例也可以作为target传入这个构造方法。

####String name
线程的名子。这个名子可以在建立Thread实例后通过Thread类的setName方法设置。如果不设置线程的名子，线程就使用默认的线程名：Thread-N，N是线程建立的顺序，是一个不重复的正整数。

####ThreadGroup group
当前建立的线程所属的线程组。如果不指定线程组，所有的线程都被加到一个默认的线程组中。

#### long stackSize
线程栈的大小，这个值一般是CPU页面的整数倍。如x86的页面大小是4KB.在x86平台下，默认的线程栈大小是12KB。

##Thread 6种状态
线程从创建到最终的消亡，要经历若干个状态。一般来说，线程包括以下这几个状态：创建(new)、就绪(runnable)、运行(running)、阻塞(blocked)、time waiting、waiting、消亡（dead）。

当需要新起一个线程来执行某个子任务时，就创建了一个线程。但是线程创建之后，不会立即进入就绪状态，因为线程的运行需要一些条件（比如内存资源，在前面的JVM内存区域划分一篇博文中知道程序计数器、Java栈、本地方法栈都是线程私有的，所以需要为线程分配一定的内存空间），只有线程运行需要的所有条件满足了，才进入就绪状态。

当线程进入就绪状态后，不代表立刻就能获取CPU执行时间，也许此时CPU正在执行其他的事情，因此它要等待。当得到CPU执行时间之后，线程便真正进入运行状态。

线程在运行状态过程中，可能有多个原因导致当前线程不继续运行下去，比如用户主动让线程睡眠（睡眠一定的时间之后再重新执行）、用户主动让线程等待，或者被同步块给阻塞，此时就对应着多个状态：time waiting（睡眠或等待一定的事件）、waiting（等待被唤醒）、blocked（阻塞）。

当由于突然中断或者子任务执行完毕，线程就会被消亡。

Thread线程的6种状态为一个 **java.lang.Thread.State** 的枚举类型，通过 Thread 的 **State static getState()** 方法获取。

####State.NEW
至今尚未启动的线程的状态。

当使用new一个新线程时，如new Thread(r)，但还没有执行start(),线程还没有开始运行，这时线程的状态就是NEW。

####State.RUNNABLE
可运行线程的线程状态。

当start()方法被调用时，线程就进入RUNNABLE状态。此时的线程可能正在运行，也可能没有运行。

####State.BLOCKED
受阻塞并且正在等待监视器锁的某一线程的线程状态。

下列情况会进入阻塞状态：

　　1.等待某个操作的返回，例如IO操作，该操作返回之前，线程不会继续下面的代码。

　　2.等待某个“锁”，在其他线程或程序释放这个“锁”之前，线程不会继续执行。

　　3.等待一定的触发条件。

　　4.线程执行了sleep方法。

　　5.线程被suspend()方法挂起。

一个被阻塞的线程在下列情况下会被重新激活：

　　1.执行了sleep()方法，睡眠时间已到。

　　2.等待的其他线程或程序持有的“锁”已被释放。

　　3.正在等待触发条件的线程，条件得到满足。

　　4.执行了suspend()方法，被调用了resume()方法。

　　5.等待的操作返回的线程，操作正确返回。

####State.WAITING
某一等待线程的线程状态。

线程因为调用了Object.wait()或Thread.join()而未运行，就会进入WAITING状态。

####State.TIMED_WAITING
具有指定等待时间的某一等待线程的线程状态。

线程因为调用了Thread.sleep()，或者加上超时值来调用Object.wait()或Thread.join()而未运行，则会进入TIMED_WAITING状态。

####State.TERMINATED
已终止线程的线程状态。

线程已运行完毕。它的run()方法已正常结束或通过抛出异常而结束。

**下面这副图描述了线程从创建到消亡之间的状态：**

![](http://oci7omhv3.bkt.clouddn.com/github/061045374695226.jpg)

##上下文切换

对于单核CPU来说（对于多核CPU，此处就理解为一个核），CPU在一个时刻只能运行一个线程，当在运行一个线程的过程中转去运行另外一个线程，这个叫做线程上下文切换（对于进程也是类似）。

由于可能当前线程的任务并没有执行完毕，所以在切换时需要保存线程的运行状态，以便下次重新切换回来时能够继续切换之前的状态运行。举个简单的例子：比如一个线程A正在读取一个文件的内容，正读到文件的一半，此时需要暂停线程A，转去执行线程B，当再次切换回来执行线程A的时候，我们不希望线程A又从文件的开头来读取。

因此需要记录线程A的运行状态，那么会记录哪些数据呢？因为下次恢复时需要知道在这之前当前线程已经执行到哪条指令了，所以需要记录程序计数器的值，另外比如说线程正在进行某个计算的时候被挂起了，那么下次继续执行的时候需要知道之前挂起时变量的值时多少，因此需要记录CPU寄存器的状态。所以一般来说，线程上下文切换过程中会记录程序计数器、CPU寄存器状态等数据。

说简单点的：对于线程的上下文切换实际上就是 存储和恢复CPU状态的过程，它使得线程执行能够从中断点恢复执行。

虽然多线程可以使得任务执行的效率得到提升，但是由于在线程切换时同样会带来一定的开销代价，并且多个线程会导致系统资源占用的增加，所以在进行多线程编程时要注意这些因素。

## 常用 Api
Thread 部分源码如下：
```
public class Thread inplements Runnable {

	private static native void registerNatives();

    static {
    	registerNatives();
    }

    private char name[];

    private int priority;

    private Thread threadQ;

    private boolean daemon = false;

    private Runnable target;

    private ThreadGroup group;
}
```
Thread类实现了Runnable接口，在 Thread 类中，有一些比较关键的属性：
name 是表示Thread的名字，可以通过 Thread 类的构造器中的参数来指定线程名字。
priority 表示线程的优先级（最大值为10，最小值为1，默认值为5）。
daemon 表示线程是否是守护线程。
target 表示要执行的任务。

###线程基本属性方法
> getId()

```
public long getId()
```
用来获取线程的ID

> getName()、setName(String name)

```
public final String getName()

public final void setName(String name)
```
用来得到或者设置线程名称，这两个方法为 final 修饰，子类不能进行重写。

> getPriority()、setPriority(int newPriority)

```
public final int getPriority()

public final void setPriority(int newPriority)
```
用来获取和设置线程优先级。

> setDaemon()、isDaemon()

```
public final void setDaemon(boolean on)

public final boolean isDaemon()
```
用来设置线程是否成为守护线程和判断线程是否是守护线程，这两个方法为 final 修饰，子类不能进行重写。

**守护线程和用户线程的区别在于：守护线程依赖于创建它的线程，而用户线程则不依赖。**

举个简单的例子：如果在main线程中创建了一个守护线程，当main方法运行完毕之后，守护线程也会随着消亡。而用户线程则不会，用户线程会一直运行直到其运行完毕。在JVM中，像垃圾收集器线程就是守护线程。

###线程运行状态方法

> start()

```
public void start()
```
start()用来启动一个线程，当调用start方法后，系统才会开启一个新的线程来执行用户定义的子任务，在这个过程中，会为相应的线程分配需要的资源。

> run()

```
public void run()
```
实现 Runnable interface 方法

run()方法是不需要用户来调用的，当通过start方法启动一个线程之后，当线程获得了CPU执行时间，便进入run方法体去执行具体的任务。注意，继承Thread类必须重写run方法，在run方法中定义具体要执行的任务。

> sleep()

```
public static void sleep(long millis) throws InterruptedException

public static void sleep(long millis,int nanos) throws InterruptedException
```
sleep相当于让线程睡眠，交出CPU，让CPU去执行其他的任务。

但是有一点要非常注意，sleep方法不会释放锁，也就是说如果当前线程持有对某个对象的锁，则即使调用sleep方法，其他线程也无法访问这个对象。

例子如下：
```
public static void main(String[] args) throws InterruptedException {
    Object obj = new Object();

    new Thread(new Runnable() {
        @Override
        public void run() {
            synchronized (obj) {
                System.out.println(Thread.currentThread().getName() + " 获取了 obj 锁");
                // sleep 1000ms
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(Thread.currentThread().getName() + " 释放了 obj 锁");
            }
        }
    }).start();

    Thread.sleep(100); // 保证Thread-0 可以顺利拿到 obj 的锁

    new Thread(new Runnable() {
        @Override
        public void run() {
            synchronized (obj) {
                System.out.println(Thread.currentThread().getName() + " 获取了 obj 锁");
                System.out.println(Thread.currentThread().getName() + " 释放了 obj 锁");
            }
        }
    }).start();
}
```
输出结果如下：
```
Thread-0 获取了 obj 锁
Thread-0 释放了 obj 锁
Thread-1 获取了 obj 锁
Thread-1 释放了 obj 锁
```
从上面输出结果可以看出，当Thread-0进入睡眠状态之后，Thread-1并没有去执行具体的任务。只有当Thread-0执行完之后，此时Thread-0释放了对象锁，Thread-1才开始执行。

注意，如果调用了sleep方法，必须捕获InterruptedException异常或者将该异常向上层抛出。当线程睡眠时间满后，不一定会立即得到执行，因为此时可能CPU正在执行其他的任务。所以说调用sleep方法相当于让线程进入阻塞状态。

> yield()

```
public static void yield()
```
调用yield方法会让当前线程交出CPU权限，让CPU去执行其他的线程。它跟sleep方法类似，同样不会释放锁。

但是yield不能控制具体的交出CPU的时间，另外，yield方法只能让拥有相同优先级的线程有获取CPU执行时间的机会。

注意，调用yield方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新获取CPU执行时间，这一点是和sleep方法不一样的。

> join()

```
public final void join() throws InterruptedException

public final void join(long millis) throws InterruptedException

public final void join(long millis, int nanos) throws InterruptedException
```
假如在main线程中，调用thread.join方法，则main方法会等待thread线程执行完毕或者等待一定的时间。如果调用的是无参join方法，则等待thread执行完毕，如果调用的是指定了时间参数的join方法，则等待一定的事件。

例子如下：
```
public static void main(String[] args) throws InterruptedException {
    Thread t = new Thread(new Runnable() {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + " thread runing");
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + " thread over");
        }
    });
    t.start();
    t.join();

    System.out.println(Thread.currentThread().getName() + " thread over");
}
```
输出如下：
```
Thread-0 thread runing
Thread-0 thread over
main thread over
```
我们可以看到存在两个线程：主线程和线程t，主线程调用线程t的Join方法，导致主线程阻塞，直到t线程执行完毕，才返回到主线程中。

简单理解，在主线程中调用t.Join()，也就是在主线程中加入了t线程的代码，必须让t线程执行完毕之后，主线程（调用方）才能正常执行。

**实际上调用join方法是调用了Object的wait方法，这个可以通过查看源码得知。**

**wait方法会让线程进入阻塞状态，并且会释放线程占有的锁，并交出CPU执行权限。**

**由于wait方法会让线程释放对象锁，所以join方法同样会让线程释放对一个对象持有的锁。**

> interrupt()

```
public void interrupt()

```
interrupt，顾名思义，即中断的意思。单独调用interrupt方法可以使得处于阻塞状态的线程抛出一个异常，也就说，它可以用来中断一个正处于阻塞状态的线程；另外，通过interrupt方法和isInterrupted()方法来停止正在运行的线程。

例子如下：
```
public static void main(String[] args) throws InterruptedException {
    Thread t = new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + " 进入睡眠");
                Thread.sleep(1000);
                System.out.println(Thread.currentThread().getName() + " 睡眠完毕");
            } catch (InterruptedException e) {
                e.printStackTrace();
                System.out.println(Thread.currentThread().getName() + " 获得中断异常");
            }
            System.out.println(Thread.currentThread().getName() + " 执行完毕");
        }
    });
    t.start();

    Thread.sleep(200); // 确保Thread-0正常启动
    t.interrupt();
}
```
输出如下：
```
Thread-0 进入睡眠
java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at com.hp.ThreadTest$1.run(ThreadTest.java:79)
	at java.lang.Thread.run(Unknown Source)
Thread-0 获得中断异常
Thread-0 执行完毕
```
从这里可以看出，通过interrupt方法可以中断处于阻塞状态的线程。那么能不能中断处于非阻塞状态的线程呢？

看下面这个例子：
```
public static void main(String[] args) throws InterruptedException {
    Thread t = new Thread(new Runnable() {
        @Override
        public void run() {
            int i = 0;
            while (true) {
                System.out.println(i++);
            }
        }
    });
    t.start();

    Thread.sleep(100);
    t.interrupt();
}
```
运行该程序会发现，while循环会一直运行，所以说直接调用interrupt方法不能中断正在运行中的线程。

但是如果配合isInterrupted()能够中断正在运行的线程，因为调用interrupt方法相当于将中断标志位置为true，那么可以通过调用isInterrupted()判断中断标志是否被置位来中断线程的执行。

比如下面这段代码：
```
public static void main(String[] args) throws InterruptedException {
    Thread t = new Thread(new Runnable() {
        @Override
        public void run() {
            int i = 0;
            while(!Thread.currentThread().isInterrupted()) {
                System.out.println(i++);
            }
        }
    });
    t.start();

    Thread.sleep(1000);
    t.interrupt();
}
```
运行会发现，打印若干个值之后，while循环就停止打印了。

但是一般情况下不建议通过这种方式来中断线程，一般会在MyThread类中增加一个属性 isStop来标志是否结束while循环，然后再在while循环中判断isStop的值。

> stop()

stop方法已经是一个废弃的方法，它是一个不安全的方法。因为调用stop方法会直接终止run方法的调用，并且会抛出一个ThreadDeath错误，如果线程持有某个对象锁的话，会完全释放锁，导致对象状态不一致。所以stop方法基本是不会被用到的。

> destroy()

destroy方法也是废弃的方法。基本不会被使用到。

那么Thread类中的方法调用到底会引起线程状态发生怎样的变化呢？下面一幅图就是在上面的图上进行改进而来的：

![](http://oci7omhv3.bkt.clouddn.com/github/061046391107893.jpg)