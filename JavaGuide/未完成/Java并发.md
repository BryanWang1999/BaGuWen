### 线程与进程

+ **进程是一个正在运行的程序实例。程序由指令和数据组成，这些指令要运行，数据要读写，就必须将指令加载至 CPU，数据加载至内存。在指令运行过程中还需要用到磁盘、网络等设备。进程就是用来加载指令、管理内存、管理 IO 的。当一个程序被运行，从磁盘加载这个程序的代码至内存，这时就开启了一个进程。**
+ **线程是进程内的一个执行单元，一个进程可以包含多个线程。一个线程就是一个指令流，将指令流中的一条条指令以一定的顺序交给 CPU 执行 。Java 中，线程作为最小调度单位，进程作为资源分配的最小单位。**



+ **线程通信相对简单，因为它们共享进程中的内存**
+ **由于线程共享资源，一个线程的错误可能会影响到其他线程**
+ **线程上下文切换成本较低**



**在Java中，当我们启动一个Java程序时，实际上会启动一个JVM进程。**

**`main`函数通常是Java程序的入口点，当JVM启动后，它会从`main`函数开始执行程序。`main`函数所在的线程被称为主线程，它是这个JVM进程中的一个线程，主线程执行`main`函数中的代码。**

**除了主线程外，Java程序还可以创建和管理其他线程。**



### 线程上下文切换

**因为一些原因导致 cpu 不再执行当前的线程，转而执行另一个线程的代码**



### 并行与并发

+ **并行：多个任务在同一时刻真正同时执行，每个任务都在不同的处理单元上执行，各任务之间相互独立，任务之间没有时间片的切换。**
+ **并发：同一个时间段内交替执行多个任务，通过快速切换来模拟同时执行的效果。**



### 异步调用

- **方法的调用者需要等待结果返回，才能继续运行就是同步**
- **方法的调用者不需要等待结果返回，就能继续运行就是异步**



### 创建线程方式 - 继承 `Thread` 类

```java
package com.bowang.JUC.JavaThread;

import lombok.extern.slf4j.Slf4j;

@Slf4j(topic = "c.Test1")
public class Test1 {
    public static void main(String[] args) {
        Thread t = new Thread() {
            @Override
            public void run() {
                log.debug("running");
            }
        };

        t.setName("t1");
        t.start();
        log.debug("running");
    }
}
```



### 创建线程方式 - 实现 `Runnable` 接口

**把【线程】和【任务】(要执行的代码)分开**

- **`Thread` 代表线程**
- **`Runnable` 可运行的任务(线程要执行的代码)**

```java
package com.bowang.JUC.JavaThread;

import lombok.extern.slf4j.Slf4j;

@Slf4j(topic = "c.Test2")
public class Test2 {
    public static void main(String[] args) {
        Runnable r = new Runnable() {
            @Override
            public void run() {
                log.debug("running");
            }
        };

        Thread t = new Thread(r, "t2");
        t.start();
        log.debug("running");
    }
}
```

```java
package com.bowang.JUC.JavaThread;

import lombok.extern.slf4j.Slf4j;

@Slf4j(topic = "c.Test2Lambda")
public class Test2Lambda {
    public static void main(String[] args) {
        Runnable r = () -> log.debug("hello");

        Thread t = new Thread(r, "t2Lambda");
        t.start();
        log.debug("running");
    }
}
```



### 创建线程方式 - 实现 `Callable` 接口

**`FutureTask` 能够接收 `Callable` 类型的参数，用来处理有返回结果的情况**

```java
package com.bowang.JUC.JavaThread;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

@Slf4j(topic = "c.Test3")
public class Test3 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> task = new FutureTask<>(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                log.debug("running");
                Thread.sleep(1000);
                return 100;
            }
        });

        Thread t3 = new Thread(task, "t3");
        t3.start();

        log.debug("{}", task.get());
    }
}
```



### `Runnable` 和 `Callable` 有什么区别

