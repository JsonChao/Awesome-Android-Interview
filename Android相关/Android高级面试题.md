## Android高级面试题


**一、图片**

**图片库对比**

http://stackoverflow.com/questions/29363321/picasso-v-s-imageloader-v-s-fresco-vs-glide
http://www.trinea.cn/android/android-image-cache-compare/


图片库的源码分析

图片框架缓存实现

LRUCache原理

图片加载原理

自己去实现图片库，怎么做？

****Glide源码解析****

http://www.lightskystreet.com/2015/10/12/glide_source_analysis/
http://frodoking.github.io/2015/10/10/android-glide/
http://ethanhua.cn/archives/243

Glide使用什么缓存？

Glide内存缓存如何控制大小？

谈谈对Fresco理解？

**Fresco与Glide的对比：**

Glide：相对轻量级，用法简单优雅，支持Gif动态图，适合用在那些对图片依赖不大的App中。
Fresco：采用匿名共享内存来保存图片，也就是Native堆，有效的的避免了OOM，功能强大，但是库体积过大，适合用在对图片依赖比较大的App中。

Fresco的整体架构如下图所示：

![image](https://github.com/guoxiaoxing/android-open-framwork-analysis/raw/master/art/fresco/fresco_structure.png)

DraweeView：继承于ImageView，只是简单的读取xml文件的一些属性值和做一些初始化的工作，图层管理交由Hierarchy负责，图层数据获取交由负责。
DraweeHierarchy：由多层Drawable组成，每层Drawable提供某种功能（例如：缩放、圆角）。
DraweeController：控制数据的获取与图片加载，向pipeline发出请求，并接收相应事件，并根据不同事件控制Hierarchy，从DraweeView接收用户的事件，然后执行取消网络请求、回收资源等操作。
DraweeHolder：统筹管理Hierarchy与DraweeHolder。
ImagePipeline：Fresco的核心模块，用来以各种方式（内存、磁盘、网络等）获取图像。
Producer/Consumer：Producer也有很多种，它用来完成网络数据获取，缓存数据获取、图片解码等多种工作，它产生的结果由Consumer进行消费。
IO/Data：这一层便是数据层了，负责实现内存缓存、磁盘缓存、网络缓存和其他IO相关的功能。
纵观整个Fresco的架构，DraweeView是门面，和用户进行交互，DraweeHierarchy是视图层级，管理图层，DraweeController是控制器，管理数据。它们构成了整个Fresco框架的三驾马车。当然还有我们 幕后英雄Producer，所有的脏活累活都是它干的，最佳劳模👍

理解了Fresco整体的架构，我们还有了解在这套矿建里发挥重要作用的几个关键角色，如下所示：

Supplier：提供一种特定类型的对象，Fresco里有很多以Supplier结尾的类都实现了这个接口。
SimpleDraweeView：这个我们就很熟悉了，它接收一个URL，然后调用Controller去加载图片。该类继承于GenericDraweeView，GenericDraweeView又继承于DraweeView，DraweeView是Fresco的顶层View类。
PipelineDraweeController：负责图片数据的获取与加载，它继承于AbstractDraweeController，由PipelineDraweeControllerBuilder构建而来。AbstractDraweeController实现了DraweeController接口，DraweeController 是Fresco的数据大管家，所以的图片数据的处理都是由它来完成的。
GenericDraweeHierarchy：负责SimpleDraweeView上的图层管理，由多层Drawable组成，每层Drawable提供某种功能（例如：缩放、圆角），该类由GenericDraweeHierarchyBuilder进行构建，该构建器 将placeholderImage、retryImage、failureImage、progressBarImage、background、overlays与pressedStateOverlay等 xml文件或者Java代码里设置的属性信息都传入GenericDraweeHierarchy中，由GenericDraweeHierarchy进行处理。
DraweeHolder：该类是一个Holder类，和SimpleDraweeView关联在一起，DraweeView是通过DraweeHolder来统一管理的。而DraweeHolder又是用来统一管理相关的Hierarchy与Controller
DataSource：类似于Java里的Futures，代表数据的来源，和Futures不同，它可以有多个result。
DataSubscriber：接收DataSource返回的结果。
ImagePipeline：用来调取获取图片的接口。
Producer：加载与处理图片，它有多种实现，例如：NetworkFetcherProducer，LocalAssetFetcherProducer，LocalFileFetchProducer。从这些类的名字我们就可以知道它们是干什么的。 Producer由ProducerFactory这个工厂类构建的，而且所有的Producer都是像Java的IO流那样，可以一层嵌套一层，最终只得到一个结果，这是一个很精巧的设计👍
Consumer：用来接收Producer产生的结果，它与Producer组成了生产者与消费者模式。
注：Fresco源码里的类的名字都比较长，但是都是按照一定的命令规律来的，例如：以Supplier结尾的类都实现了Supplier接口，它可以提供某一个类型的对象（factory, generator, builder, closure等）。 以Builder结尾的当然就是以构造者模式创建对象的类。

**怎样计算一张图片的大小，加载bitmap过程（怎样保证不产生内存溢出），二级缓存，LRUCache算法。**

    计算一张图片的大小
    
    
    图片占用内存的计算公式：图片高度 * 图片宽度 * 一个像素占用的内存大小.所以，计算图片占用内存大小的时候，要考虑图片所在的目录跟设备密度，这两个因素其实影响的是图片的高宽，android会对图片进行拉升跟压缩。
    
    
    加载bitmap过程（怎样保证不产生内存溢出）
    
    
    由于Android对图片使用内存有限制，若是加载几兆的大图片便内存溢出。Bitmap会将图片的所有像素（即长x宽）加载到内存中，如果图片分辨率过大，会直接导致内存OOM，只有在BitmapFactory加载图片时使用BitmapFactory.Options对相关参数进行配置来减少加载的像素。
    
    
    BitmapFactory.Options相关参数详解
    
    
    (1).Options.inPreferredConfig值来降低内存消耗。
    
    比如：默认值ARGB_8888改为RGB_565,节约一半内存。
    
    
    (2).设置Options.inSampleSize 缩放比例，对大图片进行压缩 。
    
    
    (3).设置Options.inPurgeable和inInputShareable：让系统能及时回 收内存。
    
    A：inPurgeable：设置为True时，表示系统内存不足时可以被回 收，设置为False时，表示不能被回收。
    
    B：inInputShareable：设置是否深拷贝，与inPurgeable结合使用，inPurgeable为false时，该参数无意义。
    
    
    (4).使用decodeStream代替其他方法。
    
    decodeResource,setImageResource,setImageBitmap等方法

**LRUCache算法是怎样实现的。**

    内部存在一个LinkedHashMap和maxSize，把最近使用的对象用强引用存储在 LinkedHashMap中，给出来put和get方法，每次put图片时计算缓存中所有图片总大小，跟maxSize进行比较，大于maxSize，就将最久添加的图片移除；反之小于maxSize就添加进来。
    
    
    之前，我们会使用内存缓存技术实现，也就是软引用或弱引用，在Android 2.3（APILevel 9）开始，垃圾回收器会更倾向于回收持有软引用或弱引用的对象，这让软引用和弱引用变得不再可靠。
    
写个图片浏览器，说出你的思路

**Bitmap 压缩策略**

    加载 Bitmap 的方式：
    BitmapFactory 四类方法：
    decodeFile( 文件系统 )
    decodeResourece( 资源 )
    decodeStream( 输入流 ) 
    decodeByteArray( 字节数 )
    BitmapFactory.options 参数
    inSampleSize 采样率，对图片高和宽进行缩放，以最小比进行缩放（一般取值为 2 的指数）。通常是根据图片宽高实际的大小/需要的宽高大小，分别计算出宽和高的缩放比。但应该取其中最小的缩放比，避免缩放图片太小，到达指定控件中不能铺满，需要拉伸从而导致模糊。
    inJustDecodeBounds 获取图片的宽高信息，交给  inSampleSize 参数选择缩放比。通过 inJustDecodeBounds = true，然后加载图片就可以实现只解析图片的宽高信息，并不会真正的加载图片，所以这个操作是轻量级的。当获取了宽高信息，计算出缩放比后，然后在将 inJustDecodeBounds = false,再重新加载图片，就可以加载缩放后的图片。
    高效加载 Bitmap 的流程
    将 BitmapFactory.Options 的 inJustDecodeBounds 参数设为 true 并加载图片
    从 BitmapFactory.Options 中取出图片原始的宽高信息，对应于 outWidth 和 outHeight 参数
    根据采样率规则并结合目标 view 的大小计算出采样率 inSampleSize
    将 BitmapFactory.Options 的 inJustDecodeBounds 设置为 false 重新加载图片

**Bitmap的处理：**

当使用ImageView的时候，可能图片的像素大于ImageView，此时就可以通过BitmapFactory.Option来对图片进行压缩，inSampleSize表示缩小2^(inSampleSize-1)倍。

BitMap的缓存：

1.使用LruCache进行内存缓存。

2.使用DiskLruCache进行硬盘缓存。

3.实现一个ImageLoader的流程：同步异步加载、图片压缩、内存硬盘缓存、网络拉取

    1.同步加载只创建一个线程然后按照顺序进行图片加载
    2.异步加载使用线程池，让存在的加载任务都处于不同线程
    3.为了不开启过多的异步任务，只在列表静止的时候开启图片加载

**图片加载库相关，bitmap如何处理大图，如一张30M的大图，如何预防OOM**

    
**[如何优雅的展示Bitmap大图](http://blog.csdn.net/guolin_blog/article/details/9316683)**


**二、网络和安全机制**

**Android：主流网络请求开源库的对比（Android-Async-Http、Volley、OkHttp、Retrofit）**

https://www.jianshu.com/p/050c6db5af5a

**怎么考虑数据传输的安全性**

    如果应用对传输的数据没有任何安全措施，攻击者设置的钓鱼网络中更改DNS服务器。这台服务器可以获取用户信息，或充当中间人与原服务器交换数据。在SSL/TLS通信中，客户端通过数字证书判断服务器是否可信，并采用证书的公钥与服务器进行加密通信。

访问网络如何加密
1：对称加密（ＤＥＳ，ＡＥＳ）和非对称（ＲＳＡ公钥与私钥）。（支付宝里的商户的公钥和私钥）
2：MD5（算法）
3：Base64

自己去设计网络请求框架，怎么做？

okhttp源码

Volley与OkHttp的对比：

Volley：支持HTTPS。缓存、异步请求，不支持同步请求。协议类型是Http/1.0, Http/1.1，网络传输使用的是 HttpUrlConnection/HttpClient，数据读写使用的IO。
OkHttp：支持HTTPS。缓存、异步请求、同步请求。协议类型是Http/1.0, Http/1.1, SPDY, Http/2.0, WebSocket，网络传输使用的是封装的Socket，数据读写使用的NIO（Okio）。
SPDY协议类似于HTTP，但旨在缩短网页的加载时间和提高安全性。SPDY协议通过压缩、多路复用和优先级来缩短加载时间。

Okhttp的子系统层级结构图如下所示：

![image](https://github.com/guoxiaoxing/android-open-framwork-analysis/raw/master/art/okhttp/okhttp_structure.png)

网络配置层：利用Builder模式配置各种参数，例如：超时时间、拦截器等，这些参数都会由Okhttp分发给各个需要的子系统。
重定向层：负责重定向。
Header拼接层：负责把用户构造的请求转换为发送给服务器的请求，把服务器返回的响应转换为对用户友好的响应。
HTTP缓存层：负责读取缓存以及更新缓存。
连接层：连接层是一个比较复杂的层级，它实现了网络协议、内部的拦截器、安全性认证，连接与连接池等功能，但这一层还没有发起真正的连接，它只是做了连接器一些参数的处理。
数据响应层：负责从服务器读取响应的数据。
在整个Okhttp的系统中，我们还要理解以下几个关键角色：

OkHttpClient：通信的客户端，用来统一管理发起请求与解析响应。
Call：Call是一个接口，它是HTTP请求的抽象描述，具体实现类是RealCall，它由CallFactory创建。
Request：请求，封装请求的具体信息，例如：url、header等。
RequestBody：请求体，用来提交流、表单等请求信息。
Response：HTTP请求的响应，获取响应信息，例如：响应header等。
ResponseBody：HTTP请求的响应体，被读取一次以后就会关闭，所以我们重复调用responseBody.string()获取请求结果是会报错的。
Interceptor：Interceptor是请求拦截器，负责拦截并处理请求，它将网络请求、缓存、透明压缩等功能都统一起来，每个功能都是一个Interceptor，所有的Interceptor最 终连接成一个Interceptor.Chain。典型的责任链模式实现。
StreamAllocation：用来控制Connections与Streas的资源分配与释放。
RouteSelector：选择路线与自动重连。
RouteDatabase：记录连接失败的Route黑名单。

网络请求缓存处理，okhttp如何处理网络缓存的

从网络加载一个10M的图片，说下注意事项

图片缓存、异常恢复、质量压缩

如何验证证书的合法性?

client如何确定自己发送的消息被server收到?

谈谈你对WebSocket的理解

WebSocket与socket的区别

谈谈你对安卓签名的理解。

请解释安卓为啥要加签名机制?

视频加密传输

App 是如何沙箱化，为什么要这么做？

权限管理系统（底层的权限是如何进行 grant 的）？

**三、数据库**

数据库框架对比和源码分析

数据库的优化

数据库数据迁移问题

**四、插件化、模块化、组件化、热修复、增量更新、Gradle**

**插件化相关技术，热修补技术是怎样实现的，和插件化有什么区别**

http://www.liuguangli.win/archives/366
http://www.liuguangli.win/archives/387
http://www.liuguangli.win/archives/452

    相同点:
    
    
    都使用ClassLoader来实现的加载的新的功能类，都可以使用PathClassLoader与DexClassLoader
    
    
    不同点：
    
    
    热修复因为是为了修复Bug的，所以要将新的同名类替代同名的Bug类，要抢先加载新的类而不是Bug类，所以多做两件事：在原先的app打包的时候，阻止相关类去打上CLASS_ISPREVERIFIED标志，还有在热修复时动态改变BaseDexClassLoader对象间接引用的dexElements，这样才能抢先代替Bug类，完成系统不加载旧的Bug类.
    
    
    而插件化只是增肌新的功能类或者是资源文件，所以不涉及抢先加载旧的类这样的使命，就避过了阻止相关类去打上CLASS_ISPREVERIFIED标志和还有在热修复时动态改变BaseDexClassLoader对象间接引用的dexElements.
    
    所以插件化比热修复简单，热修复是在插件化的基础上在进行替旧的Bug类
    
**了解插件化和热修复吗，它们有什么区别，理解它们的原理吗？**

插件化：插件化是体现在功能拆分方面的，它将某个功能独立提取出来，独立开发，独立测试，再插入到主应用中。依次来较少主应用的规模。
热修复：热修复是体现在bug修复方面的，它实现的是不需要重新发版和重新安装，就可以去修复已知的bug。
利用PathClassLoader和DexClassLoader去加载与bug类同名的类，替换掉bug类，进而达到修复bug的目的，原理是在app打包的时候阻止类打上CLASS_ISPREVERIFIED标志，然后在 热修复的时候动态改变BaseDexClassLoader对象间接引用的dexElements，替换掉旧的类。

目前热修复框架主要分为两大类：

Sophix：修改方法指针。
Tinker：修改dex数组元素。

**热补丁**

    原因：因为一个dvm中存储方法id用的是short类型，导致dex中方法不能超过65536个
    原理：将编译好的class文件拆分打包成两个dex，绕过dex方法数量的限制以及安装时的检查，在运行时再动态加载第二个dex文件中。使用Dexclassloader。

**动态加载(也叫插件化技术)**

    动态加载主要解决3个技术问题：
    1，使用ClassLoader加载类。
    2，资源访问。
    3，生命周期管理。
    
**模块化的好处**

https://www.jianshu.com/p/376ea8a19a17

Android 组件化的原理，还有一些组件化平时使用的问题；


对热修复和插件化的理解

插件化原理分析

模块化实现（好处，原因）

热修复,

项目组件化的理解

描述清点击 Android Studio 的 build 按钮后发生了什么

gradle熟悉么，自动打包知道么

如何加快 Gradle 的编译速度

**五、架构设计和设计模式**

架构设计

![image](http://www.jackywang.tech/AndroidInterview-Q-A/picture/architucture.png)

http://www.tianmaying.com/tutorial/AndroidMVC

谈谈你对Android设计模式的理解

MVC MVP MVVM原理和区别

你所知道的设计模式有哪些？

项目中常用的设计模式

手写生产者/消费者模式

适配器模式，装饰者模式，外观模式的异同？

用到的一些开源框架，介绍一个看过源码的，内部实现过程。

谈谈对RxJava的理解

RxJava的功能与原理实现

RxJava的作用，与平时使用的异步操作来比的优缺点

说说EventBus作用，实现方式，代替EventBus的方式

从0设计一款App整体架构，如何去做？

说一款你认为当前比较火的应用并设计(比如：直播APP，P2P金融，小视频等)

谈谈对java状态机理解

Fragment如果在Adapter中使用应该如何解耦？

Binder机制及底层实现

对于应用更新这块是如何做的？(解答：灰度，强制更新，分区域更新)？

实现一个Json解析器(可以通过正则提高速度)

统计启动时长,标准

**六、性能优化**

**Android优化**

性能优化

    1).节制的使用Service 如果应用程序需要使用Service来执行后台任务的话，只有当任务正在执行的时候才应该让Service运行起来。当启动一个Service时，系统会倾向于将这个Service所依赖的进程进行保留，系统可以在LRUcache当中缓存的进程数量也会减少，导致切换程序的时候耗费更多性能。我们可以使用IntentService，当后台任务执行结束后会自动停止，避免了Service的内存泄漏。
    
    2).当界面不可见时释放内存 当用户打开了另外一个程序，我们的程序界面已经不可见的时候，我们应当将所有和界面相关的资源进行释放。重写Activity的onTrimMemory()方法，然后在这个方法中监听TRIM_MEMORY_UI_HIDDEN这个级别，一旦触发说明用户离开了程序，此时就可以进行资源释放操作了。
    
    3).当内存紧张时释放内存 onTrimMemory()方法还有很多种其他类型的回调，可以在手机内存降低的时候及时通知我们，我们应该根据回调中传入的级别来去决定如何释放应用程序的资源。
    
    4).避免在Bitmap上浪费内存 读取一个Bitmap图片的时候，千万不要去加载不需要的分辨率。可以压缩图片等操作。
    
    5).使用优化过的数据集合 Android提供了一系列优化过后的数据集合工具类，如SparseArray、SparseBooleanArray、LongSparseArray，使用这些API可以让我们的程序更加高效。HashMap工具类会相对比较低效，因为它需要为每一个键值对都提供一个对象入口，而SparseArray就避免掉了基本数据类型转换成对象数据类型的时间。

