## Java并发面试题

### 一、线程池相关 （⭐⭐⭐）

#### 1、什么是线程池，如何使用？为什么要使用线程池？

答：线程池就是事先将多个线程对象放到一个容器中，使用的时候就不用new线程而是直接去池中拿线程即可，节
省了开辟子线程的时间，提高了代码执行效率。

#### 2、Java中的线程池共有几种？

Java有四种线程池：

第一种：newCachedThreadPool

不固定线程数量，且支持最大为Integer.MAX_VALUE的线程数量:

    public static ExecutorService newCachedThreadPool() {
        // 这个线程池corePoolSize为0，maximumPoolSize为Integer.MAX_VALUE
        // 意思也就是说来一个任务就创建一个woker，回收时间是60s
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
    }
    
可缓存线程池：

1、线程数无限制。
2、有空闲线程则复用空闲线程，若无空闲线程则新建线程。
3、一定程序减少频繁创建/销毁线程，减少系统开销。

第二种：newFixedThreadPool

一个固定线程数量的线程池:

    public static ExecutorService newFixedThreadPool(int nThreads, ThreadFactory threadFactory) {
        // corePoolSize跟maximumPoolSize值一样，同时传入一个无界阻塞队列
        // 该线程池的线程会维持在指定线程数，不会进行回收
        return new ThreadPoolExecutor(nThreads, nThreads,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>(),
                                    threadFactory);
    }
    
定长线程池：

1、可控制线程最大并发数（同时执行的线程数）。
2、超出的线程会在队列中等待。

第三种：newSingleThreadExecutor

可以理解为线程数量为1的FixedThreadPool:

    public static ExecutorService newSingleThreadExecutor() {
        // 线程池中只有一个线程进行任务执行，其他的都放入阻塞队列
        // 外面包装的FinalizableDelegatedExecutorService类实现了finalize方法，在JVM垃圾回收的时候会关闭线程池
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }

单线程化的线程池：

1、有且仅有一个工作线程执行任务。
2、所有任务按照指定顺序执行，即遵循队列的入队出队规则。

第四种：newScheduledThreadPool。

支持定时以指定周期循环执行任务:

    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
    
注意：前三种线程池是ThreadPoolExecutor不同配置的实例，最后一种是ScheduledThreadPoolExecutor的实例。


#### 3、[线程池原理？](https://itimetraveler.github.io/2018/02/13/%E3%80%90Java%E3%80%91%E7%BA%BF%E7%A8%8B%E6%B1%A0ThreadPoolExecutor%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/)

从数据结构的角度来看，线程池主要使用了阻塞队列（BlockingQueue）和HashSet集合构成。
从任务提交的流程角度来看，对于使用线程池的外部来说，线程池的机制是这样的：

    1、如果正在运行的线程数 < coreSize，马上创建核心线程执行该task，不排队等待；
    2、如果正在运行的线程数 >= coreSize，把该task放入阻塞队列；
    3、如果队列已满 && 正在运行的线程数 < maximumPoolSize，创建新的非核心线程执行该task；
    4、如果队列已满 && 正在运行的线程数 >= maximumPoolSize，线程池调用handler的reject方法拒绝本次提交。
    
理解记忆：1-2-3-4对应（核心线程->阻塞队列->非核心线程->handler拒绝提交）。

##### 线程池的线程复用：

这里就需要深入到源码addWorker()：它是创建新线程的关键，也是线程复用的关键入口。最终会执行到runWoker，它取任务有两个方式： 

- firstTask：这是指定的第一个runnable可执行任务，它会在Woker这个工作线程中运行执行任务run。并且置空表示这个任务已经被执行。 
- getTask()：这首先是一个死循环过程，工作线程循环直到能够取出Runnable对象或超时返回，这里的取的目标就是任务队列workQueue，对应刚才入队的操作，有入有出。

其实就是任务在并不只执行创建时指定的firstTask第一任务，还会从任务队列的中通过getTask()方法自己主动去取任务执行，而且是有/无时间限定的阻塞等待，保证线程的存活。

##### 信号量

semaphore 可用于进程间同步也可用于同一个进程间的线程同步。

可以用来保证两个或多个关键代码段不被并发调用。在进入一个关键代码段之前，线程必须获取一个信号量；一旦该关键代码段完成了，那么该线程必须释放信号量。其它想进入该关键代码段的线程必须等待直到第一个线程释放信号量。


#### 4、线程池都有哪几种工作队列？

1、ArrayBlockingQueue

是一个基于数组结构的有界阻塞队列，此队列按 FIFO（先进先出）原则对元素进行排序。

2、LinkedBlockingQueue

一个基于链表结构的阻塞队列，此队列按FIFO （先进先出） 排序元素，吞吐量通常要高于ArrayBlockingQueue。静态工厂方法Executors.newFixedThreadPool()和Executors.newSingleThreadExecutor使用了这个队列。

3、SynchronousQueue

一个不存储元素的阻塞队列。每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue，静态工厂方法Executors.newCachedThreadPool使用了这个队列。

4、PriorityBlockingQueue

一个具有优先级的无限阻塞队列。


#### 5、怎么理解无界队列和有界队列？

##### 有界队列

1.初始的poolSize < corePoolSize，提交的runnable任务，会直接做为new一个Thread的参数，立马执行 。
2.当提交的任务数超过了corePoolSize，会将当前的runable提交到一个block queue中。
3.有界队列满了之后，如果poolSize < maximumPoolsize时，会尝试new 一个Thread的进行救急处理，立马执行对应的runnable任务。
4.如果3中也无法处理了，就会走到第四步执行reject操作。

##### 无界队列

与有界队列相比，除非系统资源耗尽，否则无界的任务队列不存在任务入队失败的情况。当有新的任务到来，系统的线程数小于corePoolSize时，则新建线程执行任务。当达到corePoolSize后，就不会继续增加，若后续仍有新的任务加入，而没有空闲的线程资源，则任务直接进入队列等待。若任务创建和处理的速度差异很大，无界队列会保持快速增长，直到耗尽系统内存。
当线程池的任务缓存队列已满并且线程池中的线程数目达到maximumPoolSize，如果还有任务到来就会采取任务拒绝策略。


#### 6、[多线程中的安全队列一般通过什么实现？](https://blog.csdn.net/bieleyang/article/details/78027032)

Java提供的线程安全的Queue可以分为阻塞队列和非阻塞队列，其中阻塞队列的典型例子是BlockingQueue，非阻塞队列的典型例子是ConcurrentLinkedQueue.

