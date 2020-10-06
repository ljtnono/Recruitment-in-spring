## Capter01 

### Semaphore类的使用

Semaphore是信号量的意思，Semaphore所提供的功能完全是synchronized关键字的升级版，但是它提供的功能更加的强大与方便，主要的作用就是控制线程并发的数量，而这一点，单纯的使用synchronized关键字是做不到的。

Semaphore的API列表如下图：

![image-20200201150711981](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200201150711981.png)

多线程中的同步概念其实就是排着队去执行一个任务，执行任务是一个一个执行，并不能并行执行，这样的优点是有助于逻辑的正确性，不会出现非线程安全问题，保证软件系统功能上的运行稳定性。

案例：使用Semaphore类测试同步性

```java
// Service.java
package cn.ljtnono.juc.capter01;


import java.util.concurrent.Semaphore;

public class Service {

    private Semaphore semaphore = new Semaphore(1);

    public void test() {
        try {
            // 获取一个信号量, 这里只允许一个线程进入，当第二个线程进入的时候会阻塞
            semaphore.acquire();
            System.out.println(Thread.currentThread().getName() + "begin timer = " + System.currentTimeMillis());
            Thread.sleep(5000);
            System.out.println(Thread.currentThread().getName() + "end timer = " + System.currentTimeMillis());
            semaphore.release();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
// Run.java
package cn.ljtnono.juc.capter01;

public class Run {

    public static void main(String[] args) {
        Service service = new Service();
        Thread a = new Thread(service::test, " thread --- a ");
        Thread b = new Thread(service::test, " thread --- b ");
        Thread c = new Thread(service::test, " thread --- c ");
        a.start();
        b.start();
        c.start();
    }
}
// 结果
 thread --- a begin timer = 1580541803863
 thread --- a end timer = 1580541808863
 thread --- b begin timer = 1580541808863
 thread --- b end timer = 1580541813863
 thread --- c begin timer = 1580541813863
 thread --- c end timer = 1580541818864
```

从程序的结果来看，当三个线程都启动时，只有a线程进入了，当a线程结束了，b线程立即就进入，当b线程结束了，c立即进入，即验证了Semaphore能够保证同步性。

#### Semaphore类的构造方法permits参数作用

```java
private Semaphore semaphore = new Semaphore(1);
```

其中1（permits）参数代表的是 **同一时间内最多允许的线程数可以执行acquire()和release()之间的方法**，这里也可以>1，代表同一时间内可以多个线程执行相关代码，验证如下：

```java
// 将以上Sevice类中的Semaphore改为下面代码，其他不变
private Semaphore semaphore = new Semaphore(2);
// 结果
 thread --- a begin timer = 1580543009913
 thread --- b begin timer = 1580543009913
 thread --- a end timer = 1580543014913
 thread --- b end timer = 1580543014913
 thread --- c begin timer = 1580543014913
 thread --- c end timer = 1580543019914
```

从结果可以看出来线程a和线程b同时进入了，而且也是同时结束，当a、b结束之后c立刻就开始执行了，这就验证了构造函数参数的意义。

#### 方法acquire(int permits)参数作用及动态添加permits许可数量

有参方法acquire(int permits)的功能是每调用1次此方法，就使用x个许可

```java
// Service.java
package cn.ljtnono.juc.capter01;

import java.util.concurrent.Semaphore;

public class Service {

    private Semaphore semaphore = new Semaphore(10);

    public void test() {
        try {
            // 获取一个信号量
            semaphore.acquire(2);
            System.out.println(Thread.currentThread().getName() + "begin timer = " + System.currentTimeMillis());
            int sleepValue = (int) (Math.random() * 10000);
            System.out.println(Thread.currentThread().getName() + " 停止了" + (sleepValue / 1000) + "秒");
            Thread.sleep(sleepValue);
            System.out.println(Thread.currentThread().getName() + "end timer = " + System.currentTimeMillis());
            semaphore.release(2);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
// Run.java
package cn.ljtnono.juc.capter01;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Run {

    public static void main(String[] args) {
        Service service = new Service();
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 10; i++) {
            executorService.execute(service::test);
        }
        executorService.shutdown();
    }
}
// 结果
pool-1-thread-1begin timer = 1580544588900
pool-1-thread-2begin timer = 1580544588901
pool-1-thread-2 停止了4秒
pool-1-thread-1 停止了0秒
pool-1-thread-3begin timer = 1580544588901
pool-1-thread-3 停止了9秒
pool-1-thread-4begin timer = 1580544588901
pool-1-thread-4 停止了9秒
pool-1-thread-5begin timer = 1580544588902
pool-1-thread-5 停止了4秒
pool-1-thread-1end timer = 1580544589668
pool-1-thread-6begin timer = 1580544589668
pool-1-thread-6 停止了0秒
pool-1-thread-6end timer = 1580544590662
pool-1-thread-7begin timer = 1580544590663
pool-1-thread-7 停止了5秒
pool-1-thread-5end timer = 1580544593322
pool-1-thread-8begin timer = 1580544593322
pool-1-thread-8 停止了1秒
pool-1-thread-2end timer = 1580544593473
pool-1-thread-9begin timer = 1580544593474
pool-1-thread-9 停止了1秒
pool-1-thread-9end timer = 1580544594645
pool-1-thread-10begin timer = 1580544594645
pool-1-thread-10 停止了7秒
pool-1-thread-8end timer = 1580544595228
pool-1-thread-7end timer = 1580544596301
pool-1-thread-3end timer = 1580544598721
pool-1-thread-4end timer = 1580544598833
pool-1-thread-10end timer = 1580544602120
```