+ **`Runnable` 接口的 `run` 方法没有返回值。任务执行过程中抛出的异常需要在任务内部进行处理，无法通过返回值传递给调用者。**
+ **`Callable` 接口的 `call` 方法有返回值。任务执行过程中抛出的异常可以传递给调用者。**



### 线程的 `run()` 和 `start()` 有什么区别？

+ **`run()`：定义线程启动以后要执行的代码。不会启动一个新的线程**
  + **直接执⾏ `run()` ⽅法，会把 `run()` ⽅法当成⼀个主线程下的普通⽅法去执⾏，并不会在某个线程中执⾏它，所以这并不是多线程⼯作。**
+ **`start()`：启动一个新线程并使线程进⼊就绪状态。当分配到时间⽚后就可以开始运⾏ `run()` 方法中的代码。每个线程对象的 `start()` 方法只能调用一次**



### `sleep()`

+ **在哪个线程里被调用，就会让哪个线程休眠**

+ **调用 `sleep` 会让当前线程从 `Runnable` 进入 `Timed Waiting` 阻塞状态 → 有时限的等待状态**

+ **其它线程可以使用 `interrupt` 方法打断正在休眠的线程，这时 `sleep` 方法会抛出 `InterruptedException`**

+ **睡眠结束后的线程未必会立刻得到执行 → 也许CPU正在执行其他线程的代码，所以要等任务调度器把新的时间片分给这个线程以后才能运行**

```java
Thread t1 = new Thread("t1") {
    @Override
    public void run() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        log.debug("running...");
    }
};
```



### `yield()`

**调用 `yield()` 会暂停当前线程的执行，然后调度执行其它线程**



- **`Runnable` 就绪状态 → 还是有机会被任务调度器分配的**
- **`Timed Waiting` 阻塞状态 → 不会被任务调度器分配**



### `join()`

+ **`join()`：等待线程运行结束**
+ **`join(long n)`：等待线程运行结束，最多等待n毫秒**



### `interrupt()`

+ **打断阻塞状态的线程(`sleep()` & `wait()` & `join()`)，会清除打断标记**
+ **打断正常运行的线程, 不会清除打断标记。打断不会影响线程的正常运行，只会知道有其他线程在进行打断**
+ **打断 `park()` 线程, 不会清除打断标记**



+ **`isInterrupted()`：判断是否被打断，不会清除打断标记**
+ **`interrupted()`：判断是否被打断，会清除打断标记**



### **主线程与守护线程**

**默认情况下，Java 进程需要等待所有线程都运行结束，才会结束。**

**有一种特殊的线程叫做守护线程，只要其它非守护线程运行结束了，即使守护线程的代码没有执行完，也会强制结束。**

**垃圾回收器线程就是一种守护线程。**



### 临界区

**一段代码块内如果存在对共享资源的多线程读写操作，称这段代码块为临界区**



### `synchronized`

**采用互斥的方式让同一 时刻至多只有一个线程能持有对象锁，其它线程再想获取这个对象锁时就会阻塞住。这样就能保证拥有锁的线程可以安全的执行临界区内的代码，保证了临界区内代码的原子性，不用担心线程上下文切换**

```java
synchronized(对象) { // 需要保证多个线程对同一个对象加synchronized
    临界区 
}
```

+ **加在方法上等价于锁住实例对象**
+ **加在静态方法上等价于锁住类对象**



### Monitor

**每个 `Java` 对象都可以关联一个 Monitor 对象，如果使用 `synchronized` 给对象上锁（重量级）之后，该对象头的 Mark Word 中就被设置为指向 Monitor 对象的指针**

**对象头：**

<img src="Java%E5%B9%B6%E5%8F%91.assets/Screen%20Shot%202023-08-26%20at%2011.15.37%20PM.png" alt="Screen Shot 2023-08-26 at 11.15.37 PM" style="zoom: 50%;" />

<img src="Java%E5%B9%B6%E5%8F%91.assets/Screen%20Shot%202023-08-26%20at%2011.16.20%20PM.png" alt="Screen Shot 2023-08-26 at 11.16.20 PM" style="zoom:50%;" />



### **轻量级锁**