对于BlockingQueue，想要实现阻塞功能，需要调用put(e) take() 方法。而ConcurrentLinkedQueue是基于链接节点的、无界的、线程安全的非阻塞队列。


### 二、Synchronized、volatile、Lock(ReentrantLock)相关 （⭐⭐⭐）

#### 1、synchronized的原理？

##### lock 和 synchronized

- synchronized 是 Java 关键字，内置特性；Lock 是一个接口
- synchronized 会自动释放锁；lock 需要手动释放，所以需要写到 try catch 块中并在 finally 中释放锁
- synchronized 无法中断等待锁；lock 可以中断
- Lock 可以提高多个线程进行读/写操作的效率
- 竞争资源激烈时，lock 的性能会明显的优于 synchronized

synchronized 代码块是由一对儿 monitorenter/monitorexit 指令实现的，Monitor 对象是同步的基本实现，而 synchronized 同步方法使用了ACC_SYNCHRONIZED访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用。

在 Java 6 之前，Monitor 的实现完全是依靠操作系统内部的互斥锁，因为需要进行用户态到内核态的切换，所以同步操作是一个无差别的重量级操作。

现代的（Oracle）JDK 中，JVM 对此进行了大刀阔斧地改进，提供了三种不同的 Monitor 实现，也就是常说的三种不同的锁：偏斜锁（Biased Locking）、轻量级锁和重量级锁，大大改进了其性能。

所谓锁的升级、降级，就是 JVM 优化 synchronized 运行的机制，当 JVM 检测到不同的竞争状况时，会自动切换到适合的锁实现，这种切换就是锁的升级、降级。

当没有竞争出现时，默认会使用偏斜锁。JVM 会利用 CAS 操作，在对象头上的 Mark Word 部分设置线程 ID，以表示这个对象偏向于当前线程，所以并不涉及真正的互斥锁。这样做的假设是基于在很多应用场景中，大部分对象生命周期中最多会被一个线程锁定，使用偏斜锁可以降低无竞争开销。

如果有另外的线程试图锁定某个已经被偏斜过的对象，JVM 就需要撤销（revoke）偏斜锁，并切换到轻量级锁实现。轻量级锁依赖 CAS 操作 Mark Word 来试图获取锁，如果重试成功，就使用普通的轻量级锁；否则，进一步升级为重量级锁（可能会先进行自旋锁升级，如果失败再尝试重量级锁升级）。

我注意到有的观点认为 Java 不会进行锁降级。实际上据我所知，锁降级确实是会发生的，当 JVM 进入安全点（SafePoint）的时候，会检查是否有闲置的 Monitor，然后试图进行降级。


#### 2、Synchronized优化后的锁机制简单介绍一下，包括自旋锁、偏向锁、轻量级锁、重量级锁？

自旋锁：

线程自旋说白了就是让cpu在做无用功，比如：可以执行几次for循环，可以执行几条空的汇编指令，目的是占着CPU不放，等待获取锁的机会。如果旋的时间过长会影响整体性能，时间过短又达不到延迟阻塞的目的。

偏向锁

偏向锁就是一旦线程第一次获得了监视对象，之后让监视对象“偏向”这个线程，之后的多次调用则可以避免CAS操作，说白了就是置个变量，如果发现为true则无需再走各种加锁/解锁流程。

轻量级锁：

轻量级锁是由偏向所升级来的，偏向锁运行在一个线程进入同步块的情况下，当第二个线程加入锁竞争用的时候，偏向锁就会升级为轻量级锁；

重量级锁

重量锁在JVM中又叫对象监视器（Monitor），它很像C中的Mutex，除了具备Mutex(0|1)互斥的功能，它还负责实现了Semaphore(信号量)的功能，也就是说它至少包含一个竞争锁的队列，和一个信号阻塞队列（wait队列），前者负责做互斥，后一个用于做线程同步。


#### 3、谈谈对Synchronized关键字涉及到的类锁，方法锁，重入锁的理解？

synchronized修饰静态方法获取的是类锁(类的字节码文件对象)。

synchronized修饰普通方法或代码块获取的是对象锁。这种机制确保了同一时刻对于每一个类实例，其所有声明为 synchronized 的成员函数中至多只有一个处于可执行状态，从而有效避免了类成员变量的访问冲突。

它俩是不冲突的，也就是说：获取了类锁的线程和获取了对象锁的线程是不冲突的！

    public class Widget {

        // 锁住了
        public synchronized void doSomething() {
            ...
        }
    }

    public class LoggingWidget extends Widget {

        // 锁住了
        public synchronized void doSomething() {
            System.out.println(toString() + ": calling doSomething");
            super.doSomething();
        }
    }

因为锁的持有者是“线程”，而不是“调用”。

线程A已经是有了LoggingWidget实例对象的锁了，当再需要的时候可以继续**“开锁”**进去的！

这就是内置锁的可重入性。


#### 4、wait、sleep的区别和notify运行过程。

##### wait、sleep的区别

- sleep 是 Thread 的静态方法，可以在任何地方调用
- wait 是 Object 的成员方法，只能在 synchronized 代码块中调用，否则会报 IllegalMonitorStateException 非法监控状态异常
- sleep 不会释放共享资源锁，wait 会释放共享资源锁

最大的不同是在等待时 wait 会释放锁，而 sleep 一直持有锁。wait 通常被用于线程间交互，sleep 通常被用于暂停执行。    

- 首先，要记住这个差别，“sleep是Thread类的方法,wait是Object类中定义的方法”。尽管这两个方法都会影响线程的执行行为，但是本质上是有区别的。
- Thread.sleep不会导致锁行为的改变，如果当前线程是拥有锁的，那么Thread.sleep不会让线程释放锁。如果能够帮助你记忆的话，可以简单认为和锁相关的方法都定义在Object类中，因此调用Thread.sleep是不会影响锁的相关行为。
- Thread.sleep和Object.wait都会暂停当前的线程，对于CPU资源来说，不管是哪种方式暂停的线程，都表示它暂时不再需要CPU的执行时间。OS会将执行时间分配给其它线程。区别是，调用wait后，需要别的线程执行notify/notifyAll才能够重新获得CPU执行时间。
- 线程的状态参考 Thread.State的定义。新创建的但是没有执行（还没有调用start())的线程处于“就绪”，或者说Thread.State.NEW状态。
- Thread.State.BLOCKED（阻塞）表示线程正在获取锁时，因为锁不能获取到而被迫暂停执行下面的指令，一直等到这个锁被别的线程释放。BLOCKED状态下线程，OS调度机制需要决定下一个能够获取锁的线程是哪个，这种情况下，就是产生锁的争用，无论如何这都是很耗时的操作。