总共有10个许可，每次使用acquire(2)消耗2个许可，所以同一时间内最多5个线程执行代码。

Semaphore类有一个很好玩的特性，就是许可的个数并不一定是初始化的个数，每调用一次release方法就会多出来一个许可个数，验证如下：

```java
Semaphore semaphore = new Semaphore(10);
semaphore.release(20);
System.out.println(semaphore.availablePermits()); //availablePermits()方法返回当前许可个数
// 结果
30
```



#### acquireUninterruptibly()的使用

如果单独使用acquire()方法的话，那么在线程执行同步代码的时候可能会中断，要想让线程在执行的时候不中断，那么可以使用**acquireUninterruptibly()**方法，下面是一个会中断的测试：

```java
// Service.java
package cn.ljtnono.juc.capter01;

import java.util.concurrent.Semaphore;

public class Service {

    private Semaphore semaphore = new Semaphore(1);

    public void test() {
        try {
            semaphore.acquire();
            System.out.println(Thread.currentThread().getName() + "begin timer = " + System.currentTimeMillis());
            for (int i = 0; i < Integer.MAX_VALUE / 50; i++) {
                String s = new String();
                Math.random();
            }
            System.out.println(Thread.currentThread().getName() + "end timer = " + System.currentTimeMillis());
            semaphore.release();
        } catch (InterruptedException e) {
            System.out.println("线程" + Thread.currentThread().getName() + "进入了catch块");
            e.printStackTrace();
        }
    }

}
// Run.java
package cn.ljtnono.juc.capter01;

public class Run {

    public static void main(String[] args) throws InterruptedException {
        Service service = new Service();
        Thread a = new Thread(service::test ,"A");
        Thread b = new Thread(service::test ,"B");
        a.start();
        b.start();
        Thread.sleep(1000);
        b.interrupt();
        System.out.println("main中断了b");

    }
}
// 结果
Abegin timer = 1580545992187
Aend timer = 1580545993349
线程B进入了catch块
main中断了b
java.lang.InterruptedException
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(AbstractQueuedSynchronizer.java:998)
	at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1304)
	at java.util.concurrent.Semaphore.acquire(Semaphore.java:312)
	at cn.ljtnono.juc.capter01.Service.test(Service.java:11)
	at java.lang.Thread.run(Thread.java:748)

```

以上结果知道b线程被main线程中断了，进入了catch块，如果将acquire改成使用acquireUninterruptibly就不会进入到catch块中：

```java
// 将Service中的acquire代码改成acquireUninterruptibly，try-catch块也不用了
semaphore.acquireUninterruptibly();
// 结果
Abegin timer = 1580546185610
Aend timer = 1580546186760
main中断了b
Bbegin timer = 1580546186760
Bend timer = 1580546188151
// 如果使用acquireUninterruptibly(int permites)方法的话，是在等待许可的情况下不允许中断，如果成功获得锁，那么将会取得指定的permit个数
```



#### availablePermits()和drainPermits()方法

availablePermits()顾名思义是获取当前可用的许可个数，而drainPermits()方法是将当前可用许可个数清零，这里就不演示这两个方法



#### getQueueLength()和hasQueuedThreads()方法

getQueueLength()方法是获取当前处于等待队列中的线程数，hasQueuedThreads()方法是判断是否有线程处于等待当中，验证如下：

