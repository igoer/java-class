#Object
Object类是所有Java类的祖先。每个类都使用 Object 作为超类。所有对象（包括数组）都实现这个类的方法。

在不明确给出超类的情况下，Java会自动把Object作为要定义类的超类。可以使用类型为Object的变量指向任意类型的对象。Object类有一个默认构造方法pubilc Object()，在构造子类实例时，都会先调用这个默认构造方法。

Object类的变量只能用作各种值的通用持有者。要对他们进行任何专门的操作，都需要知道它们的原始类型并进行类型转换。例如：
```
Object obj = new MyObject();
MyObject x = (MyObject)obj;
```

##Api
> Object()

默认构造方法。

> clone()

创建并返回此对象的一个副本。

> equals(Object obj)

指示某个其他对象是否与此对象“相等”。

> finalize()

当垃圾回收器确定不存在对该对象的更多引用时，由对象的垃圾回收器调用此方法。

> getClass()

返回一个对象的运行时类。

> hashCode()

返回该对象的哈希码值。

> toString()

返回该对象的字符串表示。

> notify()

唤醒在此对象监视器上等待的单个线程。

> notifyAll()

唤醒在此对象监视器上等待的所有线程。

> wait()

导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法。

> wait(long timeout)

导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者超过指定的时间量。

> wait(long timeout, int nanos)

导致当前的线程等待，直到其他线程调用此对象的 notify() 方法或 notifyAll() 方法，或者其他某个线程中断当前线程，或者已超过某个实际时间量。

##clone()
```
protected Object clone() throws CloneNotSupportedException
```
创建并返回此对象的一个副本。“副本”的准确含义可能依赖于对象的类。一般来说，对于任何对象 x，如果表达式：
```
x.clone() != x
```
是正确的，则表达式：
```
x.clone().getClass() == x.getClass()
```
将为 true，但这些不是绝对条件。一般情况下是：
```
x.clone().equals(x)
```
将为 true，但这不是绝对条件。

返回的对象应该通过调用 super.clone 获得。如果一个类及其所有的超类（Object 除外）都遵守此约定，则 x.clone().getClass() == x.getClass()。

此方法返回的对象应该独立于该对象（正被克隆的对象）。要获得此独立性，在 super.clone 返回对象之前，有必要对该对象的一个或多个字段进行修改。这通常意味着要复制包含正在被克隆对象的内部“深层结构”的所有可变对象，并使用对副本的引用替换对这些对象的引用。如果一个类只包含基本字段或对不变对象的引用，那么通常不需要修改 super.clone 返回的对象中的字段。

Object 类的 clone 方法执行特定的克隆操作。首先，如果此对象的类不能实现接口 **Cloneable**，则会抛出 **CloneNotSupportedException**。注意：所有的数组都被视为实现接口 Cloneable。否则，此方法会创建此对象的类的一个新实例，并像通过分配那样，严格使用此对象相应字段的内容初始化该对象的所有字段；这些字段的内容没有被自我克隆。所以，此方法执行的是该对象的“浅表复制”，而不“深层复制”操作。

Object 类本身不实现接口 Cloneable，所以在类为 Object 的对象上调用 clone 方法将会导致在运行时抛出异常。

clone() 方法是protected修饰的，覆写clone()方法的时候需要写成public，才能让类外部的代码调用。

## equals(Object obj)
```
public boolean equals(Object obj)
```
子类覆写equals()方法时，应该同时覆写hashCode()方法，反之亦然。

用于测试某个对象是否同另一个对象相等，它在Object类中的实现是判断两个对象是否指向同一块内存区域。
```
public boolean equals(Object obj) {
	return (this == obj);
}
```
这中测试用处不大，因为即使内容相同的对象，内存区域也是不同的。如果想测试对象是否相等，就需要覆盖此方法，进行更有意义的比较。

例如：
```
class Employee{
	//此例子来自《java核心技术》卷一
    public boolean equals(Object otherObj){
        //快速测试是否是同一个对象
        if(this == otherObj) return true;
        //如果显式参数为null，必须返回false
        if(otherObj == null) reutrn false;
        //如果类不匹配，就不可能相等
        if(getClass() != otherObj.getClass()) return false;

        //现在已经知道otherObj是个非空的Employee对象
        Employee other = (Employee)otherObj;
        //测试所有的字段是否相等
        return name.equals(other.name) {
            && salary == other.salary
            && hireDay.equals(other.hireDay);
		}
    }
}
```
**Java语言规范要求equals方法具有下面的特点：**
1、自反性：对于任何非空引用值 x，x.equals(x) 都应返回 true。

