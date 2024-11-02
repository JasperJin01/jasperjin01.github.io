---
title: JUC学习笔记01
layout: default
parent: Java并发
---


- TOC
{:toc}



### Java本身就是多线程的

Java的JVM是多线程的。一个简单的Java程序启动后，JVM就会创建多个线程，例如下面的例子：

```java
public class OnlyMain {
    public static void main(String[] args) {
        //Java 虚拟机线程系统的管理接口
        ThreadMXBean threadMXBean = ManagementFactory.getThreadMXBean();
        // 不需要获取同步的monitor和synchronizer信息，仅仅获取线程和线程堆栈信息
        ThreadInfo[] threadInfos = threadMXBean.dumpAllThreads(false, false);
        // 遍历线程信息，仅打印线程ID和线程名称信息
        for (ThreadInfo threadInfo : threadInfos) {
            System.out.println("[" + threadInfo.getThreadId() + "] "
                    + threadInfo.getThreadName());
        }
    }
}
```

输出：

```
[5] Monitor Ctrl-Break
[4] Signal Dispatcher
[3] Finalizer
[2] Reference Handler
[1] main
```

其中，main线程是用来执行main函数的。==Finalizer线程是管理Object中finalize方法？==



> ThreadMXBean 一个管理JVM的类，据说在监控java性能内存等经常需要



### Java创建线程

方法1：继承Thread类

```java
class StartThread {
    public static class FirstThread extends Thread {
        @Override
        public void run() {
            System.out.print("hello world, ");
            // 通过静态方法获取当前实例的名称
            System.out.println(Thread.currentThread().getName());
        }
    }
    public static void main(String[] args) {
        FirstThread thread = new FirstThread();
        thread.setName("firstThread");
        thread.start(); // 调用start方法，用多线程执行run函数
    }
}
```

> 扩展一下Thread.currentThread()
>
> `Thread.currentThread()`：这是一个Thread类的静态方法，获取执行当前代码的线程的 Thread 对象。是一个本地（native）方法，由 JVM 直接实现。这个方法会根据每个线程的执行上下文返回当前正在执行的 Thread 对象。每个线程在 JVM 级别都有唯一的线程控制块TCB，通过这个控制块，JVM 能够快速识别出当前运行的线程，从而提供一个指向该线程对象的引用。

方法2：实现Runable接口，创建Thread类，将实现Runable接口的类作为初始化参数

```java
class StartThread2 {
    public static class MyRunnable implements Runnable {
        @Override
        public void run() {
            String threadName = Thread.currentThread().getName();
            System.out.println("I am " + threadName);
        }
    }
    public static void main(String[] args) {
        Runnable myRunnable = new MyRunnable(); // 创建一个 Runnable 对象
        // Thread thread = new Thread(myRunnable);
        Thread thread = new Thread(myRunnable, "myThread"); // 传入 Runnable 对象
        thread.start();
        // thread.run();
    }
}
```

* 如果直接调用run()，这个时候就和调用一个类中的函数没区别，也就是线程main执行的run函数（不是线程myThread）

方法3：lambda 表达式创建线程

```java
class StartThread3 {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            // 函数内部相当于run函数的实现
            System.out.println("Hello lambda");
        });
        thread.start();
    }
}
```









### 守护线程 DaemonThread







### synchronized

`Synchronized`可以保证在一个时间点上只有一个线程可以执行被同步的方法或代码块，从而确保线程安全。

**同步实例方法**：在实例方法上使用`synchronized`，会锁定调用该方法的对象实例（即`this`对象），确保同一时间只有一个线程可以访问该对象的同步方法。

```java
public class Counter {
    private int count = 0;
    // 同步实例方法
    public synchronized void increment() {
        count++;
    }
    public int getCount() { return count; }
}
```

在`increment`方法上添加`synchronized`，会对`Counter`对象加锁，确保只有一个线程可以访问该方法。



**同步静态方法**：在静态方法上使用`synchronized`，会锁定整个类（`Class`对象），即使多个线程访问不同的实例，也只能有一个线程访问该同步静态方法。

```java
public class Counter {
    private static int count = 0;

    // 同步静态方法
    public static synchronized void increment() {
        count++;
    }

    public static int getCount() {
        return count;
    }
}
```

`synchronized`静态方法锁住的是`Counter`类本身，而不是某个具体的实例。



