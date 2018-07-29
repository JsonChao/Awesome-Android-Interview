## Java基础面试题
    
**1.说说你对Java反射的理解；**

    答：Java 中的反射首先是能够获取到Java 中要反射类的字节码， 获取字节码有三种方法，
    1.Class.forName(className) 2.类名.class 3.this.getClass()。然后将字节码中的方法，变量，构造函数等映射成相应的Method、Filed、Constructor 等类，这些类提供了丰富的方法可以被我们所使用。
    
**2.谈谈NIO的理解**

    
**3.你所知道的设计模式有哪些。**

    答：Java 中一般认为有23 种设计模式，我们不需要所有的都会，但是其中常用的几种设计模式应该去掌握。下面列出了所有的设计模式。需要掌握的设计模式我单独列出来了，当然能掌握的越多越好。
    总体来说设计模式分为三大类：
    创建型模式，共五种：工厂方法模式、抽象工厂模式、单例模式、建造者模式、原型模式。
    结构型模式，共七种：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。
    行为型模式，共十一种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。
    
**4.类的加载过程，Person person = new Person();为例进行说明。**

    1).因为new用到了Person.class，所以会先找到Person.class文件，并加载到内存中;

    2).执行该类中的static代码块，如果有的话，给Person.class类进行初始化;
    
    3).在堆内存中开辟空间分配内存地址;
    
    4).在堆内存中建立对象的特有属性，并进行默认初始化;
    
    5).对属性进行显示初始化;
    
    6).对对象进行构造代码块初始化;
    
    7).对对象进行与之对应的构造函数进行初始化;
    
    8).将内存地址付给栈内存中的p变量
    


**5.集合框架，list，map，set都有哪些具体的实现类，区别都是什么?**

Java集合里使用接口来定义功能，是一套完善的继承体系。Iterator是所有集合的总接口，其他所有接口都继承于它，该接口定义了集合的 遍历操作，Collection接口继承于Iterator，是集合的次级接口（Map独立存在，除外），定义了集合的一些通用操作。

Java集合的类结构图如下所示：