##### notify运行过程

当线程A（消费者）调用wait()方法后，线程A让出锁，自己进入等待状态，同时加入锁对象的等待队列。
线程B（生产者）获取锁后，调用notify方法通知锁对象的等待队列，使得线程A从等待队列进入阻塞队列。
线程A进入阻塞队列后，直至线程B释放锁后，线程A竞争得到锁继续从wait()方法后执行。


#### 5、synchronized关键字和Lock的区别你知道吗？为什么Lock的性能好一些？


类别 | synchronized | Lock（底层实现主要是Volatile + CAS） | 
---|---|---|
存在层次 | Java的关键字，在jvm层面上 | 是一个类 |
锁的释放 | 1、已获取锁的线程执行完同步代码，释放锁          2、线程执行发生异常，jvm会让线程释放锁。 | 在finally中必须释放锁，不然容易造成线程死锁。 
锁的获取 | 假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待。 | 分情况而定，Lock有多个锁获取的方式，大致就是可以尝试获得锁，线程可以不用一直等待  |
锁状态 | 无法判断 | 可以判断 |
锁类型 | 可重入 不可中断 非公平	| 可重入 可判断 可公平（两者皆可） |
性能 | 少量同步 | 大量同步 |
    
Lock（ReentrantLock）的底层实现主要是Volatile + CAS（乐观锁），而Synchronized是一种悲观锁，比较耗性能。但是在JDK1.6以后对Synchronized的锁机制进行了优化，加入了偏向锁、轻量级锁、自旋锁、重量级锁，在并发量不大的情况下，性能可能优于Lock机制。所以建议一般请求并发量不大的情况下使用synchronized关键字。


#### 6、volatile原理。

- 只能用来修饰变量，适用修饰可能被多线程同时访问的变量
- 相当于轻量级的 synchronized，volatitle 能保证有序性（禁用指令重排序）、可见性
- 变量位于主内存中，每个线程还有自己的工作内存，变量在自己线程的工作内存中有份拷贝，线程直接操作的是这个拷贝
- 被 volatile 修饰的变量改变后会立即同步到主内存，保持变量的可见性

##### 双重检查单例，为什么要加 violate？

volatile想要解决的问题是，在另一个线程中想要使用instance，发现instance!=null，但是实际上instance还未初始化完毕这个问题。将instance = newInstance();拆分为3句话是。1.分配内存2.初始化3.将instance指向分配的内存空间，volatile可以禁止指令重排序，确保先执行2，后执行3

在《Java并发编程：核心理论》一文中，我们已经提到可见性、有序性及原子性问题，通常情况下我们可以通过Synchronized关键字来解决这些个问题，不过如果对Synchonized原理有了解的话，应该知道Synchronized是一个较重量级的操作，对系统的性能有比较大的影响，所以如果有其他解决方案，我们通常都避免使用Synchronized来解决问题。
    
而volatile关键字就是Java中提供的另一种解决可见性有序性问题的方案。对于原子性，需要强调一点，也是大家容易误解的一点：对volatile变量的单次读/写操作可保证原子性的，如long和double类型变量，但是并不能保证i++这种操作的原子性，因为本质上i++是读、写两次操作。

volatile也是互斥同步的一种实现，不过它非常的轻量级。

##### volatile 的意义？

- 防止CPU指令重排序

volatile有两条关键的语义：

保证被volatile修饰的变量对所有线程都是可见的

禁止进行指令重排序

要理解volatile关键字，我们得先从Java的线程模型开始说起。如图所示：

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/native/process/java_memory_model.png)

Java内存模型规定了所有字段（这些字段包括实例字段、静态字段等，不包括局部变量、方法参数等，因为这些是线程私有的，并不存在竞争）都存在主内存中，每个线程会 有自己的工作内存，工作内存里保存了线程所使用到的变量在主内存里的副本拷贝，线程对变量的操作只能在工作内存里进行，而不能直接读写主内存，当然不同内存之间也 无法直接访问对方的工作内存，也就是说主内存是线程传值的媒介。

我们来理解第一句话：

    保证被volatile修饰的变量对所有线程都是可见的

如何保证可见性？

被volatile修饰的变量在工作内存修改后会被强制写回主内存，其他线程在使用时也会强制从主内存刷新，这样就保证了一致性。

关于“保证被volatile修饰的变量对所有线程都是可见的”，有种常见的错误理解：

- 由于volatile修饰的变量在各个线程里都是一致的，所以基于volatile变量的运算在多线程并发的情况下是安全的。

这句话的前半部分是对的，后半部分却错了，因此它忘记考虑变量的操作是否具有原子性这一问题。

举个例子：

    private volatile int start = 0;

    private void volatile Keyword() {

        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    start++;
                }
            }
        };

        for (int i = 0; i < 10; i++) {
            Thread thread = new Thread(runnable);
            thread.start();
        }
        Log.d(TAG, "start = " + start);
    }

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/native/process/volatile_thread_safe.png)

这段代码启动了10个线程，每次10次自增，按道理最终结果应该是100，但是结果并非如此。

为什么会这样？

仔细看一下start++，它其实并非一个原子操作，简单来看，它有两步：

1、取出start的值，因为有volatile的修饰，这时候的值是正确的。

2、自增，但是自增的时候，别的线程可能已经把start加大了，这种情况下就有可能把较小的start写回主内存中。
所以volatile只能保证可见性，在不符合以下场景下我们依然需要通过加锁来保证原子性：

- 运算结果并不依赖变量当前的值，或者只有单一线程修改变量的值。（要么结果不依赖当前值，要么操作是原子性的，要么只要一个线程修改变量的值）
- 变量不需要与其他状态变量共同参与不变约束
比方说我们会在线程里加个boolean变量，来判断线程是否停止，这种情况就非常适合使用volatile。

我们再来理解第二句话。

禁止进行指令重排序

什么是指令重排序？

- 指令重排序是指指令乱序执行，即在条件允许的情况下直接运行当前有能力立即执行的后续指令，避开为获取一条指令所需数据而造成的等待，通过乱序执行的技术提供执行效率。

- 指令重排序会在被volatile修饰的变量的赋值操作前，添加一个内存屏障，指令重排序时不能把后面的指令重排序移到内存屏障之前的位置。


#### 7、synchronized 和 volatile 关键字的作用和区别。

