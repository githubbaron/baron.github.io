---
layout: post
title: "Thinking in Java Concurrence"
date: 2018-02-07 09:24:01
image: 'http://res.cloudinary.com/degzyaemb/image/upload/c_scale,w_750/v1517372363/Jesse_Eisenberg.png'
description: 《Thinking in java》中关于并发的总结与思考。
category: 'java'
tags:
- java
- concurrent
- thread
twitter_text:
introduction: 《Thinking in java》中关于并发的总结与思考。

---

### 1. Runnable 和 Thread的关系

Thread可以通过extends来继承并且实现其中的run()方法，而Runnable则是通过implements来实现其中的方法。
两者看似相同，但是却存在着非常大的区别。其中我们必须遵循
*Inherit less, interface more* 的原则，就是尽量以实现来代替继承。
原因是一个类可以实现很多接口但是却只能继承一个类。

Runnable 和 Thread的区别就是Thread是线程，而Runnable是任务，其中，你对Thread类没有任何控制权。
原因是：**你通过创建任务，并通过某种方式将一个线程附着到任务上，以使得这个线程可以驱动任务。**
而两者最大的不同就是，Thread是资源不共享的而Runnable是资源共享的(通过买票的例子可以得出此结论)。
所以需要好好理解并将两者分隔开。

### 2. synchronized 与 volatile

> `synchronized`的定义： 如果某个任务处于一个对标记为`synchronized`的方法的调用中，
那么在这个线程从该方法返回之前，**其他所有**要调用类中任何标记为`synchronized`方法的线程都会被阻塞。

需要理解synchronized的定义才能明白下列代码的含义。
```java
public class AtomicityTest implements Runnable {
    private int i = 0;
    public int getValue() { return i; }
    public synchronized void evenIncrement() { i++; i++ }
    public void run() {
        while(true)
            evenIncrement();
    }

    public static void main(String []args) {
        ExecutorService exec = Executors.newCachedThreadPool();
        AtomicityTest at = new AtomicityTest();
        exec.execute(at);
        while (true) {
            int val = at.getValue();
            if (val % 2 != 0) {
                System.out.println(val);
                System.exit(0);
            }
        }
    }
} /* Output: (Sample)
191583767
*/
```
如果你盲目的应用原子性概念，就有可能写出这样的代码，该程序找到基数值并结束。
尽管 `return i` 确实是原子性的，但是缺少同步使得数值可以在处于不稳定的中间状态被读取。
除此之外，由于 i 也不是 `volatile` 的，因此还存在可视性问题。

> `volatile`的可视性：如果你将一个域声明为`volatile`的，那么只要对这个域产生了写操作，那么所有的读操作就可以看到这个修改。
即使使用了本地缓存，情况也确实如此，`volatile`域或立即被写入到主存中，而读取操作就发生在主存中。

> 基本上，如果一个域可能会被多个任务同时访问，或者这些任务中至少有一个是写入任务，那么你就应该
将这个域设置为 `volatile` 的。如果你将一个域定义为 `volatile`，那么它就会告诉编译器不要执行任何移除
读取和写入操作的优化，这些操作的目的是用线程中的局部变量维护对这个域的精确同步。

这就意味着，在上述代码中，如果将i设置成volatile的，在线程 `at` 运行 `run()` 的过程中，
i的增加直接由 `getValue()`获取得到，导致了程序会很快的判断val为基数并返回结果。
需要理解并发中的原子性、可视性以及有序性的可以看这篇blog
<a href="https://my.oschina.net/wangnian/blog/668490">《java并发之原子性、可见性、有序性》</a>


### 3. synchronized锁和Lock对象的区别是：

1. synchronized使用的代码更少，代码更加优雅。
2. 但是Lock对象对于解决某些类型的问题来说更加的灵活。（比如ReentrantLock允许用户尝试获取
但最终未获取锁，如果其他人已经获取了这个锁，那你就可以决定离开去执行其他的事情，而不是等待知道这个锁
被释放）。