![image](https://github.com/guoxiaoxing/data-structure-and-algorithm/raw/master/art/java_collection_structure.png)

List：有序、可重复；索引查询速度快；插入、删除伴随数据移动，速度慢；
Set：无序，不可重复；
Map：键值对，键唯一，值多个；

    1.List,Set都是继承自Collection接口，Map则不是;
    
    
    2.List特点：元素有放入顺序，元素可重复;
    
    Set特点：元素无放入顺序，元素不可重复，重复元素会覆盖掉，（注意：元素虽然无放入顺序，但是元素在set中的位置是有该元素的HashCode决定的，其位置其实是固定的，加入Set 的Object必须定义equals()方法;
    
    另外list支持for循环，也就是通过下标来遍历，也可以用迭代器，但是set只能用迭代，因为他无序，无法用下标来取得想要的值）。

    3.Set和List对比：
    
    Set：检索元素效率低下，删除和插入效率高，插入和删除不会引起元素位置改变。
    
    List：和数组类似，List可以动态增长，查找元素效率高，插入删除元素效率低，因为会引起其他元素位置改变。
    
    4.Map适合储存键值对的数据。
    
    5.线程安全集合类与非线程安全集合类
    
    LinkedList、ArrayList、HashSet是非线程安全的，Vector是线程安全的;
    
    HashMap是非线程安全的，HashTable是线程安全的;
    
    StringBuilder是非线程安全的，StringBuffer是线程安全的。
    
    
    下面是这些类具体的使用介绍：
    
    ArrayList与LinkedList的区别和适用场景
    
    Arraylist：
    
    优点：ArrayList是实现了基于动态数组的数据结构,因为地址连续，一旦数据存储好了，查询操作效率会比较高（在内存里是连着放的）。
    
    缺点：因为地址连续， ArrayList要移动数据,所以插入和删除操作效率比较低。
    
    LinkedList：
    
    优点：LinkedList基于链表的数据结构,地址是任意的，所以在开辟内存空间的时候不需要等一个连续的地址，对于新增和删除操作add和remove，LinedList比较占优势。LinkedList 适用于要头尾操作或插入指定位置的场景。
    
    缺点：因为LinkedList要移动指针,所以查询操作性能比较低。
    
    适用场景分析：
    
    当需要对数据进行对此访问的情况下选用ArrayList，当需要对数据进行多次增加删除修改时采用LinkedList。
    
    ArrayList与Vector的区别和适用场景
    
    ArrayList有三个构造方法：
    
    public ArrayList(int initialCapacity)//构造一个具有指定初始容量的空列表。    public ArrayList()//构造一个初始容量为10的空列表。    public ArrayList(Collection<? extends E> c)//构造一个包含指定 collection 的元素的列表   
    Vector有四个构造方法：
    
    public Vector() // 使用指定的初始容量和等于零的容量增量构造一个空向量。    public Vector(int initialCapacity) // 构造一个空向量，使其内部数据数组的大小，其标准容量增量为零。    public Vector(Collection<? extends E> c)// 构造一个包含指定 collection 中的元素的向量    public Vector(int initialCapacity,int capacityIncrement)//使用指定的初始容量和容量增量构造一个空的向量
       
    
    ArrayList和Vector都是用数组实现的，主要有这么三个区别：
    
    1).Vector是多线程安全的，线程安全就是说多线程访问同一代码，不会产生不确定的结果。而ArrayList不是，这个可以从源码中看出，Vector类中的方法很多有synchronized进行修饰，这样就导致了Vector在效率上无法与ArrayList相比；
    
    2).两个都是采用的线性连续空间存储元素，但是当空间不足的时候，两个类的增加方式是不同。
    
    3).Vector可以设置增长因子，而ArrayList不可以。
    
    4).Vector是一种老的动态数组，是线程同步的，效率很低，一般不赞成使用。
    
    适用场景：
    
    1.Vector是线程同步的，所以它也是线程安全的，而ArrayList是线程异步的，是不安全的。如果不考虑到线程的安全因素，一般用ArrayList效率比较高。
    
    2.如果集合中的元素的数目大于目前集合数组的长度时，在集合中使用数据量比较大的数据，用Vector有一定的优势。
    
    HashSet与Treeset的适用场景
    
    1.TreeSet 是二叉树（红黑树的树据结构）实现的,Treeset中的数据是自动排好序的，不允许放入null值。
    
    2.HashSet 是哈希表实现的,HashSet中的数据是无序的，可以放入null，但只能放入一个null，两者中的值都不能重复，就如数据库中唯一约束。
    
    3.HashSet要求放入的对象必须实现HashCode()方法，放入的对象，是以hashcode码作为标识的，而具有相同内容的String对象，hashcode是一样，所以放入的内容不能重复。但是同一个类的对象可以放入不同的实例。
    
    适用场景分析:
    
    HashSet是基于Hash算法实现的，其性能通常都优于TreeSet。为快速查找而设计的Set，我们通常都应该使用HashSet，在我们需要排序的功能时，我们才使用TreeSet。
    
    
    HashMap与TreeMap、HashTable的区别及适用场景
    
    
    HashMap 非线程安全  
    
    HashMap：基于哈希表(散列表)实现。使用HashMap要求添加的键类明确定义了hashCode()和equals()[可以重写hashCode()和equals()]，为了优化HashMap空间的使用，您可以调优初始容量和负载因子。其中散列表的冲突处理主要分两种，一种是开放定址法，另一种是链表法。HashMap的实现中采用的是链表法。
    
    TreeMap：非线程安全基于红黑树实现。TreeMap没有调优选项，因为该树总处于平衡状态。
    
    适用场景分析：
    
    HashMap和HashTable:HashMap去掉了HashTable的contains方法，但是加上了containsValue()和containsKey()方法。HashTable同步的，而HashMap是非同步的，效率上比HashTable要高。HashMap允许空键值，而HashTable不允许。
    
    HashMap：适用于Map中插入、删除和定位元素。
    
    Treemap：适用于按自然顺序或自定义顺序遍历键(key)。

    (ps:其实我们工作的过程中对集合的使用是很频繁的,稍加注意并总结积累一下,在面试的时候应该会回答的很轻松)


**6.JAVA常量池**

    Interger中的128(-128~127)
    
    a.当数值范围为-128~127时：如果两个new出来Integer对象，即使值相同，通过“==”比较结果为false，但两个对象直接赋值，则通过“==”比较结果为“true，这一点与String非常相似。
    
    b.当数值不在-128~127时，无论通过哪种方式，即使两个对象的值相等，通过“==”比较，其结果为false；
    
    c.当一个Integer对象直接与一个int基本数据类型通过“==”比较，其结果与第一点相同；
    
    d.Integer对象的hash值为数值本身；
    
    为什么是-128-127?
    
    在Integer类中有一个静态内部类IntegerCache，在IntegerCache类中有一个Integer数组，用以缓存当数值范围为-128~127时的Integer对象。

