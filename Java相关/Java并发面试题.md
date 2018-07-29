## Java并发面试题

**1.什么是线程池，如何使用?**

    答：线程池就是事先将多个线程对象放到一个容器中，当使用的时候就不用new 线程而是直接去池中拿线程即可，节
    省了开辟子线程的时间，提高的代码执行效率。
    
**2.在Java中wait和seelp方法的不同；**

    答：最大的不同是在等待时wait 会释放锁，而sleep 一直持有锁。wait 通常被用于线程间交互，sleep 通常被用于暂停执行。
    
**3.synchronized 和volatile 关键字的作用；**

    答：1）保证了不同线程对这个变量进行操作时的可见性，即一个线程修改了某个变量的值，这新值对其他线程来说是立即可见的。2）禁止进行指令重排序。
    volatile 本质是在告诉jvm 当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；synchronized 则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
    1.volatile 仅能使用在变量级别；synchronized 则可以使用在变量、方法、和类级别的
    2.volatile 仅能实现变量的修改可见性，并不能保证原子性；synchronized 则可以保证变量的修改可见性和原子性
    3.volatile 不会造成线程的阻塞；synchronized 可能会造成线程的阻塞。
    4.volatile 标记的变量不会被编译器优化；synchronized 标记的变量可以被编译器优化
    
**4. concurrentHashmap原理，原子类。**

    ConcurrentHashMap作为一种线程安全且高效的哈希表的解决方案，尤其其中的"分段锁"的方案，相比HashTable的全表锁在性能上的提升非常之大.

**5.volatile原理**

    在《Java并发编程：核心理论》一文中，我们已经提到过可见性、有序性及原子性问题，通常情况下我们可以通过Synchronized关键字来解决这些个问题，不过如果对Synchronized原理有了解的话，应该知道Synchronized是一个比较重量级的操作，对系统的性能有比较大的影响，所以，如果有其他解决方案，我们通常都避免使用Synchronized来解决问题。
    
    
    而volatile关键字就是Java中提供的另一种解决可见性和有序性问题的方案。对于原子性，需要强调一点，也是大家容易误解的一点：对volatile变量的单次读/写操作可以保证原子性的，如long和double类型变量，但是并不能保证i++这种操作的原子性，因为本质上i++是读、写两次操作。

**6.多线程的使用场景**

    使用多线程就一定效率高吗？ 有时候使用多线程并不是为了提高效率，而是使得CPU能够同时处理多个事件。
    
    
    1).为了不阻塞主线程,启动其他线程来做好事的事情,比如APP中耗时操作都不在UI中做.
    
    
    2).实现更快的应用程序,即主线程专门监听用户请求,子线程用来处理用户请求,以获得大的吞吐量.感觉这种情况下，多线程的效率未必高。 这种情况下的多线程是为了不必等待， 可以并行处理多条数据。
    
    比如JavaWeb的就是主线程专门监听用户的HTTP请求，然后启动子线程去处理用户的HTTP请求。
    
    
    3).某种虽然优先级很低的服务，但是却要不定时去做。
    
    比如Jvm的垃圾回收。
    
    
    4.)某种任务，虽然耗时，但是不耗CPU的操作时，开启多个线程，效率会有显著提高。
    
    比如读取文件，然后处理。 磁盘IO是个很耗费时间，但是不耗CPU计算的工作。 所以可以一个线程读取数据，一个线程处理数据。肯定比一个线程读取数据，然后处理效率高。 因为两个线程的时候充分利用了CPU等待磁盘IO的空闲时间。
    

**7.ConcurrentHashMap加锁机制是什么，详细说一下？**

HashTable容器在竞争激烈的并发环境下表现出效率低下的原因，是因为所有访问HashTable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。


**8.Synchronized优化后的锁机制简单介绍一下，包括自旋锁、偏向锁、轻量级锁、重量级锁？**

自旋锁：

线程自旋说白了就是让cpu在做无用功，比如：可以执行几次for循环，可以执行几条空的汇编指令，目的是占着CPU不放，等待获取锁的机会。如果旋的时间过长会影响整体性能，时间过短又达不到延迟阻塞的目的。

偏向锁

偏向锁就是一旦线程第一次获得了监视对象，之后让监视对象“偏向”这个线程，之后的多次调用则可以避免CAS操作，

