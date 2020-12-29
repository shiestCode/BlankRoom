## Java多线程

------



### 1.Java线程基础

#### 1.1What is Java线程

一个轻量级(对比进程)的任务执行单位,(同一进程下的)线程之间可以共享数据,也可以独立调度。

Java的线程 1.在java.langThread下 2.已经执行过start()的实例.

#### 1.2线程的实现

##### 1.2.1继承Thread

```java
public class Thread implements Runnable {
    /* What will be run. */
    private Runnable target;
    
    @Override
    public void run() {
        if (target != null) {
            target.run();
        }
    }
}
```

##### 1.2.2实现Runnable接口

```java
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     * <p>
     * The general contract of the method <code>run</code> is that it may
     * take any action whatsoever.
     *
     * @see     java.lang.Thread#run()
     */
    public abstract void run();
}
```

##### 1.2.3实现Callable<>，run()方法可以存在返回值

```java
public static class CallerTask implements Callable<String> {
    @Override
    public String call() throws Exception{
        return "hello";
    }
}

public static void main(String[] args) {
    //创建异步任务
    FutureTask<String> futureTask = new FutureTask<>(new CallerTask());
    //启动线程
    new Thread(futureTask).start();
    try{
        //等待任务执行完毕,并返回结果
        String result = futureTask.get();
        System.out.println(result);
    } catch(ExecutionException e) {
        e.printSatckTrace();
    }
}
```



#### 1.3线程的状态

```java
public enum State {
        /**
         * 线程刚刚被创建出来时
         * MyThread thread = new MyThread();
         */
        NEW,

        /**
         * 线程已经准备好在JVM中进行执行.注意这里并不是说准备好可以执行就是执行
         * thread.start();---->I am ready!
         */
        RUNNABLE,

        /**
         * 线程正在等待获取   锁  (synchronized修饰的代码块或方法,在获得锁之前就是BLOCKED)
         * **长时间状态为 BLOCKED 就要考虑是不是死锁了	
         */
    	BLOCKED,
         
        /**
         * 线程进入这个状态,一定执行了
         * Object.wait();
         * Thread.join();
         * LockSupport.park();
         * 线程进入无限等待
         * 进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）
         */
        WAITING,

        /**
         * 有限的等待,在过指定时间后,会重新唤醒
         */
        TIMED_WAITING,

        /**
         * Thread state for a terminated thread.
         * The thread has completed execution.
         */
        TERMINATED;
    }
```