**7. 简单介绍一下java中的泛型，泛型擦除以及相关的概念,解析与分派**

    泛型是Java SE 1.5的新特性，泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法。 Java语言引入泛型的好处是安全简单。
    
    在Java SE 1.5之前，没有泛型的情况的下，通过对类型Object的引用来实现参数的“任意化”，“任意化”带来的缺点是要做显式的强制类型转换，而这种转换是要求开发者对实际参数类型可以预知的情况下进行的。对于强制类型转换错误的情况，编译器可能不提示错误，在运行的时候才出现异常，这是一个安全隐患。
    
    
    泛型的好处是在编译的时候检查类型安全，并且所有的强制转换都是自动和隐式的，提高代码的重用率。
    
    1、泛型的类型参数只能是类类型（包括自定义类），不能是简单类型。
    
    2、同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。
    
    3、泛型的类型参数可以有多个。
    
    4、泛型的参数类型可以使用extends语句，例如<T extends superclass>。习惯上称为“有界类型”。
    
    5、泛型的参数类型还可以是通配符类型。例如Class<?> classType = Class.forName("java.lang.String");
    
    
泛型擦除以及相关的概念
    
    泛型信息只存在代码编译阶段，在进入JVM之前，与泛型相关的信息都会被擦除掉。

在类型擦除的时候，如果泛型类里的类型参数没有指定上限，例如：，则会被转成Object类型，如果指定了上限，例如：，则会 被传换成对应的类型上限。
    
    Java中的泛型基本上都是在编译器这个层次来实现的。在生成的Java字节码中是不包含泛型中的类型信息的。使用泛型的时候加上的类型参数，会在编译器在编译的时候去掉。这个过程就称为类型擦除。
    
    
    类型擦除引起的问题及解决方法
    
    
    1、先检查，在编译，以及检查编译的对象和引用传递的问题
    
    2、自动类型转换
    
    3、类型擦除与多态的冲突和解决方法
    
    4、泛型类型变量不能是基本数据类型
    
    5、运行时类型查询
    
    6、异常中使用泛型的问题
    
    7、数组（这个不属于类型擦除引起的问题）
    
    9、类型擦除后的冲突
    
    10、泛型在静态方法和静态类中的问题
    

**8.在重写equals方法时，需要遵循哪些约定，具体介绍一下？**

重写equals方法时需要遵循通用约定：自反性、对称性、传递性、一致性.、非空性

1）自反性

对于任何非null的引用值x,x.equals(x)必须返回true。---这一点基本上不会有啥问题

2）对称性

对于任何非null的引用值x和y，当且仅当x.equals(y)为true时，y.equals(x)也为true。

3）传递性

对于任何非null的引用值x、y、z。如果x.equals(y)==true,y.equals(z)==true,那么x.equals(z)==true。

4） 一致性

对于任何非null的引用值x和y，只要equals的比较操作在对象所用的信息没有被修改，那么多次调用x.eqals(y)就会一致性地返回true,或者一致性的返回false。

5）非空性

所有比较的对象都不能为空。


**9.set集合从原理上如何保证不重复**

1）在往set中添加元素时，如果指定元素不存在，则添加成功。也就是说，如果set中不存在(e==null ? e1==null : e.queals(e1))的元素e1,则e1能添加到set中。

2）具体来讲：当向HashSet中添加元素的时候，首先计算元素的hashcode值，然后用这个（元素的hashcode）%（HashMap集合的大小）+1计算出这个元素的存储位置，如果这个位置位空，就将元素添加进去；如果不为空，则用equals方法比较元素是否相等，相等就不添加，否则找一个空位添加。

**10.HashMap和HashTable的主要区别是什么？，两者底层实现的数据结构是什么？**

HashMap和HashTable的区别：

http://www.233.com/ncre2/JAVA/jichu/20100717/084230917.html

二者都实现了Map 接口，是将惟一键映射到特定的值上；主要区别在于：

1)HashMap 没有排序，允许一个null 键和多个null 值,而Hashtable 不允许；

2)HashMap 把Hashtable 的contains 方法去掉了，改成containsvalue 和

containsKey,因为contains 方法容易让人引起误解；

3)Hashtable 继承自Dictionary 类，HashMap 是Java1.2 引进的Map 接口的实现；

4)Hashtable 的方法是Synchronize 的，而HashMap 不是，在多个线程访问Hashtable 时，不需要自己为它的方法实现同步，而HashMap 就必须为之提供外同步。Hashtable 和HashMap 采用的hash/rehash 算法大致一样，所以性能不会有很大的差异。

HashMap和HashTable的底层实现数据结构：

HashMap和Hashtable的底层实现都是数组+链表结构实现的

**11.HashMap何时扩容，扩容的算法是什么？**