说白了就是置个变量，如果发现为true则无需再走各种加锁/解锁流程。

轻量级锁：

轻量级锁是由偏向所升级来的，偏向锁运行在一个线程进入同步块的情况下，当第二个线程加入锁争用的时候，偏向锁就会升级为轻量级锁；

重量级锁

重量锁在JVM中又叫对象监视器（Monitor），它很像C中的Mutex，除了具备Mutex(0|1)互斥的功能，它还负责实现了Semaphore(信号量)的功能，也就是说它至少包含一个竞争锁的队列，和一个信号阻塞队列（wait队列），前者负责做互斥，后一个用于做线程同步。

偏向锁、轻量级锁、重量级锁的对比：

Redis和Memcache区别对比？如何选择这两个技术？

区别：

1） Redis和Memcache都是将数据存放在内存中，都是内存数据库。不过memcache还可用于缓存其他东西，例如图片、视频等等。

2）Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，hash等数据结构的存储。

3）虚拟内存--Redis当物理内存用完时，可以将一些很久没用到的value 交换到磁盘

4）过期策略--memcache在set时就指定，例如set key1 0 0 8,即永不过期。Redis可以通过例如expire 设定，例如expire name 10

5）分布式--设定memcache集群，利用magent做一主多从;redis可以做一主多从。都可以一主一从

6）存储数据安全--memcache挂掉后，数据没了；redis可以定期保存到磁盘（持久化）

7）灾难恢复--memcache挂掉后，数据不可恢复; redis数据丢失后可以通过aof恢复

8）Redis支持数据的备份，即master-slave模式的数据备份。

选型：

若是简单的存取key-value这样的数据用memcache好一些

若是要支持数据持久化，多数据类型(如集合、散列之类的)，用列表类型做队列之类的高级应用，就用redis

Redis的持久化机制是什么？各自的优缺点？

redis提供两种持久化机制RDB和AOF机制。

1）RDB持久化方式：

是指用数据集快照的方式记录redis数据库的所有键值对。

优点：

　　1.只有一个文件dump.rdb，方便持久化。

　　2.容灾性好，一个文件可以保存到安全的磁盘。

　　3.性能最大化，fork子进程来完成写操作，让主进程继续处理命令，所以是IO最大化。

　　4.相对于数据集大时，比AOF的启动效率更高。

缺点：

　　1.数据安全性低。

2）AOF持久化方式：

是指所有的命令行记录以redis命令请求协议的格式保存为aof文件。

优点：

　　1.数据安全，aof持久化可以配置appendfsync属性，有always，每进行一次命令操作就记录到aof文件中一次。

　　2.通过append模式写文件，即使中途服务器宕机，可以通过redis-check-aof工具解决数据一致性问题。

　　3.AOF机制的rewrite模式。

缺点：

　　1.文件会比RDB形式的文件大。

　　2.数据集大的时候，比rdb启动效率低。
　　
**9.Java中的线程池共有几种？**

Java四种线程池

第一种：newCachedThreadPool

　　创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。

第二种：newFixedThreadPool

　　创建一个指定工作线程数量的线程池

第三种：newScheduledThreadPool

创建一个线程池，它可安排在给定延迟后运行命令或者定期地执行。

第四种：newSingleThreadExecutor

　　创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。

**10.volatile和synchronized区别**

volatile和synchronized简介：

在Java中,为了保证多线程读写数据时保证数据的一致性,可以采用两种方式：

　 1）使用synchronized关键字

　 2）使用volatile关键字：用一句话概括volatile,它能够使变量在值发生改变时能尽快地让其他线程知道。

两者的区别：

1）volatile本质是在告诉jvm当前变量在寄存器中的值是不确定的,需要从主存中读取,synchronized则是锁定当前变量,只有当前线程可以访问该变量,其他线程被阻塞住.

2）volatile仅能使用在变量级别,synchronized则可以使用在变量,方法.

3）volatile仅能实现变量的修改可见性,而synchronized则可以保证变量的修改可见性和原子性.

4）volatile不会造成线程的阻塞,而synchronized可能会造成线程的阻塞.

**11.单机上一个线程池正在处理服务，如果忽然断电了怎么办（正在处理和阻塞队列里的请求怎么处理）？**