##### Volatile

1）保证了不同线程对这个变量进行操作时的可见性即一个线程修改了某个变量的值，这新值对其他线程来是立即可见的。

2）禁止进行指令重排序。

##### 作用

volatile 本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需从主存中读取；synchronized则是锁定当前变量，只有当前线程可以访问该变量，其它线程被阻塞住。

##### 区别

1.volatile 仅能使用在变量级别；synchronized则可以使用在变量、方法、和类级别的。

2.volatile 仅能实现变量的修改可见性，并不能保证原子性；synchronized 则可以保证变量的修改可见性和原子性。

3.volatile 不会造成线程的阻塞；synchronized 可能会造成线程的阻塞。

4.volatile 标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化。


#### 8、[ReentrantLock的内部实现](https://www.cnblogs.com/chengxiao/p/7255941.html)。

ReentrantLock实现的前提就是AbstractQueuedSynchronizer，简称AQS，是java.util.concurrent的核心，CountDownLatch、FutureTask、Semaphore、ReentrantLock等都有一个内部类是这个抽象类的子类。由于AQS是基于FIFO队列的实现，因此必然存在一个个节点，Node就是一个节点，Node有两种模式：共享模式和独占模式。ReentrantLock是基于AQS的，AQS是Java并发包中众多同步组件的构建基础，它通过一个int类型的状态变量state和一个FIFO队列来完成共享资源的获取，线程的排队等待等。AQS是个底层框架，采用模板方法模式，它定义了通用的较为复杂的逻辑骨架，比如线程的排队，阻塞，唤醒等，将这些复杂但实质通用的部分抽取出来，这些都是需要构建同步组件的使用者无需关心的，使用者仅需重写一些简单的指定的方法即可（其实就是对于共享变量state的一些简单的获取释放的操作）。AQS的子类一般只需要重写tryAcquire(int arg)和tryRelease(int arg)两个方法即可。

##### ReentrantLock的处理逻辑：

其内部定义了三个重要的静态内部类，Sync，NonFairSync，FairSync。Sync作为ReentrantLock中公用的同步组件，继承了AQS（要利用AQS复杂的顶层逻辑嘛，线程排队，阻塞，唤醒等等）；NonFairSync和FairSync则都继承Sync，调用Sync的公用逻辑，然后再在各自内部完成自己特定的逻辑（公平或非公平）。

接着说下这两者的lock()方法实现原理：

##### NonFairSync（非公平可重入锁）

1.先获取state值，若为0，意味着此时没有线程获取到资源，CAS将其设置为1，设置成功则代表获取到排他锁了；

2.若state大于0，肯定有线程已经抢占到资源了，此时再去判断是否就是自己抢占的，是的话，state累加，返回true，重入成功，state的值即是线程重入的次数；

3.其他情况，则获取锁失败。

##### FairSync（公平可重入锁）

可以看到，公平锁的大致逻辑与非公平锁是一致的，不同的地方在于有了!hasQueuedPredecessors()这个判断逻辑，即便state为0，也不能贸然直接去获取，要先去看有没有还在排队的线程，若没有，才能尝试去获取，做后面的处理。反之，返回false，获取失败。

最后，说下ReentrantLock的tryRelease()方法实现原理：

若state值为0，表示当前线程已完全释放干净，返回true，上层的AQS会意识到资源已空出。若不为0，则表示线程还占有资源，只不过将此次重入的资源的释放了而已，返回false。

ReentrantLock是一种可重入的，可实现公平性的互斥锁，它的设计基于AQS框架，可重入和公平性的实现逻辑都不难理解，每重入一次，state就加1，当然在释放的时候，也得一层一层释放。至于公平性，在尝试获取锁的时候多了一个判断：是否有比自己申请早的线程在同步队列中等待，若有，去等待；若没有，才允许去抢占。　　

#### 9、ReentrantLock 、synchronized 和 volatile 比较？

synchronized是互斥同步的一种实现。

synchronized：当某个线程访问被synchronized标记的方法或代码块时，这个线程便获得了该对象的锁，其他线暂时无法访问这个方法，只有等待这个方法执行完毕或代码块执行完毕，这个线程才会释放该对象的锁，其他线程才能执行这个方法代码块。

前面我们已经说了volatile关键字，这里我们举个例子来综合分析volatile与synchronized关键字的使用。