HashMap何时扩容：

当向容器添加元素的时候，会判断当前容器的元素个数，如果大于等于阈值---即当前数组的长度乘以加载因子的值的时候，就要自动扩容

扩容的算法是什么：

扩容(resize)就是重新计算容量，向HashMap对象里不停的添加元素，而HashMap对象内部的数组无法装载更多的元素时，对象就需要扩大数组的长度，以便能装入更多的元素。当然Java里的数组是无法自动扩容的，方法是使用一个新的数组代替已有的容量小的数组


**12.Java中对异常是如何进行分类的？**

异常整体分类：

1）Java异常结构中定义有Throwable类。 Exception和Error为其子类。

2）其中Exception表示由于网络故障、文件损坏、设备错误、用户输入非法情况导致的异常；

3）而Error标识Java运行时环境出现的错误，例如：JVM内存耗尽。


**13.为什么jdk8用metaspace数据结构用来替代perm？**

**14.简单谈谈堆外内存以及你的理解和认识。**

**15.怎么理解栈、堆？堆中存什么？栈中存什么？**

**16.为什么要把堆和栈区分出来呢？栈中不是也可以存储数据吗？**

**17.在Java中，什么是是栈的起始点，同是也是程序的起始点？**

**18.为什么不把基本类型放堆中呢？**

**19.Java中的参数传递时传值呢？还是传引用？**

**20.Java中有没有指针的概念？**

**21.Java中，栈的大小通过什么参数来设置？**

**22.一个空Object对象的占多大空间？**

**23.对象引用类型分为哪几类？**


**24.java里带$的函数见过么，是什么意思**





**25.jdk1.5？SparseArray和ArrayMap各自的数据结构，前者的查找是怎么实现的，与HashMap的区别**
    
**26.hash冲突及处理算法；**



**27.Java中实现多态的机制是什么；**



**28.服务器只提供数据接收接口，在多线程或多进程条件下，如何保证数据的有序到达；**

**29.静态属性和静态方法是否可以被继承？是否可以被重写？以及原因？**

**30.修改对象A的equals方法的签名，那么使用HashMap存放这个对象实例的时候，会调用哪个equals方法；**

**31.Java的异常体系；**

Error是程序无法处理的错误，比如OutOfMemoryError、ThreadDeath等。这些异常发生时， Java虚拟机（JVM）一般会选择线程终止。

Exception是程序本身可以处理的异常，这种异常分两大类运行时异常和非运行时异常，程序中应当尽可能去处理这些异常。运行时异常都是RuntimeException类及其子类异常，如NullPointerException、IndexOutOfBoundsException等， 这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的， 程序应该从逻辑角度尽可能避免这类异常的发生。


**32.Java中对象的生命周期**

**33.并发集合了解哪些**


**34.HashMap**



**35.arraylist和linkedlist的区别，以及应用场景**

**36.TreeMap具体实现**

**37.手写生产者/消费者模式**

**38.适配器模式，装饰者模式，外观模式的异同？**



**39.List,Set,Map的区别**

**40.HashSet与HashMap怎么判断集合元素重复**

**41.String 为什么要设计成不可变的？**

String是不可变的（修改String时，不会在原有的内存地址修改，而是重新指向一个新对象），String用final修饰，不可继承，String本质上是个final的char[]数组，所以char[]数组的内存地址不会被修改，而且String 也没有对外暴露修改char[]数组的方法。不可变性可以保证线程安全以及字符串串常量池的实现。
StringBuffer是线程安全的。
StringBuilder是非线程安全的。

Java里的幂等性了解吗？

幂等性原本是数学上的一个概念，即：f(x) = f(f(x))，对同一个系统，使用同样的条件，一次请求和重复的多次请求对系统资源的影响是一致的。
    
    幂等性最为常见的应用就是电商的客户付款，试想一下，如果你在付款的时候因为网络等各种问题失败了，然后又去重复的付了一次，是一种多么糟糕的体验。幂等性 就是为了解决这样的问题。

实现幂等性可可以使用Token机制。

    核心思想是为每一次操作生成一个唯一性的凭证，也就是token。一个token在操作的每一个阶段只有一次执行权，一旦执行成功则保存执行结果。对 重复的请求，返回同一个结果。

例如：电商平台上的订单id就是最适合的token。当用户下单时，会经历多个环节，比如生成订单，减库存，减优惠券等等。每一个环节执行时都先检 测一下该订单id是否已经执行过这一步骤，对未执行的请求，执行操作并缓存结果，而对已经执行过的id，则直接返回之前的执行结果，不做任何操