**如果一个对象虽然有多线程要加锁，但加锁的时间是错开的(也就是没有竞争)，那么可以使用轻量级锁来优化。** 



+ **创建锁记录（Lock Record）对象，每个线程的栈帧都包含一个锁记录的结构，内部可以存储锁定对象的 Mark Word** 
+ **让锁记录中 Object reference 指向锁定对象，并尝试用 cas 替换锁定对象的 Mark Word，将 Mark Word 的值存入锁记录**
+ **如果 cas 替换成功，表示该线程成功给对象加锁**
+ **如果 cas 失败，有两种情况**
  + **如果是其它线程已经持有了该锁定对象的轻量级锁，这时表明有竞争，进入锁膨胀过程**
  + **如果是自己执行了 `synchronized` 锁重入，那么再添加一条 Lock Record 作为重入的计数(锁重入 → 当前线程又一次给同一个对象加锁)**
+ **解锁**
  + **当退出 `synchronized` 代码块时，如果有取值为 null 的锁记录，表示有重入，这时重置锁记录，表示重入计数减一**
  + **当退出 `synchronized` 代码块时锁记录的值不为 null，这时尝试用 cas 将 Mark Word 的值恢复给对象头**
    + **成功，则解锁成功**
    + **失败，说明轻量级锁进行了锁膨胀或已经升级为重量级锁，进入重量级锁解锁流程**



### 锁膨胀

**如果在尝试加轻量级锁的过程中，cas 操作无法成功，这时一种情况就是有其它线程为此对象加上了轻量级锁（有竞争），这时需要进行锁膨胀，将轻量级锁变为重量级锁。**

+ **当 Thread-1 进行轻量级加锁时，Thread-0 已经对该对象加了轻量级锁**
+ **这时 Thread-1 加轻量级锁失败，进入锁膨胀流程**
  + **为锁定对象申请 Monitor 锁，让锁定对象指向重量级锁地址**
  + **然后自己进入 Monitor 的 EntryList BLOCKED**
+ **当 Thread-0 退出同步块解锁时，使用 cas 将 Mark Word 的值恢复给对象头，失败。这时会进入重量级解锁 流程，即按照 Monitor 地址找到 Monitor 对象，设置 Owner 为 null，唤醒 EntryList 中的 BLOCKED 线程**



### 自旋优化

**自旋指在一个循环中不断检查资源是否可用。重量级锁竞争的时候，可以使用自旋来进行优化，如果当前线程自旋成功（即这时候持锁线程已经退出了同步块，释放了锁），这时当前线程就可以避免阻塞。**

- **在 Java 6 之后自旋锁是自适应的，比如对象刚刚的一次自旋操作成功过，那么认为这次自旋成功的可能性会 高，就多自旋几次；反之，就少自旋甚至不自旋**
- **Java 7 之后不能控制是否开启自旋功能**



### 偏向锁

+ **轻量级锁在没有竞争时，每次重入仍然需要执行 CAS 操作。**

- **Java 6 中引入了偏向锁来做进一步优化：在第一次加锁时使用 CAS 将线程 ID 设置到对象头的 Mark Word 上，之后发现这个线程 ID 是自己的就表示没有竞争，不用重新 CAS。以后只要不发生竞争，这个对象就归该线程所有**

```java
static final Object obj = new Object();
public static void method1() {
    synchronized( obj ) {
        // 同步块 A
        method2();
    }
}

public static void method2() {
    synchronized (obj) {
        // 同步块 B
        method3();
    }
}

public static void method3() {
    synchronized (obj) {
        // 同步块 C
    }
}
```

+ **调用 hashCode 会导致偏向锁被撤销**
+ **当有其它线程使用偏向锁对象时，会将偏向锁升级为轻量级锁**
+ **wait/notify 等待通知机制只有重量级锁才有，调用 wait/notify 会导致偏向锁被撤销**



### wait & notify