**12.为什么要使用线程池？**

**13.线程池有什么作用？**

**14.说说几种常见的线程池及使用场景。**

**15.线程池都有哪几种工作队列？**

**16.怎么理解无界队列和有界队列？**

**17.线程池中的几种重要的参数及流程说明。**

**18.介绍介绍CMS。**

**19.如何控制某个方法允许并发访问线程的个数；**

**20.多进程开发以及多进程应用场景；**

**21.Synchronize关键字后面跟类或者对象有什么不同。**

**22.Java的线程模型；**

**23.线程死锁的4个条件？**

**24.多线程断点续传原理**

**25.死锁的概念，怎么避免死锁**


**26.ConCurrentHashMap实现**

**27.CAS介绍**

**28.开启线程的三种方式,run()和start()方法区别**

**29.ReentrantLock (ReentrantLock的内部实现)**


**30.方法的多线程访问和作用，同一个类里面两个synchronized方法，两个线程同时访问的问题**

**31.wait/notify**

**32.如何保证多线程读写文件的安全？**


**33.线程如何关闭，以及如何防止线程的内存泄漏**

**34.多线程：怎么用、有什么问题要注意（关于AsyncTask缺陷引发的思考）；Android线程有没有上限，然后提到线程池的上限**

**35.为什么要有线程，而不是仅仅用进程？**


**36.多个线程如何同时请求，返回的结果如何等待所有线程数据完成后合成一个数据？**

**37.Sychronized参数是实例对象和class对象时候的区别（对象锁 类锁），二者区别，使用类锁的具体作用**


**38.进程和线程的区别**

简而言之,一个程序至少有一个进程,一个进程至少有一个线程。

线程的划分尺度小于进程，使得多线程程序的并发性高。

另外，进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率。

线程在执行过程中与进程还是有区别的。每个独立的线程有一个程序运行的入口、顺序执行序列和程序的出口。但是线程不能够独立执行，必须依存在应用程序中，由应用程序提供多个线程执行控制。

从逻辑角度来看，多线程的意义在于一个应用程序中，有多个执行部分可以同时执行。但操作系统并没有将多个线程看做多个独立的应用，来实现进程的调度和管理以及资源分配。这就是进程和线程的重要区别。

进程是具有一定独立功能的程序关于某个数据集合上的一次运行活动,进程是系统进行资源分配和调度的一个独立单位.

线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位.线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源.

一个线程可以创建和撤销另一个线程;同一个进程中的多个线程之间可以并发执行.

进程和线程的主要差别在于它们是不同的操作系统资源管理方式。进程有独立的地址空间，一个进程崩溃后，在保护模式下不会对其它进程产生影响，而线程只是一个进程中的不同执行路径。线程有自己的堆栈和局部变量，但线程之间没有单独的地址空间，一个线程死掉就等于整个进程死掉，所以多进程的程序要比多线程的程序健壮，但在进程切换时，耗费资源较大，效率要差一些。但对于一些要求同时进行并且又要共享某些变量的并发操作，只能用线程，不能用进程。如果有兴趣深入的话，我建议你们看看《现代操作系统》或者《操作系统的设计与实现》。对就个问题说得比较清楚。


**39.开启线程的三种方式？**

**40.为什么要有线程，而不是仅仅用进程？**

**41.run()和start()方法区别**

**42.如何控制某个方法允许并发访问线程的个数？**

**43.在Java中wait和seelp方法的不同；**

**44.谈谈wait/notify关键字的理解**

**45.什么导致线程阻塞？**

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

**46.线程如何关闭？**

**47.讲一下java中的同步的方法**

**48.数据一致性如何保证？**

**49.如何保证线程安全？**

**50.如何实现线程同步？**

http://www.itzhai.com/java-based-notebook-thread-synchronization-problem-solving-synchronization-problems-synchronized-block-synchronized-methods.html#read-more

http://www.juwends.com/tech/android/android-inter-thread-comm.html

单例

    public class Singleton{
    private volatile static Singleton mSingleton;
    private Singleton(){
    }
    public static Singleton getInstance(){
      if(mSingleton == null){\\A
        synchronized(Singleton.class){\\C
         if(mSingleton == null)
          mSingleton = new Singleton();\\B
          }
        }
        return mSingleton;
      }
    }
    