2、对称性：对于任何非空引用值 x 和 y，当且仅当 y.equals(x) 返回 true 时，x.equals(y) 才应返回 true。

3、传递性：对于任何非空引用值 x、y 和 z，如果 x.equals(y) 返回 true，并且 y.equals(z) 返回 true，那么 x.equals(z) 应返回 true。

4、一致性：对于任何非空引用值 x 和 y，多次调用 x.equals(y) 始终返回 true 或始终返回 false，前提是对象上 equals 比较中所用的信息没有被修改。

5、对于任何非空引用值 x，x.equals(null) 都应返回 false。

只有当继承Object的类覆写（override）了equals()方法之后，继承类实现了用equals()方法比较两个对象是否相等，才可以说equals()方法与==的不同。

##hashcode()
```
public int hashCode()
```
当你覆写（override）了equals()方法之后，必须也覆写hashCode()方法，反之亦然。

这个方法返回一个整型值（hash code value），如果两个对象被equals()方法判断为相等，那么它们就应该拥有同样的hash code。

Object类的hashCode()方法为不同的对象返回不同的值，Object类的hashCode值表示的是对象的地址。

**hashCode的一般性契约（需要满足的条件）如下：**

1、在Java应用的一次执行过程中，如果对象用于equals比较的信息没有被修改，那么同一个对象多次调用hashCode()方法应该返回同一个整型值。

应用的多次执行中，这个值不需要保持一致，即每次执行都是保持着各自不同的值。

2、如果equals()判断两个对象相等，那么它们的hashCode()方法应该返回同样的值。

3、并没有强制要求如果equals()判断两个对象不相等，那么它们的hashCode()方法就应该返回不同的值。

即，两个对象用equals()方法比较返回false，它们的hashCode可以相同也可以不同。但是，应该意识到，为两个不相等的对象产生两个不同的hashCode可以改善哈希表的性能。

##getClass()
```
public final Class<?> getClass()
```
返回此 Object 的运行时类。

利用这个方法就可以获得一个实例的类型类。类型类指的是代表一个类型的类，因为一切皆是对象，类型也不例外，在Java使用类型类来表示一个类型。所有的类型类都是Class类的实例。

例如，有如下一段代码：
```
A a = new A();

if(a.getClass()==A.class) {
	System.out.println("equal");
} else {
	System.out.println("unequal");
}
```
输出：
```
equal
```
可以看到，对象a是A的一个实例，A是某一个类，在if语句中使用a.getClass()返回的结果正是A的类型类，在Java中表示一个特定类型的类型类可以用“类型.class”的方式获得，因为a.getClass()获得是A的类型类，也就是A.class，因此上面的代码执行的结果就是打印出“equal”。


##toString()
```
public String toString()
```
当打印引用，如调用System.out.println()时，会自动调用对象的toString()方法，打印出引用所指的对象的toString()方法的返回值，因为每个类都直接或间接地继承自Object，因此每个类都有toString()方法。

Object类中的toString()方法定义如下：
```
public String toString() {
    return getClass().getName() + "@" + Integer.toHexString(hashCode());
}
```

##wait, notfiy, notfiyAll
```
/**
 * 随机选择一个在该对象上调用wait方法的线程，解除其阻塞状态。该方法只能在同步方法或同步块内部调用。
 * 如果当前线程不是锁的持有者，该方法抛出一个IllegalMonitorStateException异常。
 */
public final native void notify();

/**
 * 解除所有那些在该对象上调用wait方法的线程的阻塞状态。该方法只能在同步方法或同步块内部调用。
 * 如果当前线程不是锁的持有者，该方法抛出一个IllegalMonitorStateException异常。
 */
public final native void notifyAll();

/**
 * 导致线程进入等待状态，直到它被其他线程通过notify()或者notifyAll唤醒。
 * 如果当前线程不是锁的持有者，该方法抛出一个IllegalMonitorStateException异常。
 */
public final native void wait() throws InterruptedException;

/**
 * 导致线程进入等待状态直到它被其他线程通过notify()或者notifyAll唤醒或者经过指定的时间。
 * 如果当前线程不是锁的持有者，该方法抛出一个IllegalMonitorStateException异常。
 */
public final native void wait(long timeout) throws InterruptedException;

/**
 * 导致线程进入等待状态直到它被其他线程通过notify()或者notifyAll唤醒或者经过指定的时间。
 * 如果当前线程不是锁的持有者，该方法抛出一个IllegalMonitorStateException异常。
 */
public final native void wait(long timeout, int nano) throws InterruptedException;
```
从这三个方法的文字描述可以知道以下几点信息：