+ **Owner 线程发现条件不满足，调用 wait 方法，即可进入 WaitSet 变为 WAITING 状态，处于阻塞状态，不占用 CPU 时间片**
+ **BLOCKED 线程会在 Owner 线程释放锁时唤醒**
+ **WAITING 线程会在 Owner 线程调用 notify 或 notifyAll 时唤醒，但唤醒后并不意味者立刻获得锁，仍需进入 EntryList 重新竞争**



- **`obj.wait()` →  让已经获得锁的线程到 waitSet 等待**
  - **`wait()` → 无限制等待，直到 notify 为止**
  - **`wait(Long n)` → 有时限的等待, 到 n 毫秒后结束等待，或是被 notify**
- **`obj.notify()` → 在正在 waitSet 等待的线程中挑一个唤醒**
- **`obj.notifyAll()` → 让正在 waitSet 等待的线程全部唤醒**

**必须获得此对象的锁，才能调用这几个方法**



### `sleep(Long n)` 和 `wait(Long n)` 的区别

- **`sleep` 是 Thread 方法，而 `wait` 是 Object 的方法**
- **`sleep` 不需要强制和 `synchronized` 配合使用，但 `wait` 需要 和 `synchronized` 一起用**
- **`sleep` 在睡眠的同时，不会释放对象锁的，但 `wait` 在等待的时候会释放对象锁**
- **共同点：它们的状态都是 TIMED_WAITING**



### Park & Unpark

**它们是 LockSupport 类中的方法**

```java
// 暂停当前线程 
LockSupport.park();

// 恢复某个线程的运行
LockSupport.unpark(暂停线程对象);
```

```Java
@Slf4j(topic = "c.TestParkUnpark")
public class TestParkUnpark {
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            log.debug("start...");
            sleep(1);
            log.debug("park...");
            LockSupport.park();
            log.debug("resume...");
        }, "t1");
        t1.start();

        sleep(2);
        log.debug("unpark...");
        LockSupport.unpark(t1);
    }
}
```

- **`wait`，`notify` 和 `notifyAll` 必须配合 Object Monitor 一起使用，而 `park`，`unpark` 不必**
- **`park` & `unpark` 是以线程为单位来阻塞和唤醒线程，而 `notify` 只能随机唤醒一个等待线程，`notifyAll`是唤醒所有等待线程，就不那么精确**
- **`park` & `unpark` 可以先 `unpark`，而 `wait` & `notify` 不能先 `notify`**



### 线程状态

+ **NEW 初始状态，线程被创建，但是还没有调用 `start()` 方法**
+ **RUNNABLE  可运行状态**
+ **BLOCKED 阻塞状态，表示线程阻塞于锁**
+ **WAITING 等待状态**
+ **TIMED_WAITING 指定时间等待状态**
+ **TERMINATED 终止状态**



### **线程状态转换**

+ **NEW --> RUNNABLE**
  + **当调用 `t.start()` 方法时，由 NEW --> RUNNABLE 可以交给CPU调度执行**
+ **RUNNABLE <--> WAITING**
  + **t 线程用 `synchronized(obj)` 获取了对象锁后**
    - **调用 `obj.wait()` 方法时，t 线程从 `RUNNABLE --> WAITING`**
    - **调用 `obj.notify()`，`obj.notifyAll()`，`t.interrupt()` 时**
      - **竞争锁成功，t 线程从 `WAITING --> RUNNABLE`**
      - **竞争锁失败，t 线程从 `WAITING --> BLOCKED`**

+ **RUNNABLE <--> WAITING**
  + **当前线程调用 `t.join()` 方法时，当前线程从 `RUNNABLE --> WAITING`**
    - **注意是当前线程在t 线程对象的监视器上等待**
  + **t 线程运行结束，或调用了当前线程的 `interrupt()` 时，当前线程从 `WAITING --> RUNNABLE`**

+ **RUNNABLE <--> WAITING**
  + **当前线程调用 `LockSupport.park()` 方法会让当前线程从 `RUNNABLE --> WAITING`**
  + **调用 `LockSupport.unpark(目标线程)` 或调用了线程 的 `interrupt()` ，会让目标线程从 `WAITING -→ RUNNABLE`**