作。这样可以在最大程度上避免操作的重复执行问题，缓存起来的执行结果也能用于事务的控制等。

**42.List 和 Map 的实现方式以及存储方式。**

**43.静态内部类的设计意图。**


**44.arraylist 与 linkedlist 异同？**

**45.object类的equal 和hashcode 方法重写，为什么？**



**46.java中==和equals和hashCode的区别**

http://blog.csdn.net/tiantiandjava/article/details/46988461

https://www.cnblogs.com/Qian123/p/5703507.html

**47.int、char、long各占多少字节数**

**48.int与integer的区别**

为什么Java里的匿名内部类只能访问final修饰的外部变量？

匿名内部类用法

    public class TryUsingAnonymousClass {
        public void useMyInterface() {
            final Integer number = 123;
            System.out.println(number);
    
            MyInterface myInterface = new MyInterface() {
                @Override
                public void doSomething() {
                    System.out.println(number);
                }
            };
            myInterface.doSomething();
    
            System.out.println(number);
        }
    }
编译后的结果

    class TryUsingAnonymousClass$1
            implements MyInterface {
        private final TryUsingAnonymousClass this$0;
        private final Integer paramInteger;
    
        TryUsingAnonymousClass$1(TryUsingAnonymousClass this$0, Integer paramInteger) {
            this.this$0 = this$0;
            this.paramInteger = paramInteger;
        }
    
        public void doSomething() {
            System.out.println(this.paramInteger);
        }
    }
因为匿名内部类最终用会编译成一个单独的类，而被该类使用的变量会以构造函数参数的形式传递给该类，例如：Integer paramInteger，如果变量 不定义成final的，paramInteger在匿名内部类被可以被修改，进而造成和外部的paramInteger不一致的问题，为了避免这种不一致的情况，因为Java 规定匿名内部类只能访问final修饰的外部变量。

讲一下Java的编码方式？

为什么需要编码

    计算机存储信息的最小单元是一个字节即8bit，所以能表示的范围是0~255，这个范围无法保存所有的字符，所以需要一个新的数据结构char来表示这些字符，从char到byte需要编码。

常见的编码方式有以下几种：

ASCII：总共有 128 个，用一个字节的低 7 位表示，031 是控制字符如换行回车删除等；32126 是打印字符，可以通过键盘输入并且能够显示出来。
GBK：码范围是 8140~FEFE（去掉 XX7F）总共有 23940 个码位，它能表示 21003 个汉字，它的编码是和 GB2312 兼容的，也就是说用 GB2312 编码的汉字可以用 GBK 来解码，并且不会有乱码。
UTF-16：UTF-16 具体定义了 Unicode 字符在计算机中存取方法。UTF-16 用两个字节来表示 Unicode 转化格式，这个是定长的表示方法，不论什么字符都可以用两个字节表示，两个字节是 16 个 bit，所以叫 UTF-16。UTF-16 表示字符非常方便，每两个字节表示一个字符，这个在字符串操作时就大大简化了操作，这也是 Java 以 UTF-16 作为内存的字符存储格式的一个很重要的原因。
UTF-8：统一采用两个字节表示一个字符，虽然在表示上非常简单方便，但是也有其缺点，有很大一部分字符用一个字节就可以表示的现在要两个字节表示，存储空间放大了一倍，在现在的网络带宽还非常有限的今天，这样会增大网络传输的流量，而且也没必要。而 UTF-8 采用了一种变长技术，每个编码区域有不同的字码长度。不同类型的字符可以是由 1~6 个字节组成。
Java中需要编码的地方一般都在字符到字节的转换上，这个一般包括磁盘IO和网络IO。

    Reader 类是 Java 的 I/O 中读字符的父类，而 InputStream 类是读字节的父类，InputStreamReader 类就是关联字节到字符的桥梁，它负责在 I/O 过程中处理读取字节到字符的转换，而具体字节到字符的解码实现它由 StreamDecoder 去实现，在 StreamDecoder 解码过程中必须由用户指定 Charset 编码格式。

**49.谈谈对java多态的理解**

多态是指父类的某个方法被子类重写时，可以产生自己的功能行为，同一个操作作用于不同对象，可以有不同的解释，产生不同的执行结果。

多态的三个必要条件：

继承父类。
重写父类的方法。
父类的引用指向子类对象。

Java多态性理解

Java中多态性的实现

什么是多态

面向对象的三大特性：封装、继承、多态。从一定角度来看，封装和继承几乎都是为多态而准备的。这是我们最后一个概念，也是最重要的知识点。

多态的定义：指允许不同类的对象对同一消息做出响应。即同一消息可以根据发送对象的不同而采用多种不同的行为方式。（发送消息就是函数调用）