volatile 的意义？

防止CPU指令重排序

**51.两个进程同时要求写或者读，能不能实现？如何防止进程的同步？**

**52.线程间操作List**

**53.Synchronized用法**

**54.synchronize的原理**

**55.谈谈对Synchronized关键字，类锁，方法锁，重入锁的理解**

**56.static synchronized 方法的多线程访问和作用**

**57.同一个类里面两个synchronized方法，两个线程同时访问的问题**

**58.volatile的原理**

volatile也是互斥同步的一种实现，不过它非常的轻量级。

volatile有两条关键的语义：

保证被volatile修饰的变量对所有线程都是可见的
禁止进行指令重排序
要理解volatile关键字，我们得先从Java的线程模型开始说起。如图所示：

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/native/process/java_memory_model.png)

Java内存模型规定了所有字段（这些字段包括实例字段、静态字段等，不包括局部变量、方法参数等，因为这些是线程私有的，并不存在竞争）都存在主内存中，每个线程会 有自己的工作内存，工作内存里保存了线程所使用到的变量在主内存里的副本拷贝，线程对变量的操作只能在工作内存里进行，而不能直接读写主内存，当然不同内存之间也 无法直接访问对方的工作内存，也就是说主内存时线程传值的媒介。

我们来理解第一句话：

    保证被volatile修饰的变量对所有线程都是可见的

如何保证可见性？🤔

被volatile修饰的变量在工作内存修改后会被强制写回主内存，其他线程在使用时也会强制从主内存刷新，这样就保证了一致性。

关于“保证被volatile修饰的变量对所有线程都是可见的”，有种常见的错误理解：

    错误理解：由于volatile修饰的变量在各个线程里都是一致的，所以基于volatile变量的运算在多线程并发的情况下是安全的。

这句话的前半部分是对的，后半部分却错了，因此它忘记考虑变量的操作是否具有原子性这一问题。