+ **RUNNABLE <--> TIMED_WAITING**
  + **t 线程用 `synchronized(obj)` 获取了对象锁后**
    - **调用 `obj.wait(long n)` 方法时，t 线程从 `RUNNABLE --> TIMED_WAITING`**
    - **t 线程等待时间超过了 n 毫秒，或调用 `obj.notify()`，`obj.notifyAll()`，`t.interrupt()` 时**
      - **竞争锁成功，t 线程从 `TIMED_WAITING --> RUNNABLE`**
      - **竞争锁失败，t 线程从 `TIMED_WAITING --> BLOCKED`**

+ **RUNNABLE <--> TIMED_WAITING**
  + **当前线程调用 `t.join(long n)` 方法时，当前线程从 `RUNNABLE --> TIMED_WAITING`**
    - **注意是当前线程在t 线程对象的监视器上等待**
  + **t 线程等待时间超过了 n 毫秒，或t 线程运行结束，或调用了当前线程的 `interrupt()` 时，当前线程从 `TIMED_WAITING --> RUNNABLE`**

+ **RUNNABLE <--> TIMED_WAITING**
  + **当前线程调用 `Thread.sleep(long n)` ，当前线程从 `RUNNABLE --> TIMED_WAITING`**
  + **当前线程等待时间超过了 n 毫秒，当前线程从 `TIMED_WAITING --> RUNNABLE`**

+ **RUNNABLE <--> TIMED_WAITING**
  + **当前线程调用 `LockSupport.parkNanos(long nanos)` 或 `LockSupport.parkUntil(long millis)` 时，当前线程从 `RUNNABLE --> TIMED_WAITING`**
  + **调用 `LockSupport.unpark(目标线程)` 或调用了线程 的 `interrupt()`，或是等待超时，会让目标线程从 `TIMED_WAITING--> RUNNABLE`**

+ **RUNNABLE <--> BLOCKED**
  + **t 线程用 `synchronized(obj)` 获取了对象锁时如果竞争失败，从 `RUNNABLE --> BLOCKED`**
  + **持 obj 锁线程的同步代码块执行完毕，会唤醒该对象上所有 `BLOCKED` 的线程重新竞争，如果其中 t 线程竞争 成功，从 `BLOCKED --> RUNNABLE` ，其它失败的线程仍然 `BLOCKED`**

+ **RUNNABLE <--> TERMINATED**
  + **当前线程所有代码运行完毕，进入 TERMINATED**



### `ThreadLocal`

**`ThreadLocal`是Java中的一个类，用于在多线程环境中存储线程本地的变量。每个线程都可以独立地访问自己的线程本地变量**



### Java 内存模型 JMM

- **主存 → 所有线程共享的数据**
- **工作内存 → 各个线程私有的数据，包括局部变量等**



- **原子性 - 确保多个线程操作共享变量的时候，每个操作都是不可分割的，保证指令不会受到线程上下文切换的影响 **
- **可见性 - 确保一个线程对共享变量的修改对其他线程是可见的**
- **有序性 - 确保程序执行的顺序与代码中的顺序一致**





### `volatile`

+ **`volatile` 可以用来修饰成员变量和静态成员变量，可以避免线程从自己的工作内存中获取变量的值，必须到主存中获取它的值，线程操作 `volatile` 变量都是直接操作主存**
  + **不能解决指令交错**
  
  + **`synchronized` 语句块既可以保证代码块的原子性，也同时保证代码块内变量的可见性。但缺点是 `synchronized` 是属于重量级操作，性能相对更低**
  
+ **`volatile` 可以禁用指令重排**



### `volatile` 原理

**`volatile` 的底层实现原理是内存屏障**

- **对 `volatile` 变量的写指令后加入写屏障**
  - **写屏障保证在该屏障之前的对共享变量的改动，都会同步到主存当中 -> 可见性**
  - **写屏障会确保指令重排序时，不会将写屏障之前的代码排在写屏障之后 -> 有序性**