==问题：这个锁指的是什么啊？java里面的锁机制吗？java里面有什么锁机制啊？==



**同步代码块**：只锁定synchronized指定的代码块，相比同步方法，具有更细粒度的锁。同时可以指定锁的对象（即synchronized小括号中的内容）

```java
public class Counter {
    private int count = 0;

    public void increment() {
        // 给当前对象实例加锁，仅锁定代码块部分（而不是整个方法）
        synchronized (this) {
            count++;
        }
    }
    public int getCount() { return count; }
}

```



```java
```





> 复习：Java final关键字
>
> `final`关键字可以用于**变量**、**方法**和**类**，主要作用是使被修饰的元素不可更改或被继承，从而增强代码的安全性和可读性。
>
> **final 修饰基本数据类型**：一旦赋值，就不能修改其值。
>
> **final 修饰引用类型（对象）**：一旦初始化，引用地址不能改变，但对象本身的内容可以修改。
>
> **final 修饰方法**：当`final`修饰一个方法时，该方法不能被子类重写。
>
> **final 修饰类**：当`final`修饰一个类时，该类不能被继承





### Sleep方法不会释放锁

sleep()方法具有不会释放锁的特性。也就是说在同步代码块中调用`Thread.sleep()`让线程休眠而不释放锁，其他等待该锁的线程只能等待，锁才会释放。

```java
class SleepLock {
    private final Object lock = new Object();

    public static void main(String[] args) {
        SleepLock sleepTest = new SleepLock();
        Thread threadA = sleepTest.new ThreadSleep();
        threadA.setName("ThreadSleep");
        Thread threadB = sleepTest.new ThreadNotSleep();
        threadB.setName("ThreadNotSleep");

        threadA.start();
        try {
            Thread.sleep(1000); // 保证线程B在线程A的
            System.out.println(" Main slept");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        threadB.start();
    }

    private class ThreadSleep extends Thread {
        @Override
        public void run() {
            String threadName = Thread.currentThread().getName();
            System.out.println(threadName + " will take the lock!!!!!");
            try {
                synchronized (lock) {
                    System.out.println(threadName + " taking the lock!!!!!");
                    Thread.sleep(5000);
                    System.out.println("Finish the work: " + threadName + "!!!!!");
                }
            } catch (InterruptedException e) {
                //e.printStackTrace();
            }
        }
    }

    private class ThreadNotSleep extends Thread {
        @Override
        public void run() {
            String threadName = Thread.currentThread().getName();
            System.out.println(threadName + " will take the lock time=" + System.currentTimeMillis());
            synchronized (lock) {
                System.out.println(threadName + " taking the lock time=" + System.currentTimeMillis());
                System.out.println("Finish the work: " + threadName);
            }
        }
    }
}

```



### Java 不推荐的一些管理线程函数

不建议在Java中调用`Thread`类中的`destroy()`、`stop()`、`suspend()`等方法。原因在于，这些方法存在严重的安全性和资源管理问题，可能会导致程序状态不一致、资源泄漏等难以预料的后果。以下是对这些方法的具体分析以及推荐的替代方法。

**`stop()` 方法**：`stop()`方法在任何时刻强制终止线程，不会执行finally块或释放锁资源。这会导致线程持有的锁不被释放，导致程序死锁或数据不一致的问题。

```java
Thread t = new Thread(() -> {
    synchronized (SomeObject.class) {
        // 执行某些操作
    }
});

t.start();
t.stop(); // 不推荐，可能会导致 SomeObject.class 的锁永远不被释放
```

**替代方案**：使用一个**中断标志位**来安全终止线程。例如，可以使用`volatile`修饰的布尔变量作为标志，让线程定期检查标志的状态，决定是否终止。