![](https://upload-images.jianshu.io/upload_images/4840092-f85e70e2262b7878.png?imageMogr2/auto-orient/strip|imageView2/2/w/1155/format/webp)

```tex
1.初始状态  New        继承Thread或实现Runnable接口可以得到一个线程类,new出一个实例,线程就进入了初始状态
2.就绪状态  Ready      就绪状态说明该线程有资格运行,但是还要靠cpu的调度
3.运行中    Running    线程正在运行
4.阻塞状态  Blocked    线程堵塞在进入synchronized关键字修饰的方法/代码块时的状态
5.等待      Waiting    该状态下的线程不会被cpu分配时间,只能等待显式地唤醒,否则就会无限期的等待
6.终止状态  Terminated 线程一旦终止,就不能重生了
```

#### 1.4线程常用函数

```java
//线程休眠
sleep(long millis);
//Waits for this thread to die.  
//使用场景：一个主线程中启动了很多子线程,如果这些子线程是执行耗时很多的,会导致主线程先结束,使用join()可以保证在子线程结束之后,主线程才会结束
t.join();
//暂停正在执行的线程,并执行其他线程,让Running状态的线程变成Runnable状态,但是实际中可能不会让正在执行的线程进行让步,因为让步的线程很可能会是下一个运行的线程
t.yield();
//MIN_PRIORITY = 1; MAX_PRIORITY = 10; NORMAL_PRIORITY = 5;
t.setPriority();
//线程直接中断,线程很可能所持有的资源都没有释放
t.interrput();
//obj.wait();方法是一套的使用,obj.notify(),synchronized(obj)
synchronized(obj) {
    //线程在获得obj锁后,进行了obj释放,同时线程进行休眠
    obj.wait();
    //唤醒wait的线程
    obj.notify();
}

```

#### 1.5线程通知与等待

```java
Object类是所有类的父类:所有类都需要的方法都放在那里面:
	Class<?> getClass();
	int hashCode();
	boolean equals(Object);
	Object clone();
	String toString();
	void notify();
	void noitfyAll();
	void wait();
	void finalize();
1.wait()函数
    wait():当一个线程调用一个共享变量的wait()时,调用线程会被阻塞挂起,直到以下这几种情况
    	1.其他线程调用了这个共享变量的notify()或者notifyAll();
	    2.其他线程调用了该线程的interrupt()方法,该线程抛出InterruptedException
    调用 wait()要事先获取该对象的监视器锁
那么问题来了,一个线程如何获取一个共享变量的监视器锁呢?
    方法1：使用同步代码块,参数为共享变量
        synchronized(共享变量) {
            //do something
        } 
	方法2：调用共享变量,使用synchronized修饰
        synchronized void add(int a, int b) {
        //do something
    }
*******
但是一个被挂起的线程是可以变为可执行状态的,没经过notify(),notifyAll(),interrput(),这成为线程的虚假唤醒,
	所以为了避免虚假唤醒我们可以这样
    synchronized(obj) {
    	while(唤醒条件不满足) {
            obj.wait();
        }   
    }
不满足唤醒条件就会一直挂起

    
2.notify()函数
    一个线程调用共享变量的 notify() 后,会唤醒一个线程(调用了该共享变量的 wait() 方法而挂起的线程),notify 唤醒的线程是随机的
    但是唤醒他的线程释放了共享变量的锁,被唤醒的线程也不一定能获取到共享变量的锁,因为被唤醒的线程会和其他线程一起竞争共享变量的锁,只有竞争到的才可以继续执行.
3.notifyAll()函数
   唤醒所有因为该共享变量而挂起的线程,他们一起去竞争共享变量的锁
```

#### 1.6简单的生产者消费者模型

​	说明:queue为共享变量,生产者线程在调用queue的wait()前，使用synchronized获取到queue的监视器锁，如果当前队列没有空闲容量就会调用queue的wait()挂起当前线程,使用循环是为了避免线程的虚假唤醒

```java
//生产者
synchronized (queue) {
    //队列满,线程就会挂起,等待线程空闲
    while(queue.size() == MAX_SIZE) {
        try {
            //挂起当前线程,释放通过同步块获得的queue监视器锁,让消费者线程可以获取到queue的监视器锁,
            queue.wait();
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
    //空闲则生产,并通知消费者线程
    queue.add(ele);
    queue.notifyAll();
}
//消费者
synchronized (queue) {
    //如果队列是空的，消费者就没办法消费了,就得把自己挂起,等待生产者生产完的notifyAll()
    while(queue.size() == 0) {
        try {
            queue.wait();
        } catch(Exception e) {
            e.printStackTrace();
        }
    }
    queue.remove(ele);
    queue.notifyAll();
}
```

```java
public class ThreadTest {

    //创建共享资源
    private static volatile Object resourceA = new Object();
    private static volatile Object resourceB = new Object();

    /**
     * @desc: threadA调用resourceA的wait();挂起了自己,释放了resourceA的共享资源监视器锁,
     * 但是threadA依然还是保有resourceB的共享资源监视器锁，所以threadB的synchronized(resourceB)没办法获取到
     * @author: liuzicheng
     * @date: 2020/6/18 10:18
     * @return: void
     */
    public static void main(String[] args) throws InterruptedException {
        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                try{
                    //获取resourceA的共享资源监视器锁
                    synchronized (resourceA) {
                        System.out.println("threadA获取到resourceA的共享资源监视器锁");
                        //获取resourceB的共享资源监视器锁
                        synchronized (resourceB) {
                            System.out.println("threadA获取到resourceB的共享资源监视器锁");
                            //threadA释放resourceA的锁
                            System.out.println("threadA释放resourceA的共享资源监视器锁");
                            resourceA.wait();
                        }
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });

        Thread threadB = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    Thread.sleep(1000);
                    synchronized (resourceA) {
                        System.out.println("threadB获取到resourceA的共享资源监视器锁");
                        System.out.println("threadB尝试获得resourceB的共享资源监视器锁");
                        synchronized (resourceB) {
                            System.out.println("threadB获取到resourceB的共享资源监视器锁");
                            System.out.println("threadB阻塞,释放resourceA的共享资源监视器锁");
                            resourceA.wait();
                        }
                    }
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
        //启动线程
        threadA.start();
        threadB.start();
        //等待两个线程结束
        threadA.join();
        threadB.join();
    }

}
```

```java
public class WaitNotifyInterupt {

    static Object obj = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    System.out.println("--begin--");
                    synchronized (obj) {
                        obj.wait();
                    }
                    System.out.println("--end--");
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });

        threadA.start();

        Thread.sleep(1000);
        System.out.println("--begin interrput threadA--");
        threadA.interrupt();
        System.out.println("--end interrput threadA--");
    }
}
```

------

等待线程执行中止的join()

```java
public class JoinThreadTest {

    public static void main(String[] args) throws InterruptedException {
        Thread threadOne = new Thread(new Runnable() {
            @Override
            public void run() {
                try{
                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                System.out.println("child threadOne over");
            }
        });

        Thread threadTwo = new Thread(new Runnable() {
            @Override
            public void run() {
                try{
                    Thread.sleep(1000);
                } catch (Exception e) {
                    e.printStackTrace();
                }
                System.out.println("child threadTwo over");
            }
        });
        //启动子线程
        threadOne.start();
        threadTwo.start();

        System.out.println("wait all child thread over");
        //等待所有子线程执行完毕
        threadOne.join();
        threadTwo.join();
        System.out.println("all child thread over");
    }
    /**
     * @desc:
     *       主线程先启动了两个子线程,
     *       然后调用threadOne的join()，主线程就会阻塞,直到threadOne线程结束
     *       主线程恢复
     *       又调用threadTwo的join(),主线程又阻塞了,直到threadTwo线程结束
     */
}
```

------

让线程睡觉的sleep()方法

执行行中的线程调用sleep()，调用线程会让出指定时间的执行权,在此段时间内不参与CPU调度，但是线程获取的共享资源监视器锁就不会释放的,指定时间之后线程就恢复就绪状态

```java

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/**
 * 在睡眠时间之内锁也是在当前线程所拥有的
 * 子线程在睡眠期间,其他线程中断了他,子线程会抛出InterruptedException异常
 * @author liuzicheng
 */
public class SleepThreadTest {

    /**
     * @desc: 创建一个独占锁
     */
    private static final Lock lock = new ReentrantLock();

    public static void main(String[] args) {

        //创建一个子线程
        Thread threadA = new Thread(new Runnable() {
            @Override
            public void run() {
                //获取独占锁
                lock.lock();
                try{
                    System.out.println("child threadA is in sleep");

                    Thread.sleep(10000);

                    System.out.println("child threadA is in awaked");
                } catch (Exception e) {
                    e.printStackTrace();
                } finally {
                    //释放锁
                    lock.unlock();
                }
            }
        });

        //创建线程B
        Thread threadB = new Thread(new Runnable() {
            @Override
            public void run() {
                //获取独占锁
                lock.lock();
                try{
                    System.out.println("child threadB is in sleep");

                    Thread.sleep(10000);

                    System.out.println("child threadB is in awaked");
                } catch(Exception e) {
                    e.printStackTrace();
                } finally {
                    //释放锁
                    lock.unlock();
                }
             }
        });
        //启动线程
        threadA.start();
        threadB.start();
    }
}

```

------

让出CPU执行权的yield方法

当线程执行yield()方法时, 当前线程 向 线程调度器 暗示 请求让出自己的CPU使用,但是线程调度器可以无条件忽略这个暗示。让出之后,线程就成为就绪状态

```java
public class YieldThreadTest {

    public static class YieldTest implements Runnable{

        public YieldTest () {

            //创建线程并启动
            Thread t = new Thread(this);
            t.start();
        }

        @Override
        public void run() {
            for (int i = 0; i < 5; i++) {
                //当i=0时让出CPU执行权,放弃时间片,进行下一阶段调度
                if ((i % 5) == 0) {
                    System.out.println(Thread.currentThread() + "yield CPU ...");
                    //当前线程让出CPU执行权，放弃时间片,进行下一轮调度
                    Thread.yield();
                }
            }
            System.out.println(Thread.currentThread() + "is over.");
        }
    }

    public static void main(String[] args) {
        new YieldTest();
        new YieldTest();
    }
}
```

sleep和yield的区别：

| yield | 让出当前是CPU执行片段      | 不被阻塞 | 是就绪状态,等待下一轮的调度     |
| ----- | :------------------------- | -------- | ------------------------------- |
| sleep | 线程调度器不会调度这些线程 | 直接阻塞 | 是阻塞状态,睡完了才能到就绪状态 |

------

线程中断

Java中的线程中断是一种线程间的**协作模式**,通过设置线程的**中断标志**,并不直接终止该线程的执行,而是,线程自己跟据中断状态来**自行中断**.

```java
public void run () {
	try {
        //线程退出条件
        while(!Thread.currentThread().isInterrupted && more work to do){
            // do something
        } 
    }catch (Exception e) {
        
    } finally {
        
    }
}
```

```java
public class InterruptThreadTest {

    public static void main(String[] args) throws InterruptedException {
        Thread threadOne = new Thread(new Runnable() {
            @Override
            public void run() {
                try{
                    System.out.println("threadOne begin sleep for 2000 s");
                    Thread.sleep(2000000);
                    System.out.println("threadOne awaked");
                } catch(Exception e) {
                    System.out.println("threadOne is interrupted while sleeping");
                    return;
                }
                System.out.println("threadOne-leaving normally");
            }
        });
        //启动线程
        threadOne.start();
        //确保子线程进入休眠状态
        Thread.sleep(1000);
        //打断子线程的休眠,让子线程从sleep函数返回
        threadOne.interrupt();
        //等待子线程运行结束
        threadOne.join();
        System.out.println("main thread is over");
    }
}
```

#### 1.7线程死锁

##### 1.7.1死锁定义:

两个或两个以上的线程,因争夺资源而造成相互等待,在无外力作用下,线程会一直等待。

##### 1.7.2死锁的产生条件

| 互斥条件       | 资源同时只能由一个线程占用                                   |
| -------------- | ------------------------------------------------------------ |
| 请求并持有条件 | 一个线程已经持有至少一个线程,但又提出新的资源请求，而新的资源请求被占用 |
| 不可剥夺条件   | 线程在获得资源到自己使用完之前不能被其他线程占用             |
| 环路等待条件   |                                                              |

##### 1.7.3避免死锁的产生

避免死锁只能从两个产生条件下手:请求并持有条件,环路等待条件

请求并持有条件：资源请求顺序必须一致,线程A,线程B都需要资源1,2,3,4...n,我们需要对资源进行排序,线程A和线程B只有在获取资源n-1时,才能获取资源n

#### 1.8守护线程与用户线程

Java中的线程分为两类:守护线程(daemon),用户线程(user)

当最后一个非守护线程结束,JVM就会正常退出,守护线程是否结束并不影响JVM的退出.

#### 1.9ThreadLocal

线程的本地化变量,如果你创建了一个ThreadLocal变量,那么访问这个变量的每个线程都会有一个本地副本.多个线程操作这个变量时,实际操作的是自己本地内存中的变量

```java
public class ThreadLocalTest {

    //创建ThreadLocal变量
    static ThreadLocal<String> localVariable = new ThreadLocal<>();
    //
    static void print(String string) {
        //打印当前线程本地内存中的localVariable变量的值
        System.out.println(string + ":" + localVariable.get());
        //清楚当前线程本地内存中的localVariable
        localVariable.remove();
    }

    public static void main(String[] args) {
        Thread threadOne = new Thread(new Runnable() {
            @Override
            public void run() {
                //设置当前线程中本地变量localVariable的值
                localVariable.set("threadOne local variable");
                print("threadOne");
                System.out.println("threadOne remove after" + ":" + localVariable.get());
            }
        });

        Thread threadTwo = new Thread(new Runnable() {
            @Override
            public void run() {
                localVariable.set("threadTwo local variable");
                print("threadTwo");
                System.out.println("threadTwo remove after" + ":" + localVariable.get());
            }
        });

        threadOne.start();
        threadTwo.start();
    }
}
```

#### 1.9.1ThreadLocal原理

Thread中存在两个ThreadLocalMap类型的变量 threadLocals ,inheritableThreadLocals ,每个线程的本地变量存放在threadLocals中。

ThreadLocalMap就是一个改装的HashMap,ThreadLocal的set()就是把value放到调用线程的threadLocals中

常用方法就是set，get，remove

```java
void set(T value) {
    //获取到当前线程
    Thread t = Thread.currentThread();
    //将当前线程当做key,去查找对应的线程变量,找到则设置
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value);
    }  else {
        //第一次调用就创建当前线程对应的HashMap
        createMap(t, value);
    }
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

void createMap(Thread t, T firstValue) {
    return new ThreadLocalMap(this, firstValue);
}
```

```java
public T get() {
    //获取当前线程
    Thread t = THread.getCurrentThread();
    //获取当线程的threadLocals变量
    ThreadLocalMap map = getMap(t);
    //如果ThreadLocals不为null,则返回对应本地变量的值
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            
            T result = (T)e.value;
            return result;
        }
    }
    //threadLocals为空则初始化当前线程的threadLocals成员变量
    return setInitialValue();
}

private T setInitialValue() {
    //初始化为null
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    //如果当前线程的threadLocals变量不为空
    if(map != null) {
        map.set(this, value);
    } else {
        createMap(t, value);
    }
    return value;
}

protected T initialValue() {
    return null;
}
```

```java
public void remove() {
    ThreadLocalMap m = getMap(Thread.currentThread());
    if(m != null) 
        m.remove(this);
}
```

每个线程内部都有一个名为threadLocals的成员变量,该变量列为HashMap,key为我们定义的ThreadLocal变量的引用,value是set方法设置的值.只要线程不消亡,threadLocal就不会消失,所以可能会OOM

#### 1.9.2ThreadLocal不支持继承

#### 1.9.3InheritableThreadLocal可继承

### 2.并发List

#### 2.1CopyOnWriteArrayList

![](C:\Users\liuzicheng\Downloads\未命名文件.png)

需要实现一个写时复制的线程安全的List需要考虑以下:

什么时候初始化list?初始化的size是多少?list是有限的吗?

如何保证线程安全?

如何保证使用迭代器遍历时的数据一致性