- **对 `volatile` 变量的读指令前加入读屏障**
  - **读屏障保证在该屏障之后对共享变量的读取，加载的是主存中最新数据 -> 可见性**
  - **读屏障会确保指令重排序时，不会将读屏障之后的代码排在读屏障之前 -> 有序性**



### `volatile` 与 `synchronized` 的区别

+ **`volatile` 只能修饰变量⽽ `synchronized` 可以修饰⽅法以及代码块**
+ **`volatile` 可以保证可见性、有序性，⽽ `synchronized` 可以保证可见性、有序性、原子性**



### **CAS**

**Compare And Swap 比较并交换，体现了乐观锁的思想，在无锁情况下保证多线程操作共享数据的原子性**

**`synchronized` -> 悲观锁**



### Atomic 原子类

**使用 CAS 保证操作的原子性，从而避免了锁机制所带来的性能开销**



### 死锁

**死锁指两个或多个线程相互等待对方释放资源，从而导致所有线程都无法继续执行的情况**

- **互斥：一个资源一次只能被一个线程使用**
- **占有并等待：一个线程至少持有一个资源，并在等待另一个被其它线程持有的资源**
  - **破坏方式：一个线程开始运行之前，必须一次性申请它所需要的全部资源**

- **非抢占：已经分配给一个线程的资源不能被强制性抢占，只能由线程任务结束之后自愿释放**
- **循环等待：若干线程之间形成一种头尾相接的环形等待资源关系，该环路中的每个线程都在等待下一个线程所占有的资源**
  - **破坏方式：对资源进行排序，每个线程按照顺序请求资源**



### AQS

+ **AQS是构建阻塞式锁和相关的同步器工具的框架，ReentrantLock、Semaphore都是基于AQS实现的**
+ **内部维护了一个先进先出的双向队列，队列中存储排队的线程**
+ **用 state 属性来表示资源的状态，分独占模式和共享模式**
  + **独占模式：只有一个线程能够访问资源**
    + **公平锁：按照线程在队列中的排队顺序，先到者先获得锁**
    + **⾮公平锁：⽆视排队顺序直接去抢锁，不保证公平性**
  + **共享模式：允许多个线程访问资源**
+ **在对 state 修改的时候使用的 cas 操作，保证多个线程修改 state 时的线程安全**
+ **使用条件变量来实现等待、唤醒机制，支持多个条件变量**



### AQS 组件

+ **`Semaphore`：是一种基于计数的同步器，用于控制同时访问某个资源的线程数量。允许多个线程同时访问**
+ **`CountDownLatch`：是一种同步工具，⽤来协调多个线程之间的同步，让某⼀个线程等待直到倒计时结束，再开始执⾏**
+ **`CyclicBarrier`：`CountdownLatch` 不能重用，`CyclicBarrier` 可以重用，计数可以恢复到初始值**



### ReentrantLock

+ **可中断**
  + **允许线程在等待锁时被中断 `lock.lockInterruptibly()`**
+ **可设置超时时间**
  + **防止无限期等待下去 `lock.tryLock(2, TimeUnit.SECONDS)`**
+ **可设置为公平锁，默认为非公平锁**
+ **支持多个条件变量**
  + **`await()` & `signal()`**
+ **与 `synchronized` 一样，都支持可重入**
+ **CAS + AQS 实现**
  + **线程尝试使用 cas 修改 state 状态，如果成功修改状态为1，则表示获取锁成功**
  + **如果修改状态失败，则会进入双向队列中等待锁的释放**

```java
// 获取锁
reentrantLock.lock();
try {
    // 临界区
} finally {
    // 释放锁
    reentrantLock.unlock();
}
```



### **`synchronized` 和 Lock 有什么区别**

+ **`synchronized` 是关键字，用c++语言实现；Lock 是接口，用java语言实现**
+ **使用 `synchronized` 时，退出同步代码块锁会自动释放；使用 Lock 时，需要手动调用 unlock 方法释放锁**
+ **Lock 提供了许多 `synchronized` 不具备的功能，例如可中断、可设置超时时间、可设置为公平锁、支持多个条件变量**



### 线程池核心参数