1）wait()、notify()和notifyAll()方法是本地方法，并且为final方法，无法被重写。

2）调用某个对象的 wait() 方法能让当前线程阻塞，并且当前线程必须拥有此对象的monitor（即锁）。

3）调用某个对象的 notify() 方法能够唤醒一个正在等待这个对象的 monitor 的线程，如果有多个线程都在等待这个对象的 monitor，则只能唤醒其中一个线程。

4）调用 notifyAll() 方法能够唤醒所有正在等待这个对象的 monitor 的线程。

> PS：
> 有朋友可能会有疑问：为何这三个不是Thread类声明中的方法，而是Object类中声明的方法（当然由于Thread类继承了Object类，所以Thread也可以调用者三个方法）？其实这个问题很简单，由于每个对象都拥有monitor（即锁），所以让当前线程等待某个对象的锁，当然应该通过这个对象来操作了。而不是用当前线程来操作，因为当前线程可能会等待多个线程的锁，如果通过线程来操作，就非常复杂了。

上面已经提到，如果调用某个对象的wait()方法，当前线程必须拥有这个对象的monitor（即锁），因此调用wait()方法必须在同步块或者同步方法中进行（synchronized块或者synchronized方法）。

调用某个对象的wait()方法，相当于让当前线程交出此对象的monitor，然后进入等待状态，等待后续再次获得此对象的锁（Thread类中的sleep方法使当前线程暂停执行一段时间，从而让其他线程有机会继续执行，但它并不释放对象锁）。

notify()方法能够唤醒一个正在等待该对象的monitor的线程，当有多个线程都在等待该对象的monitor的话，则只能唤醒其中一个线程，具体唤醒哪个线程则不得而知。

nofityAll()方法能够唤醒所有正在等待该对象的monitor的线程，这一点与notify()方法是不同的。

同样地，调用某个对象的notify()方法，当前线程也必须拥有这个对象的monitor，因此调用notify()方法必须在同步块或者同步方法中进行（synchronized块或者synchronized方法）

这里要注意一点：notify()和notifyAll()方法只是唤醒等待该对象的monitor的线程，并不决定哪个线程能够获取到monitor。

>举个简单的例子：
假如有三个线程Thread1、Thread2和Thread3都在等待对象objectA的monitor，此时Thread4拥有对象objectA的monitor，当在Thread4中调用objectA.notify()方法之后，Thread1、Thread2和Thread3只有一个能被唤醒。注意，被唤醒不等于立刻就获取了objectA的monitor。假若在Thread4中调用objectA.notifyAll()方法，则Thread1、Thread2和Thread3三个线程都会被唤醒，至于哪个线程接下来能够获取到objectA的monitor就具体依赖于操作系统的调度了。

上面尤其要注意一点，一个线程被唤醒不代表立即获取了对象的monitor，只有等调用完notify()或者notifyAll()并退出synchronized块，释放对象锁后，其余线程才可获得锁执行。

下面看一个简单的示例：
```
public class XS {

	private Object obj = new Object();

    public static void main(String[] args) throws InterruptedException  {
    	XS xs = new XS();
    	T1 t1 = xs.new T1();
    	T2 t2 = xs.new T2();

    	// t1线程先启动, 并且获取到obj的对象锁
    	t1.start();
    	// 确保t1能够顺利的获取到obj对象锁
    	Thread.sleep(1000);
    	t2.start();
    }

    class T1 extends Thread {
    	@Override
    	public void run() {
    		synchronized (obj) {
				try {
					System.out.println(Thread.currentThread().getName() + " 开始等待唤醒");
					// 调用wait()当前线程进入等待状态, 并交出obj对象锁
					obj.wait();
					System.out.println(Thread.currentThread().getName() + " 被唤醒, 获取了锁");
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
    	}
    }

 class T2 extends Thread {
    	@Override
    	public void run() {
    		synchronized (obj) {
				obj.notify();
				System.out.println(Thread.currentThread().getName() + " 调用了notify");
				System.out.println(Thread.currentThread().getName() + " 释放锁");
			}
    	}
    }

}
```
输出结果：
```
Thread-0 开始等待唤醒
Thread-1 调用了notify
Thread-1 释放锁
Thread-0 被唤醒, 获取了锁
```