public class SafeStopExample implements Runnable {
    private volatile boolean running = true;

```java
public class SafeStopExample implements Runnable {
    private volatile boolean running = true;

    public void stopRunning() {
        running = false;
    }

    @Override
    public void run() {
        while (running) {
            // 线程的任务逻辑
        }
        System.out.println("Thread has stopped safely.");
    }

    public static void main(String[] args) throws InterruptedException {
        SafeStopExample example = new SafeStopExample();
        Thread t = new Thread(example);
        t.start();

        Thread.sleep(1000);
        example.stopRunning(); // 安全停止线程
    }
}
```
**`suspend()` 和 `resume()` 方法**：`suspend()`会直接挂起线程，但线程在挂起时不会释放所持有的锁，导致可能的死锁问题。

**替代方案**：可以使用`wait()`和`notify()`，并控制线程的执行来实现类似的挂起和恢复功能。

```java
public class SafeSuspendExample {
    private final Object lock = new Object();
    private volatile boolean suspended = false;

    public void suspend() {
        suspended = true;
    }

    public void resume() {
        synchronized (lock) {
            suspended = false;
            lock.notify();
        }
    }

    public void doWork() {
        synchronized (lock) {
            while (suspended) {
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            }
            // 线程的任务逻辑
        }
    }
}
```

==不太理解啊？给我讲讲什么意思？==



**`destroy()`方法**更极端，它会立刻销毁线程对象。由于线程的生命周期强行终止，线程的状态无法得到恢复，且任何未执行的`finally`块都不会运行。由于安全风险太大，这个方法从未实现过，直接被弃用。





### 线程中断(interrupt)

一个线程内部有一个记录是否中断的标志位 interrupt_flag，当这个标志位是 true 时，表示当前线程应该是中断的。当然java并不会像suspend()或destroy()那样直接挂起或销毁线程，而是允许程序在run函数中处理中断，主动结束线程。

关于线程中断有三个主要函数：interrupt()、interrupted() 和 isInterrupted()

* `interrupt()`：调用目标线程的 `interrupt()` 方法，表示通知该线程应尽快停止当前任务。例如，在main线程中想要中断thread这个线程时，只需要在main中调用thread.interrupt()。<br>这个方法可以理解为**把中断标志位 interrupt_flag 设为了true**。<br>而当**线程处于阻塞状态（例如`sleep()`、`wait()`、`join()`等）**，调用`interrupt()`会导致这些方法立即抛出`InterruptedException`异常，通知线程它被中断。
* `isInterrupted()`：判断中断标志位 interrupt_flag
* `Thread.interrupted()`：这是一个静态内部类，他会在interrupt_flag为true时返回true，同时把标志位interrupt_flag重设为false，即**重置中断标志**。<br>此外，InterruptedException也会重置中断标志，即捕获到InterruptedException异常后，

==为什么要重置中断标志？重置中断标志有什么作用？？==

```java
// Thread.java 中部分代码的节选
 public static boolean interrupted() {
    return currentThread().isInterrupted(true);
}

public boolean isInterrupted() {
    return isInterrupted(false);
}

private native boolean isInterrupted(boolean ClearInterrupted);
```



**<font color=red>阻塞方法中抛出InterruptedException异常后，如果需要继续中断，需要手动再中断一次。</font>**



一定要区分下面两个代码段的区别：

```java
public static void main(String[] args) throws InterruptedException {
    Thread thread1 = new Thread(()->{
        while (!Thread.currentThread().isInterrupted()) {
            System.out.print("1");
        }
    });
    thread1.start();
    sleep(100);
    thread1.interrupt();
}
```



```java
public static void main(String[] args) throws InterruptedException {
	Thread thread2 = new Thread(()->{
        while (!Thread.currentThread().isInterrupted()) {
            System.out.println("1111");
            try {
                sleep(100);
            } catch (InterruptedException e) {
                System.out.println("当前进程被中断");
                Thread.currentThread().interrupt();
            }
            System.out.println("2222");
        }
    });
    thread2.start();
    sleep(1000);
    System.out.println("interrupt thread2");
    thread2.interrupt();
    System.out.println("main end");
}
```

thread1和thread2的区别在于，当main线程调用interrupt()中断时，**thread1正处于运行状态，而thread2处于阻塞状态**。

* **如果线程在运行过程中（非阻塞状态）**，调用`interrupt()`只会设置线程的中断标志位为`true`，线程可以通过`Thread.interrupted()`或`isInterrupted()`方法来检测这个标志位的状态。

* **如果线程在阻塞状态（例如`sleep()`、`wait()`、`join()`等）**，调用`interrupt()`会导致这些方法立即抛出`InterruptedException`异常，通知线程它被中断，同时会**重置中断标志位**为`false`。

在代码段1中，调用interrupt()使得





### 两阶段锁





### volatile





### ThreadLocal