```java
// Service.java
package cn.ljtnono.juc.capter01;

import java.util.concurrent.Semaphore;

public class Service {

    private Semaphore semaphore = new Semaphore(1);

    public void test() {
        semaphore.acquireUninterruptibly();
        System.out.println(Thread.currentThread().getName() + "begin timer = " + System.currentTimeMillis());
        if (semaphore.hasQueuedThreads()) {
            System.out.println("当前有" + semaphore.getQueueLength() + "个线程在等待");
        } else {
            System.out.println("当前没有线程在等待");
        }
        System.out.println(Thread.currentThread().getName() + "end timer = " + System.currentTimeMillis());
        semaphore.release();
    }

}
// Run.java
package cn.ljtnono.juc.capter01;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Run {

    public static void main(String[] args) throws InterruptedException {
        Service service = new Service();
        ExecutorService executorService = Executors.newFixedThreadPool(10);
        for (int i = 0; i < 20; i++) {
            executorService.execute(service::test);
        }
        executorService.shutdown();
    }
}
// 结果
pool-1-thread-1begin timer = 1580547561343
当前有1个线程在等待
pool-1-thread-1end timer = 1580547561343
pool-1-thread-2begin timer = 1580547561343
当前有2个线程在等待
pool-1-thread-2end timer = 1580547561344
pool-1-thread-3begin timer = 1580547561344
当前有3个线程在等待
pool-1-thread-3end timer = 1580547561344
pool-1-thread-3begin timer = 1580547561344
当前有3个线程在等待
pool-1-thread-3end timer = 1580547561344
pool-1-thread-3begin timer = 1580547561344
当前有3个线程在等待
pool-1-thread-3end timer = 1580547561344
pool-1-thread-4begin timer = 1580547561344
当前有4个线程在等待
pool-1-thread-4end timer = 1580547561344
pool-1-thread-4begin timer = 1580547561344
当前有4个线程在等待
pool-1-thread-4end timer = 1580547561344
pool-1-thread-1begin timer = 1580547561344
当前有5个线程在等待
pool-1-thread-1end timer = 1580547561344
pool-1-thread-1begin timer = 1580547561344
当前有6个线程在等待
pool-1-thread-1end timer = 1580547561344
pool-1-thread-1begin timer = 1580547561344
当前有6个线程在等待
pool-1-thread-1end timer = 1580547561344
pool-1-thread-1begin timer = 1580547561344
当前有6个线程在等待
pool-1-thread-1end timer = 1580547561344
pool-1-thread-2begin timer = 1580547561344
当前有5个线程在等待
pool-1-thread-2end timer = 1580547561344
pool-1-thread-3begin timer = 1580547561345
当前有4个线程在等待
pool-1-thread-3end timer = 1580547561345
pool-1-thread-7begin timer = 1580547561345
当前有4个线程在等待
pool-1-thread-7end timer = 1580547561345
pool-1-thread-5begin timer = 1580547561345
当前有3个线程在等待
pool-1-thread-5end timer = 1580547561345
pool-1-thread-6begin timer = 1580547561345
当前有3个线程在等待
pool-1-thread-6end timer = 1580547561345
pool-1-thread-4begin timer = 1580547561345
当前有2个线程在等待
pool-1-thread-4end timer = 1580547561345
pool-1-thread-10begin timer = 1580547561345
当前有2个线程在等待
pool-1-thread-10end timer = 1580547561345
pool-1-thread-8begin timer = 1580547561346
当前有1个线程在等待
pool-1-thread-8end timer = 1580547561346
pool-1-thread-9begin timer = 1580547561346
当前没有线程在等待
pool-1-thread-9end timer = 1580547561346
```

从结果可以看出当线程数超过permit时，线程将会进入到等待队列中去，使用这两个方法可以判断等待队列中是否有线程和获取等待队列中的线程数。



## Capter04

### Executor接口介绍

Executor接口只有一个方法，但是Executor是接口，并不能直接使用，所以还得需要实现类，以下是Executor接口相关的类继承结构。

```java
public interface Executor {
    void execute(Runnable command);
}
```

![image-20200419151640906](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200419151640906.png)

接口ExecutorService是Executor的子接口，在内部添加了比较多的方法，以下是ExecutorService中定义的方法

![image-20200419152338540](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200419152338540.png)

由于ExecutorService是接口，我们可以通过其子类AbstractExecutorService的子类ThreadPoolExecutor这个类来创建线程池。由于ThreadPoolExecutor这个类的参数比较复杂，所以我们可以先熟悉一下Executors这个类，这个类是专门用于创建ThreadPoolExecutor线程池的工厂类，有很多好用的方法，下面是这个类的方法列表：

![image-20200419161233881](https://ljtnono.oss-cn-beijing.aliyuncs.com/images/image-20200201150711981.png)

### 使用Thread





### ThreadPoolExecutor的使用

类ThreadPoolExecutor可以非常方便的创建线程池对象，而不需要程序员设计大量的new实例化Thread相关的代码。使用Executors工厂类的newXXXThreadExecutors()方法可以快速方便地创建线程池，但是创建的细节未知，查看源码得知都是new ThreadPoolExecutor这个对象来实现的，所以研究ThreadPoolExecutor这个对象十分有必要。