实现多态的技术称为：动态绑定（dynamic binding），是指在执行期间判断所引用对象的实 际类型，根据其实际的类型调用其相应的方法。

多态的作用：消除类型之间的耦合关系。

现实中，关于多态的例子不胜枚举。比方说按下 F1 键这个动作，如果当前在 Flash 界面下弹出的就是 AS 3 的帮助文档；如果当前在 Word 下弹出的就是 Word 帮助；在 Windows 下弹出的就是 Windows 帮助和支持。同一个事件发生在不同的对象上会产生不同的结果。 下面是多态存在的三个必要条件，要求大家做梦时都能背出来！

多态存在的三个必要条件 一、要有继承； 二、要有重写； 三、父类引用指向子类对象。

多态的好处：

1.可替换性（substitutability）。多态对已存在代码具有可替换性。例如，多态对圆Circle类工作，对其他任何圆形几何体，如圆环，也同样工作。

2.可扩充性（extensibility）。多态对代码具有可扩充性。增加新的子类不影响已存在类的多态性、继承性，以及其他特性的运行和操作。实际上新加子类更容易获得多态功能。例如，在实现了圆锥、半圆锥以及半球体的多态基础上，很容易增添球体类的多态性。

3.接口性（interface-ability）。多态是超类通过方法签名，向子类提供了一个共同接口，由子类来完善或者覆盖它而实现的。如图8.3 所示。图中超类Shape规定了两个实现多态的接口方法，computeArea()以及computeVolume()。子类，如Circle和Sphere为了实现多态，完善或者覆盖这两个接口方法。

4.灵活性（flexibility）。它在应用中体现了灵活多样的操作，提高了使用效率。

5.简化性（simplicity）。多态简化对应用软件的代码编写和修改过程，尤其在处理大量对象的运算和操作时，这个特点尤为突出和重要。

Java中多态的实现方式：接口实现，继承父类进行方法重写，同一个类中进行方法重载。

**50.String、StringBuffer、StringBuilder区别**

**51.什么是内部类？内部类的作用**

内部类可以用多个实例，每个实例都有自己的状态信息，并且与其他外围对象的信息相互独立。

在单个外围类中，可以让多个内部类以不同的方式实现同一个接口，或者继承同一个类。

创建内部类对象的时刻并不依赖于外围类对象的创建。

内部类并没有令人迷惑的“is-a”关系，他就是一个独立的实体。

内部类提供了更好的封装，除了该外围类，其他类都不能访问

**52.抽象类和接口区别**

共同点

是上层的抽象层。
都不能被实例化。
都能包含抽象的方法，这些抽象的方法用于描述类具备的功能，但是不比提供具体的实现。
区别

在抽象类中可以写非抽象的方法，从而避免在子类中重复书写他们，这样可以提高代码的复用性，这是抽象类的优势，接口中只能有抽象的方法。
一个类只能继承一个直接父类，这个父类可以是具体的类也可是抽象类，但是一个类可以实现多个接口。

默认的方法实现 抽象类可以有默认的方法实现完全是抽象的。接口根本不存在方法的实现

实现 子类使用extends关键字来继承抽象类。如果子类不是抽象类的话，它需要提供抽象类中所有声明的方法的实现。
子类使用关键字implements来实现接口。它需要提供接口中所有声明的方法的实现

构造器
抽象类可以有构造器
接口不能有构造器

与正常Java类的区别
除了你不能实例化抽象类之外，它和普通Java类没有任何区 接口是完全不同的类型

访问修饰符
抽象方法可以有public、protected和default这些修饰符 接口方法默认修饰符是public。你不可以使用其它修饰符。

main方法
抽象方法可以有main方法并且我们可以运行它
接口没有main方法，因此我们不能运行它。

多继承
抽象类在java语言中所表示的是一种继承关系，一个子类只能存在一个父类，但是可以存在多个接口。

速度
它比接口速度要快
接口是稍微有点慢的，因为它需要时间去寻找在类中实现的方法。

添加新方法
如果你往抽象类中添加新的方法，你可以给它提供默认的实现。因此你不需要改变你现在的代码。
如果你往接口中添加方法，那么你必须改变实现该接口的类。

**53.抽象类的意义**

为其子类提供一个公共的类型 封装子类中得重复内容 定义抽象方法，子类虽然有不同的实现 但是定义是一致的

**54.抽象类与接口的应用场景**

**55.抽象类是否可以没有方法和属性？**

**56.接口的意义**

规范、扩展、回调

**57.泛型中extends和super的区别**

**58.父类的静态方法能否被子类重写**

