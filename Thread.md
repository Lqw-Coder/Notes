# 多线程

## 进程与线程的关系：

* 进程：应用程序的执行实例；有独立的内存空间 和系统资源
* 将进程可以进一步细分 为线程，CPU调度和分派的最小单位

举例，QQ是一个进程，QQ又可以细分多个功能（接收消息、发送消息），每个功能都可以通过一个线程来实现。

多线程：前提为单核，**宏观并行，微观串行**，并不存在真正意义的并行

## 多线程的好处：

​	多线程可以充分的利用CPU资源，提高效率；

> 补充：多线程并不是 越多越好，原因： 每开启一个线程 会占用1M左右的内存，因此太多的线程会占用 大量的内存资源。 单线程有时候 性能也非常不错，例如NodeJs、Netty、NIO都是基于单线程的 多路复用技术，性能不比多线程差。

多线程 并不是绝对的可以提高效率。

## JDK中的线程安全/非安全的支持

Hashtable：线程安全，效率低

HashMap：非线程安全，效率高

StringBuffer：线程安全

StringBuilder: 非线程安全

ArrayList：非线程安全

* 非线程安全，可能会出现一些数据不安全的问题，甚至报错，例如并发添加数据时会因增容问题出现越界
* 设计之初没有过多的考虑并发问题,使用迭代器迭代时修改数据会出现**ConcurrentModificationException**

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class ArrayListDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("aaa") ;
        list.add("bb") ;
        list.add("cc") ;
        Iterator<String> iter = list.iterator();
        while(iter.hasNext()){
            System.out.println(iter.next());

            list.add("xxx");
        }
    }
}
```

以下为iter.next()方法

```java
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }
```

* 问题原因
  * modCoutn是迭代器实际的指向位置；expectedModCount是迭代器期望的指向位置；为了防止数据被篡改，arraylist会判断二者是否相等。
  * 通过源码可知：ArrayList的add()会增加modCount 。正常流程：先add()，并且在add()时会 增加modCount，然后iter.next()来检查modCount\expectedModCount是否相等；
  * 本程序的错误流程：先add()，再iter.next() 、又add()、又iter.next()
  * 以上：if(modCount != expectedModCount)非常类似 版本控制技术，严格来讲 被称为  "fail-fast策略"。

解决：使用juc中相应对ArrayList的并发支持：CopyOnWriteArrayList

## JUC

java.util.concurrent：包名，从jdk.1.5之后提供，包含了很多 用于并发编程的工具类

* CopyOnWriteArrayList：在**读写同时进行**的时候，当write的时候，对ArrayList进行一次copy

执行时机：写操作，访问的是”copy后的新集合“；读操作，访问的是”copy钱的新集合“

```java

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class ArrayListDemo {
    public static void main(String[] args) {
        CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
        list.add("aaa") ;
        list.add("bb") ;
        list.add("cc") ;
        Iterator<String> iter = list.iterator();
        while(iter.hasNext()){
            System.out.println(iter.next());
            list.add("xxx");
        }
        System.out.println("---");

        Iterator<String> iter2 = list.iterator();
        while(iter2.hasNext()){
            System.out.println(iter2.next());
        }
    }
}
console:
aaabbcc---
aaabbccxxxxxxxxx
```

* ConcurrentHashMap：大量使用了volatile（线程可见性）、synchronized（锁）、CAS无锁算法实现的。CAS无锁算法即每次获取CPU时判断公共资源是否被修改，若被修改则重新抢夺cpu

* 读写锁：在一般的项目中，都是读多写少，并且只有“写”可能引起并发冲突。 
  * 读锁：readLock()。加了读锁的资源，可以在没有写锁的情况下 被多个线程共享。即：如果t1线程已经加了读写，那么此时：

    * 如果t2要申请写锁，则t2会一直等待t1解锁
    * 如果t2要申请读写，则t2会直接加读锁。也就是说，t1和t2可以共同加读锁。

    小结：如果一个线程a加了读写，另一些线程只有在 申请加写锁时，才需要等待线程a解锁。

  * 写锁：writeLock()，独占锁。如果一个线程a加了写锁，则其他线程必须等待a释放锁。

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockDemo {
    //可重入的读写锁
    ReentrantReadWriteLock rwl = new ReentrantReadWriteLock() ;


    public static void main(String[] args) {
        ReadWriteLockDemo demo = new  ReadWriteLockDemo();
            new Thread(  () ->{
                //读
                demo.readMethod(Thread.currentThread());
                //写
                demo.writeMethod(Thread.currentThread());

            },"t1" ).start( );


            new Thread(   () ->{
                //读
                demo.readMethod(Thread.currentThread());
                //写
                demo.writeMethod(Thread.currentThread());

            },"t2" ).start( );
    }

    //读
    private void readMethod(Thread t ){
        rwl.readLock().lock();
        try {
            for (int i = 0; i < 9999; i++) {
                System.out.println(t.getName() + "正在执行读操作...");
            }
            System.out.println(t.getName() + "读操作【完毕】...");
        }finally {
            rwl.readLock().unlock();
        }

    }
    //写
    private void writeMethod(Thread t){
        rwl.writeLock().lock();
        try {
            for (int i = 0; i < 9999; i++) {
                System.out.println(t.getName() + "正在执行写操作...");
            }
            System.out.println(t.getName() + "写操作【完毕】...");
        }finally {
            rwl.writeLock().unlock();
        }
    }
}
```

## 线程通信

多个线程在争夺同一个资源时，为了保证协同工作，可以进行线程通信。