☝️举个栗子

    private volatile int start = 0;

    private void volatileKeyword() {

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

    指令重排序是值指令乱序执行，即在条件允许的情况下，直接运行当前有能力立即执行的后续指令，避开为获取下一条指令所需数据而造成的等待，通过乱序执行的技术，提供执行效率。

指令重排序绘制被volatile修饰的变量的赋值操作前，添加一个内存屏障，指令重排序时不能把后面的指令重排序的内存屏障之前的位置。

关于指令重排序不是本篇文章重点讨论的内容，更多细节可以参考指令重排序。

如何防止反射、序列化攻击单例？

枚举单例

    public enum Singleton {
        INSTANCE {
    
            @Override
            protected void read() {
                System.out.println("read");
            }
    
            @Override
            protected void write() {
                System.out.println("write");
            }
    
        };
        protected abstract void read();
        protected abstract void write();
    }
class文件：

    public abstract class Singleton extends Enum
    {
    
        private Singleton(String s, int i)
        {
            super(s, i);
        }
    
        protected abstract void read();
    
        protected abstract void write();
    
        public static Singleton[] values()
        {
            Singleton asingleton[];
            int i;
            Singleton asingleton1[];
            System.arraycopy(asingleton = ENUM$VALUES, 0, asingleton1 = new Singleton[i = asingleton.length], 0, i);
            return asingleton1;
        }
    
        public static Singleton valueOf(String s)
        {
            return (Singleton)Enum.valueOf(singleton/Singleton, s);
        }
    
        Singleton(String s, int i, Singleton singleton)
        {
            this(s, i);
        }
    
        public static final Singleton INSTANCE;
        private static final Singleton ENUM$VALUES[];
    
        static 
        {
            INSTANCE = new Singleton("INSTANCE", 0) {
    
                protected void read()
                {
                    System.out.println("read");
                }
    
                protected void write()
                {
                    System.out.println("write");
                }
    
            };
            ENUM$VALUES = (new Singleton[] {
                INSTANCE
            });
        }
    }
类的修饰abstract，所以没法实例化，反射也无能为力。
关于线程安全的保证，其实是通过类加载机制来保证的，我们看看INSTANCE的实例化时机，是在static块中，JVM加载类的过程显然是线程安全的。
对于防止反序列化生成新实例的问题还不是很明白，一般的方法我们会在该类中添加上如下方法，不过枚举中也没有显示的写明该方法。
    //readResolve to prevent another instance of Singleton
    private Object readResolve(){
        return INSTANCE;
    }

**59.谈谈volatile关键字的用法**

**60.谈谈volatile关键字的作用**



**61.synchronized 和volatile 关键字的区别**

**62.synchronized与Lock的区别**

**63.ReentrantLock 、synchronized和volatile比较**

synchronized是互斥同步的一种实现。

    synchronized：当某个线程访问被synchronized标记的方法或代码块时，这个线程便获得了该对象的锁，其他线程暂时无法访问这个方法，只有等待这个方法执行完毕或者代码块执行完毕，这个 线程才会释放该对象的锁，其他线程才能执行这个方法或代码块。

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

    对象锁：Java的所有对象都含有1个互斥锁，这个锁由JVM自动获取和释放。线程进入synchronized方法的时候获取该对象的锁，当然如果已经有线程获取了这个对象的锁，那么当前线 程会等待；synchronized方法正常返回或者抛异常而终止，JVM会自动释放对象锁。这里也体现了用synchronized来加锁的好处，方法抛异常的时候，锁仍然可以由JVM来自动释放。

    类锁：对象锁是用来控制实例方法之间的同步，类锁是用来控制静态方法（或静态变量互斥体）之间的同步。其实类锁只是一个概念上的东西，并不是真实存在的，它只是用来帮助我们理 解锁定实例方法和静态方法的区别的。我们都知道，java类可能会有很多个对象，但是只有1个Class对象，也就是说类的不同实例之间共享该类的Class对象。Class对象其实也仅仅是1个 java对象，只不过有点特殊而已。由于每个java对象都有1个互斥锁，而类的静态方法是需要Class对象。所以所谓的类锁，不过是Class对象的锁而已。获取类的Class对象有好几种，最简 单的就是MyClass.class的方式。 类锁和对象锁不是同一个东西，一个是类的Class对象的锁，一个是类的实例的锁。也就是说：一个线程访问静态synchronized的时候，允许另一个线程访 问对象的实例synchronized方法。反过来也是成立的，因为他们需要的锁是不同的。

**64.ReentrantLock的内部实现**

**65.lock原理**

**66.死锁的四个必要条件？**

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

互斥条件：一个资源每次只能被一个线程使用。
请求与保持条件：一个线程因请求资源而阻塞时，对已获得的资源保持不放。
不剥夺条件：线程已获得的资源，在未使用完之前，不能强行剥夺。
循环等待条件：若干线程之间形成一种头尾相接的循环等待资源关系。
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

**67.对象锁和类锁是否会互相影响？**

**68.Java的并发、多线程、线程模型**

**69.谈谈对多线程的理解**

**70.多线程有什么要注意的问题？**

**71.谈谈你对并发编程的理解并举例说明**

**72.谈谈你对多线程同步机制的理解？**

**73.如何保证多线程读写文件的安全？**

**74.线程状态：**
https://my.oschina.net/mingdongcheng/blog/139263

**75.锁：**
https://blog.csdn.net/vking_wang/article/details/9952063
http://wiki.jikexueyuan.com/project/java-concurrent/locks-in-java.html

**76.并发编程：**

并发包
http://www.blogjava.net/xylz/archive/2010/07/08/325587.html

锁
http://blog.csdn.net/vking_wang/article/details/9952063
http://wiki.jikexueyuan.com/project/java-concurrent/locks-in-java.html
https://blog.csdn.net/javazejian/article/details/72828483
https://www.jianshu.com/p/12192b13990f

并发编程
http://www.cnblogs.com/dolphin0520/p/3920357.html
http://blog.51cto.com/lavasoft/27069
https://www.jianshu.com/p/053943a425c3#
http://www.cnblogs.com/chenssy/p/4701027.html
http://www.infoq.com/cn/articles/ConcurrentHashMap#



**77.synchrolzie关键字和Lock的区别你知道吗？为什么Lock的性能好一些？**


**78.Java线程安全涉及的概念和分类。**

http://www.iteye.com/topic/806990


**79.线程的状态和优先级。**

**80.启动线程和安全的终止线程。（interrupt）**

**81.ThreadLocal的使用**

**82.Java中的锁（偏向锁，轻量锁，重量级锁）**

**83.Java重入锁ReentrantLock和Condition。**

**84.Synchronized和锁的等级（方法锁、对象锁、类锁）。**

**85.Synchronized的wait（sleep的区别）和notify运行过程。**

**86.Java中的并发工具（CountDownLatch，CyclicBarrier等）**


**87.CopyOnWriteArrayList的了解。**

**88.TreeMap、HashMap、LindedHashMap、ArrayMap的区别。**

**89.可重入锁的实现，公平锁非公平锁都是什么定义？**

**90.进程线程在操作系统中的实现**

**91.线程的生命周期**

线程状态流程图图

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/native/process/java_thread_state.png)