### 4.在其他对象上同步(synchronized同步控制块)

理解synchronized(otherObject)的这种 *在其他对象上同步* 的形式首先需要理解synchronized的定义，就是synchronized(otherObject)
这种方法可以使得在同个类中两个方法在运行的时候相互独立而不会堵塞而导致另一个方法无法运行。

```java
class DualSynch {
    private Object syncObject = new Object();
    public synchronized void f() {
        for (int i = 0; i < 5; i++) {
            System.out.println("f()");
            Thread.yield();
        }
    }
    public void g() {
        synchronized (syncObject) {
            for (int i = 0; i < 5; i++) {
                System.out.println("g()");
                Thread.yield();
            }
        }
    }
}

public class SyncObject {
    public static void main(String []args) {
        final DualSynch ds = new DualSynch();
        new Thread() {
            public void run() {
                ds.f();
            }
        }.start();
        ds.g();
    }
}
```

DualSyn.f()在this同步，而g()有一个在syncObject上同步的synchronized块。因此，这两个同步是相互独立的。
如果将g()方法中的syncObject该成this，**那么这两个方法就会相互堵塞**。

#### 进入阻塞状态的几种方式

1. sleep(milliseconds)
2. wait()将线程挂起，直到得到了notify()和notifyAll()的消息。
3. 任务在等待某个输入/输出完成。
4. 任务试图在某个对象上调用同步控制方法，但是对象锁不可用，因为另一个任务已经获取了这个锁。

#### shutdown() 和 shutdownNow()的区别是

这里截取自stackOverflow的高票回答：

> * shutdown() will just tell the executor service that it can't accept new tasks, but the already submitted tasks continue to run
> * shutdownNow() will do the same AND will try to cancel the already submitted tasks by interrupting the relevant threads. Note that if your tasks ignore the interruption, shutdownNow will behave exactly the same way as shutdown.

#### 中断

```java
class SleepBlocked implements Runnable {

    @Override
    public void run() {
        try {
            TimeUnit.SECONDS.sleep(100);
        } catch (InterruptedException e) {
            print("InterruptedException");
        }
        print("Exiting Sleeping.run()");
    }
}

// 通过传入System.in并不传入任何值使其陷入等待中
class IOBlocked implements Runnable {
    private InputStream in;
    public IOBlocked(InputStream is) {
        in = is;
    }
    @Override
    public void run() {
        try {
            print("Waiting for read()");
            in.read();
        } catch (IOException e) {
            if (Thread.currentThread().isInterrupted())
                print("Interrupted from blocked I/O");
            else
                throw new RuntimeException(e);
        }
        print("Exiting IOBlocked.run()");
    }
}

// 通过在构造函数中创建线程占用f()方法来使其陷入死锁状态
class SynchronizedBlocked implements Runnable {
    private synchronized void f() {
        while (true)
            Thread.yield();
    }
    public SynchronizedBlocked() {
        new Thread() {
            public void run() {
                f();
            }
        }.start();
    }
    @Override
    public void run() {
        print("Trying to call f()");
        f();
        print("Exiting SynchronizedBlocked.run()");
    }
}

public class Interrupting {
    private static ExecutorService exec = Executors.newCachedThreadPool();
    static void test(Runnable r) throws InterruptedException {
        Future<?> f = exec.submit(r);
        TimeUnit.MILLISECONDS.sleep(100);
        print("Interrupting " + r.getClass().getName());
        f.cancel(true);
        print("Interrupted send to " + r.getClass().getName());
    }

    public static void main(String []args) throws Exception {
        test(new SleepBlocked());
        test(new IOBlocked(System.in));
        test(new SynchronizedBlocked());
        TimeUnit.SECONDS.sleep(3);
        print("Aborting with System.exit(0)");
        System.exit(0);
    }
}

```
通过SleepBlocked、IOBlocked和SynchronizedBlocked代码可以看出
阻塞中的任务是可以中断的，但是I/O和在synchronized块上等待的是不可中断的。