<img src="Java%E5%B9%B6%E5%8F%91.assets/Screen%20Shot%202023-09-03%20at%202.08.08%20PM.png" alt="Screen Shot 2023-09-03 at 2.08.08 PM" style="zoom: 50%;" />

+ **corePoolSize 核心线程数**
+ **maximumPoolSize 最大线程数 (核心线程数 + 救急线程数)**
+ **keepAliveTime 救急线程生存时间(生存时间内没有新任务，此线程资源会释放)**
+ **unit 时间单位 - 救急线程生存时间单位**
+ **workQueue 阻塞队列(核心线程用完后，新的任务将会加入阻塞队列中排队等待，队列满会创建救急线程执行任务)**
+ **threadFactory 线程工厂(可以定制线程对象的创建，例如设置线程名宇、是否是守护线程等)**
+ **handler 拒绝策略(当所有线程都在繁忙，阻塞队列也放满时，会触发拒绝策略)**
  + **AbortPolicy 让调用者抛出异常，这是默认策略**
  + **CallerRunsPolicy 让调用者执行任务**
  + **DiscardPolicy 放弃本次任务**
  + **DiscardOldestPolicy 放弃阻塞队列中最早的任务，并执行当前任务**




### 如何确定核心线程数

+ **CPU 密集型运算 → cpu 核数 + 1**
+ **I/O 密集型运算 → 核数 * 期望 CPU 利用率 * 总时间(CPU计算时间+等待时间) / CPU 计算时间**



### 线程池种类有哪些

+ **`newFixedThreadPool` 固定大小线程池**
  + **核心线程数 = 最大线程数（没有救急线程被创建）**
  + **阻塞队列采用了 `LinkedBlockingQueue`，是无界的，可以放任意数量的任务**
  + **适用于任务量已知，且相对耗时的任务(线程不会频繁地创建和销毁。这可以降低线程创建和上下文切换的开销)**
+ **`newCachedThreadPool`**
  + **核心线程数是 0，创建的全部都是救急线程，救急线程可以无限创建**
  + **阻塞队列采用了 `SynchronousQueue`，每个插入操作都必须等待一个移出操作**
  + **适合任务数比较密集，但每个任务执行时间较短的情况**
+ **``newSingleThreadExecutor`` 单线程线程池**
  + **核心线程数和最大线程数都是1**
  + **阻塞队列采用了 `LinkedBlockingQueue`，是无界的，可以放任意数量的任务**
  + **适用于按照顺序执行的任务**
+ **`newScheduledThreadPool`**
  + **支持延迟和周期执行功能**



### 线程池好处

+ **线程重用：通过重复利⽤已创建的线程降低线程的创建和销毁开销**
+ **资源管理：限制并发线程的数量，防止无限制地创建新线程，从而避免资源耗尽和系统性能下降**
+ **任务调度：进行有效的调度和管理任务执行顺序**



### 执⾏ `execute()`⽅法和 `submit()` ⽅法的区别

+ **`execute()` 方法用于没有返回值的任务**
+ **`submit()` 方法用于有返回值的任务，返回一个`Future`对象**



#### `ConcurrentHashMap`

+ **JDK1.7：底层采⽤ 分段的数组+链表 实现**
  + **使用了一种分段锁的机制，其底层数据结构是 `Segment` 数组，使用 `ReentrantLock` 保证线程安全，确保了多线程对不同 `Segment` 的数据进行访问时不会存在锁竞**
+ **JDK1.8：底层采⽤ 数组+链表/红⿊树 实现**
  + **使⽤ `synchronized` 和 CAS 来实现并发安全**
  + **CAS 控制数组节点的添加**
  + **`synchronized` 只锁定当前链表或红黑树的首节点，只要不存在哈希冲突，就不会产生并发问题**



#### **`CopyOnWriteArrayList`**

+ **采用了 写入时拷贝 的思想，增删改操作会将底层数组拷贝一份，更改操作在新数组上执行，这时不影响其它线程的并发读，读写分离。**

+ **适合读多写少的应用场景**
+ **`get` 方法弱一致性**







