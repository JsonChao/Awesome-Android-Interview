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


### 二、Synchronized、volatile、ReentrantLock、Lock相关 （⭐⭐⭐）

#### 1、synchronized的原理？

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

    类别              synchronized	                                                Lock（底层实现主要是Volatile + CAS）
    存在层次：	Java的关键字，在jvm层面上	是一个类
    锁的释放	1、以获取锁的线程执行完同步代码，释放锁 2、线程执行发生异常，jvm会让线程释放锁	在finally中必须释放锁，不然容易造成线程死锁
       锁的获取	假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待	分情况而定，Lock有多个锁获取的方式，具体下面会说道，大致就是可以尝试获得锁，线程可以不用一直等待
    锁状态	无法判断	可以判断
    锁类型	可重入 不可中断 非公平	可重入 可判断 可公平（两者皆可）
    性能	少量同步	大量同步
    
Lock（ReentrantLock）的底层实现主要是Volatile + CAS（乐观锁），而Synchronized是一种悲观锁，比较耗性能。但是在JDK1.6以后对Synchronized的锁机制进行了优化，加入了偏向锁、轻量级锁、自旋锁、重量级锁，在并发量不大的情况下，性能可能优于Lock机制。所以建议一般请求并发量不大的情况下使用synchronized关键字。


#### 6、volatile原理。

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

如何保证可见性？🤔

被volatile修饰的变量在工作内存修改后会被强制写回主内存，其他线程在使用时也会强制从主内存刷新，这样就保证了一致性。

关于“保证被volatile修饰的变量对所有线程都是可见的”，有种常见的错误理解：

- 由于volatile修饰的变量在各个线程里都是一致的，所以基于volatile变量的运算在多线程并发的情况下是安全的。

这句话的前半部分是对的，后半部分却错了，因此它忘记考虑变量的操作是否具有原子性这一问题。

☝️举个栗子

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

为什么会这样？🤔

仔细看一下start++，它其实并非一个原子操作，简单来看，它有两步：

取出start的值，因为有volatile的修饰，这时候的值是正确的。
自增，但是自增的时候，别的线程可能已经把start加大了，这种情况下就有可能把较小的start写回主内存中。
所以volatile只能保证可见性，在不符合以下场景下我们依然需要通过加锁来保证原子性：

运算结果并不依赖变量当前的值，或者只有单一线程修改变量的值。（要么结果不依赖当前值，要么操作是原子性的，要么只要一个线程修改变量的值）
变量不需要与其他状态变量共同参与不变约束
比方说我们会在线程里加个boolean变量，来判断线程是否停止，这种情况就非常适合使用volatile。

我们再来理解第二句话。

禁止进行指令重排序
什么是指令重排序？🤔

- 指令重排序是值指令乱序执行，即在条件允许的情况下直接运行当前有能力立即执行的后续指令，避开为获取一条指令所需数据而造成的等待，通过乱序执行的技术提供执行效率。

- 指令重排序绘制被volatile修饰的变量的赋值操作前，添加一个内存屏障，指令重排序时不能把后面的指令重排序的内存屏障之前的位置。

关于指令重排序不是本篇文章重点讨论的内容，更多细节可以参考指令重排序。
 
#### 12、synchronized 和 volatile 关键字的作用；

答：

1）保证了不同线程对这个变量进行操作时的可见性即一个线程修改了某个变量的值，这新值对其他线程来是立即可见的。

2）禁止进行指令重排序。

volatile 本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需从主存中读取；synchronized则是锁定当前变量，只有当前线程可以访问该变量，其它线程被阻塞住。

1.volatile 仅能使用在变量级别；synchronized则可以使用在变量、方法、和类级别的

2.volatile仅能实现变量的修改可见性，并不能保证原子性；synchrnized 则可以保证变量的修改可见性和原子性

3.volatile 不会造成线程的阻塞；synchronized可能会造成线程的阻塞。

4.volatile 标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化

#### 13、ReentrantLock的内部实现。

#### 14、Java重入锁ReentrantLock和Condition。

#### 15、ReentrantLock 、synchronized和volatile比较？

synchronized是互斥同步的一种实现。

synchronized：当某个线程访问被synchronized标记的法或代码块时，这个线程便获得了该对象的锁，其他线暂时无法访问这个方法，只有等待这个方法执行完毕或代码块执行完毕，这个线程才会释放该对象的锁，其他线程才能执行这个方法代码块。

前面我们已经说了volatile关键字，这里我们举个例子来综合分析volatile与synchronized关键字的使用。