不能

子类继承父类后，用相同的静态方法和非静态方法，这时非静态方法覆盖父类中的方法（即方法重写），父类的该静态方法被隐藏（如果对象是父类则调用该隐藏的方法），另外子类可继承父类的静态与非静态方法，至于方法重载我觉得它其中一要素就是在同一类中，不能说父类中的什么方法与子类里的什么方法是方法重载的体现

**59.final，finally，finalize的区别**

**60.序列化的方式**

**61.Serializable 和Parcelable 的区别**

**62.静态属性和静态方法是否可以被继承？是否可以被重写？以及原因？**

**63.静态内部类的设计意图**

**64.成员内部类、静态内部类、局部内部类和匿名内部类的理解，以及项目中的应用**

静态内部类：只是为了降低包的深度，方便类的使用，静态内部类适用于包含类当中，但又不依赖与外在的类，不用使用外在类的非静态属性和方法，只是为了方便管理类结构而定义。在创建静态内部类的时候，不需要外部类对象的引用。
非静态内部类：持有外部类的引用，可以自由使用外部类的所有变量和方法。

**65.闭包和局部内部类的区别**

    「函数」和「函数内部能访问到的变量」（也叫环境）的总和，就是一个闭包。

    fun main(args: Array<String>) {
        test
    }
    val test = if (5 > 3) {
        print("yes")
    } else {
        print("no")
    }

**66.string 转换成 integer的方式及原理**


**67.讲一下常见编码方式？**

**68.utf-8编码中的中文占几个字节；int型几个字节？**

**69.静态代理和动态代理的区别，什么场景使用？**

静态代理与动态代理的区别在于代理类生成的时间不同，如果需要对多个类进行代理，并且代理的功能都是一样的，用静态代理重复编写代理类就非常的麻烦，可以用动态代理动态的生成代理类。

    // 为目标对象生成代理对象
    public Object getProxyInstance() {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(),
                new InvocationHandler() {
    
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        System.out.println("开启事务");
    
                        // 执行目标对象方法
                        Object returnValue = method.invoke(target, args);
    
                        System.out.println("提交事务");
                        return null;
                    }
                });
    }

**70.谈谈你对解析与分派的认识。**

**71.修改对象A的equals方法的签名，那么使用HashMap存放这个对象实例的时候，会调用哪个equals方法？**

**72.Java中实现多态的机制是什么？**

**73.如何将一个Java对象序列化到文件里？**

**74.说说你对Java反射的理解**

**75.说说你对Java注解的理解**

注解相当于一种标记，在程序中加了注解就等于为程序打上了某种标记。程序可以利用ava的反射机制来了解你的类及各种元素上有无何种标记，针对不同的标记，就去做相 应的事件。标记可以加在包，类，字段，方法，方法的参数以及局部变量上。

**76.说说你对依赖注入的理解**

**77.说一下泛型原理，并举例说明**

**78.Java中String的了解**

**79.Object类的equal和hashCode方法重写，为什么？**


**80.HashMap底层实现，hashCode如何对应bucket?**


**81.hashMap的2倍扩容机制为什么是2倍**

**82.在java8和java7中，hashMap的hash函数有什么不同**


**83.ArrayList和LinkedList你知道吗？你知道它怎么动态扩容的吗？**

**84.Java 中的 Error、 Exception 的区别；**    

**85.Java 中内部类为什么可以访问外部类**

**86.Java为何引入泛型，泛型边界****

**87.ArrayMap跟SparseArray在HashMap上面的改进**

**88.Java的四种引用及使用场景**


**89.Class文件结构（常量池）。**





**90.运行时栈帧结构（主要是局部变量表，理解栈堆）。**

**91.Java泛型了解吗，知道它的运行机制吗？**

    泛型是为了参数化类型。

为什么使用泛型？

相对于使用Object这种简单粗暴的方式，泛型提供了一种参数化的能力，使得数据的类型可以像参数一样被传递进来，这提供了一种扩展能力。
当数据类型确定以后，提供了一种类型检测机制，只有相匹配的数据才可以正常赋值，否则编译错误，增强了安全性。
泛型提高了代码的可读性，不必等到运行时采取执行类型转换，在编写代码阶段，程序员就可以通过参数书写正确的数据类型。
除了用 表示泛型外，还有 <?> 这种形式。？ 被称为通配符。

被称作无限定的通配符。
被称作有上限的通配符。
被称作有下限的通配符。



**92.Java重排序和顺序一致性。（as-if-serial和happens-before）**


**93.Java生产者和消费者模型。**

生成者消费者模型