布局优化


    1).重用布局文件
    
    标签可以允许在一个布局当中引入另一个布局，那么比如说我们程序的所有界面都有一个公共的部分，这个时候最好的做法就是将这个公共的部分提取到一个独立的布局中，然后每个界面的布局文件当中来引用这个公共的布局。
    
    
    Tips:如果我们要在标签中覆写layout属性，必须要将layout_width和layout_height这两个属性也进行覆写，否则覆写效果将不会生效。
    
    
    标签是作为标签的一种辅助扩展来使用的，它的主要作用是为了防止在引用布局文件时引用文件时产生多余的布局嵌套。布局嵌套越多，解析起来就越耗时，性能就越差。因此编写布局文件时应该让嵌套的层数越少越好。
    
    
    举例：比如在LinearLayout里边使用一个布局。里边又有一个LinearLayout，那么其实就存在了多余的布局嵌套，使用merge可以解决这个问题。
    
    
    2).仅在需要时才加载布局
    
    
    某个布局当中的元素不是一起显示出来的，普通情况下只显示部分常用的元素，而那些不常用的元素只有在用户进行特定操作时才会显示出来。
    
    
    举例：填信息时不是需要全部填的，有一个添加更多字段的选项，当用户需要添加其他信息的时候，才将另外的元素显示到界面上。用VISIBLE性能表现一般，可以用ViewStub。
    
    
    ViewStub也是View的一种，但是没有大小，没有绘制功能，也不参与布局，资源消耗非常低，可以认为完全不影响性能。

![image](https://user-gold-cdn.xitu.io/2017/11/21/15fdc4cfd6f2531a?imageslim)

    tips：ViewStub所加载的布局是不可以使用标签的，因此这有可能导致加载出来出来的布局存在着多余的嵌套结构。
    
    高性能编码优化
    
    
    都是一些微优化，在性能方面看不出有什么显著的提升的。使用合适的算法和数据结构是优化程序性能的最主要手段。
    
    
    1).避免创建不必要的对象 不必要的对象我们应该避免创建：
    
    如果有需要拼接的字符串，那么可以优先考虑使用StringBuffer或者StringBuilder来进行拼接，而不是加号连接符，因为使用加号连接符会创建多余的对象，拼接的字符串越长，加号连接符的性能越低。
    
    
    当一个方法的返回值是String的时候，通常需要去判断一下这个String的作用是什么，如果明确知道调用方会将返回的String再进行拼接操作的话，可以考虑返回一个StringBuffer对象来代替，因为这样可以将一个对象的引用进行返回，而返回String的话就是创建了一个短生命周期的临时对象。
    
    
    尽可能地少创建临时对象，越少的对象意味着越少的GC操作。
    
    
    2).在没有特殊原因的情况下，尽量使用基本数据类型来代替封装数据类型，int比Integer要更加有效，其它数据类型也是一样。
    
    
    基本数据类型的数组也要优于对象数据类型的数组。另外两个平行的数组要比一个封装好的对象数组更加高效，举个例子，Foo[]和Bar[]这样的数组，使用起来要比Custom(Foo,Bar)[]这样的一个数组高效的多。
    
    
    3).静态优于抽象
    
    
    如果你并不需要访问一个对系那个中的某些字段，只是想调用它的某些方法来去完成一项通用的功能，那么可以将这个方法设置成静态方法，调用速度提升15%-20%，同时也不用为了调用这个方法去专门创建对象了，也不用担心调用这个方法后是否会改变对象的状态(静态方法无法访问非静态字段)。
    
    4).对常量使用static final修饰符
    
    static int intVal = 42;  static String strVal = "Hello, world!";  
    编译器会为上面的代码生成一个初始方法，称为方法，该方法会在定义类第一次被使用的时候调用。这个方法会将42的值赋值到intVal当中，从字符串常量表中提取一个引用赋值到strVal上。当赋值完成后，我们就可以通过字段搜寻的方式去访问具体的值了。


    final进行优化:
    
    static final int intVal = 42;  static final String strVal = "Hello, world!";  
    这样，定义类就不需要方法了，因为所有的常量都会在dex文件的初始化器当中进行初始化。当我们调用intVal时可以直接指向42的值，而调用strVal会用一种相对轻量级的字符串常量方式，而不是字段搜寻的方式。
    
    
    这种优化方式只对基本数据类型以及String类型的常量有效，对于其他数据类型的常量是无效的。
    
    
    5).使用增强型for循环语法
    
    static class Counter {      int mCount;  }  Counter[] mArray = ...  public void zero() {      int sum = 0;      for (int i = 0; i < mArray.length; ++i) {          sum += mArray[i].mCount;      }  }  public void one() {      int sum = 0;      Counter[] localArray = mArray;      int len = localArray.length;      for (int i = 0; i < len; ++i) {          sum += localArray[i].mCount;      }  }  public void two() {      int sum = 0;      for (Counter a : mArray) {          sum += a.mCount;      }  }
    zero()最慢，每次都要计算mArray的长度，one()相对快得多，two()fangfa在没有JIT(Just In Time Compiler)的设备上是运行最快的，而在有JIT的设备上运行效率和one()方法不相上下，需要注意这种写法需要JDK1.5之后才支持。
    
    Tips:ArrayList手写的循环比增强型for循环更快，其他的集合没有这种情况。
    
    因此默认情况下使用增强型for循环，而遍历ArrayList使用传统的循环方式。
    
    6).多使用系统封装好的API
    
    系统提供不了的Api完成不了我们需要的功能才应该自己去写，因为使用系统的Api很多时候比我们自己写的代码要快得多，它们的很多功能都是通过底层的汇编模式执行的。 
    
    举个例子，实现数组拷贝的功能，使用循环的方式来对数组中的每一个元素一一进行赋值当然可行，但是直接使用系统中提供的System.arraycopy()方法会让执行效率快9倍以上。
    
    7).避免在内部调用Getters/Setters方法
    
    面向对象中封装的思想是不要把类内部的字段暴露给外部，而是提供特定的方法来允许外部操作相应类的内部字段。但在Android中，字段搜寻比方法调用效率高得多，我们直接访问某个字段可能要比通过getters方法来去访问这个字段快3到7倍。
    
    
    但是编写代码还是要按照面向对象思维的，我们应该在能优化的地方进行优化，比如避免在内部调用getters/setters方法。

如何对Android 应用进行性能分析以及优化?

ddms 和 traceView

性能优化如何分析systrace

用IDE如何分析内存泄漏

Java多线程引发的性能问题，怎么解决

启动页白屏及黑屏解决？

启动太慢怎么解决

怎么保证应用启动不卡顿？

App启动崩溃异常捕捉

自定义View注意事项

渲染帧率、内存

现在下载速度很慢,试从网络协议的角度分析原因,并优化(提示：网络的5层都可以涉及)。

Https请求慢的解决办法（提示：DNS，携带数据，直接访问IP）

如何保持应用的稳定性

RecyclerView和ListView的性能对比

ListView的优化

RecycleView优化

View渲染

Bitmap如何处理大图，如一张30M的大图，如何预防OOM

java中的四种引用的区别以及使用场景

强引用置为null，会不会被回收？

**七、NDK、jni、Binder、AIDL、进程通信有关**

**AIDL的全称是什么?如何工作?能处理哪些类型的数据?**

http://blog.csdn.net/singwhatiwanna/article/details/17041691