NEW：创建状态，线程创建之后，但是还未启动。
RUNNABLE：运行状态，处于运行状态的线程，但有可能处于等待状态，例如等待CPU、IO等。
WAITING：等待状态，一般是调用了wait()、join()、LockSupport.spark()等方法。
TIMED_WAITING：超时等待状态，也就是带时间的等待状态。一般是调用了wait(time)、join(time)、LockSupport.sparkNanos()、LockSupport.sparkUnit()等方法。
BLOCKED：阻塞状态，等待锁的释放，例如调用了synchronized增加了锁。
TERMINATED：终止状态，一般是线程完成任务后退出或者异常终止。
NEW、WAITING、TIMED_WAITING都比较好理解，我们重点说一说RUNNABLE运行态和BLOCKED阻塞态。

线程进入RUNNABLE运行态一般分为五种情况：

线程调用sleep(time)后查出了休眠时间
线程调用的阻塞IO已经返回，阻塞方法执行完毕
线程成功的获取了资源锁
线程正在等待某个通知，成功的获得了其他线程发出的通知
线程处于挂起状态，然后调用了resume()恢复方法，解除了挂起。
线程进入BLOCKED阻塞态一般也分为五种情况：

线程调用sleep()方法主动放弃占有的资源
线程调用了阻塞式IO的方法，在该方法返回前，该线程被阻塞。
线程视图获得一个资源锁，但是该资源锁正被其他线程锁持有。
线程正在等待某个通知
线程调度器调用suspend()方法将该线程挂起
我们再来看看和线程状态相关的一些方法。

sleep()方法让当前正在执行的线程在指定时间内暂停执行，正在执行的线程可以通过Thread.currentThread()方法获取。
yield()方法放弃线程持有的CPU资源，将其让给其他任务去占用CPU执行时间。但放弃的时间不确定，有可能刚刚放弃，马上又获得CPU时间片。
wait()方法是当前执行代码的线程进行等待，将当前线程放入预执行队列，并在wait()所在的代码处停止执行，知道接到通知或者被中断为止。该方法可以使得调用该方法的线程释放共享资源的锁， 然后从运行状态退出，进入等待队列，直到再次被唤醒。该方法只能在同步代码块里调用，否则会抛出IllegalMonitorStateException异常。
wait(long millis)方法等待某一段时间内是否有线程对锁进行唤醒，如果超过了这个时间则自动唤醒。
notify()方法用来通知那些可能等待该对象的对象锁的其他线程，该方法可以随机唤醒等待队列中等同一共享资源的一个线程，并使该线程退出等待队列，进入可运行状态。
notifyAll()方法可以是所有正在等待队列中等待同一共享资源的全部线程从等待状态退出，进入可运行状态，一般会是优先级高的线程先执行，但是根据虚拟机的实现不同，也有可能是随机执行。
join()方法可以让调用它的线程正常执行完成后，再去执行该线程后面的代码，它具有让线程排队的作用。


**92.多线程中的安全队列一般通过什么实现？线程池原理？**

线程
https://blog.csdn.net/u011240877/article/details/57202704

线程状态
https://my.oschina.net/mingdongcheng/blog/139263

线程池
https://blog.csdn.net/u011240877/article/details/58756137
https://blog.csdn.net/u011240877/article/details/73440993 
https://itimetraveler.github.io/2018/02/13/%E3%80%90Java%E3%80%91%E7%BA%BF%E7%A8%8B%E6%B1%A0ThreadPoolExecutor%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/