☝️举个栗子

    public class Singleton {
    
        //volatile保证了：1 instance在多线程并发的可见性 2 禁止instance在操作是的指令重排序
        private volatile static Singleton instance;
    
        private Singleton(){}
    
        public static Singleton getInstance() {
            //第一次判空，保证不必要的同步
            if (instance == null) {
                //synchronized对Singleton加全局所，保证每次只要一个线程创建实例
                synchronized (Singleton.class) {
                    //第二次判空时为了在null的情况下创建实例
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

- 对象锁：Java的所有对象都含有1个互斥锁，这个锁由JV自动获取和释放。线程进入synchronized方法的时候获该对象的锁，当然如果已经有线程获取了这个对象的锁那么当前线程会等待；synchronized方法正常返回或者抛异常而终止，JVM会自动释放对象锁。这里也体现了用synchronized来加锁的好处，方法抛异常的时候，锁仍然可以由JVM来自动释放。

- 类锁：对象锁是用来控制实例方法之间的同步，类锁是来控制静态方法（或静态变量互斥体）之间的同步。其类锁只是一个概念上的东西，并不是真实存在的，它只用来帮助我们理解锁定实例方法和静态方法的区别的。我们都知道，java类可能会有很多个对象，但是只有1个Class对象，也就说类的不同实例之间共享该类的Class对象。Class对象实也仅仅是1个java对象，只不过有点特殊而已。由于每个java对象都有个互斥锁，而类的静态方法是需要Class对象。所以所谓类锁，不过是Class对象的锁而已。获取类的Class对象好几种，最简 单的就是MyClass.class的方式。类锁和对象锁不是同一个东西，一个是类的Class对象的，一个是类的实例的锁。也就是说：一个线程访问静态sychronized的时候，允许另一个线程访问对象的实例synchronized方法。反过来也是成立的，为他们需要的锁是不同的。

### 三、其它 （⭐⭐⭐）

#### 1、多线程的使用场景？

使用多线程就一定效率高吗？有时候使用多线程并不是为了提高效率，而是使得CPU能同时处理多个事件。

- 为了不阻塞主线程,启动其他线程来做好事的事情,比APP中耗时操作都不在UI中做.

- 实现更快的应用程序,即主线程专门监听用户请求,子程用来处理用户请求,以获得大的吞吐量.感觉这种情况，多线程的效率未必高。这种情况下的多线程是为了不必等待，可以并行处理多条数据。比如JavaWeb的就是主线程专门监听用户的HTTP请求，然启动子线程去处理用户的HTTP请求。

- 某种虽然优先级很低的服务，但是却要不定时去做。比如Jvm的垃圾回收。

- 某种任务，虽然耗时，但是不耗CPU的操作时，开启个线程，效率会有显著提高。比如读取文件，然后处理。磁盘IO是个很耗费时间，但是不耗CPU计算的工作。所以可以一个线程读取数据，一个线程处理数据。肯定一个线程读取数据，然后处理效率高。因为两个线程的时候充分利用了CPU等待磁盘IO的空闲时。
    
#### 2、在Java中wait和seelp方法的不同；

答：最大的不同是在等待时wait 会释放锁，而sleep一直持有锁。wait 通常被用于线程间交互，sleep通常被用于暂停执行。    

#### 3、CopyOnWriteArrayList的了解。
    
#### 4、ConcurrentHashMap加锁机制是什么，详细说一下？

ConcurrentHashMap作为一种线程安全且高效的哈希表的决方案，尤其其中的"分段锁"的方案，相比HashTable的表锁在性能上的提升非常之大.

HashTable容器在竞争激烈的并发环境下表现出效率低下的原因，是因为所有访问HashTable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。

#### 5、单机上一个线程池正在处理服务，如果忽然断电了怎么办（正在处理和阻塞队列里的请求怎么处理）？

#### 6、介绍介绍CMS。

#### 7、如何控制某个方法允许并发访问线程的个数；

#### 8、多进程开发以及多进程应用场景；

#### 9、Java的线程模型；

#### 10、死锁的概念，怎么避免死锁

#### 11、线程死锁的4个条件？

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

在并发程序中，避免了逻辑中出现复数个线程互相持有对方线程所需要的独占锁的的情况，就可以避免死锁，如下所示：

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

#### 12、多线程断点续传原理

#### 13、CAS介绍

#### 14、如何保证多线程读写文件的安全？

#### 15、线程如何关闭，以及如何防止线程的内存泄漏

#### 16、为什么要有线程，而不是仅仅用进程？

#### 17、多个线程如何同时请求，返回的结果如何等待所有线程数据完成后合成一个数据？

#### 18、进程和线程的区别

简而言之,一个程序至少有一个进程,一个进程至少有一个线程。

线程的划分尺度小于进程，使得多线程程序的并发性高。

另外，进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。

线程在执行过程中与进程还是有区别的。每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。

从逻辑角度来看，多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。但操作系统并没有将多个线程看做多个独立的应用，来实现进程的调度和管理以及资源分配。这就是进程和线程的重要区别。

进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位.

线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位.线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源.

一个线程可以创建和撤销另一个线程;同一个进程中的多个线程之间可以并发执行.

进程和线程的主要差别在于它们是不同的操作系统资源管理方式。进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些。但对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能用进程。如果有兴趣深入的话，我建议你们看看《现代操作系统》或者《操作系统的设计与实现》。对就个问题说得比较清楚。

#### 19、run()和start()方法区别

#### 20、谈谈wait/notify关键字的理解

#### 21、什么导致线程阻塞？

线程的阻塞

为了解决对共享存储区的访问冲突，Java 引入了同步机制，现在让我们来考察多个线程对共享资源的访问，显然同步机制已经不够了，因为在任意时刻所要求的资源不一定已经准备好了被访问，反过来，同一时刻准备好了的资源也可能不止一个。为了解决这种情况下的访问控制问题，Java 引入了对阻塞机制的支持.

阻塞指的是暂停一个线程的执行以等待某个条件发生（如某资源就绪），学过操作系统的同学对它一定已经很熟悉了。Java 提供了大量方法来支持阻塞，下面让我们逐一分析。

sleep() 方法：sleep() 允许 指定以毫秒为单位的一段时间作为参数，它使得线程在指定的时间内进入阻塞状态，不能得到CPU 时间，指定的时间一过，线程重新进入可执行状态。 典型地，sleep() 被用在等待某个资源就绪的情形：测试发现条件不满足后，让线程阻塞一段时间后重新测试，直到条件满足为止。

suspend() 和 resume() 方法：两个方法配套使用，suspend()使得线程进入阻塞状态，并且不会自动恢复，必须其对应的resume() 被调用，才能使得线程重新进入可执行状态。典型地，suspend() 和 resume() 被用在等待另一个线程产生的结果的情形：测试发现结果还没有产生后，让线程阻塞，另一个线程产生了结果后，调用 resume() 使其恢复。

yield() 方法：yield() 使得线程放弃当前分得的 CPU 时间，但是不使线程阻塞，即线程仍处于可执行状态，随时可能再次分得 CPU 时间。调用 yield() 的效果等价于调度程序认为该线程已执行了足够的时间从而转到另一个线程.

wait() 和 notify() 方法：两个方法配套使用，wait() 使得线程进入阻塞状态，它有两种形式，一种允许 指定以毫秒为单位的一段时间作为参数，另一种没有参数，前者当对应的 notify() 被调用或者超出指定时间时线程重新进入可执行状态，后者则必须对应的 notify() 被调用.
初看起来它们与 suspend() 和 resume() 方法对没有什么分别，但是事实上它们是截然不同的。区别的核心在于，前面叙述的所有方法，阻塞时都不会释放占用的锁（如果占用了的话），而这一对方法则相反。

上述的核心区别导致了一系列的细节上的区别。

首先，前面叙述的所有方法都隶属于 Thread 类，但是这一对却直接隶属于 Object 类，也就是说，所有对象都拥有这一对方法。初看起来这十分不可思议，但是实际上却是很自然的，因为这一对方法阻塞时要释放占用的锁，而锁是任何对象都具有的，调用任意对象的 wait() 方法导致线程阻塞，并且该对象上的锁被释放。而调用 任意对象的notify()方法则导致因调用该对象的 wait() 方法而阻塞的线程中随机选择的一个解除阻塞（但要等到获得锁后才真正可执行）。

其次，前面叙述的所有方法都可在任何位置调用，但是这一对方法却必须在 synchronized 方法或块中调用，理由也很简单，只有在synchronized 方法或块中当前线程才占有锁，才有锁可以释放。同样的道理，调用这一对方法的对象上的锁必须为当前线程所拥有，这样才有锁可以释放。因此，这一对方法调用必须放置在这样的 synchronized 方法或块中，该方法或块的上锁对象就是调用这一对方法的对象。若不满足这一条件，则程序虽然仍能编译，但在运行时会出现IllegalMonitorStateException 异常。

wait() 和 notify() 方法的上述特性决定了它们经常和synchronized 方法或块一起使用，将它们和操作系统的进程间通信机制作一个比较就会发现它们的相似性：synchronized方法或块提供了类似于操作系统原语的功能，它们的执行不会受到多线程机制的干扰，而这一对方法则相当于 block 和wakeup 原语（这一对方法均声明为 synchronized）。它们的结合使得我们可以实现操作系统上一系列精妙的进程间通信的算法（如信号量算法），并用于解决各种复杂的线程间通信问题。

关于 wait() 和 notify() 方法最后再说明两点：

第一：调用 notify() 方法导致解除阻塞的线程是从因调用该对象的 wait() 方法而阻塞的线程中随机选取的，我们无法预料哪一个线程将会被选择，所以编程时要特别小心，避免因这种不确定性而产生问题。

第二：除了 notify()，还有一个方法 notifyAll() 也可起到类似作用，唯一的区别在于，调用 notifyAll() 方法将把因调用该对象的 wait() 方法而阻塞的所有线程一次性全部解除阻塞。当然，只有获得锁的那一个线程才能进入可执行状态。

谈到阻塞，就不能不谈一谈死锁，略一分析就能发现，suspend() 方法和不指定超时期限的 wait() 方法的调用都可能产生死锁。遗憾的是，Java 并不在语言级别上支持死锁的避免，我们在编程中必须小心地避免死锁。

以上我们对 Java 中实现线程阻塞的各种方法作了一番分析，我们重点分析了 wait() 和 notify() 方法，因为它们的功能最强大，使用也最灵活，但是这也导致了它们的效率较低，较容易出错。实际使用中我们应该灵活使用各种方法，以便更好地达到我们的目的。

#### 22、线程如何关闭？

#### 23、数据一致性如何保证？

#### 24、两个进程同时要求写或者读，能不能实现？如何防止进程的同步？

#### 25、谈谈对多线程的理解并举例说明

#### 26、线程的状态和优先级。

#### 27、启动线程和安全的终止线程。（interrupt）

#### 28、ThreadLocal的使用

#### 29、Java中的并发工具（CountDownLatch，CyclicBarrier等）

#### 30、进程线程在操作系统中的实现

#### 31、线程的生命周期

线程状态流程图图

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/native/process/java_thread_state.png)

- NEW：创建状态，线程创建之后，但是还未启动。
- RUNNABLE：运行状态，处于运行状态的线程，但有可能处于等待状态，例如等待CPU、IO等。
- WAITING：等待状态，一般是调用了wait()、join()、LockSupport.spark()等方法。
- TIMED_WAITING：超时等待状态，也就是带时间的等待状态。一般是调用了wait(time)、join(time)、LockSupport.sparkNanos()、LockSupport.sparkUnit()等方法。
- BLOCKED：阻塞状态，等待锁的释放，例如调用了synchronized增加了锁。
- TERMINATED：终止状态，一般是线程完成任务后退出或者异常终止。

NEW、WAITING、TIMED_WAITING都比较好理解，我们重点说一说RUNNABLE运行态和BLOCKED阻塞态。

线程进入RUNNABLE运行态一般分为五种情况：

- 线程调用sleep(time)后查出了休眠时间
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

- wait()方法是当前执行代码的线程进行等待，将当前线程放入预执行队列，并在wait()所在的代码处停止执行，知道接到通知或者被中断为止。该方法可以使得调用该方法的线程释放共享资源的锁， 然后从运行状态退出，进入等待队列，直到再次被唤醒。该方法只能在同步代码块里调用，否则会抛出IllegalMonitorStateException异常。wait(long millis)方法等待某一段时间内是否有线程对锁进行唤醒，如果超过了这个时间则自动唤醒。

- notify()方法用来通知那些可能等待该对象的对象锁的其他线程，该方法可以随机唤醒等待队列中等同一共享资源的一个线程，并使该线程退出等待队列，进入可运行状态。

- notifyAll()方法可以是所有正在等待队列中等待同一共享资源的全部线程从等待状态退出，进入可运行状态，一般会是优先级高的线程先执行，但是根据虚拟机的实现不同，也有可能是随机执行。

- join()方法可以让调用它的线程正常执行完成后，再去执行该线程后面的代码，它具有让线程排队的作用。

#### 32、乐观锁与悲观锁

#### 33、双线程通过线程同步的方式打印12121212.......

#### 34、java线程，场景实现，多个线程如何同时请求，返回的结果如何等待所有线程数据完成后合成一个数据

#### 35、服务器只提供数据接收接口，在多线程或多进程条件下，如何保证数据的有序到达？