AIDL (Android Interface Definition Language) 是一种IDL 语言，用于生成可以在Android设备上两个进程之间进行进程间通信(interprocess communication, IPC)的代码。如果在一个进程中（例如Activity）要调用另一个进程中（例如Service）对象的操作，就可以使用AIDL生成可序列化的参数。 AIDL IPC机制是面向接口的，像COM或Corba一样，但是更加轻量级。它是使用代理类在客户端和实现端传递数据。

    AIDL全称Android Interface Definition Language（Android接口描述语言）是一种接口描述语言; 编译器可以通过aidl文件生成一段代码，通过预先定义的接口达到两个进程内部通信进程跨界访问对象的目的.AIDL的IPC的机制和COM或CORBA类似, 是基于接口的，但它是轻量级的。它使用代理类在客户端和实现层间传递值. 如果要使用AIDL, 需要完成2件事情: 1. 引入AIDL的相关类.; 2. 调用aidl产生的class.
    理论上, 参数可以传递基本数据类型和String, 还有就是Bundle的派生类, 不过在Eclipse中,目前的ADT不支持Bundle做为参数,
    具体实现步骤如下:
    1、创建AIDL文件, 在这个文件里面定义接口, 该接口定义了可供客户端访问的方法和属性。
    2、编译AIDL文件, 用Ant的话, 可能需要手动, 使用Eclipse plugin的话,可以根据adil文件自动生产java文件并编译, 不需要人为介入.
    3、在Java文件中, 实现AIDL中定义的接口. 编译器会根据AIDL接口, 产生一个JAVA接口。这个接口有一个名为Stub的内部抽象类，它继承扩展了接口并实现了远程调用需要的几个方法。接下来就需要自己去实现自定义的几个接口了.
    4、向客户端提供接口ITaskBinder, 如果写的是service，扩展该Service并重载onBind ()方法来返回一个实现上述接口的类的实例。
    5、在服务器端回调客户端的函数. 前提是当客户端获取的IBinder接口的时候,要去注册回调函数, 只有这样, 服务器端才知道该调用那些函数
    AIDL语法很简单,可以用来声明一个带一个或多个方法的接口，也可以传递参数和返回值。 由于远程调用的需要, 这些参数和返回值并不是任何类型.下面是些AIDL支持的数据类型:
    
    不需要import声明的简单Java编程语言类型(int,boolean等)
    String, CharSequence不需要特殊声明
    List, Map和Parcelables类型, 这些类型内所包含的数据成员也只能是简单数据类型, String等其他比支持的类型.
    (另外: 我没尝试Parcelables, 在Eclipse+ADT下编译不过, 或许以后会有所支持).
    实现接口时有几个原则:
    1.抛出的异常不要返回给调用者. 跨进程抛异常处理是不可取的.
    2.IPC调用是同步的。如果你知道一个IPC服务需要超过几毫秒的时间才能完成地话，你应该避免在Activity的主线程中调用。也就是IPC调用会挂起应用程序导致界面失去响应. 这种情况应该考虑单起一个线程来处理.
    3.不能在AIDL接口中声明静态属性。
    IPC的调用步骤:
    声明一个接口类型的变量，该接口类型在.aidl文件中定义。
    实现ServiceConnection。
    调用ApplicationContext.bindService(),并在ServiceConnection实现中进行传递.
    在ServiceConnection.onServiceConnected()实现中，你会接收一个IBinder实例(被调用的Service). 调用
    YourInterfaceName.Stub.asInterface((IBinder)service)将参数转换为YourInterface类型。
    调用接口中定义的方法。你总要检测到DeadObjectException异常，该异常在连接断开时被抛出。它只会被远程方法抛出。
    断开连接，调用接口实例中的ApplicationContext.unbindService()
    aidl主要就是帮助我们完成了包装数据和解包的过程，并调用了transact过程，而用来传递的数据包我们就称为parcel
    
    AIDL: xxx.aidl->xxx.java,注册service
    
    用aidl定义需要被调用方法接口
    实现这些方法
    调用这些方法

说下binder序列化与反序列化的过程，与使用过程

**Android 上的 Inter-Process-Communication 跨进程通信时如何工作的**

跨进程通信主要靠Binder

https://blog.csdn.net/carson_ho/article/details/73560642

**Android IPC:Binder原理。**

IPC:

![image](https://user-gold-cdn.xitu.io/2017/10/10/a1cd0604f7807e215047053498e6daad?imageslim)

    通信，利用进程间可共享的内核内存空间来完成底层通信工作的，Client 端与 Server 端进程往往采用 ioctl 等方法跟内核空间的驱动进行交互。
    Binder 原理:
    Binder 通信采用 C/S 架构，包含 Client、Server、ServiceManager 以及 binder 驱动，其中 ServiceManager 用于管理系统中的各种服务。
架构图如下所示：

![image](http://note.youdao.com/favicon.icohttps://user-gold-cdn.xitu.io/2017/10/10/1c04668dd0b841d99f96a9f82d3eb345?imageslim)

    Binder 四个角色：
    Client 进程：使用服务的进程
    server 进程：提供服务的进程
    ServiceManager 进程：将字符型是的 Binder 名字转为 Client 中对该 Binder 的引用，使得 Client 能够通过 Binder 名字获取到 Server 中 Binder 实体的引用。
    Binder 驱动：进程间 Binder 通信的建立，Binder 在进程间的传递，Binder 引用计数管理，数据包在进程间传递和交互等一系列底层支持。
    Binder 运行机制：
    注册服务：Server 在 ServiceManager 注册服务
    获取服务：Client 从 ServiceManager 获取相应的 Service
    使用服务：Client 根据得到的 Service 信息建立与 Service 所在的 Server 进程通信的通路可与 Server 进行交互。
    
Android Binder是用来做进程通信的，Android的各个应用以及系统服务都运行在独立的进程中，它们的通信都依赖于Binder。

为什么选用Binder，在讨论这个问题之前，我们知道Android也是基于Linux内核，Linux现有的进程通信手段有以下几种：

管道：在创建时分配一个page大小的内存，缓存区大小比较有限；
消息队列：信息复制两次，额外的CPU消耗；不合适频繁或信息量大的通信；
共享内存：无须复制，共享缓冲区直接付附加到进程虚拟地址空间，速度快；但进程间的同步问题操作系统无法实现，必须各进程利用同步工具解决；
套接字：作为更通用的接口，传输效率低，主要用于不通机器或跨网络的通信；
信号量：常作为一种锁机制，防止某进程正在访问共享资源时，其他进程也访问该资源。因此，主要作为进程间以及同一进程内不同线程之间的同步手段。6. 信号: 不适用于信息交换，更适用于进程中断控制，比如非法内存访问，杀死某个进程等；
既然有现有的IPC方式，为什么重新设计一套Binder机制呢。主要是出于以上三个方面的考量：

高性能：从数据拷贝次数来看Binder只需要进行一次内存拷贝，而管道、消息队列、Socket都需要两次，共享内存不需要拷贝，Binder的性能仅次于共享内存。
稳定性：上面说到共享内存的性能优于Binder，那为什么不适用共享内存呢，因为共享内存需要处理并发同步问题，控制负责，容易出现死锁和资源竞争，稳定性较差。而Binder基于C/S架构，客户端与服务端彼此独立，稳定性较好。
安全性：我们知道Android为每个应用分配了UID，用来作为鉴别进程的重要标志，Android内部也依赖这个UID进行权限管理，包括6.0以前的固定权限和6.0以后的动态权限，传荣IPC只能由用户在数据包里填入UID/PID，这个标记完全 是在用户空间控制的，没有放在内核空间，因此有被恶意篡改的可能，因此Binder的安全性更高。


**Binder机制**

https://www.zhihu.com/question/39440766
http://gityuan.com/2016/09/04/binder-start-service/
http://baronzhang.com/blog/Android/%E5%86%99%E7%BB%99-Android-%E5%BA%94%E7%94%A8%E5%B7%A5%E7%A8%8B%E5%B8%88%E7%9A%84-Binder-%E5%8E%9F%E7%90%86%E5%89%96%E6%9E%90/
https://blog.csdn.net/prike/article/details/70195608
http://blog4jimmy.com/2018/01/356.html

老罗：
https://blog.csdn.net/luoshengyang/article/details/6618363

    在Android系统中，每一个应用程序都运行在独立的进程中，这也保证了当其中一个程序出现异常而不会影响另一个应用程序的正常运转。在许多情况下，我们activity都会与各种系统的service打交道，很显然，我们写的程序中activity与系统service肯定不是同一个进程，但是它们之间是怎样实现通信的呢？


    所以Binder是android中一种实现进程间通信（IPC）的方式之一。


1).首先，Binder分为Client和Server两个进程。


    注意，Client和Server是相对的。谁发消息，谁就是Client，谁接收消息，谁就是Server。
    
    
    举个例子，两个进程A和B之间使用Binder通信，进程A发消息给进程B，那么这时候A是Binder Client，B是Binder Server；进程B发消息给进程A，那么这时候B是Binder Client，A是Binder Server——其实这么说虽然简单了，但还是不太严谨，我们先这么理解着。


2).其次，我们看下面这个图（摘自田维术的博客），基本说明白了Binder的组成解构：

![image](https://user-gold-cdn.xitu.io/2017/11/21/15fdc4cfd565735a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

图中的IPC就是进程间通信的意思。

图中的ServiceManager，负责把Binder Server注册到一个容器中。


    有人把ServiceManager比喻成电话局，存储着每个住宅的座机电话，还是很恰当的。张三给李四打电话，拨打电话号码，会先转接到电话局，电话局的接线员查到这个电话号码的地址，因为李四的电话号码之前在电话局注册过，所以就能拨通；没注册，就会提示该号码不存在。
    
    
    对照着Android Binder机制，对着上面这图，张三就是Binder Client，李四就是Binder Server，电话局就是ServiceManager，电话局的接线员在这个过程中做了很多事情，对应着图中的Binder驱动.


3).接下来我们看Binder通信的过程，还是摘自田维术博客的一张图：

![image](https://user-gold-cdn.xitu.io/2017/11/21/15fdc4cfd6cf0739?imageslim)

注：图中的SM也就是ServiceManager。


    我们看到，Client想要直接调用Server的add方法，是不可以的，因为它们在不同的进程中，这时候就需要Binder来帮忙了。
    
    
    首先是Server在SM这个容器中注册。
    
    其次，Client想要调用Server的add方法，就需要先获取Server对象， 但是SM不会把真正的Server对象返回给Client，而是把Server的一个代理对象返回给Client，也就是Proxy。
    
    然后，Client调用Proxy的add方法，SM会帮他去调用Server的add方法，并把结果返回给Client。
    
    
    以上这3步，Binder驱动出了很多力，但我们不需要知道Binder驱动的底层实现，涉及到C++的代码了——把有限的时间去做更有意义的事情。

2.为什么android选用Binder来实现进程间通信？

    1).可靠性。在移动设备上，通常采用基于Client-Server的通信方式来实现互联网与设备间的内部通信。目前linux支持IPC包括传统的管道，System V IPC，即消息队列/共享内存/信号量，以及socket中只有socket支持Client-Server的通信方式。Android系统为开发者提供了丰富进程间通信的功能接口，媒体播放，传感器，无线传输。
    
    
    这些功能都由不同的server来管理。开发都只关心将自己应用程序的client与server的通信建立起来便可以使用这个服务。毫无疑问，如若在底层架设一套协议来实现Client-Server通信，增加了系统的复杂性。在资源有限的手机 上来实现这种复杂的环境，可靠性难以保证。
    
    
    2).传输性能。socket主要用于跨网络的进程间通信和本机上进程间的通信，但传输效率低，开销大。消息队列和管道采用存储-转发方式，即数据先从发送方缓存区拷贝到内核开辟的一块缓存区中，然后从内核缓存区拷贝到接收方缓存区，其过程至少有两次拷贝。虽然共享内存无需拷贝，但控制复杂。比较各种IPC方式的数据拷贝次数。共享内存：0次。Binder：1次。Socket/管道/消息队列：2次。
    
    
    3).安全性。Android是一个开放式的平台，所以确保应用程序安全是很重要的。Android对每一个安装应用都分配了UID/PID,其中进程的UID是可用来鉴别进程身份。传统的只能由用户在数据包里填写UID/PID，这样不可靠，容易被恶意程序利用。而我们要求由内核来添加可靠的UID。
    
    
    所以，出于可靠性、传输性、安全性。android建立了一套新的进程间通信方式。

Binder原理：

    1.在Activity和Service进行通讯的时候，用到了Binder。
        1.当属于同个进程我们可以继承Binder然后在Activity中对Service进行操作
        2.当不属于同个进程，那么要用到AIDL让系统给我们创建一个Binder，然后在Activity中对远端的Service进行操作。
    2.系统给我们生成的Binder：
        1.Stub类中有:接口方法的id，有该Binder的标识，有asInterface(IBinder)(让我们在Activity中获取实现了Binder的接口，接口的实现在Service里，同进程时候返回Stub否则返回Proxy)，有onTransact()这个方法是在不同进程的时候让Proxy在Activity进行远端调用实现Activity操作Service
        2.Proxy类是代理，在Activity端，其中有:IBinder mRemote(这就是远端的Binder)，两个接口的实现方法不过是代理最终还是要在远端的onTransact()中进行实际操作。
    3.哪一端的Binder是副本，该端就可以被另一端进行操作，因为Binder本体在定义的时候可以操作本端的东西。所以可以在Activity端传入本端的Binder，让Service端对其进行操作称为Listener，可以用RemoteCallbackList这个容器来装Listener，防止Listener因为经历过序列化而产生的问题。
    4.当Activity端向远端进行调用的时候，当前线程会挂起，当方法处理完毕才会唤醒。
    5.如果一个AIDL就用一个Service太奢侈，所以可以使用Binder池的方式，建立一个AIDL其中的方法是返回IBinder，然后根据方法中传入的参数返回具体的AIDL。
    6.IPC的方式有：Bundle（在Intent启动的时候传入，不过是一次性的），文件共享(对于SharedPreference是特例，因为其在内存中会有缓存)，使用Messenger(其底层用的也是AIDL，同理要操作哪端，就在哪端定义Messenger)，AIDL，ContentProvider(在本进程中继承实现一个ContentProvider，在增删改查方法中调用本进程的SQLite，在其他进程中查询)，Socket