生产者和消费者在同一时间段内共用同一个存储空间，生产者往存储空间中添加产品，消费者从存储空间中取走产品，当存储空间为空时，消费者阻塞，当存储空间满时，生产者阻塞。

wait()和notify()方法的实现生成者消费者模型，缓冲区满和为空时都调用wait()方法等待，当生产者生产了一个产品或者消费者消费了一个产品之后会唤醒所有线程。

    public class ProducerAndCustomerModel {
        
        private static Integer count = 0;
        private static final Integer FULL = 10;
        private static String LOCK = "lock";
        
        public static void main(String[] args) {
            Test1 test1 = new Test1();
            new Thread(test1.new Producer()).start();
            new Thread(test1.new Consumer()).start();
            new Thread(test1.new Producer()).start();
            new Thread(test1.new Consumer()).start();
            new Thread(test1.new Producer()).start();
            new Thread(test1.new Consumer()).start();
            new Thread(test1.new Producer()).start();
            new Thread(test1.new Consumer()).start();
        }
        class Producer implements Runnable {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(3000);
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                    synchronized (LOCK) {
                        while (count == FULL) {
                            try {
                                LOCK.wait();
                            } catch (Exception e) {
                                e.printStackTrace();
                            }
                        }
                        count++;
                        System.out.println(Thread.currentThread().getName() + "生产者生产，目前总共有" + count);
                        LOCK.notifyAll();
                    }
                }
            }
        }
        class Consumer implements Runnable {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(3000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    synchronized (LOCK) {
                        while (count == 0) {
                            try {
                                LOCK.wait();
                            } catch (Exception e) {
                            }
                        }
                        count--;
                        System.out.println(Thread.currentThread().getName() + "消费者消费，目前总共有" + count);
                        LOCK.notifyAll();
                    }
                }
            }
        }
    }
    

**94.java 7 8 9 10的区别**

**95.lambda原理**

    Lambda 表达式俗称匿名函数。Kotlin 的 Lambda表达式更“纯粹”一点， 因为它是真正把Lambda抽象为了一种类型，而 Java 8 的 Lambda 只是单方法匿名接口实现的语法糖罢了。
    
    val printMsg = { msg: String -> 
    	println(msg) 
    }
    
    fun main(args: Array<String>) {
      printMsg("hello")
    }
    
高阶函数了解吗？

当定义一个闭包作为参数的函数，称这个函数为高阶函数。

fun main(args: Array<String>) {
    log("world", printMsg)
}

val printMsg = { str: String ->
    println(str)
}

val log = { str: String, printLog: (String) -> Unit ->
    printLog(str)
}

**96.为什么java 7中不能用lambda**

**97.数组扩容**

**98.sparearray原理（拆分包）**

**99.Java反射**

Java对象的生命周期

加载：将类的信息加载到JVM的方法区，然后在堆区中实例化一个java.lang.Class对象，作为方法去中这个类的信息入口。
连接：验证：验证类是否合法。准备：为静态变量分配内存并设置JVM默认值，非静态变量不会分配内存。解析：将常量池里的符号引用转换为直接引用。
初始化：初始化类的静态赋值语句和静态代码块，主动引用会被触发类的初始化，被动引用不会触发类的初始化。
使用：执行类的初始化，主动引用会被触发类的初始化，被动引用不会触发类的初始化。
卸载：卸载过程就是清楚堆里类的信息，以下情况会被卸载：① 类的所有实例都已经被回收。② 类的ClassLoader被回收。③ 类的CLass对象没有被任何地方引用，无法在任何地方通过 反射访问该类。

**100.集合类以及集合框架**

http://alexyyek.github.io/2015/04/06/Collection/
http://www.cnblogs.com/CarpenterLee/p/5545987.html

**101.HashMap的实现原理**

HashMap概述： HashMap是基于哈希表的Map接口的非同步实现。此实现提供所有可选的映射操作，并允许使用null值和null键。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。
HashMap的数据结构： 在java编程语言中，最基本的结构就是两种，一个是数组，另外一个是模拟指针（引用），所有的数据结构都可以用这两个基本结构来构造的，HashMap也不例外。HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。

![image](http://www.jackywang.tech/AndroidInterview-Q-A/picture/hashmap.jpg)

从上图中可以看出，HashMap底层就是一个数组结构，数组中的每一项又是一个链表。当新建一个HashMap的时候，就会初始化一个数组。

**102.ConcurrentHashMap的实现原理**

**103.[动态代理和静态代理](http://a.codekk.com/detail/Android/Caij/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E5%8A%A8%E6%80%81%E4%BB%A3%E7%90%86)**