举个例子：

    public class Singleton {
    
        // volatile保证了：1 instance在多线程并发的可见性 2 禁止instance在操作是的指令重排序
        private volatile static Singleton instance;
    
        private Singleton(){}
    
        public static Singleton getInstance() {
            // 第一次判空，保证不必要的同步
            if (instance == null) {
                // synchronized对Singleton加全局锁，保证每次只要一个线程创建实例
                synchronized (Singleton.class) {
                    // 第二次判空时为了在null的情况下创建实例
                    if (instance == null) {
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }
    }
    
这是一个经典的DCL单例。

它的字节码如下：

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/native/process/synchronized_bytecode.png)

可以看到被synchronized同步的代码块，会在前后分别加上monitorenter和monitorexit，这两个字节码都需要指定加锁和解锁的对象。

关于加锁和解锁的对象：

synchronized代码块 ：同步代码块，作用范围是整个代码块，作用对象是调用这个代码块的对象。

synchronized方法 ：同步方法，作用范围是整个方法，作用对象是调用这个方法的对象。

synchronized静态方法 ：同步静态方法，作用范围是整个静态方法，作用对象是调用这个类的所有对象。

synchronized(this)：作用范围是该对象中所有被synchronized标记的变量、方法或代码块，作用对象是对象本身。

synchronized(ClassName.class) ：作用范围是静态的方法或者静态变量，作用对象是Class对象。

synchronized(this)添加的是对象锁，synchronized(ClassName.class)添加的是类锁，它们的区别如下：

- 对象锁：Java的所有对象都含有1个互斥锁，这个锁由JVM自动获取和释放。线程进入synchronized方法的时候获取该对象的锁，当然如果已经有线程获取了这个对象的锁那么当前线程会等待；synchronized方法正常返回或者抛异常而终止，JVM会自动释放对象锁。这里也体现了用synchronized来加锁的好处，方法抛异常的时候，锁仍然可以由JVM来自动释放。

- 类锁：对象锁是用来控制实例方法之间的同步，类锁是来控制静态方法（或静态变量互斥体）之间的同步。其实类锁只是一个概念上的东西，并不是真实存在的，它只用来帮助我们理解锁定实例方法和静态方法的区别的。我们都知道，java类可能会有很多个对象，但是只有1个Class对象，也就说类的不同实例之间共享该类的Class对象。Class对象其实也仅仅是1个java对象，只不过有点特殊而已。由于每个java对象都有个互斥锁，而类的静态方法是需要Class对象。所以所谓类锁，不过是Class对象的锁而已。获取类的Class对象有好几种，最简单的就是MyClass.class的方式。类锁和对象锁不是同一个东西，一个是类的Class对象的锁，一个是类的实例的锁。也就是说：一个线程访问静态sychronized的时候，允许另一个线程访问对象的实例synchronized方法。反过来也是成立的，为他们需要的锁是不同的。


### 三、其它 （⭐⭐⭐）

#### 1、多线程的使用场景？

使用多线程就一定效率高吗？有时候使用多线程并不是为了提高效率，而是使得CPU能同时处理多个事件。

- 为了不阻塞主线程,启动其他线程来做事情,比如APP中的耗时操作都不在UI线程中做。

- 实现更快的应用程序,即主线程专门监听用户请求,子线程用来处理用户请求,以获得大的吞吐量.感觉这种情况，多线程的效率未必高。这种情况下的多线程是为了不必等待，可以并行处理多条数据。比如JavaWeb的就是主线程专门监听用户的HTTP请求，然启动子线程去处理用户的HTTP请求。

- 某种虽然优先级很低的服务，但是却要不定时去做。比如Jvm的垃圾回收。

- 某种任务，虽然耗时，但是不消耗CPU的操作时间，开启个线程，效率会有显著提高。比如读取文件，然后处理。磁盘IO是个很耗费时间，但是不耗CPU计算的工作。所以可以一个线程读取数据，一个线程处理数据。肯定比一个线程读取数据，然后处理效率高。因为两个线程的时候充分利用了CPU等待磁盘IO的空闲时间。


#### 2、CopyOnWriteArrayList的了解。

##### Copy-On-Write 是什么？

在计算机中就是当你想要对一块内存进行修改时，我们不在原有内存块中进行写操作，而是将内存拷贝一份，在新的内存中进行写操作，写完之后呢，就将指向原来内存指针指向新的内存，原来的内存就可以被回收掉。

##### 原理：

CopyOnWriteArrayList这是一个ArrayList的线程安全的变体，CopyOnWriteArrayList 底层实现添加的原理是先copy出一个容器(可以简称副本)，再往新的容器里添加这个新的数据，最后把新的容器的引用地址赋值给了之前那个旧的的容器地址，但是在添加这个数据的期间，其他线程如果要去读取数据，仍然是读取到旧的容器里的数据。

##### 优点和缺点:

优点:

1.据一致性完整，为什么？因为加锁了，并发数据不会乱。

2.解决了像ArrayList、Vector这种集合多线程遍历迭代问题，记住，Vector虽然线程安全，只不过是加了synchronized关键字，迭代问题完全没有解决！

缺点:

1.内存占有问题:很明显，两个数组同时驻扎在内存中，如果实际应用中，数据比较多，而且比较大的情况下，占用内存会比较大，针对这个其实可以用ConcurrentHashMap来代替。

2.数据一致性:CopyOnWrite容器只能保证数据的最终一致性，不能保证数据的实时一致性。所以如果你希望写入的的数据，马上能读到，请不要使用CopyOnWrite容器。

##### 使用场景：

1、读多写少（白名单，黑名单，商品类目的访问和更新场景），为什么？因为写的时候会复制新集合。

2、集合不大，为什么？因为写的时候会复制新集合。

3、实时性要求不高，为什么，因为有可能会读取到旧的集合数据。

    
#### 3、ConcurrentHashMap加锁机制是什么，详细说一下？

##### Java7 ConcurrentHashMap

ConcurrentHashMap作为一种线程安全且高效的哈希表的解决方案，尤其其中的"分段锁"的方案，相比HashTable的表锁在性能上的提升非常之大。HashTable容器在竞争激烈的并发环境下表现出效率低下的原因，是因为所有访问HashTable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

ConcurrentHashMap 是一个 Segment 数组，Segment 通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全。

concurrencyLevel：并行级别、并发数、Segment 数。默认是 16，也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。其中的每个 Segment 很像 HashMap，不过它要保证线程安全，所以处理起来要麻烦些。

初始化槽: ensureSegment

ConcurrentHashMap 初始化的时候会初始化第一个槽 segment[0]，对于其他槽来说，在插入第一个值的时候进行初始化。对于并发操作使用 CAS 进行控制。

##### Java8 ConcurrentHashMap

抛弃了原有的 Segment 分段锁，而采用了 CAS + synchronized 来保证并发安全性。结构上和 Java8 的 HashMap（数组+链表+红黑树） 基本上一样，不过它要保证线程安全性，所以在源码上确实要复杂一些。1.8 在 1.7 的数据结构上做了大的改动，采用红黑树之后可以保证查询效率（O(logn)），甚至取消了 ReentrantLock 改为了 synchronized，这样可以看出在新版的 JDK 中对 synchronized 优化是很到位的。


#### 4、线程死锁的4个条件？

死锁是如何发生的，如何避免死锁？

当线程A持有独占锁a，并尝试去获取独占锁b的同时，线程B持有独占锁b，并尝试获取独占锁a的情况下，就会发生AB两个线程由于互相持有对方需要的锁，而发生的阻塞现象，我们称为死锁。

    public class DeadLockDemo {
    
        public static void main(String[] args) {
            // 线程a
            Thread td1 = new Thread(new Runnable() {
                public void run() {
                    DeadLockDemo.method1();
                }
            });
            // 线程b
            Thread td2 = new Thread(new Runnable() {
                public void run() {
                    DeadLockDemo.method2();
                }
            });
    
            td1.start();
            td2.start();
        }
    
        public static void method1() {
            synchronized (String.class) {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("线程a尝试获取integer.class");
                synchronized (Integer.class) {
    
                }
            }
        }
    
        public static void method2() {
            synchronized (Integer.class) {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("线程b尝试获取String.class");
                synchronized (String.class) {
    
                }
            }
        }
    }
    
造成死锁的四个条件：

- 互斥条件：一个资源每次只能被一个线程使用。
- 请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放。
- 不剥夺条件：线程已获得的资源，在未使用完之前，不能强行剥夺。
- 循环等待条件：若干线程之间形成一种头尾相接的循环等待资源关系。

在并发程序中，避免了逻辑中出现数个线程互相持有对方线程所需要的独占锁的的情况，就可以避免死锁，如下所示：

    public class BreakDeadLockDemo {
    
        public static void main(String[] args) {
            // 线程a
            Thread td1 = new Thread(new Runnable() {
                public void run() {
                    DeadLockDemo2.method1();
                }
            });
            // 线程b
            Thread td2 = new Thread(new Runnable() {
                public void run() {
                    DeadLockDemo2.method2();
                }
            });
    
            td1.start();
            td2.start();
        }
    
        public static void method1() {
            synchronized (String.class) {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("线程a尝试获取integer.class");
                synchronized (Integer.class) {
                    System.out.println("线程a获取到integer.class");
                }
    
            }
        }
    
        public static void method2() {
            // 不再获取线程a需要的Integer.class锁。
            synchronized (String.class) {
                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("线程b尝试获取Integer.class");
                synchronized (Integer.class) {
                    System.out.println("线程b获取到Integer.class");
                }
            }
        }
    }


#### 5、CAS介绍？

##### Unsafe

Unsafe是CAS的核心类。因为Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门，JDK中有一个类Unsafe，它提供了硬件级别的原子操作。

##### CAS

CAS，Compare and Swap即比较并交换，设计并发算法时常用到的一种技术，java.util.concurrent包全完建立在CAS之上，没有CAS也就没有此包，可见CAS的重要性。当前的处理器基本都支持CAS，只不过不同的厂家的实现不一样罢了。并且CAS也是通过Unsafe实现的，由于CAS都是硬件级别的操作，因此效率会比普通加锁高一些。

##### CAS的缺点

CAS看起来很美，但这种操作显然无法涵盖并发下的所有场景，并且CAS从语义上来说也不是完美的，存在这样一个逻辑漏洞：如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，那我们就能说明它的值没有被其他线程修改过了吗？如果在这段期间它的值曾经被改成了B，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个漏洞称为CAS操作的"ABA"问题。java.util.concurrent包为了解决这个问题，提供了一个带有标记的原子引用类"AtomicStampedReference"，它可以通过控制变量值的版本来保证CAS的正确性。不过目前来说这个类比较"鸡肋"，大部分情况下ABA问题并不会影响程序并发的正确性，如果需要解决ABA问题，使用传统的互斥同步可能回避原子类更加高效。

##### 底层原理

借助CPU底层指令cmpxchg实现原子操作。


#### 6、进程和线程的区别？

简而言之,一个程序至少有一个进程,一个进程至少有一个线程。

- 1、线程的划分尺度小于进程，使得多线程程序的并发性高。

- 2、进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。

- 3、线程在执行过程中与进程还是有区别的。每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。

- 4、从逻辑角度来看，多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。但操作系统并没有将多个线程看做多个独立的应用，来实现进程的调度和管理以及资源分配。这就是进程和线程的重要区别。

- 5、进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位。线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位.线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源。

- 6、一个线程可以创建和撤销另一个线程;同一个进程中的多个线程之间可以并发执行。

- 7、进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些。


#### 7、什么导致线程阻塞？

线程的阻塞

为了解决对共享存储区的访问冲突，Java 引入了同步机制，现在让我们来考察多个线程对共享资源的访问，显然同步机制已经不够了，因为在任意时刻所要求的资源不一定已经准备好了被访问，反过来，同一时刻准备好了的资源也可能不止一个。为了解决这种情况下的访问控制问题，Java 引入了对阻塞机制的支持.

阻塞指的是暂停一个线程的执行以等待某个条件发生（如某资源就绪），学过操作系统的同学对它一定已经很熟悉了。Java 提供了大量方法来支持阻塞，下面让我们逐一分析。

sleep() 方法：sleep() 允许 指定以毫秒为单位的一段时间作为参数，它使得线程在指定的时间内进入阻塞状态，不能得到CPU 时间，指定的时间一过，线程重新进入可执行状态。 典型地，sleep() 被用在等待某个资源就绪的情形：测试发现条件不满足后，让线程阻塞一段时间后重新测试，直到条件满足为止。

suspend() 和 resume() 方法：两个方法配套使用，suspend()使得线程进入阻塞状态，并且不会自动恢复，必须其对应的resume() 被调用，才能使得线程重新进入可执行状态。典型地，suspend() 和 resume() 被用在等待另一个线程产生的结果的情形：测试发现结果还没有产生后，让线程阻塞，另一个线程产生了结果后，调用 resume() 使其恢复。

yield() 方法：yield() 使得线程放弃当前分得的 CPU 时间，但是不使线程阻塞，即线程仍处于可执行状态，随时可能再次分得 CPU 时间。调用 yield() 的效果等价于调度程序认为该线程已执行了足够的时间从而转到另一个线程。

wait() 和 notify() 方法：两个方法配套使用，wait() 使得线程进入阻塞状态，它有两种形式，一种允许指定以毫秒为单位的一段时间作为参数，另一种没有参数，前者当对应的 notify() 被调用或者超出指定时间时线程重新进入可执行状态，后者则必须对应的 notify() 被调用。初看起来它们与 suspend() 和 resume() 方法对没有什么分别，但是事实上它们是截然不同的。区别的核心在于，前面叙述的所有方法，阻塞时都不会释放占用的锁（如果占用了的话），而这一对方法则相反。

上述的核心区别导致了一系列的细节上的区别。

首先，前面叙述的所有方法都隶属于 Thread 类，但是这一对却直接隶属于 Object 类，也就是说，所有对象都拥有这一对方法。初看起来这十分不可思议，但是实际上却是很自然的，因为这一对方法阻塞时要释放占用的锁，而锁是任何对象都具有的，调用任意对象的 wait() 方法导致线程阻塞，并且该对象上的锁被释放。而调用 任意对象的notify()方法则导致因调用该对象的 wait() 方法而阻塞的线程中随机选择的一个解除阻塞（但要等到获得锁后才真正可执行）。

其次，前面叙述的所有方法都可在任何位置调用，但是这一对方法却必须在 synchronized 方法或块中调用，理由也很简单，只有在synchronized 方法或块中当前线程才占有锁，才有锁可以释放。同样的道理，调用这一对方法的对象上的锁必须为当前线程所拥有，这样才有锁可以释放。因此，这一对方法调用必须放置在这样的 synchronized 方法或块中，该方法或块的上锁对象就是调用这一对方法的对象。若不满足这一条件，则程序虽然仍能编译，但在运行时会出现IllegalMonitorStateException 异常。

wait() 和 notify() 方法的上述特性决定了它们经常和synchronized 方法或块一起使用，将它们和操作系统的进程间通信机制作一个比较就会发现它们的相似性：synchronized方法或块提供了类似于操作系统原语的功能，它们的执行不会受到多线程机制的干扰，而这一对方法则相当于 block 和wakeup 原语（这一对方法均声明为 synchronized）。它们的结合使得我们可以实现操作系统上一系列精妙的进程间通信的算法（如信号量算法），并用于解决各种复杂的线程间通信问题。（此外，线程间通信的方式还有多个线程通过synchronized关键字这种方式来实现线程间的通信、while轮询、使用java.io.PipedInputStream 和 java.io.PipedOutputStream进行通信的管道通信）。

##### wait、notify、notifyAll

- 锁池：某个对象的锁已被线程A拥有，其他线程要执行该对象的 synchronized 方法获取锁时就会进入该对象的锁池，锁池中的线程回去竞争该对象的锁
- 等待池：某个线程调用了某个对象的 wait 方法，该线程就会释放该对象的锁，进入该对象的等待池，等待池中的线程不会去竞争该对象的锁
- 调用 notify 会随机唤醒等待池中的一个线程，唤醒后会进入到锁池
- 调用 notifyAll 会唤醒等待池中的所有线程，唤醒后会都进入到锁池

关于 wait() 和 notify() 方法最后再说明两点：

第一：调用 notify() 方法导致解除阻塞的线程是从调用该对象的 wait() 方法而阻塞的线程中随机选取的，我们无法预料哪一个线程将会被选择，所以编程时要特别小心，避免因这种不确定性而产生问题。

第二：除了 notify()，还有一个方法 notifyAll() 也可起到类似作用，唯一的区别在于，调用 notifyAll() 方法将把因调用该对象的 wait() 方法而阻塞的所有线程一次性全部解除阻塞。当然，只有获得锁的那一个线程才能进入可执行状态。

谈到阻塞，就不能不谈一谈死锁，略一分析就能发现，suspend() 方法和不指定超时期限的 wait() 方法的调用都可能产生死锁。遗憾的是，Java 并不在语言级别上支持死锁的避免，我们在编程中必须小心地避免死锁。

以上我们对 Java 中实现线程阻塞的各种方法作了一番分析，我们重点分析了 wait() 和 notify() 方法，因为它们的功能最强大，使用也最灵活，但是这也导致了它们的效率较低，较容易出错。实际使用中我们应该灵活使用各种方法，以便更好地达到我们的目的。


#### 8、线程的生命周期

线程状态流程图

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/native/process/java_thread_state.png)

- NEW：创建状态，线程创建之后，但是还未启动。
- RUNNABLE：运行状态，处于运行状态的线程，但有可能处于等待状态，例如等待CPU、IO等。
- WAITING：等待状态，一般是调用了wait()、join()、LockSupport.spark()等方法。
- TIMED_WAITING：超时等待状态，也就是带时间的等待状态。一般是调用了wait(time)、join(time)、LockSupport.sparkNanos()、LockSupport.sparkUnit()等方法。
- BLOCKED：阻塞状态，等待锁的释放，例如调用了synchronized增加了锁。
- TERMINATED：终止状态，一般是线程完成任务后退出或者异常终止。

NEW、WAITING、TIMED_WAITING都比较好理解，我们重点说一说RUNNABLE运行态和BLOCKED阻塞态。

线程进入RUNNABLE运行态一般分为五种情况：

- 线程调用sleep(time)后结束了休眠时间
- 线程调用的阻塞IO已经返回，阻塞方法执行完毕
- 线程成功的获取了资源锁
- 线程正在等待某个通知，成功的获得了其他线程发出的通知
- 线程处于挂起状态，然后调用了resume()恢复方法，解除了挂起。

线程进入BLOCKED阻塞态一般也分为五种情况：

- 线程调用sleep()方法主动放弃占有的资源
- 线程调用了阻塞式IO的方法，在该方法返回前，该线程被阻塞。
- 线程视图获得一个资源锁，但是该资源锁正被其他线程锁持有。
- 线程正在等待某个通知
- 线程调度器调用suspend()方法将该线程挂起

我们再来看看和线程状态相关的一些方法。

- sleep()方法让当前正在执行的线程在指定时间内暂停执行，正在执行的线程可以通过Thread.currentThread()方法获取。

- yield()方法放弃线程持有的CPU资源，将其让给其他任务去占用CPU执行时间。但放弃的时间不确定，有可能刚刚放弃，马上又获得CPU时间片。

- wait()方法是当前执行代码的线程进行等待，将当前线程放入预执行队列，并在wait()所在的代码处停止执行，直到接到通知或者被中断为止。该方法可以使得调用该方法的线程释放共享资源的锁， 然后从运行状态退出，进入等待队列，直到再次被唤醒。该方法只能在同步代码块里调用，否则会抛出IllegalMonitorStateException异常。wait(long millis)方法等待某一段时间内是否有线程对锁进行唤醒，如果超过了这个时间则自动唤醒。

- notify()方法用来通知那些可能等待该对象的对象锁的其他线程，该方法可以随机唤醒等待队列中等同一共享资源的一个线程，并使该线程退出等待队列，进入可运行状态。

- notifyAll()方法可以使所有正在等待队列中等待同一共享资源的全部线程从等待状态退出，进入可运行状态，一般会是优先级高的线程先执行，但是根据虚拟机的实现不同，也有可能是随机执行。

- join()方法可以让调用它的线程正常执行完成后，再去执行该线程后面的代码，它具有让线程排队的作用。


#### 9、[乐观锁与悲观锁](https://juejin.im/post/5b4977ae5188251b146b2fc8)。

- 悲观锁：线程一旦得到锁，其他线程就挂起等待，适用于写入操作频繁的场景；synchronized 就是悲观锁
- 乐观锁：假设没有冲突，不加锁，更新数据时判断该数据是否过期，过期的话则不进行数据更新，适用于读取操作频繁的场景
- 乐观锁 CAS：Compare And Swap，更新数据时先比较原值是否相等，不相等则表示数据过去，不进行数据更新
- 乐观锁实现：AtomicInteger、AtomicLong、AtomicBoolean

##### 悲观锁

总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁（共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程）。Java中synchronized和ReentrantLock等独占锁就是悲观锁思想的实现。

##### 乐观锁

总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号机制和CAS算法实现。乐观锁适用于多读的应用类型，这样可以提高吞吐量。在Java中java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式CAS实现的。

##### 使用场景

乐观锁适用于写比较少的情况下（多读场景），而一般多写的场景下用悲观锁就比较合适。

##### 乐观锁常见的两种实现方式

1、版本号机制

一般是在数据表中加上一个数据版本号version字段，表示数据被修改的次数，当数据被修改时，version值会加1。当线程A要更新数据值时，在读取数据的同时也会读取version值，在提交更新时，若刚才读取到的version值为当前数据库中的version值相等时才更新，否则重试更新操作，直到更新成功。

2、CAS算法

即compare and swap（比较与交换），是一种有名的无锁算法。CAS有3个操作数，内存值V，旧的预期值A，要修改的新值B。当且仅当预期值A和内存值V相同时，将内存值V修改为B，否则什么都不做。 一般情况下是一个自旋操作，即不断的重试。

##### 乐观锁的缺点

1、ABA 问题

如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，那我们就能说明它的值没有被其他线程修改过了吗？很明显是不能的，因为在这段时间它的值可能被改为其他值，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个问题被称为CAS操作的 "ABA"问题。

JDK 1.5 以后的 AtomicStampedReference 类一定程度上解决了这个问题，其中的 compareAndSet 方法就是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

2、自旋CAS（也就是不成功就一直循环执行直到成功）如果长时间不成功，会给CPU带来非常大的执行开销。

3、CAS 只对单个共享变量有效，当操作涉及跨多个共享变量时 CAS 无效。但是从 JDK 1.5开始，提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行 CAS 操作.所以我们可以使用锁或者利用AtomicReference类把多个共享变量合并成一个共享变量来操作。


#### 10、run()和start()方法区别？

1.start()方法来启动线程，真正实现了多线程运行，这时无需等待run方法体代码执行完毕而直接继续执行下面的代码：

通过调用Thread类的start()方法来启动一个线程，
这时此线程是处于就绪状态，
并没有运行。
然后通过此Thread类调用方法run()来完成其运行操作的，
这里方法run()称为线程体，
它包含了要执行的这个线程的内容，
Run方法运行结束，
此线程终止，
而CPU再运行其它线程，在Android中一般是主线程。
 
2.run()方法当作普通方法的方式调用，程序还是要顺序执行，还是要等待run方法体执行完毕后才可继续执行下面的代码：

而如果直接用Run方法，
这只是调用一个方法而已，
程序中依然只有主线程--这一个线程，
其程序执行路径还是只有一条，
这样就没有达到写线程的目的。


#### 11、多线程断点续传原理。

在本地下载过程中要使用数据库实时存储到底存储到文件的哪个位置了，这样点击开始继续传递时，才能通过HTTP的GET请求中的setRequestProperty("Range","bytes=startIndex-endIndex");方法可以告诉服务器，数据从哪里开始，到哪里结束。同时在本地的文件写入时，RandomAccessFile的seek()方法也支持在文件中的任意位置进行写入操作。同时通过广播或事件总线机制将子线程的进度告诉Activity的进度条。关于断线续传的HTTP状态码是206，即HttpStatus.SC_PARTIAL_CONTENT。


#### 12、怎么安全停止一个线程任务？原理是什么？线程池里有类似机制吗？

##### 终止线程

1、使用violate boolean变量退出标志，使线程正常退出，也就是当run方法完成后线程终止。（推荐）

2、使用interrupt()方法中断线程，但是线程不一定会终止。

3、使用stop方法强行终止线程。不安全主要是：thread.stop()调用之后，创建子线程的线程就会抛出ThreadDeatherror的错误，并且会释放子线程所持有的所有锁。

##### 终止线程池

ExecutorService线程池就提供了shutdown和shutdownNow这样的生命周期方法来关闭线程池自身以及它拥有的所有线程。

1、shutdown关闭线程池

线程池不会立刻退出，直到添加到线程池中的任务都已经处理完成，才会退出。 

2、shutdownNow关闭线程池并中断任务

终止等待执行的线程，并返回它们的列表。试图停止所有正在执行的线程，试图终止的方法是调用Thread.interrupt()，但是大家知道，如果线程中没有sleep 、wait、Condition、定时锁等应用, interrupt()方法是无法中断当前的线程的。所以，ShutdownNow()并不代表线程池就一定立即就能退出，它可能必须要等待所有正在执行的任务都执行完成了才能退出。


#### 13、堆内存，栈内存理解，栈如何转换成堆？

- 在函数中定义的一些基本类型的变量和对象的引用变量都是在函数的栈内存中分配。
- 堆内存用于存放由new创建的对象和数组。JVM里的“堆”（heap）特指用于存放Java对象的内存区域。所以根据这个定义，Java对象全部都在堆上。JVM的堆被同一个JVM实例中的所有Java线程共享。它通常由某种自动内存管理机制所管理，这种机制通常叫做“垃圾回收”（garbage collection，GC）。
- 堆主要用来存放对象的，栈主要是用来执行程序的。
- 实际上，栈中的变量指向堆内存中的变量，这就是 Java 中的指针!


#### 14、多线程间的特性

##### 多线程间的有序性、可见性和原子性是什么意思？


- 原子性：执行一个或者多个操作的时候，要么全部执行，要么都不执行，并且中间过程中不会被打断。Java中的原子性可以通过独占锁和CAS去保证
- 可见性：指多线程访问同一个变量的时候，一个线程修改了变量的值，其他线程能够立刻看得到修改的值。锁和volatile能够保证可见性
- 有序性：程序执行的顺序按照代码先后的顺序执行。锁和volatile能够保证有序性


##### happens-before原则有哪些？


Java内存模型具有一些先天的有序性，它通常叫做happens-before原则。如果两个操作的先后顺序不能通过happens-before原则推倒出来，那就不能保证它们的先后执行顺序，虚拟机就可以随意打乱执行指令。happens-before原则有（前四条规则比较重要）：


- 程序次序规则：单线程程序的执行结果得和看上去代码执行的结果要一致。
- 锁定规则：一个锁的lock操作一定发生在上一个unlock操作之后。
- volatile规则：对volatile变量的写操作一定先行于后面对这个变量的对操作。
- 传递规则：A发生在B前面，B发生在C前面，那么A一定发生在C前面。
- 线程启动规则：线程的start方法先行发生于线程中的每个动作。
- 线程中断规则：对线程的interrupt操作先行发生于中断线程的检测代码。
- 线程终结原则：线程中所有的操作都先行发生于线程的终止检测。
- 对象终止原则：一个对象的初始化先行发生于他的finalize()方法的执行。