通信主要依赖于3个方法:

- wait（）【自己】：使当前线程处理等待状态（阻塞），一直的线程调用此对象的notify()或notifyAll()方法
- notify()【别人】：唤醒一个正在等待线程；如果有多个线程正在等待，则随机唤醒一个。
- notifyAll()【别人】：唤醒全部正在等待的线程

注意：

1. 线程在等待时，会被一个监听器所监听。因此，当其他线程调用notify()、notifyAll（）时，实际是通过监听器来唤醒线程。
2. 因为任何对象都可以使用synchronized加锁，而线程通信又依赖于加锁，因此线程通信属于“任何对象”的使用范围，即对象级，所以是在Object中定义的。
3. 这三个方法，必须定义在synchronized中定义，否则报异常
4. jdk建议：为了避免一些并发问题，建议将wait()方法写在循环中，因为被唤醒后必须在一次对条件进行判断，否则容易出现问题。

```java
     * As in the one argument version, interrupts and spurious wakeups are
     * possible, and this method should always be used in a loop:
     * <pre>
     *     synchronized (obj) {
     *         while (&lt;condition does not hold&gt;)
     *             obj.wait();
     *         ... // Perform action appropriate to condition
     *     }
     * </pre>
```

## Lock

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
Lock lock = new ReentrantLock();//可调用lock(),unlock()方法加锁，解锁
Condition condition = lock.newCondition();//signal(),signalAll()方法等价与notify(),notifyAll()
```

|      | synchronized                    | Lock                               |
| ---- | ------------------------------- | ---------------------------------- |
| 形式 | 关键字                          | 接口                               |
| 加锁 | {                               | lock()，多种方式（尝试一段时间等） |
| 解锁 | }，即方法正常结束;  或 出现异常 | unlock()                           |
| 状态 | 无法判断                        | 可以判断                           |
| 中断 | 不支持                          | 支持                               |

**中断等待：如果线程t要访问某个一资源，但此资源已经被其他线程长期加了锁，则线程t可以被其他线程结束等待状态**

```java
package com.tanzhou.thread;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class MyThread extends Thread {
    WaitInterrupt waitInterrupt;

    public MyThread(WaitInterrupt waitInterrupt) {
        this.waitInterrupt = waitInterrupt;
    }


    @Override
    public void run() {
        try {
            waitInterrupt.myLock(Thread.currentThread());//当前线程休眠3秒
        } catch (Exception e) {
            System.out.println(Thread.currentThread().getName() + "被中断");
        }
    }
}

public class WaitInterrupt {
    private Lock lock = new ReentrantLock();

    public static void main(String[] args) throws Exception {

        WaitInterrupt inter = new WaitInterrupt();
        MyThread t1 = new MyThread(inter);
        t1.setName("t1");
        MyThread t2 = new MyThread(inter);
        t2.setName("t2");

        t1.start();//休眠3秒
        t2.start();//休眠3秒
        Thread.sleep(1000);//main线程休眠1秒
        System.out.println("t1线程的状态" + t1.getState());
        System.out.println("t2线程的状态" + t2.getState());
        t2.interrupt();//中断t2
        System.out.println(111);
    }


    public void myLock(Thread thread) throws InterruptedException {
        try {

            lock.lockInterruptibly();//加了一把可被中断的锁
            System.out.println(thread.getName() + "加锁");
            Thread.sleep(10000);
        } finally {
            lock.unlock();
            System.out.println(thread.getName() + "解锁");
        }
    }

}
console
t1加锁
t1线程的状态TIMED_WAITING
t2线程的状态WAITING
111
t2被中断
t1解锁
```

## 无锁算法CAS

为了保证并发环境下的 数据安全，可以使用锁，也可以使用 一种无锁算法CAS算法。

CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。

加锁：缺点，消耗系统资源比较大（切换上下文对象，造成CPU调度很频繁）。即将cpu从t1切换到t2是一种比较消耗CPU资源的操作

加锁：是一种悲观策略，线程在访问资源时  总认为会和其他资源造成冲突，因此需要加锁。

CAS：是一种乐观策略，线程在访问资源时  总认为不会和其他资源造成冲突，因此有两种可能：

- 假设成功（的确没有和其他线程产生冲突）

  OK

- 假设失败（实际和其他线程产生了冲突）

  再试：重新获取最新的资源，然后再次访问（如果发现要访问的资源不是最新的，则一直尝试；直到拿到最新数据为止）

## 线程通信：信号量Semaphor

Semaphor编程：默认所有的线程都是阻塞；有个一许可acquire()，只有获取了许可的线程 ，才能运行。并且许可的数量，可以通过persmits属性设置。release()释放许可

以下示例：有10个线程尝试执行，但只有3个许可证。

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class SemaphoreDemo {
    public static void main(String[] args) {

        ExecutorService executor = Executors.newCachedThreadPool();
        Semaphore sem = new Semaphore(3) ;//同一时间 最多有3个线程可以并发运行
        //假设有10个线程
        for( int i=0;i<10;i++){
            final int tId = i ;
            executor.execute(
                         () ->{
                             try{
                                  sem.acquire();//获取许可（同一时间 只有3个许可）
                                 System.out.println("线程" + tId +"获取到了许可，正在执行");
                                Thread.sleep((long)(5000*Math.random()));
                                sem.release();//释放许可
                             }catch (InterruptedException e){

                             }catch(Exception e){
                                 e.printStackTrace();
                             }
                         }
            );//给线程中 加入一个新线程
        }
//            executor.shutdown();
    }
}
```