请介绍一下NDK

什么是NDK库?

jni用过吗

如何在jni中注册native函数，有几种注册方式


Java如何调用c、c++语言？

**Java调用C++**

在Java中声明Native方法（即需要调用的本地方法）
编译上述 Java源文件javac（得到 .class文件） 3。 通过 javah 命令导出JNI的头文件（.h文件）
使用 Java需要交互的本地代码 实现在 Java中声明的Native方法
编译.so库文件
通过Java命令执行 Java程序，最终实现Java调用本地代码
C++调用Java

从classpath路径下搜索ClassMethod这个类，并返回该类的Class对象。
获取类的默认构造方法ID。
查找实例方法的ID。
创建该类的实例。
调用对象的实例方法。
    JNIEXPORT void JNICALL Java_com_study_jnilearn_AccessMethod_callJavaInstaceMethod  
    (JNIEnv *env, jclass cls)  
    {  
        jclass clazz = NULL;  
        jobject jobj = NULL;  
        jmethodID mid_construct = NULL;  
        jmethodID mid_instance = NULL;  
        jstring str_arg = NULL;  
        // 1、从classpath路径下搜索ClassMethod这个类，并返回该类的Class对象  
        clazz = (*env)->FindClass(env, "com/study/jnilearn/ClassMethod");  
        if (clazz == NULL) {  
            printf("找不到'com.study.jnilearn.ClassMethod'这个类");  
            return;  
        }  
    
        // 2、获取类的默认构造方法ID  
        mid_construct = (*env)->GetMethodID(env,clazz, "<init>","()V");  
        if (mid_construct == NULL) {  
            printf("找不到默认的构造方法");  
            return;  
        }  
    
        // 3、查找实例方法的ID  
        mid_instance = (*env)->GetMethodID(env, clazz, "callInstanceMethod", "(Ljava/lang/String;I)V");  
        if (mid_instance == NULL) {  
    
            return;  
        }  
    
        // 4、创建该类的实例  
        jobj = (*env)->NewObject(env,clazz,mid_construct);  
        if (jobj == NULL) {  
            printf("在com.study.jnilearn.ClassMethod类中找不到callInstanceMethod方法");  
            return;  
        }  
    
        // 5、调用对象的实例方法  
        str_arg = (*env)->NewStringUTF(env,"我是实例方法");  
        (*env)->CallVoidMethod(env,jobj,mid_instance,str_arg,200);  
    
        // 删除局部引用  
        (*env)->DeleteLocalRef(env,clazz);  
        (*env)->DeleteLocalRef(env,jobj);  
        (*env)->DeleteLocalRef(env,str_arg);  
    }  
    
八、其它高频面试题

**事件传递机制**

![image](https://upload-images.jianshu.io/upload_images/2911038-5349d6ebb32372da)

    1).Android事件分发机制的本质是要解决：点击事件由哪个对象发出，经过哪些对象，最终达到哪个对象并最终得到处理。这里的对象是指Activity、ViewGroup、View.
    
    2).Android中事件分发顺序：Activity（Window） -> ViewGroup -> View.
    
    3).事件分发过程由dispatchTouchEvent() 、onInterceptTouchEvent()和onTouchEvent()三个方法协助完成


设置Button按钮来响应点击事件事件传递情况：（如下图）

布局如下:

![image](https://user-gold-cdn.xitu.io/2017/11/21/15fdc4cfaed36b0e?imageslim)
    
    最外层：Activiy A，包含两个子View：ViewGroup B、View C
    
    中间层：ViewGroup B，包含一个子View：View C
    
    最内层：View C
    
    
    假设用户首先触摸到屏幕上View C上的某个点（如图中黄色区域），那么Action_DOWN事件就在该点产生，然后用户移动手指并最后离开屏幕。
    
    按钮点击事件:
    
    
    DOWN事件被传递给C的onTouchEvent方法，该方法返回true，表示处理这个事件;
    
    因为C正在处理这个事件，那么DOWN事件将不再往上传递给B和A的onTouchEvent()；
    
    该事件列的其他事件（Move、Up）也将传递给C的onTouchEvent();

![image](https://user-gold-cdn.xitu.io/2017/11/21/15fdc4cfd00ab478?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

![image](https://upload-images.jianshu.io/upload_images/2911038-5349d6ebb32372da)

(记住这个图的传递顺序,面试的时候能够画出来,就很详细了)


请写出四种以上你知道的设计模式（例如Android中哪里使用了观察者模式，单例模式相关），并介绍下实现原理
          
安卓子线程是否能更新UI，如果能请说明具体细节。

**广播发送和接收的原理了解吗？**

继承BroadcastReceiver，重写onReceive()方法。
通过Binder机制向ActivityManagerService注册广播。
通过Binder机制向ActivityMangerService发送广播。
ActivityManagerService查找符合相应条件的广播（IntentFilter/Permission）的BroadcastReceiver，将广播发送到BroadcastReceiver所在的消息队列中。
BroadcastReceiver所在消息队列拿到此广播后，回调它的onReceive()方法。

**View的绘制流程**

    View的绘制流程：OnMeasure()——>OnLayout()——>OnDraw()
    
    各步骤的主要工作：
    
    
    OnMeasure()：
    
    
    测量视图大小。从顶层父View到子View递归调用measure方法，measure方法又回调OnMeasure。
    
    
    OnLayout()：
    
    
    确定View位置，进行页面布局。从顶层父View向子View的递归调用view.layout方法的过程，即父View根据上一步measure子View所得到的布局大小和布局参数，将子View放在合适的位置上。
    
    
    OnDraw()：
    
    
    绘制视图:ViewRoot创建一个Canvas对象，然后调用OnDraw()。六个步骤：①、绘制视图的背景；②、保存画布的图层（Layer）；③、绘制View的内容；④、绘制View子视图，如果没有就不用；⑤、还原图层（Layer）；⑥、绘制滚动条。

事件分发中的onTouch 和onTouchEvent 有什么区别，又该如何使用？

View和ViewGroup分别有哪些事件分发相关的回调方法

View刷新机制

HttpUrlConnection 和 okhttp关系

Bitmap对象的理解

ActivityThread，AMS，WMS的工作原理

AstncTask+HttpClient 与 AsyncHttpClient有什么区别？


View绘制流程

http://www.codekk.com/blogs/detail/54cfab086c4761e5001b253f
https://www.jianshu.com/p/5a71014e7b1b

自定义控件原理

自定义View如何提供获取View属性的接口？

Android代码中实现WAP方式联网

AsyncTask机制

AsyncTask原理及不足

如何取消AsyncTask？

LeakCanary 实现原理

http://blog.csdn.net/cloud_huan/article/details/53081120

内存泄露的本质

无法回收无用的对象

你认为Rxjava的线程池与你们自己实现任务管理框架有什么区别？

处理有序数组为什么比无序数组更快 参考StackOverflow

Integer类是不是线程安全的，为什么

Android动画框架实现原理

Android各个版本API的区别

**Requestlayout，onlayout，onDraw，DrawChild区别与联系**

requestLayout()方法 ：会导致调用measure()过程 和 layout()过程 。 将会根据标志位判断是否需要ondraw

onLayout()方法(如果该View是ViewGroup对象，需要实现该方法，对每个子视图进行布局)

调用onDraw()方法绘制视图本身 (每个View都需要重载该方法，ViewGroup不需要实现该方法)

drawChild()去重新回调每个子视图的draw()方法

invalidate和postInvalidate的区别及使用

Activity-Window-View三者的差别

**如何优化自定义View**

为了加速你的view，对于频繁调用的方法，需要尽量减少不必要的代码。先从onDraw开始，需要特别注意不应该在这里做内存分配的事情，因为它会导致GC，从而导致卡顿。在初始化或者动画间隙期间做分配内存的动作。不要在动画正在执行的时候做内存分配的事情。

你还需要尽可能的减少onDraw被调用的次数，大多数时候导致onDraw都是因为调用了invalidate().因此请尽量减少调用invaildate()的次数。如果可能的话，尽量调用含有4个参数的invalidate()方法而不是没有参数的invalidate()。没有参数的invalidate会强制重绘整个view。

另外一个非常耗时的操作是请求layout。任何时候执行requestLayout()，会使得Android UI系统去遍历整个View的层级来计算出每一个view的大小。如果找到有冲突的值，它会需要重新计算好几次。另外需要尽量保持View的层级是扁平化的，这样对提高效率很有帮助。

如果你有一个复杂的UI，你应该考虑写一个自定义的ViewGroup来执行他的layout操作。与内置的view不同，自定义的view可以使得程序仅仅测量这一部分，这避免了遍历整个view的层级结构来计算大小。这个PieChart 例子展示了如何继承ViewGroup作为自定义view的一部分。PieChart 有子views，但是它从来不测量它们。而是根据他自身的layout法则，直接设置它们的大小。

低版本SDK如何实现高版本api？


计算一个view的嵌套层级

图片加载原理

统计启动时长,标准

如何保持应用的稳定性

SpareArray原理

性能优化,怎么保证应用启动不卡顿

SP是进程同步的吗?有什么方法做到同步

线程间 操作 List

App启动流程，从点击桌面开始

OSGI

描述清点击 Android Studio 的 build 按钮后发生了什么

**大体说清一个应用程序安装到手机上时发生了什么；**

APK的安装流程如下所示：

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/app/package/apk_install_structure.png)

复制APK到/data/app目录下，解压并扫描安装包。
资源管理器解析APK里的资源文件。
解析AndroidManifest文件，并在/data/data/目录下创建对应的应用数据目录。
然后对dex文件进行优化，并保存在dalvik-cache目录下。
将AndroidManifest文件解析出的四大组件信息注册到PackageManagerService中。
安装完成后，发送广播。

Android 上的 Inter-Process-Communication 跨进程通信时如何工作的；

App 是如何沙箱化，为什么要这么做；

权限管理系统（底层的权限是如何进行 grant 的）

进程和 Application 的生命周期；

系统启动流程 Zygote进程 –> SystemServer进程 –> 各种系统服务 –> 应用进程

recycleview listview 的区别,性能

数据库数据迁移问题

项目组件化的理解

Android系统为什么会设计ContentProvider，进程共享和线程安全问题

Android相关优化（如内存优化、网络优化、布局优化、电量优化、业务优化）

EventBus作用，实现方式，EventBus实现原理,代替EventBus的方式

封装view的时候怎么知道view的大小

下拉状态栏是不是影响activity的生命周期，如果在onStop的时候做了网络请求，onResume的时候怎么恢复

view渲染

逻辑地址与物理地址，为什么使用逻辑地址

RecycleView的使用，原理，RecycleView优化

App中唤醒其他进程的实现方式



**//暂未分类的面试题**

**1.软键盘顶起布局**

**2.ViewHolder有什么用？**

**3.Gradle的Flavor能否配置sourceset？**

**4.线程池核心线程数一般定义多少，为什么？**

**5.View在屏幕中的移动底层是如何实现的**

**6.setContentView都干了啥**

**7.Bitmap在decode的时候申请的内存如何复用，释放时机**

**8.注解如何实现一个findViewById**

**9.说说你对Context的理解**

**10.由A启动BActivity，A为栈内复用模式，B为标准模式，然后再次启动A或者杀死B，说说A，B的生命周期变化，为什么**

**11.区别Animation和Animator的用法，概述其原理**

**12.如何加载NDK库？如何在jni中注册native函数，有几种注册方法？**

**13.操作系统中进程和线程有什么联系和区别？系统会在什么情况下会在用户态好内核态中切换。**

**14.对于Android APP闪退，可能的原因有哪些？请针对每种情况简述分析过程。**

**15.listview跟recyclerview上拉加载的时候分别应该如何处理**

**16.不用锁如何保证int自增安全**

**17.如何自动化部署打包发包流程**

**18.微信支付宝支付调用时上层是如何封装AIDL的**

**19.如何实现一个推送，极光推送原理**

**20.图片框架选型**

**21.图片加载原理**

**22.统计启动时长**

**23.如何保持应用的稳定性**

**24.adb install 和 pms scan 的区别有哪些？**

**25.Runtime permission,如何把一个预置的app默认给它权限，不要授权。**

**26.一个图片在app中调用R.id后是如何找到的？**

**27.Android权限管理的技术是什么？**

**28.开机流程和关机流程请描述下？**

**29.Android ++ 智能指针相关使用介绍？**

**30.PowerManagerService主要做了哪些相关的操作？系统亮灭屏都有哪些流程？**


**31.MVC的情况下怎么把Activity的C和V抽离**

**32.各个网络框架之间的差异和优缺点，网络框架代替进化的原因**

**33.图片缓存框架的差异和优缺点，有没有比Glide更好的图片加载框架？**

**34.项目框架里有没有Base类，BaseActivity和BaseFragment这种封装导致的问题，以及解决方法
**

**35.为什么不推荐软引用，软引用在dvm上的垃圾回收机制和jvm上一样吗？**

**36.L6UCache的删除条件，LRU是什么意思**

**37.启动页缓存设计 白屏问题**

**38.网络图片怎么加载？Glide如何确定图片加载完毕**

**39.项目框架中对多View的支持**

**40.Arouter的原理**

**41.组件化原理，组件化中路由的实现**

**42.应用跟系统之间通信什么时候用Socket什么时候用Binder**

**43.Debug跟Release的APK的区别**

**44.对谷歌新推出的Room架构**

**45.SplashActivity中进行初始化MainActivity的参数，Splash没有初始化，AMS直接启动了MainActivity怎么办**

**46.设计一个多线程，可以同时读，读的时候不能写，写的时候不能读(读写锁)**


**47.AMS是如何管理Activity的**

**48.Hook以及插桩技术**

**49.Android的签名机制，APK包含哪些东西**

**50.点击Launcher跟点击微信支付启动微信有什么区别**

**51.没有给权限如何定位，特定机型定位失败，如何解决**

**52.Gradle生命周期**

**53.ACTION_CANCEL什么时候触发，触摸button然后滑动到外部抬起会触发点击事件吗，在+ + 滑动回去抬起会么**

**54.怎么处理嵌套View的滑动冲突问题**

**55.热修复相关的原理，框架熟悉么**

**56.gradle打包流程熟悉么**

**57.任意提问环节：其实可以问之前面试中遇到的问题：比如，多模块开发的时候不同的负责人可能会引入重复资源，相同的字符串，相同的icon等但是文件名并不一样，怎样去重？**

**58.Canvas的底层机制，绘制框架，硬件加速是什么原理，canvas lock的缓冲区是怎么回事**

**59.surfaceview， suface，surfacetexure等相关的，以及底层原理**

**60.android文件存储，各版本存储位置的权限控制的演进，外部存储，内部存储**

**61.上层业务activity和fragment的遇到什么坑？？页面展示上的一些坑和优化经验**

**62.网络请求的开源框架：OKHttp介绍，写过拦截器么**

**63.数据层有统一的管理么，数据缓存是怎么做的，http请求等有提供统一管理么？**

**64.有用什么模式么，逻辑什么的都在Activity层？怎么分离的**

**65.如果用了一些解耦的策略，怎么管理生命周期的？**

**66.有什么提高编译速度的方法？**

**67.对应用里的线程有做统一管理么？**

**68.jni的算法提供都是主线程的？是不是想问服务类的啊**

**69.边沿检测用的啥？深度学习相关的有了解么？**

**70.上线后的app性能分析检测有做么**

**71.进程间通信方式？Binder的构成有几部分？**


**72.想改变listview的高度，怎么做**


**73.Fragment的懒加载实现，参数传递与保存** 

**74.ViewPager的缓存实现 **

**75.Android里的内存缓存和磁盘缓存是怎么实现的。**

内存缓存基于LruCache实现，磁盘缓存基于DiskLruCache实现。这两个类都基于Lru算法和LinkedHashMap来实现。

LRU算法可以用一句话来描述，如下所示：

    LRU是Least Recently Used的缩写，最近最久未使用算法，从它的名字就可以看出，它的核心原则是如果一个数据在最近一段时间没有使用到，那么它在将来被 访问到的可能性也很小，则这类数据项会被优先淘汰掉。

LruCache的原理是利用LinkedHashMap持有对象的强引用，按照Lru算法进行对象淘汰。具体说来假设我们从表尾访问数据，在表头删除数据，当访问的数据项在链表中存在时，则将该数据项移动到表尾，否则在表尾新建一个数据项。当链表容量超过一定阈值，则移除表头的数据。

为什么会选择LinkedHashMap呢？

这跟LinkedHashMap的特性有关，LinkedHashMap的构造函数里有个布尔参数accessOrder，当它为true时，LinkedHashMap会以访问顺序为序排列元素，否则以插入顺序为序排序元素。

DiskLruCache与LruCache原理相似，只是多了一个journal文件来做磁盘文件的管理和迎神，如下所示：

    libcore.io.DiskLruCache
    1
    1
    1
    
    DIRTY 1517126350519
    CLEAN 1517126350519 5325928
    REMOVE 1517126350519
注：这里的缓存目录是应用的缓存目录/data/data/pckagename/cache，未root的手机可以通过以下命令进入到该目录中或者将该目录整体拷贝出来：

    //进入/data/data/pckagename/cache目录
    adb shell
    run-as com.your.packagename 
    cp /data/data/com.your.packagename/
    
    //将/data/data/pckagename目录拷贝出来
    adb backup -noapk com.your.packagename
我们来分析下这个文件的内容：

第一行：libcore.io.DiskLruCache，固定字符串。
第二行：1，DiskLruCache源码版本号。
第三行：1，App的版本号，通过open()方法传入进去的。
第四行：1，每个key对应几个文件，一般为1.
第五行：空行
第六行及后续行：缓存操作记录。
第六行及后续行表示缓存操作记录，关于操作记录，我们需要了解以下三点：

DIRTY 表示一个entry正在被写入。写入分两种情况，如果成功会紧接着写入一行CLEAN的记录；如果失败，会增加一行REMOVE记录。注意单独只有DIRTY状态的记录是非法的。
当手动调用remove(key)方法的时候也会写入一条REMOVE记录。
READ就是说明有一次读取的记录。
CLEAN的后面还记录了文件的长度，注意可能会一个key对应多个文件，那么就会有多个数字。

**76.RecyclerView和ListView有什么区别？局部刷新？前者使用时多重type场景下怎么避免滑动卡顿。懒加载怎么实现，怎么优化滑动体验。**

**77.开放问题：如果提高启动速度，设计一个延迟加载框架或者sdk的方法和注意的问题。**

**78.Scroller有什么方法，怎么使用的。**

**79.webwiew了解？怎么实现和javascript的通信？相互双方的通信。@JavascriptInterface在？版本有bug，除了这个还有其他调用android方法的方案吗？**


**80.android api层的源码熟悉哪些？解释一下**

**81.线程sleep对消息的影响**

**82.如果在当前线程内使用Handler postdelayed 两个消息，一个延迟5s，一个延迟10s，然后使当前线程sleep 5秒，以上消息的执行时间会如何变化？**

答：照常执行

扩展：sleep时间<=5 对两个消息无影响，5< sleep时间 <=10 对第一个消息有影响，第一个消息会延迟到sleep后执行，sleep时间>10 对两个时间都有影响，都会延迟到sleep后执行。

**83.对Dalvik、ART虚拟机有什么了解？**

**84.虚拟机原理，如何自己设计一个虚拟机(内存管理，类加载，双亲委派)**

**85.Ubuntu编译安卓系统**

**86.系统启动流程是什么？（提示：Zygote进程 –> SystemServer进程 –> 各种系统服务 –> 应用进程）**

**87.大体说清一个应用程序安装到手机上时发生了什么**

**88.简述Activity启动全部过程**

**89.App启动流程，从点击桌面开始**

**90.逻辑地址与物理地址，为什么使用逻辑地址？**

**91.Android中进程内存的分配，能不能自己分配定额内存？**

**92.如何保证一个后台服务不被杀死？（相同问题：如何保证service在后台不被kill？）比较省电的方式是什么？**

一、onStartCommand方法，返回START_STICKY

START_STICKY 在运行onStartCommand后service进程被kill后，那将保留在开始状态，但是不保留那些传入的intent。不久后service就会再次尝试重新创建，因为保留在开始状态，在创建 service后将保证调用onstartCommand。如果没有传递任何开始命令给service，那将获取到null的intent。

START_NOT_STICKY 在运行onStartCommand后service进程被kill后，并且没有新的intent传递给它。Service将移出开始状态，并且直到新的明显的方法（startService）调用才重新创建。因为如果没有传递任何未决定的intent那么service是不会启动，也就是期间onstartCommand不会接收到任何null的intent。

START_REDELIVER_INTENT 在运行onStartCommand后service进程被kill后，系统将会再次启动service，并传入最后一个intent给onstartCommand。直到调用stopSelf(int)才停止传递intent。如果在被kill后还有未处理好的intent，那被kill后服务还是会自动启动。因此onstartCommand不会接收到任何null的intent。

二、提升service进程优先级

Android中的进程是托管的，当系统进程空间紧张的时候，会依照优先级自动进行进程的回收。Android将进程分为6个等级,它们按优先级顺序由高到低依次是:

前台进程( FOREGROUND_APP)
可视进程(VISIBLE_APP )
次要服务进程(SECONDARY_SERVER )
后台进程 (HIDDEN_APP)
内容供应节点(CONTENT_PROVIDER)
空进程(EMPTY_APP)
当service运行在低内存的环境时，将会kill掉一些存在的进程。因此进程的优先级将会很重要，可以使用startForeground 将service放到前台状态。这样在低内存时被kill的几率会低一些。

三、onDestroy方法里重启service

service +broadcast 方式，就是当service走ondestory的时候，发送一个自定义的广播，当收到广播的时候，重新启动service；

四、Application加上Persistent属性

五、监听系统广播判断Service状态

通过系统的一些广播，比如：手机重启、界面唤醒、应用状态改变等等监听并捕获到，然后判断我们的Service是否还存活，别忘记加权限啊。


**93.Activity的启动模式有哪些？栈里是A-B-C，先想直接到A，BC都清理掉，有几种方法可以做到？这几种方法产生的结果是有几个A的实例？**

**94.有什么工具可以看到Activity栈信息么？多个栈话，有方法分别得到各个栈的Activity列表么**

**95.都熟悉哪些命令？知道怎么用命令启动一个Activity么?**
    
**96.Android性能优化方法**

http://www.trinea.cn/android/performance/



    布局优化：尽量减少布局文件的层级
    尽量减少布局文件的层级
    删除布局中无用的控件和层次，有选择的使用性能较高的 ViewGroup
    采用标签，ViewStub，布局重用可降低布局的层级（ViewStub提供了按需加载的功能，当需要时才会将 ViewStub 中布局加载到内存，提高了初始化效率）
    避免过度绘制
    
    绘制优化：View 的 onDraw() 方法避免执行大量操作。
    onDraw() 中不要创建新的布局对象。
    onDraw() 中不做耗时任务，大量循环十分抢占 cpu 时间片，造成 View 的绘制不流畅。
    
    内存泄漏优化
    避免写出内存泄漏代码
    通过分析工具（MAT、LeakCannary）找出潜在的内存泄漏方法，然后解决。
    导致内存泄漏的原因:
    集合类的泄漏
    单例 / 静态变量造成内存泄漏
    匿名内部类/非静态内部类造成内存泄漏
    资源未关闭
    
    响应速度的优化：避免在主线程做耗时的操作
    
    ListView / RecyclerView优化：
    使用 ViewHolder 模式来提高效率
    异步加载：耗时操作放在异步线程
    滑动时停止加载和分页加载
    
    线程优化：采用线程池，避免程序中存在大量 Thread 。
    
    其他性能优化：
    不要过多的创建对象。
    不要过多使用枚举类，枚举占用内存空间要比整型大。
    常量使用 static final 修饰。
    使用 Android 特有的数据结构。
    适当采用软引用和弱引用。
    采用内存缓存和磁盘缓存。
    尽量采用静态内部类，可避免潜在由于内部类导致的内存泄漏。

**97.Android中软引用与弱引用的应用场景。**

Java 引用类型分类：

![image](https://user-gold-cdn.xitu.io/2017/10/10/29c884389e96babb2759b95014628aae?imageslim)

    在 Android 应用的开发中，为了防止内存溢出，在处理一些占用内存大而且声明周期较长的对象时候，可以尽量应用软引用和弱引用技术。
    软/弱引用可以和一个引用队列（ReferenceQueue）联合使用，如果软引用所引用的对象被垃圾回收器回收，Java 虚拟机就会把这个软引用加入到与之关联的引用队列中。利用这个队列可以得知被回收的软/弱引用的对象列表，从而为缓冲器清除已失效的软 / 弱引用。
    如果只是想避免 OOM 异常的发生，则可以使用软引用。如果对于应用的性能更在意，想尽快回收一些占用内存比较大的对象，则可以使用弱引用。
    可以根据对象是否经常使用来判断选择软引用还是弱引用。如果该对象可能会经常使用的，就尽量用软引用。如果该对象不被使用的可能性更大些，就可以用弱引用。


    
**Android长连接，怎么处理心跳机制。**

    长连接：长连接是建立连接之后, 不主动断开. 双方互相发送数据, 发完了也不主动断开连接, 之后有需要发送的数据就继续通过这个连接发送.
    心跳包：其实主要是为了防止NAT超时，客户端隔一段时间就主动发一个数据，探测连接是否断开
    服务器处理心跳包：假如客户端心跳间隔是固定的, 那么服务器在连接闲置超过这个时间还没收到心跳时, 可以认为对方掉线, 关闭连接. 如果客户端心跳会动态改变,  应当设置一个最大值, 超过这个最大值才认为对方掉线. 还有一种情况就是服务器通过TCP连接主动给客户端发消息出现写超时, 可以直接认为对方掉线.
    
**Zygote的启动过程**

    在 Android 系统里面，zygote 是一个进程的名字。Android 是基于 Linux System 的，当你的手机开机的时候，Linux 的内核加载完成之后就会启动一个叫 “init“ 的进程。在 Linux System 里面，所有的进程都是由 init 进程 fork 出来的，我们的zygote进程也不例外。

    
**安卓view绘制机制和加载过程，请详细说下整个流程**

    1.ViewRootImpl会调用performTraversals(),其内部会调用performMeasure()、performLayout、performDraw()。
    2.performMeasure()会调用最外层的ViewGroup的measure()-->onMeasure(),ViewGroup的onMeasure()是抽象方法，但其提供了measureChildren()，这之中会遍历子View然后循环调用measureChild()这之中会用getChildMeasureSpec()+父View的MeasureSpec+子View的LayoutParam一起获取本View的MeasureSpec，然后调用子View的measure()到View的onMeasure()-->setMeasureDimension(getDefaultSize(),getDefaultSize()),getDefaultSize()默认返回measureSpec的测量数值，所以继承View进行自定义的wrap_content需要重写。
    3.performLayout()会调用最外层的ViewGroup的layout(l,t,r,b),本View在其中使用setFrame()设置本View的四个顶点位置。在onLayout(抽象方法)中确定子View的位置，如LinearLayout会遍历子View，循环调用setChildFrame()-->子View.layout()。
    4.performDraw()会调用最外层ViewGroup的draw():其中会先后调用background.draw()(绘制背景)、onDraw()(绘制自己)、dispatchDraw()(绘制子View)、onDrawScrollBars()(绘制装饰)。
    5.MeasureSpec由2位SpecMode(UNSPECIFIED、EXACTLY(对应精确值和match_parent)、AT_MOST(对应warp_content))和30位SpecSize组成一个int,DecorView的MeasureSpec由窗口大小和其LayoutParams决定，其他View由父View的MeasureSpec和本View的LayoutParams决定。ViewGroup中有getChildMeasureSpec()来获取子View的MeasureSpec。
    6.三种方式获取measure()后的宽高：
    1.Activity#onWindowFocusChange()中调用获取
    2.view.post(Runnable)将获取的代码投递到消息队列的尾部。
    3.ViewTreeObservable.
    
**activty的加载过程 请详细介绍下:**

    1.Activity中最终到startActivityForResult()（mMainThread.getApplicationThread()传入了一个ApplicationThread检查APT）
    ->Instrumentation#execStartActivity()和checkStartActivityResult()(这是在启动了Activity之后判断Activity是否启动成功，例如没有在AM中注册那么就会报错)
    ->ActivityManagerNative.getDefault().startActivity()（类似AIDL，实现了IAM，实际是由远端的AMS实现startActivity()）
    ->ActivityStackSupervisor#startActivityMayWait()
    ->ActivityStack#resumeTopActivityInnerLocked
    ->ActivityStackSupervisor#realStartActivityLocked()（在这里调用APT的scheduleLaunchActivity,也是AIDL，不过是在远端调起了本进程Application线程）
    ->ApplicationThread#scheduleLaunchActivity()（这是本进程的一个线程，用于作为Service端来接受AMS client端的调起）
    ->ActivityThread#handleLaunchActivity()（接收内部类H的消息，ApplicationThread线程发送LAUNCH_ACTIVITY消息给H）
    ->最终在ActivityThread#performLaunchActivity()中实现Activity的启动完成了以下几件事：
    2.从传入的ActivityClientRecord中获取待启动的Activity的组件信息
    3.创建类加载器，使用Instrumentation#newActivity()加载Activity对象
    4.调用LoadedApk.makeApplication方法尝试创建Application，由于单例所以不会重复创建。
    5.创建Context的实现类ContextImpl对象，并通过Activity#attach()完成数据初始化和Context建立联系，因为Activity是Context的桥接类，
    最后就是创建和关联window，让Window接收的事件传给Activity，在Window的创建过程中会调用ViewRootImpl的performTraversals()初始化View。
    6.Instrumentation#callActivityOnCreate()->Activity#performCreate()->Activity#onCreate().onCreate()中会通过Activity#setContentView()调用PhoneWindow的setContentView()
    更新界面。    

**android重要术语解释**

    1.ActivityManagerServices，简称AMS，服务端对象，负责系统中所有Activity的生命周期
    2.ActivityThread，App的真正入口。当开启App之后，会调用main()开始运行，开启消息循环队列，这就是传说中的UI线程或者叫主线程。与ActivityManagerServices配合，一起完成Activity的管理工作
    3.ApplicationThread，用来实现ActivityManagerService与ActivityThread之间的交互。在ActivityManagerService需要管理相关Application中的Activity的生命周期时，通过ApplicationThread的代理对象与ActivityThread通讯。
    4.ApplicationThreadProxy，是ApplicationThread在服务器端的代理，负责和客户端的ApplicationThread通讯。AMS就是通过该代理与ActivityThread进行通信的。
    5.Instrumentation，每一个应用程序只有一个Instrumentation对象，每个Activity内都有一个对该对象的引用。Instrumentation可以理解为应用进程的管家，ActivityThread要创建或暂停某个Activity时，都需要通过Instrumentation来进行具体的操作。
    6.ActivityStack，Activity在AMS的栈管理，用来记录已经启动的Activity的先后关系，状态信息等。通过ActivityStack决定是否需要启动新的进程。
    7.ActivityRecord，ActivityStack的管理对象，每个Activity在AMS对应一个ActivityRecord，来记录Activity的状态以及其他的管理信息。其实就是服务器端的Activity对象的映像。
    8.TaskRecord，AMS抽象出来的一个“任务”的概念，是记录ActivityRecord的栈，一个“Task”包含若干个ActivityRecord。AMS用TaskRecord确保Activity启动和退出的顺序。如果你清楚Activity的4种launchMode，那么对这个概念应该不陌生。
    
**理解Window和WindowManager**

    1.Window用于显示View和接收各种事件，Window有三种类型：应用Window(每个Activity对应一个Window)、子Window(不能单独存在，附属于特定Window)、系统window(Toast和状态栏)
    2.Window分层级，应用Window在1-99、子Window在1000-1999、系统Window在2000-2999.WindowManager提供了增删改View三个功能。
    3.Window是个抽象概念：每一个Window对应着一个View和ViewRootImpl，Window通过ViewRootImpl来和View建立联系，View是Window存在的实体，只能通过WindowManager来访问Window。
    4.WindowManager的实现是WindowManagerImpl其再委托给WindowManagerGlobal来对Window进行操作，其中有四个List分别储存对应的View、ViewRootImpl、WindowManger.LayoutParams和正在被删除的View
    5.Window的实体是存在于远端的WindowMangerService中，所以增删改Window在本端是修改上面的几个List然后通过ViewRootImpl重绘View，通过WindowSession(每个应用一个)在远端修改Window。
    6.Activity创建Window：Activity会在attach()中创建Window并设置其回调(onAttachedToWindow()、dispatchTouchEvent()),Activity的Window是由Policy类创建PhoneWindow实现的。然后通过Activity#setContentView()调用PhoneWindow的setContentView。


**CrashHandler实现原理**

    获取app crash的信息保存在本地然后在下一次打开app的时候发送到服务器。

**动态权限适配方案，权限组的概念**


**启动一个程序，可以主界面点击图标进入，也可以从一个程序中跳转过去，二者有什么区别？** 

    是因为启动程序（主界面也是一个app），发现了在这个程序中存在一个设置为
    
    <category android:name="android.intent.category.LAUNCHER" />
    的activity,
    所以这个launcher会把icon提出来，放在主界面上。当用户点击icon的时候，发出一个Intent：
    
    Intent intent = mActivity.getPackageManager().getLaunchIntentForPackage(packageName);
    mActivity.startActivity(intent);   
    跳过去可以跳到任意允许的页面，如一个程序可以下载，那么真正下载的页面可能不是首页（也有可能是首页），这时还是构造一个Intent，startActivity.
    这个intent中的action可能有多种view,download都有可能。系统会根据第三方程序向系统注册的功能，为你的Intent选择可以打开的程序或者页面。所以唯一的一点
    不同的是从icon的点击启动的intent的action是相对单一的，从程序中跳转或者启动可能样式更多一些。本质是相同的。


**FC(Force Close)**

什么时候会出现

Error
OOM，内存溢出
StackOverFlowError
Runtime,比如说空指针异常
解决的办法

注意内存的使用和管理
使用Thread.UncaughtExceptionHandler接口

**界面优化** 

    太多重叠的背景(overdraw)
    
    这个问题其实最容易解决，建议就是检查你在布局和代码中设置的背景，有些背景是隐藏在底下的，它永远不可能显示出来，这种没必要的背景一定要移除，因为它很可能会严重影响到app的性能。如果采用的是selector的背景，将normal状态的color设置为”@android:color/transparent”,也同样可以解决问题。
    
    太多重叠的View
    
    第一个建议是 ：使用ViewStub来加载一些不常用的布局，它是一个轻量级且默认是不可见的视图，可以动态的加载一个布局，只要你用到这个重叠着的View的时候才加载，推迟加载的时间。
    
    第二个建议是：如果使用了类似Viewpager＋Fragment这样的组合或者有多个Fragment在一个界面上，需要控制Fragment的显示和隐藏，尽量使用动态的Inflation view，它的性能要比SetVisibility好。
    
    复杂的Layout层级
    
    这里的建议比较多一些，首先推荐使用Android提供的布局工具Hierarchy Viewer来检查和优化布局。第一个建议是：如果嵌套的线性布局加深了布局层次，可以使用相对布局来取代。第二个建议是：用标签来合并布局。第三个建议是：用标签来重用布局，抽取通用的布局可以让布局的逻辑更清晰明了。记住，这些建议的最终目的都是使得你的Layout在Hierarchy Viewer里变得宽而浅，而不是窄而深。
    
    总结：可以考虑多使用merge和include，ViewStub。尽量使布局浅平，根布局尽量少使用RelactivityLayout,因为RelactivityLayout每次需要测量2次。

**内存优化** 

    核心思想：减少内存使用，能不new的不new，能少分配的少分配。因为分配更多的内存就意味着发生更多的GC，每次触发GC都会占用CPU时间，影响性能。
    
    集合优化：Android提供了一系列优化过后的数据集合工具类，如SparseArray、SparseBooleanArray、LongSparseArray，使用这些API可以让我们的程序更加高效。HashMap工具类会相对比较低效，因为它需要为每一个键值对都提供一个对象入口，而SparseArray就避免掉了基本数据类型转换成对象数据类型的时间。
    Bitmap优化：读取一个Bitmap图片的时候，千万不要去加载不需要的分辨率。可以压缩图片等操作。
    尽量避免使用依赖注入框架。
    避免创作不必要的对象：字符串拼接使用StringBuffer，StringBuilder。
    onDraw方法里面不要执行对象的创建.
    重写onTrimMemory，根据传入的参数，进行内存释放。
    使用static final 优化成员变量。
    
**移动端获取网络数据优化的几个点**

    连接复用：节省连接建立时间，如开启 keep-alive。
    对于Android来说默认情况下HttpURLConnection和HttpClient都开启了keep-alive。只是2.2之前HttpURLConnection存在影响连接池的Bug，具体可见：Android HttpURLConnection及HttpClient选择
    
    请求合并：即将多个请求合并为一个进行请求，比较常见的就是网页中的CSS Image Sprites。如果某个页面内请求过多，也可以考虑做一定的请求合并。
    
    减少请求数据的大小：对于post请求，body可以做gzip压缩的，header也可以做数据压缩(不过只支持http 2.0)。
    返回数据的body也可以做gzip压缩，body数据体积可以缩小到原来的30%左右。（也可以考虑压缩返回的json数据的key数据的体积，尤其是针对返回数据格式变化不大的情况，支付宝聊天返回的数据用到了）
    根据用户的当前的网络质量来判断下载什么质量的图片（电商用的比较多）

**Android系统启动过程，App启动过程** 
    
    从桌面点击到activity启动的过程
    
    1.Launcher线程捕获onclick的点击事件，调用Launcher.startActivitySafely,进一步调用Launcher.startActivity，最后调用父类Activity的startActivity。
    
    2.Activity和ActivityManagerService交互，引入Instrumentation，将启动请求交给Instrumentation，调用Instrumentation.execStartActivity。
    
    3.调用ActivityManagerService的startActivity方法，这里做了进程切换（具体过程请查看源码）。
    
    4.开启Activity，调用onCreate方法

**Android系统的架构**

![image](https://upload-images.jianshu.io/upload_images/2893137-1047c70c15c1589b.png?imageMogr2/auto-orient)

    android的系统架构和其操作系统一样，采用了分层的架构。从架构图看，android分为四个层，从高层到低层分别是应用程序层、应用程序框架层、系统运行库层和linux核心层。
    　　1.应用程序 
    　　Android会同一系列核心应用程序包一起发布，该应用程序包包括email客户端，SMS短消息程序，日历，地图，浏览器，联系人管理程序等。所有的应用程序都是使用JAVA语言编写的。
    　　2.应用程序框架 
    　　开发人员也可以完全访问核心应用程序所使用的API框架。该应用程序的架构设计简化了组件的重用;任何一个应用程序都可以发布它的功能块并且任何其它的应用程序都可以使用其所发布的功能块(不过得遵循框架的安全性限制)。同样，该应用程序重用机制也使用户可以方便的替换程序组件。
    　　隐藏在每个应用后面的是一系列的服务和系统, 其中包括;
    　　* 丰富而又可扩展的视图(Views)，可以用来构建应用程序， 它包括列表(lists)，网格(grids)，文本框(text boxes)，按钮(buttons)， 甚至可嵌入的web浏览器。
    　　* 内容提供器(Content Providers)使得应用程序可以访问另一个应用程序的数据(如联系人数据库)， 或者共享它们自己的数据
    　　* 资源管理器(Resource Manager)提供 非代码资源的访问，如本地字符串，图形，和布局文件( layout files )。
    　　* 通知管理器 (Notification Manager) 使得应用程序可以在状态栏中显示自定义的提示信息。
    　　* 活动管理器( Activity Manager) 用来管理应用程序生命周期并提供常用的导航回退功能。
    　　有关更多的细节和怎样从头写一个应用程序，请参考 如何编写一个 Android 应用程序.
    3.系统运行库 
    　　1)程序库
    　　Android 包含一些C/C++库，这些库能被Android系统中不同的组件使用。它们通过 Android 应用程序框架为开发者提供服务。以下是一些核心库：
    　　系统 C 库 - 一个从 BSD 继承来的标准 C 系统函数库( libc )， 它是专门为基于 embedded linux 的设备定制的。
    　　 媒体库 - 基于 PacketVideo OpenCORE;该库支持多种常用的音频、视频格式回放和录制，同时支持静态图像文件。编码格式包括MPEG4, H.264, MP3, AAC, AMR, JPG, PNG 。
    　　 Surface Manager - 对显示子系统的管理，并且为多个应用程序提 供了2D和3D图层的无缝融合。
    　　 LibWebCore - 一个最新的web浏览器引擎用，支持Android浏览器和一个可嵌入的web视图。
    　　SGL- 底层的2D图形引擎
    　　 3D libraries - 基于OpenGL ES 1.0 APIs实现;该库可以使用硬件 3D加速(如果可用)或者使用高度优化的3D软加速。
    　　 FreeType -位图(bitmap)和矢量(vector)字体显示。
    　　* SQLite - 一个对于所有应用程序可用，功能强劲的轻型关系型数据库引擎。
    　　2)Android 运行库
    　　Android 包括了一个核心库，该核心库提供了JAVA编程语言核心库的大多数功能。
    　　每一个Android应用程序都在它自己的进程中运行，都拥有一个独立的Dalvik虚拟机实例。Dalvik被设计成一个设备可以同时高效地运行多个虚拟系统。 Dalvik虚拟机执行(.dex)的Dalvik可执行文件，该格式文件针对小内存使用做了优化。同时虚拟机是基于寄存器的，所有的类都经由JAVA编译器编译，然后通过SDK中 的 “dx” 工具转化成.dex格式由虚拟机执行。
    　　Dalvik虚拟机依赖于linux内核的一些功能，比如线程机制和底层内存管理机制。
    　　4.Linux 内核 
    Android 的核心系统服务依赖于 Linux 2.6 内核，如安全性，内存管理，进程管理， 网络协议栈和驱动模型。 Linux 内核也同时作为硬件和软件栈之间的抽象层。
    
![image](https://raw.githubusercontent.com/BeesAndroid/BeesAndroid/master/art/android_system_structure.png)

从上到下依次分为四层：

Android应用框架层
Java系统框架层
C++系统框架层
Linux内核层

安卓view绘制机制和加载过程，请详细说下整个流程

activty的加载过程 请详细介绍下（不是生命周期切记）

安卓采用自动垃圾回收机制，请说下安卓内存管理的原理

说下安卓虚拟机和java虚拟机的原理和不同点 

**[RxJava中map和flatmap操作符的区别及底层实现](https://www.jianshu.com/p/af13a8278a05)**

**[RecyclerView与ListView缓存机制的不同](https://segmentfault.com/a/1190000007331249)**


**ButterKnife原理 **

ButterKnife对性能的影响很小，因为没有使用使用反射，而是使用的Annotation Processing Tool(APT),注解处理器，javac中用于编译时扫描和解析Java注解的工具。在编译阶段执行的，它的原理就是读入Java源代码，解析注解，然后生成新的Java代码。新生成的Java代码最后被编译成Java字节码，注解解析器不能改变读入的Java 类，比如不能加入或删除Java方法。

**Activity/Window/View三者的差别,fragment的特点**

    Activity像一个工匠（控制单元），Window像窗户（承载模型），View像窗花（显示视图） LayoutInflater像剪刀，Xml配置像窗花图纸。
    
    在Activity中调用attach，创建了一个Window
    创建的window是其子类PhoneWindow，在attach中创建PhoneWindow
    在Activity中调用setContentView(R.layout.xxx)
    其中实际上是调用的getWindow().setContentView()
    调用PhoneWindow中的setContentView方法
    创建ParentView：
    作为ViewGroup的子类，实际是创建的DecorView(作为FramLayout的子类）
    将指定的R.layout.xxx进行填充
    通过布局填充器进行填充【其中的parent指的就是DecorView】
    调用到ViewGroup
    调用ViewGroup的removeAllView()，先将所有的view移除掉
    添加新的view：addView()

**JVM 和Dalvik虚拟机的区别**

    JVM:
    .java -> javac -> .class -> jar -> .jar
    架构: 堆和栈的架构.
    DVM:
    .java -> javac -> .class -> dx.bat -> .dex
    架构: 寄存器(cpu上的一块高速缓存)

**事件传递机制**

    当手指触摸到屏幕时，系统就会调用相应View的onTouchEvent，并传入一系列的action。
    
    dispatchTouchEvent的执行顺序为：
    
    首先触发ACTIVITY的dispatchTouchEvent,然后触发ACTIVITY的onInterceptTouchEvent.
    然后触发LAYOUT的dispatchTouchEvent，然后触发LAYOUT的onInterceptTouchEvent
    这就解释了重写ViewGroup时必须调用super.dispatchTouchEvent();
    
    (1)dispatchTouchEvent:
    
    此方法一般用于初步处理事件，因为动作是由此分发，所以通常会调用super.dispatchTouchEvent。这样就会继续调用onInterceptTouchEvent，再由onInterceptTouchEvent决定事件流向。
    
    (2)onInterceptTouchEvent:
    
    若返回值为true事件会传递到自己的onTouchEvent();若返回值为false传递到下一个View的dispatchTouchEvent();
    
    (3)onTouchEvent():
    
    若返回值为true，事件由自己消耗，后续动作让其处理；若返回值为false，自己不消耗事件了，向上返回让其他的父View的onTouchEvent接受处理
    
    三大方法关系的伪代码：如果当前View拦截事件，就交给自己的onTouchEvent去处理，否则就丢给子View继续走相同的流程。
    
    public boolean dispatchTouchEvent(MotionEvent ev)
    {
        boolean consume = false;
        if(onInterceptTouchEvent(ev))
        {
            consume = onTouchEvent(ev);
        }
        else
        {
            consume = child.dispatchTouchEvent(ev);
        }
        return consume;
    }
    onTouchEvent的传递：
    
    当有多个层级的View时，在父层级允许的情况下，这个action会一直传递直到遇到最深层的View。所以touch事件最先调用的是最底层View的onTouchEvent，如果View的onTouchEvent接收到某个touch action并做了相应处理，最后有两种返回方式return true和return false；return true会告诉系统当前的View需要处理这次的touch事件，以后的系统发出的ACTION_MOVE,ACTION_UP还是需要继续监听并接收的，并且这次的action已经被处理掉了，父层的View是不可能触发onTouchEvent的了。所以每一个action最多只能有一个onTouchEvent接口返回true。如果返回false，便会通知系统，当前View不关心这一次的touch事件，此时这个action会传向父级，调用父级View的onTouchEvent。但是这一次的touch事件之后发出任何action，该View都不在接受，onTouchEvent在这一次的touch事件中再也不会触发，也就是说一旦View返回false，那么之后的ACTION_MOVE,ACTION_UP等ACTION就不会在传入这个View,但是下一次touch事件的action还是会传进来的。
    
    父层的onInterceptTouchEvent
    
    前面说了底层的View能够接收到这次的事件有一个前提条件：在父层允许的情况下。假设不改变父层级的dispatch方法，在系统调用底层onTouchEvent之前会调用父View的onInterceptTouchEvent方法判断，父层View是否要截获本次touch事件之后的action。如果onInterceptTouchEvent返回了true，那么本次touch事件之后的所有action都不会向深层的View传递，统统都会传给父层View的onTouchEvent，就是说父层已经截获了这次touch事件，之后的action也不必询问onInterceptTouchEvent，在这次的touch事件之后发出的action时onInterceptTouchEvent不会再被调用，直到下一次touch事件的来临。如果onInterceptTouchEvent返回false，那么本次action将发送给更深层的View，并且之后的每一次action都会询问父层的onInterceptTouchEvent需不需要截获本次touch事件。只有ViewGroup才有onInterceptTouchEvent方法，因为一个普通的View肯定是位于最深层的View，touch能够传到这里已经是最后一站了，肯定会调用View的onTouchEvent()。
    
    底层View的getParent().requestDisallowInterceptTouchEvent(true)
    
    对于底层的View来说，有一种方法可以阻止父层的View获取touch事件，就是调用getParent().requestDisallowInterceptTouchEvent(true)方法。一旦底层View收到touch的action后调用这个方法那么父层View就不会再调用onInterceptTouchEvent了，也无法截获以后的action（如果父层ViewGroup和最底层View需要截获不同焦点，或不同手势的touch，不能使用这个写死）。
    
http://gityuan.com/2015/09/19/android-touch/
https://www.jianshu.com/p/84b2e0038080
http://hanhailong.com/2015/09/24/Android-%E4%B8%89%E5%BC%A0%E5%9B%BE%E6%90%9E%E5%AE%9ATouch%E4%BA%8B%E4%BB%B6%E4%BC%A0%E9%80%92%E6%9C%BA%E5%88%B6/

**ART和Dalvik区别**

art上应用启动快，运行快，但是耗费更多存储空间，安装时间长，总的来说ART的功效就是”空间换时间”。

ART: Ahead of Time
Dalvik: Just in Time

什么是Dalvik：Dalvik是Google公司自己设计用于Android平台的Java虚拟机。Dalvik虚拟机是Google等厂商合作开发的Android移动设备平台的核心组成部分之一，它可以支持已转换为.dex(即Dalvik Executable)格式的Java应用程序的运行，.dex格式是专为Dalvik应用设计的一种压缩格式，适合内存和处理器速度有限的系统。Dalvik经过优化，允许在有限的内存中同时运行多个虚拟机的实例，并且每一个Dalvik应用作为独立的Linux进程执行。独立的进程可以防止在虚拟机崩溃的时候所有程序都被关闭。

什么是ART:Android操作系统已经成熟，Google的Android团队开始将注意力转向一些底层组件，其中之一是负责应用程序运行的Dalvik运行时。Google开发者已经花了两年时间开发更快执行效率更高更省电的替代ART运行时。ART代表Android Runtime,其处理应用程序执行的方式完全不同于Dalvik，Dalvik是依靠一个Just-In-Time(JIT)编译器去解释字节码。开发者编译后的应用代码需要通过一个解释器在用户的设备上运行，这一机制并不高效，但让应用能更容易在不同硬件和架构上运行。ART则完全改变了这套做法，在应用安装的时候就预编译字节码到机器语言，这一机制叫Ahead-Of-Time(AOT)编译。在移除解释代码这一过程后，应用程序执行将更有效率，启动更快。

ART优点：

系统性能的显著提升
应用启动更快、运行更快、体验更流畅、触感反馈更及时。
更长的电池续航能力
支持更低的硬件

ART缺点：
更大的存储空间占用，可能会增加10%-20%
更长的应用安装时间

**Scroller原理**

    Scroller执行流程里面的三个核心方法
    
    mScroller.startScroll()
    mScroller.computeScrollOffset()
    view.computeScroll()
    1、在mScroller.startScroll()中为滑动做了一些初始化准备，比如：起始坐标，滑动的距离和方向以及持续时间(有默认值)，动画开始时间等。
    
    2、mScroller.computeScrollOffset()方法主要是根据当前已经消逝的时间来计算当前的坐标点。因为在mScroller.startScroll()中设置了动画时间，那么在computeScrollOffset()方法中依据已经消逝的时间就很容易得到当前时刻应该所处的位置并将其保存在变量mCurrX和mCurrY中。除此之外该方法还可判断动画是否已经结束。

**Android中Java和JavaScript交互**

    webView.addJavaScriptInterface(new Object(){xxx}, "xxx");
    1
    答案：可以使用WebView控件执行JavaScript脚本，并且可以在JavaScript中执行Java代码。要想让WebView控件执行JavaScript，需要调用WebSettings.setJavaScriptEnabled方法，代码如下：
    
    WebView webView = (WebView)findViewById(R.id.webview);
    WebSettings webSettings = webView.getSettings();
    //设置WebView支持JavaScript
    webSettings.setJavaScriptEnabled(true);
    webView.setWebChromeClient(new WebChromeClient());
    
    JavaScript调用Java方法需要使用WebView.addJavascriptInterface方法设置JavaScript调用的Java方法，代码如下：
    
    webView.addJavascriptInterface(new Object()
    {
        //JavaScript调用的方法
        public String process(String value)
        {
            //处理代码
            return result;
        }
    }, "demo");       //demo是Java对象映射到JavaScript中的对象名
    
    可以使用下面的JavaScript代码调用process方法，代码如下：
    
    <script language="javascript">
        function search()
        {
            //调用searchWord方法
            result.innerHTML = "<font color='red'>" + window.demo.process('data') + "</font>";
        }

**SurfaceView和View的最本质的区别**

    SurfaceView是在一个新起的单独线程中可以重新绘制画面，而view必须在UI的主线程中更新画面。
    
    在UI的主线程中更新画面可能会引发问题，比如你更新的时间过长，那么你的主UI线程就会被你正在画的函数阻塞。那么将无法响应按键、触屏等消息。当使用SurfaceView由于是在新的线程中更新画面所以不会阻塞你的UI主线程。但这也带来了另外一个问题，就是事件同步。比如你触屏了一下，你需要SurfaceView中thread处理，一般就需要有一个event queue的设计来保存touchevent，这会稍稍复杂一点，因为涉及到线程安全。

**Android程序运行时权限与文件系统权限** 

1,Linux 文件系统权限。不同的用户对文件有不同的读写执行权限。在android系统中，system和应用程序是分开的，system里的数据是不可更改的。

2，Android中有3种权限，进程权限UserID，签名，应用申明权限。每次安装时，系统根据包名为应用分配唯一的userID，不同的userID运行在不同的进程里，进程间的内存是独立的，不可以相互访问，除非通过特定的Binder机制。

Android提供了如下的一种机制，可以使两个apk打破前面讲的这种壁垒。

在AndroidManifest.xml中利用sharedUserId属性给不同的package分配相同的userID，通过这样做，两个package可以被当做同一个程序，系统会分配给两个程序相同的UserID。当然，基于安全考虑，两个package需要有相同的签名，否则没有验证也就没有意义了。
    
**如何让程序自动启动** 

定义一个Braodcastreceiver，action为BOOT——COMPLETE，接受到广播后启动程序。

**[app如何保证后台服务不被杀死](https://segmentfault.com/a/1190000006251859#articleHeader1)**

**[深拷贝和浅拷贝的区别](http://www.cnblogs.com/chenssy/p/3308489.html)**

**[clone()的默认实现是深拷贝还是浅拷贝?如何让clone()实现深拷贝？](http://blog.csdn.net/zhangjg_blog/article/details/18369201)**

如何实现圆形ImageView；

如何自己实现RecyclerView的侧滑删除；

TabLayout中如何让当前标签永远位于屏幕中间；

TextView调用setText方法的内部执行流程；

Android dalvik虚拟机和Art虚拟机的优化升级点

Activity启动模式，allowReparent的特点和栈亲和性

怎么控制另外一个进程的View显示

双指缩放拖动大图

客户端网络安全实现

曲面屏的适配

能不能动态add同一个布局

手写rxjava遍历数组

aop思想

Android2个虚拟机的区别（一个5.0之前，一个5.0之后）

ActivityThread工作原理 

RecyclerView的ItemTouchHelper的实现原理

7.0 8.0 p特性及兼容


Bitmap的四种属性，如何加载大图（inJustDecodeBounds）。

如何检测一段代码的执行时间

TabLayout如何设置指示器的宽度包裹内容？

最大一次线上Bug处理措施

MVP如何管理Presenter的生命周期，何时取消网络请求

Webview性能优化

**为什么WebView加载会慢呢？**

这是因为在客户端中，加载H5页面之前，需要先初始化WebView，在WebView完全初始化完成之前，后续的界面加载过程都是被阻塞的。

优化手段围绕着以下两个点进行：

预加载WebView。
加载WebView的同时，请求H5页面数据。
因此常见的方法是：

全局WebView。
客户端代理页面请求。WebView初始化完成后向客户端请求数据。
asset存放离线包。
除此之外还有一些其他的优化手段：

脚本执行慢，可以让脚本最后运行，不阻塞页面解析。
DNS与链接慢，可以让客户端复用使用的域名与链接。
React框架代码执行慢，可以将这部分代码拆分出来，提前进行解析。

界面卡顿如何修复

滑动不流畅怎么处理

50fps 有什么办法可以提高到 60fps

**App启动流程，从点击桌面开始**

点击应用图标后会去启动应用的LauncherActivity，如果LancerActivity所在的进程没有创建，还会创建新进程，整体的流程就是一个Activity的启动流程。

Activity的启动流程图（放大可查看）如下所示：

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/app/component/activity_start_flow.png)

整个流程涉及的主要角色有：

Instrumentation: 监控应用与系统相关的交互行为。
AMS：组件管理调度中心，什么都不干，但是什么都管。
ActivityStarter：Activity启动的控制器，处理Intent与Flag对Activity启动的影响，具体说来有：1 寻找符合启动条件的Activity，如果有多个，让用户选择；2 校验启动参数的合法性；3 返回int参数，代表Activity是否启动成功。
ActivityStackSupervisior：这个类的作用你从它的名字就可以看出来，它用来管理任务栈。
ActivityStack：用来管理任务栈里的Activity。
ActivityThread：最终干活的人，是ActivityThread的内部类，Activity、Service、BroadcastReceiver的启动、切换、调度等各种操作都在这个类里完成。
注：这里单独提一下ActivityStackSupervisior，这是高版本才有的类，它用来管理多个ActivityStack，早期的版本只有一个ActivityStack对应着手机屏幕，后来高版本支持多屏以后，就 有了多个ActivityStack，于是就引入了ActivityStackSupervisior用来管理多个ActivityStack。

整个流程主要涉及四个进程：

调用者进程，如果是在桌面启动应用就是Launcher应用进程。
ActivityManagerService等所在的System Server进程，该进程主要运行着系统服务组件。
Zygote进程，该进程主要用来fork新进程。
新启动的应用进程，该进程就是用来承载应用运行的进程了，它也是应用的主线程（新创建的进程就是主线程），处理组件生命周期、界面绘制等相关事情。
有了以上的理解，整个流程可以概括如下：

点击桌面应用图标，Launcher进程将启动Activity（MainActivity）的请求以Binder的方式发送给了AMS。
AMS接收到启动请求后，交付ActivityStarter处理Intent和Flag等信息，然后再交给ActivityStackSupervisior/ActivityStack 处理Activity进栈相关流程。同时以Socket方式请求Zygote进程fork新进程。
Zygote接收到新进程创建请求后fork出新进程。
在新进程里创建ActivityThread对象，新创建的进程就是应用的主线程，在主线程里开启Looper消息循环，开始处理创建Activity。
ActivityThread利用ClassLoader去加载Activity、创建Activity实例，并回调Activity的onCreate()方法。这样便完成了Activity的启动。

![image](http://img.mp.itc.cn/upload/20170329/ca9567ce3bf04c4abdb4d124cebfee76_th.jpeg)

http://www.sohu.com/a/130814934_675634

**一个应用程序安装到手机上时发生了什么**

http://www.androidchina.net/6667.html

**对 Dalvik、ART 虚拟机有基本的了解**

https://blog.csdn.net/jason0539/article/details/50440669

http://www.jackywang.tech/2017/08/21/%E5%85%B3%E4%BA%8EDalvik%EF%BC%8C%E6%88%91%E4%BB%AC%E8%AF%A5%E7%9F%A5%E9%81%93%E4%BA%9B%E4%BB%80%E4%B9%88%EF%BC%9F/


Android中App 是如何沙箱化的,为何要这么做

**权限管理系统**

https://juejin.im/entry/57a99fba5bbb500064418fc0


如何实现右滑finish activity

如何在整个系统层面实现界面的圆角效果（即所有的APP打开界面都会是圆角，我承认，当时我懵逼了）

宽高为20dp的view，不同手机，用尺子量尺寸一样不一样；不一样的话差别是 %20， %30 还是2，3倍。

APK 包含了哪些东西，打包过程是什么；


Android 界面刷新原理

**介绍下Android应用程序启动过程**

    整个应用程序的启动过程要执行很多步骤，但是整体来看，主要分为以下五个阶段：
       一. ：Launcher通过Binder进程间通信机制通知ActivityManagerService，它要启动一个Activity；
    
       二.：ActivityManagerService通过Binder进程间通信机制通知Launcher进入Paused状态；
    
       三.：Launcher通过Binder进程间通信机制通知ActivityManagerService，它已经准备就绪进入Paused状态，于是ActivityManagerService就创建一个新的进程，用来启动一个ActivityThread实例，即将要启动的Activity就是在这个ActivityThread实例中运行；
    
       四. ：ActivityThread通过Binder进程间通信机制将一个ApplicationThread类型的Binder对象传递给ActivityManagerService，以便以后ActivityManagerService能够通过这个Binder对象和它进行通信；
    
       五 ：ActivityManagerService通过Binder进程间通信机制通知ActivityThread，现在一切准备就绪，它可以真正执行Activity的启动操作了。

**非UI线程可以更新UI吗?**

    可以
    当访问UI时，ViewRootImpl会调用checkThread方法去检查当前访问UI的线程是哪个，如果不是UI线程则会抛出异常
    执行onCreate方法的那个时候ViewRootImpl还没创建，无法去检查当前线程.ViewRootImpl的创建在onResume方法回调之后.
    void checkThread() {
        if (mThread != Thread.currentThread()) {
            throw new CalledFromWrongThreadException(
                    "Only the original thread that created a view hierarchy can touch its views.");
        }
    }
    非UI线程是可以刷新UI的，前提是它要拥有自己的ViewRoot,即更新UI的线程和创建ViewRoot是同一个,或者在执行checkThread()前更新UI.

**解决ScrollView嵌套ListView和GridView冲突的方法**

https://blog.csdn.net/btt2013/article/details/53447649

**自定义View优化策略**

    为了加速你的view，对于频繁调用的方法，需要尽量减少不必要的代码。先从onDraw开始，需要特别注意不应该在这里做内存分配的事情，因为它会导致GC，从而导致卡顿。在初始化或者动画间隙期间做分配内存的动作。不要在动画正在执行的时候做内存分配的事情。
    你还需要尽可能的减少onDraw被调用的次数，大多数时候导致onDraw都是因为调用了invalidate().因此请尽量减少调用invaildate()的次数。如果可能的话，尽量调用含有4个参数的invalidate()方法而不是没有参数的invalidate()。没有参数的invalidate会强制重绘整个view。
    另外一个非常耗时的操作是请求layout。任何时候执行requestLayout()，会使得Android UI系统去遍历整个View的层级来计算出每一个view的大小。如果找到有冲突的值，它会需要重新计算好几次。另外需要尽量保持View的层级是扁平化的，这样对提高效率很有帮助。
    如果你有一个复杂的UI，你应该考虑写一个自定义的ViewGroup来执行他的layout操作。与内置的view不同，自定义的view可以使得程序仅仅测量这一部分，这避免了遍历整个view的层级结构来计算大小。这个PieChart 例子展示了如何继承ViewGroup作为自定义view的一部分。PieChart 有子views，但是它从来不测量它们。而是根据他自身的layout法则，直接设置它们的大小。


framework方面有什么理解

Android Studio 3.0 中 Gradle 的 api 和 implementation 有什么区别；

**说说 apk 打包流程；**

Android的包文件APK分为两个部分：代码和资源，所以打包方面也分为资源打包和代码打包两个方面，这篇文章就来分析资源和代码的编译打包原理。

APK整体的的打包流程如下图所示：

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/native/vm/apk_package_flow.png)

具体说来：

通过AAPT工具进行资源文件（包括AndroidManifest.xml、布局文件、各种xml资源等）的打包，生成R.java文件。
通过AIDL工具处理AIDL文件，生成相应的Java文件。
通过Javac工具编译项目源码，生成Class文件。
通过DX工具将所有的Class文件转换成DEX文件，该过程主要完成Java字节码转换成Dalvik字节码，压缩常量池以及清除冗余信息等工作。
通过ApkBuilder工具将资源文件、DEX文件打包生成APK文件。
利用KeyStore对生成的APK文件进行签名。
如果是正式版的APK，还会利用ZipAlign工具进行对齐处理，对齐的过程就是将APK文件中所有的资源文件举例文件的起始距离都偏移4字节的整数倍，这样通过内存映射访问APK文件 的速度会更快。

Android Framework层有没有了解过，说说 Window 窗口添加的过程；

消息推送有没有做过，推送到达率的问题；


如何解决git冲突

设计一个音乐播放界面，你会如何实现，用到那些类，如何设计，如何定义接口，如何与后台交互，如何缓存与下载，如何优化(15分钟时间)


app的架构是怎么样的，并且为什么这样，有什么优缺点？

Android Native 和 JS
通信有几种方式，有没有用到什么框架之类的；

单元测试有没有做过，说说熟悉的单元测试框架；

嵌套滑动实现原理 

Fragment 在 ViewPager 里面的生命周期，滑动 ViewPager 的页面时 Fragment 的生命周期的变化；

Gradle 打包；

AOP IOC 的好处以及在 Android 开发中的应用；

Kotlin 特性，和 Java 相比有什么不同的地方；

Dagger2 框架中 @module 和 @component 的区别；

MVP 架构中 Presenter 定义为接口有什么好处；

Jenkins持续集成；

Java动态代理的使用，InvocationHandler 有什么用；

为什么 Google 会推出Fragment ，有什么好处和用途？ 直接用 View 代替不行么？

安卓安全方面了解过吗，反编译、加壳之类的；


点击事件被拦截，但是想传到下面的View，如何操作？

微信主页面的实现方式

微信上消息小红点的原理

实现stack 的pop和push接口 要求：
1.用基本的数组实现
2.考虑范型
3.考虑下同步问题
4.考虑扩容问题


介绍下先的app架构和通信

推送消息有富文本么？

apk包大小有限制么？怎么减少包大小？

工作中有没有用过或者写过什么工具？脚本，插件等等

比如：多人协同开发可能对一些相同资源都各自放了一份，有没有方法自动检测这种重复之类的

自定义View如何考虑机型适配；

自定义View如何提供获取View属性的接口；

View和ViewGroup分别有哪些事件分发相关的回调方法；

注解的作用与原理

说下四大组件的启动过程，四大组件的启动与销毁的方式。

说下冷启动与热启动是什么，区别，如何优化，使用场景等。












    





