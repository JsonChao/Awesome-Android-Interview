## Android基础面试题

**1.什么是ANR 如何避免它？**

答：在Android上，如果你的应用程序有一段时间响应不够灵敏，系统会向用户显示一个对话框，这个对话框称作应
用程序无响应（ANR：Application NotResponding）对话框。
用户可以选择让程序继续运行，但是，他们在使用你的
应用程序时，并不希望每次都要处理这个对话框。因此
，在程序里对响应性能的设计很重要这样，系统不会显
示ANR给用户。

不同的组件发生ANR的时间不一样，Activity是5秒，BroadCastReceiver是10秒，Service是20秒（均为前台）。

解决方案：

将所有耗时操作，比如访问网络，Socket通信，查询大
量SQL 语句，复杂逻辑计算等都放在子线程中去，然
后通过handler.sendMessage、runonUIThread、AsyncTask 等方式更新UI。无论如何都要确保用户界面作的流畅
度。如果耗时操作需要让用户等待，那么可以在界面上显示度条。
    
**2.Activity和Fragment生命周期有哪些？**

![image](https://upload-images.jianshu.io/upload_images/2893137-d63537703193a6d1.png?imageMogr2/auto-orient/)

![image](http://www.jackywang.tech/AndroidInterview-Q-A/picture/fragment-life.png)
    
**3.横竖屏切换时候Activity的生命周期**

不设置Activity的android:configChanges时，切屏会重新回调各个生命周期，切横屏时会执行一次，切竖屏时会执行两次。
设置Activity的android:configChanges=”orientation”时，切屏还是会调用各个生命周期，切换横竖屏只会执行一次
设置Activity的android:configChanges=”orientation |keyboardHidden”时，切屏不会重新调用各个生命周期，只会执行onConfigurationChanged方法

**4.AsyncTask的缺陷和问题**
    
关于线程池：

Asynctask对应的线程池ThreadPoolExecutr都是进程范围内共享的，都是static的，所以是Asynctask控制着进程范围内所有的子类实例。由于这个限制的存在，当使用默认线程池时，如果线程数超过线程池的最大容量，线程池就会爆掉(3.0后默认串行执行，不会出现个问题)。针对这种情况，可以尝试自定义线程池，配合Asynctask使用。

关于默认线程池：

AsyncTask里面线程池是一个核心线程数为CPU + 1，最大线程数为CPU * 2 + 1，工作队列长度为128的线程池，线程等待队列的最大等待数为28，但是可以自定义线程池。线程池是由AsyncTask来处理的，线程池允许tasks并行运行，需要注意的是并发情况下数据的一致性问题，新数据可能会被老数据覆盖掉类似volatile变量。所以希望tasks能够串行运行的话，使用SERIAL_EXECUTOR。
    
AsyncTask在不同的SDK版本中的区别：

调用AsyncTask的execute方法不能立即执行程序的原因析及改善方案通过查阅官方文档发现，AsyncTask首次引入时，异步任务是在一个独立的线程中顺序的执行，也就是说一次只执行一个任务，不能并行的执行，从1.6开始，AsyncTask引入了线程池，支持同时执行5个异步任务，也就是说时只能有5个线程运行，超过的线程只能等待，等待前的线程某个执行完了才被调度和运行。换句话说，如果个进程中的AsyncTask实例个数超过5个，那么假如前5都运行很长时间的话，那么第6个只能等待机会了。这是AsyncTask的一个限制，而且对于2.3以前的版本无法解决。如果你的应用需要大量的后台线程去执行任务，那么只能放弃使用AsyncTask，自己创建线程池来管理Thread。不得不说，虽然AsyncTask较Thread使用起来方便，但是它最多只能同时运行5个线程，这也大大局限了它的作用，你必须要小心设计你的应用，错开使用AsyncTask时间，尽力做到分时，或者保证数量不会大于5个，否就会遇到上次提到的问题。可能是Google意识到了AsynTask的局限性了，从Android 3.0开始对AsyncTask的API做出了一些调整：每次只启动一个线程执行一个任务，完了之后再执行第二个任务，也就是相当于只有一个后台线在执行所提交的任务。

1.生命周期

很多开发者会认为一个在Activity中创建的AsyncTask随着Activity的销毁而销毁。然而事实并非如此。AsynTask会一直执行，直到doInBackground()方法执行完毕，然后，如果cancel(boolean)被调用,那么onCancelled(Result result)方法会被执行；否则，执行onPostExecuteResult result)方法。如果我们的Activity销毁之前，没有取消AsyncTask，这有可能让我们的应用崩溃(crash)。因为它想要处理的view已经不存在了。所以，我们是必须确保在销毁活动之前取消任务。总之，我们使用AsyncTask需要确保AsyncTask正确的取消。

2.内存泄漏

如果AsyncTask被声明为Activity的非静态的内部类，那么AsyncTask会保留一个对Activity的引用。如果Activity已经被销毁，AsyncTask的后台线程还在执行，它将续在内存里保留这个引用，导致Activity无法被回收，引起内存泄漏。

3.结果丢失

屏幕旋转或Activity在后台被系统杀掉等情况会导致Actvity的重新创建，之前运行的AsyncTask会持有一个之前Activity的引用，这个引用已经无效，这时调用onPostExecute()再去更新界面将不再生效。

4.并行还是串行

在Android1.6之前的版本，AsyncTask是串行的，在1.6-2.3的版本，改成了并行的。在2.3之后的版本又做了修改，可以支持并行和串行，当想要串行执行时，直接行execute()方法，如果需要并行执行时，执行executeOnExecutor(Executor)。

**5.onSaveInstanceState() 与onRestoreIntanceState()**

用户或者程序员主动去销毁一个Activity的时候不会调用
，其他情况都会调动，来保存界面信息。如代码中finish()或用户按下back，不会调用。

**6.android中进程的优先级？**

1. 前台进程：

即与用户正在交互的Activity或者Activity用到的Service等，如果系统内存不足时前台进程是最晚被杀死的

2. 可见进程：

可以是处于暂停状态(onPause)的Activity或者绑定在其上的Service，即被用户可见，但由于失了焦点而不能与用户交互

3. 服务进程：

其中运行着使用startService方法启动的Service，虽然不被用户可见，但是却是用户关心的，例如用户正在非音乐界面听的音乐或者正在非下载页面下载的文件等；当系统要空间运行，前两者进程才会被终止

4. 后台进程：

其中运行着执行onStop方法而停止的程序，但是却不是用户当前关心的，例如后台挂着的QQ，这时的进程系统一旦没了有内存就首先被杀死

5. 空进程：

不包含任何应用程序的程序组件的进程，这样的进程系统是一般不会让他存在的
    
**7.Serializable和Parcelable**

序列化，表示将一个对象转换成可存储或可传输的状态，序列化后的对象可以在网络上进行传输，也可以存储到本地。

Serializable（Java自带）：

Serializable是序列化的意思，表示将一个对象转换成存储或可传输的状态。序列化后的对象可以在网络上进传输，也可以存储到本地。

Parcelable（android专用）：

除了Serializable之外，使用Parcelable也可以实现相同的效果，不过不同于将对象进行序列化，Parcelable方式的实现原理是将一个完整的对象进行分解，而分解后的每一部分都是Intent所支持的数据类型，这也就实现传递对象的功能了。

**8.动画**

- tween 补间动画。通过指定View的初末状态和变化方式，对View的内容完成一系列的图形变换来实现动画效果。 Alpha, Scale ,Translate, Rotate。
- frame 帧动画。Animation Drawable控制animation-list.xml布局
- PropertyAnimation 属性动画3.0引入，属性动画核心思想是对值的变化。

Property Animation 动画有两个步聚：

1.计算属性值

2.为目标对象的属性设置属性值，即应用和刷新动画
    
![image](https://upload-images.jianshu.io/upload_images/2893137-fe2225f697a60433.png?imageMogr2/auto-orient/)
    
计算属性分为3个过程：

过程一：

计算已完成动画分数 elapsed fraction。为了执行一个动画，你需要创建一个ValueAnimator，并且指定目标对象属性的开始、结束和持续时间。在调用 start 后的整个动画过程中，ValueAnimator 会根据已经完成的动画时间计算得到一个0 到 1 之间的分数，代表该动画的已完成动画百分比。0表示 0%，1 表示 100%。

过程二：

计算插值（动画变化率）interpolated fraction 。当 ValueAnimator计算完已完成动画分数后，它会调用当前设置的TimeInterpolator，去计算得到一个interpolated（插值）分数，在计算过程中，已完成动百分比会被加入到新的插值计算中。

过程三：

计算属性值当插值分数计算完成后，ValueAnimator会根据插值分数调用合适的 TypeEvaluator去计算运动中的属性值。
以上分析引入了两个概念：已完成动画分数（elapsed fraction）、插值分数( interpolated fraction )。

原理及特点：

1.动画的基本原理：

其实就是利用插值器和估值器，来计出各个时刻View的属性，然后通过改变View的属性来实现View的动画效果。

2.View动画:

只是影像变化，view的实际位置还在原来地方。

3.帧动画：

是在xml中定义好一系列图片之后，使用AnimatonDrawable来播放的动画。

4.View的属性动画：

1.插值器：作用是根据时间流逝的百分比来计算属性变化的百分比

2.估值器：在1的基础上由这个东西来计算出属性到底变化了多少数值的类

**9.Context相关**

- Activity和Service以及Application的Context是不一的,Activity继承自ContextThemeWraper.其他的继承自ContextWrapper.

- 每一个Activity和Service以及Application的Context是一个新的ContextImpl对象

- getApplication()用来获取Application实例的，但是个方法只有在Activity和Service中才能调用的到。那也许在绝大多数情况下我们都是在Activity或者Servic中使用Application的，但是如果在一些其它的场景，比如BroadcastReceiver中也想获得Application的实例，这时就可以借助getApplicationContext()方法，getApplicationContext()比getApplication()方法的作用域会更广一些，任何一个Context的实例，只要调用getApplicationContext()方法都可以拿到我们的Application对象。

- 创建对话框时不可以用Application的context，只能用Activity的context。

- Context的数量等于Activity的个数 + Service的个数 +1，这个1为Application

**10.Android各版本新特性**

Android5.0新特性

- MaterialDesign设计风格
- 支持多种设备
- 支持64位ART虚拟机

Android6.0新特性

- 动态权限管理
- 支持快速充电的切换
- 支持文件夹拖拽应用
- 相机新增专业模式

Android7.0新特性

- 分屏多任务
- 增强的Java8语言模式
- 夜间模式

Android8.0新特性

- 画中画
- 通知标志
- 自动填充框架
- 系统优化
- 后台限制
- 等等优化很多
    
**11.Json**

JSON的全称是JavaScript Object Notation，也就是JavaScript 对象表示法
JSON是存储和交换文本信息的语法，类似XML，但是比XML更小、更快，更易解析
JSON是轻量级的文本数据交换格式，独立于语言，具有可描述性，更易理解，对象可以包含多个名称/值对，比如：

    {"name":"zhangsan" , "age":25}
使用谷歌的GSON包进行解析，在 Android Studio 里引入依赖：

    compile 'com.google.code.gson:gson:2.7'
值得注意的是实体类中变量名称必须和json中的值名相。
json1的解析
我们这里的实体类是Student.class

    Gson gson = new Gson();
    Student student = gson.fromJson(json1,Student.class);
    
json2的解析
我们可以解析成int数组，也可以解析成Integer的List。
解析成数组：

    Gson gson = new Gson();
    int[] ages = gson.fromJson(json2, int[].class);
    
解析成List：

    Gson gson = new Gson();
    List<Integer> ages = gson.fromJson(json2,  newTypeToken<List<Integer>>(){}.getType);
    
json3的解析
同样可以解析成List或者数组，我们就直接解析成List.

    Gson gson = new Gson();
    List<Student> students = gson.fromJson(json3,     newTypeToke<List<Student>>(){}.getType);

**12.android中有哪几种解析xml的类,官方推荐哪种？以及它们的原理和区别**

DOM解析

优点:

1.XML树在内存中完整存储,因此可以直接修改其数据结构.

2.可以通过该解析器随时访问XML树中的任何一个节点.

3.DOM解析器的API在使用上也相对比较简单.

缺点:

如果XML文档体积比较大时,将文档读入内存是非消耗系统资源的.

使用场景:

- DOM 是用与平台和语言无关的方式表示 XML文档的官方 W3C 标准.
- DOM是以层次结构组织的节点的集合.这个层次结构允许开人员在树中寻找特定信息.分析该结构通常需要加载整文档和构造层次结构,然后才能进行任何工作.
- DOM是基于对象层次结构的.

SAX解析

优点:

SAX 对内存的要求比较低,因为它让开发人员自己来决所要处理的标签.特别是当开发人员只需要处理文档中包含的部分数据时,SAX 这种扩展能力得到了更好的体现.

缺点:

用SAX方式进行XML解析时,需要顺序执行,所以很难访问同一文档中的不同数据.此外,在基于该方式的解析编码程序也相对复杂.

使用场景:

对于含有数据量十分巨大,而又不用对文档的所有数据行遍历或者分析的时候,使用该方法十分有效.该方法不将整个文档读入内存,而只需读取到程序所需的文档标处即可.

Xmlpull解析

android SDK提供了xmlpullapi,xmlpull和sax类似,是基于流（stream）操作文件,后者根据节点事件回调开发者编写的处理程序.因为是基流的处理,因此xmlpull和sax都比较节约内存资源,不会dom那样要把所有节点以对象树的形式展现在内存中.xmpull比sax更简明,而且不需要扫描完整个流.

**13.Jar和Aar的区别**

Jar包里面只有代码，aar里面不光有代码还包括资源文件，比如 drawable 文件，xml资源文件。对于一些不常变动的 Android Library，我们可以直接引用 aar，加快编译速度
    
**14.Android为每个应用程序分配的内存大小是多少**

android程序内存一般限制在16M，也有的是24M。近几年手机发展较快，一般都会分配两百兆左右，和具体机型有关。
    
**15.更新UI方式**

- Activity.runOnUiThread(Runnable)
- View.post(Runnable)，View.postDelay(Runnable, long)
- Handler
- AsyncTask
- Rxjava
- LiveData

**16.ContentProvider使用方法。**

进行跨进程通信，实现进程间的数据交互和共享。通过Context 中 getContentResolver() 获得实例，通过 Uri匹配进行数据的增删改查。ContentProvider使用表的形式来组织数据，无论数据的来源是什么，ConentProvider 都会认为是一种表，然后把数据组织成表格

**17.Thread、AsyncTask、IntentService的使用场景与特点。**

1. Thread线程，独立运行与于 Activity 的，当Activity 被 finish 后，如果没有主动停止 Thread或者 run 方法没有执行完，其会一直执行下去。

2. AsyncTask 封装了两个线程池和一个Handler，（SerialExecutor用于排队，THREAD_POOL_EXECUTOR为真正的执行任务，Handler将工作线程切换到主线程），其必须在 UI线程中创建，execute 方法必须在 UI线程中执行，一个任务实例只允许执行一次，执行多次抛出异常，用于网络请求或者简单数据处理。

3. IntentService：处理异步请求，实现多线程，在onHandleIntent中处理耗时操作，多个耗时任务会依次执行，执行完毕自动结束。

**18.Merge、ViewStub 的作用。**

Merge: 减少视图层级，可以删除多余的层级，优化 UI。

ViewStub: 按需加载，减少内存使用量、加快渲染速度、不支持 merge 标签
    
**19.Json有什么优劣势。**

优点

- 轻量级的数据交换格式
- 读写更加容易
- 易于机器的解析和生成

缺点

- 语义性较差，不如 xml 直观
    
**20.动画有哪两类，各有什么特点？**

传统动画：帧动画和补间动画。

属性动画。

区别

属性动画才是真正的实现了 view 的移动，补间动画对view 的移动更像是在不同地方绘制了一个影子，实际对象还是处于原来的地方。
当动画的 repeatCount 设置为无限循环时，如果在Activity 退出时没有及时将动画停止，属性动画会导致Activity 无法释放而导致内存泄漏，而补间动画却没问题。
xml 文件实现的补间动画，复用率极高。在 Activity切换，窗口弹出时等情景中有着很好的效果。
使用帧动画时需要注意，不要使用过多特别大的图，容导致内存不足。
    
**21.Asset目录与res目录的区别。**

assets：不会在 R文件中生成相应标记，存放到这里的资源在打包时会打包到程序安装包中。（通过 AssetManager 类访问这些文件）

res：会在 R 文件中生成 id标记，资源在打包时如果使用到则打包到安装包中，未用到不会打入安装包中。

res/anim：存放动画资源

res/raw：和 asset下文件一样，打包时直接打入程序安装包中（会映射到 R文件中）
    
**22.Android怎么加速启动Activity。**

- onCreate() 中不执行耗时操作
把页面显示的 View 细分一下，放在 AsyncTask里逐步显示，用 Handler更好。这样用户的看到的就是有层次有步骤的一个个的View 的展示，不会是先看到一个黑屏，然后一下显示所有 View。最好做成动画，效果更自然。
- 利用多线程的目的就是尽可能的减少 onCreate()和onReume()的时间，使得用户能尽快看到页面，操作页面。
- 减少主线程阻塞时间
- 提高 Adapter 和 AdapterView 的效率
- 优化布局文件
    
**23.Handler机制**

Handler各个部分的作用：

1.MessageQueue：读取会自动删除消息，单链表维护，插入和删除上有优势。在其next()中会无限循环，不断断是否有消息，有就返回这条消息并移除。

2.Looper：Looper创建的时候会创建一个MessageQueue调用loop()方法的时候消息循环开始，loop()也是一个循环，会不断调用messageQueue的next()，当有消息就理，否则阻塞在messageQueue的next()中。当Looper的qit()被调用的时候会调用messageQueue的quit(),此时nxt()会返回null，然后loop()方法也跟着退出。

3.Handler：在主线程构造一个Handler，然后在其他线调用sendMessage(),此时主线程的MessageQueue中会插一条message，然后被Looper使用。

4.系统的主线程在ActivityThread的main()为入口开启线程，其中定义了内部类Activity.H定义了一系列消息型，包含四大组件的启动停止。

Android 的消息机制也就是 handler 机制,创建 handler的时候会创建一个 looper ( 通过 looper.prepare()来创建 ),looper 一般为主线程 looper.
handler 通过 send 发送消息 (sendMessage) ,当然post 一系列方法最终也是通过 send 来实现的,在 send方法中handler 会通过 enqueueMessage() 方法中的enqueueMessage(msg,millis )向消息队列 MessageQueue插入一条消息,同时会把本身的 handler 通过msg.target = this 传入.

Looper 是一个死循环,不断的读取MessageQueue中的消,loop 方法会调用 MessageQueue 的 next方法来获取新的消息,next操作是一个阻塞操作,当没有消息的时候 next方法会一直阻塞,进而导致 loop一直阻塞,当有消息的时候,Looper 就会处理消息 Looper收到消息之后就开始处理消息:msg.target.dispatchMessage(msg),当然这里的msg.target 就是上面传过来的发送这条消息的 handler对象,这样 handler发送的消息最终又交给他的dispatchMessage方法来处理了,这里不同的是,handler 的 dispatchMessage方法是在创建 Handler时所使用的 Looper中执行的,这样就成功的将代码逻辑切换到了主线程了.


Handler 处理消息的过程是:首先,检查Message 的callback 是否为 null,不为 null 就通过handleCallBack 来处理消息,Message 的 callback是一个 Runnable 对象,实际上是 handler 的 post方法所传递的 Runnable 参数.其次是检查 mCallback 是否为 null,不为 null 就调用 mCallback的handleMessage 方法来处理消息.
    
Android消息循环流程图如下所示：

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/native/process/android_handler_structure.png)
    
主要涉及的角色如下所示：

Message：消息，分为硬件产生的消息（例如：按钮、触摸）和软件产生的消息。

MessageQueue：消息队列，主要用来向消息池添加消息和取走消息。

Looper：消息循环器，主要用来把消息分发给相应的处理者。

Handler：消息处理器，主要向消息队列发送各种消息以及处理各种消息。

整个消息的循环流程还是比较清晰的，具体说来：

Handler通过sendMessage()发送消息Message到消息队列MessageQueue。
Looper通过loop()不断提取触发条件的Message，并将Message交给对应的target handler来处理。
target handler调用自身的handleMessage()方法来处理Message。
事实上，在整个消息循环的流程中，并不只有Java层参与，很多重要的工作都是在C++层来完成的。我们来看下这些类的调用关系。

![image](https://github.com/guoxiaoxing/android-open-source-project-analysis/raw/master/art/native/process/android_handler_class.png)

注：虚线表示关联关系，实线表示调用关系。

在这些类中MessageQueue是Java层与C++层维系的桥梁，MessageQueue与Looper相关功能都通过MessageQueue的Native方法来完成，而其他虚线连接的类只有关联关系，并没有 直接调用的关系，它们发生关联的桥梁是MessageQueue。
    
https://www.zhihu.com/question/34652589
https://github.com/xitu/gold-miner/blob/master/TODO/android-handler-internals.md
http://blog.csdn.net/guolin_blog/article/details/9991569
http://droidyue.com/blog/2015/11/08/make-use-of-handlerthread
http://blog.csdn.net/luoshengyang/article/deta6817933


**24.程序A能否接收到程序B的广播？接入微信支付的时候，微信是如何跟当前程序进行通信？**

**25.为什么复写equals方法的同时需要复写hashcode方法，前者相同后者是否相同，反过来呢？为什么？**

**26.Android4.0～8.0之间大的变化，如何处理？**

**27. Android中进程的级别，以及各自的区别。**

1、前台进程

用户当前正在做的事情需要这个进程。如果满足下面的件之一，一个进程就被认为是前台进程：

1).这个进程拥有一个正在与用户交互的Activity(这个Ativity的onResume()方法被调用)。

2).这个进程拥有一个绑定到正在与用户交互的activit上的Service。

3).这个进程拥有一个前台运行的Service（service调了方法startForeground()）.

4).这个进程拥有一个正在执行其任何一个生命周期回方法（onCreate(),onStart(),或onDestroy()）的Servie。

5).这个进程拥有正在执行其onReceive()方法的BroadcatReceiver。

通常，在任何时间点，只有很少的前台进程存在。它们有在达到无法调合的矛盾时才会被杀－－如内存太小而能继续运行时。通常，到了这时，设备就达到了一个内分页调度状态，所以需要杀一些前台进程来保证用户界的反应.

2、可见进程

一个进程不拥有运行于前台的组件，但是依然能影响用所见。满足下列条件时，进程即为可见：

这个进程拥有一个不在前台但仍可见的Activity(它的onause()方法被调用)。当一个前台activity启动一个对框时，就出了这种情况。

3、服务进程

一个可见进程被认为是极其重要的。并且，除非只有杀它才可以保证所有前台进程的运行，否则是不能动它的。

这个进程拥有一个绑定到可见activity的Service。

一个进程不在上述两种之内，但它运行着一个被startSevice()所启动的service。

尽管一个服务进程不直接影响用户所见，但是它们通常一些用户关心的事情（比如播放音乐或下载数据），所系统不到前台进程和可见进程活不下去时不会杀它。

4、后台进程

一个进程拥有一个当前不可见的activity(activity的ontop()方法被调用)。

这样的进程们不会直接影响到用户体验，所以系统可以任意时刻杀了它们从而为前台、可见、以及服务进程们供存储空间。通常有很多后台进程在运行。它们被保存一个LRU(最近最少使用)列表中来确保拥有最近刚被看到的activity的进程最后被杀。如果一个activity正确的实现了它的生命周期方法，并保存了它的当前状态，那么杀死它的进程将不会对用户的可视化体验造成影响。因为当用户返回到这个activity时，这个activity会恢复它所有的可见状态。
    
5、空进程

一个进程不拥有入何active组件。

保留这类进程的唯一理由是高速缓存，这样可以提高下次一个组件要运行它时的启动速度。系统经常为了平衡进程高速缓存和底层的内核高速缓存之间的整体系统资而杀死它们。

**28.线程池的相关知识。**

Android中的线程池都是之间或间接通过配置ThreadPoolxecutor来实现不同特性的线程池.Android中最常见的类具有不同特性的线程池分别为FixThreadPool、CachedhreadPool、SingleThreadPool、ScheduleThreadExecutr.

1).FixThreadPool

只有核心线程,并且数量固定的,也不会被回收,所有线都活动时,因为队列没有限制大小,新任务会等待执行.

优点:更快的响应外界请求.

2).SingleThreadPool

只有一个核心线程,确保所有的任务都在同一线程中按序完成.因此不需要处理线程同步的问题.

3).CachedThreadPool

只有非核心线程,最大线程数非常大,所有线程都活动时会为新任务创建新线程,否则会利用空闲线程(60s空闲时间,过了就会被回收,所以线程池中有0个线程的可能)处理任务.
    
优点:任何任务都会被立即执行(任务队列SynchronousQuue相当于一个空集合);比较适合执行大量的耗时较少的务.

4).ScheduledThreadPool

核心线程数固定,非核心线程(闲着没活干会被立即回收数没有限制.

优点:执行定时任务以及有固定周期的重复任务

**29.内存泄露，怎样查找，怎么产生的内存泄露。**

1).资源对象没关闭造成的内存泄漏

2).构造Adapter时，没有使用缓存的convertView

3).Bitmap对象不在使用时调用recycle()释放内存

4).试着使用关于application的context来替代和activiy相关的context

5).注册没取消造成的内存泄漏

6).集合中对象没清理造成的内存泄漏

查找内存泄漏

查找内存泄漏可以使用Android Studio 自带的AndroidProfiler工具,也可以使用Square产品的LeakCanary.

**30.类的初始化顺序依次是（静态变量、静态代码块）>（变量、代码块）>构造方法**

**31.如何实现Fragment的滑动**

**32.ViewPager使用细节，如何设置成每次只初始化当前的Fragment，其他的不初始化**

**33.ListView重用的是什么**

**34.如何取消AsyncTask**

**35.Android为什么引入Parcelable**

**36.有没有尝试简化Parcelable的使用**

**37.Bitmap 使用时候注意什么？**

**38.Oom 是否可以try catch ？**

**39.多进程场景遇见过么？**

**40.关于handler，在任何地方new handler 都是什么线程下**

**41.sqlite升级，增加字段的语句**

**42.数据库升级增加表和删除表都不涉及数据迁移，但是修改表涉及到对原有数据进行迁移。升级的方法如下所示：**

将现有表命名为临时表。
创建新表。
将临时表的数据导入新表。
删除临时表。
重写

如果是跨版本数据库升级，可以由两种方式，如下所示：

逐级升级，确定相邻版本与现在版本的差别，V1升级到V2,V2升级到V3，依次类推。
跨级升级，确定每个版本与现在数据库的差别，为每个case编写专门升级大代码。

**43.bitmap recycler 相关**

**44.强引用置为null，会不会被回收？**

**45.glide 使用什么缓存？**

**46.Glide 内存缓存如何控制大小？**

**47.请描述安卓四大组件之间的关系，并说下安卓MVC的设计模式。**

**48.ContentProvider的权限管理(读写分离，权限控制-精确到表级，URL控制)；**

**49.Fragment状态保存**

**50.startActivityForResult是哪个类的方法，在什么情况下使用，如果在Adapter中使用应该如何解耦**

**51.如何计算一个Bitmap占用内存的大小，怎么保证加载Bitmap不产生内存溢出？**

    Bitamp 占用内存大小 = 宽度像素 x （inTargetDensity / inDensity） x 高度像素 x （inTargetDensity / inDensity）x 一个像素所占的内存

注：这里inDensity表示目标图片的dpi（放在哪个资源文件夹下），inTargetDensity表示目标屏幕的dpi，所以你可以发现inDensity和inTargetDensity会对Bitmap的宽高 进行拉伸，进而改变Bitmap占用内存的大小。

在Bitmap里有两个获取内存占用大小的方法。

getByteCount()：API12 加入，代表存储 Bitmap 的像素需要的最少内存。
getAllocationByteCount()：API19 加入，代表在内存中为 Bitmap 分配的内存大小，代替了 getByteCount() 方法。
在不复用 Bitmap 时，getByteCount() 和 getAllocationByteCount 返回的结果是一样的。在通过复用 Bitmap 来解码图片时，那么 getByteCount() 表示新解码图片占用内存的大 小，getAllocationByteCount() 表示被复用 Bitmap真实占用的内存大小（即 mBuffer 的长度）。

为了保证在加载Bitmap的时候不产生内存溢出，可以受用BitmapFactory进行图片压缩，主要有以下几个参数：

BitmapFactory.Options.inPreferredConfig：将ARGB_8888改为RGB_565，改变编码方式，节约内存。
BitmapFactory.Options.inSampleSize：缩放比例，可以参考Luban那个库，根据图片宽高计算出合适的缩放比例。
BitmapFactory.Options.inPurgeable：让系统可以内存不足时回收内存。

**52.对于应用更新这块是如何做的？(灰度，强制更新，分区域更新)**

**53.Handler机制，请写出一种更新UI的方法和代码**

**54.请解释安卓为啥要加签名机制。**

**55.十六进制数据怎么和十进制和二进制之间转换**

**56.activty和Fragmengt之间怎么通信，Fragmengt和Fragmengt怎么通信**

**57.自定义view效率高于xml定义吗？说明理由。**

**58.广播注册一般有几种，各有什么优缺点**

**59.服务启动一般有几种，服务和activty之间怎么通信，服务和服务之间怎么通信**

**60.数据库的知识，包括本地数据库优化点。**

**61.ddms 和 traceView的区别；**

**62.Fragment生命周期；Fragment状态保存；**

**63..startActivityForResult是哪个类的方法，在什么情况下使用，如果在Adapter中使用应该如何解耦；**

**64.AsyncTask原理及不足；**

**65.IntentService原理；**

**66.Activity 怎么和Service 绑定，怎么在Activity 中启动自己对应的Service；**

**67.AstncTask + HttpClient与AsyncHttpClient有什么区别；**

**68.如何保证一个后台服务不被杀死；比较省电的方式是什么；**

**69.如何通过广播拦截和abort一条短信；广播是否可以请求网络；广播引起anr的时间限制；**

**70.BroadcastReceiver，LocalBroadcastReceiver 区别；**

**71.请介绍下ContentProvider 是如何实现数据共享的；**

**72.ListView 中图片错位的问题是如何产生的；**

**73.说说Activity、Intent、Service 是什么关系；**

**74.ApplicationContext和ActivityContext的区别；**

**75.Serializable 和Parcelable 的区别；**

**76.AsyncTask 如何使用；**

**77.对于应用更新这块是如何做的？(灰度，强制更新，分区域更新)；**

**78.两个Activity之间跳转时必然会执行的是哪几个方法？**

答：一般情况下比如说有两个activity,分别叫A,B,当在A里面激活B 组件的时候, A 会调用onPause()方法,然后B调用onCreate() ,onStart(), onResume()。
这个时候B 覆盖了窗体, A 会调用onStop()方法. 如果B是个透明的,或者是对话框的样式, 就不会调用A 的
onStop()方法。
    
**79.如何选择第三方，从那些方面考虑？**

**80.简单说下接入支付的流程，是否自己接入过支付功能**

**81.说下你对多进程的理解，什么情况下要使用多进程，为什么要使用多进程，在多进程的情况下为什么要使用进程通讯。**

**82.说下handler为什么会出现内存泄漏，为什么继承Handle就不会出现内存泄漏**

**83.说下你对广播的理解**

**84.说下你对服务的理解，如何杀死一个服务。**

**85.如何查看模拟器中的SP与SQList文件。如何可视化查看布局嵌套层数与加载时间。**

**86.各大平台打包上线的流程与审核时间，常见问题(主流的应用市场说出3-4)**

**87.如何收集anr信息**

**88.说说ContentProvider、ContentResolver、ContentObserver 之间的关系**

ContentProvider：管理数据，提供数据的增删改查操作，数据源可以是数据库、文件、XML、网络等，ContentProvider为这些数据的访问提供了统一的接口，可以用来做进程间数据共享。
ContentResolver：ContentResolver可以不同URI操作不同的ContentProvider中的数据，外部进程可以通过ContentResolver与ContentProvider进行交互。
ContentObserver：观察ContentProvider中的数据变化，并将变化通知给外界。

**89.AlertDialog,popupWindow,Activity区别**

**90.Android属性动画特性**

如果你的需求中只需要对View进行移动、缩放、旋转和淡入淡出操作，那么补间动画确实已经足够健全了。但是很显然，这些功能是不足以覆盖所有的场景的，一旦我们的需求超出了移动、缩放、旋转和淡入淡出这四种对View的操作，那么补间动画就不能再帮我们忙了，也就是说它在功能和可扩展方面都有相当大的局限性，那么下面我们就来看看补间动画所不能胜任的场景。

注意上面我在介绍补间动画的时候都有使用“对View进行操作”这样的描述，没错，补间动画是只能够作用在View上的。也就是说，我们可以对一个Button、TextView、甚至是LinearLayout、或者其它任何继承自View的组件进行动画操作，但是如果我们想要对一个非View的对象进行动画操作，抱歉，补间动画就帮不上忙了。可能有的朋友会感到不能理解，我怎么会需要对一个非View的对象进行动画操作呢？这里我举一个简单的例子，比如说我们有一个自定义的View，在这个View当中有一个Point对象用于管理坐标，然后在onDraw()方法当中就是根据这个Point对象的坐标值来进行绘制的。也就是说，如果我们可以对Point对象进行动画操作，那么整个自定义View的动画效果就有了。显然，补间动画是不具备这个功能的，这是它的第一个缺陷。

然后补间动画还有一个缺陷，就是它只能够实现移动、缩放、旋转和淡入淡出这四种动画操作，那如果我们希望可以对View的背景色进行动态地改变呢？很遗憾，我们只能靠自己去实现了。说白了，之前的补间动画机制就是使用硬编码的方式来完成的，功能限定死就是这些，基本上没有任何扩展性可言。

最后，补间动画还有一个致命的缺陷，就是它只是改变了View的显示效果而已，而不会真正去改变View的属性。什么意思呢？比如说，现在屏幕的左上角有一个按钮，然后我们通过补间动画将它移动到了屏幕的右下角，现在你可以去尝试点击一下这个按钮，点击事件是绝对不会触发的，因为实际上这个按钮还是停留在屏幕的左上角，只不过补间动画将这个按钮绘制到了屏幕的右下角而已。

**91.如何导入外部数据库?**

把原数据库包括在项目源码的 res/raw

android系统下数据库应该存放在 /data/data/com..（package name）/ 目录下，所以我们需要做的是把已有的数据库传入那个目录下.操作方法是用FileInputStream读取原数据库，再用FileOutputStream把读取到的东西写入到那个目录.

**92.LinearLayout、RelativeLayout、FrameLayout的特性及对比，并介绍使用场景。**

**93.谈谈对接口与回调的理解**

**94.屏幕适配的处理技巧都有哪些?**

**95。服务器只提供数据接收接口，在多线程或多进程条件下，如何保证数据的有序到达？**

**96.动态布局的理解**

**97.怎么去除重复代码？**

**98.Recycleview和ListView的区别**

**99.ListView图片加载错乱的原理和解决方案**

**100.动态权限适配方案，权限组的概念**

**101.Android系统为什么会设计ContentProvider？**

**102.下拉状态栏是不是影响activity的生命周期**

**103.如果在onStop的时候做了网络请求，onResume的时候怎么恢复？**

**104.Bitmap使用时候注意什么？**

**105.Bitmap的recycler**()

**106.Android中开启摄像头的主要步骤**

**107.ViewPager使用细节，如何设置成每次只初始化当前的Fragment，其他的不初始化？**


**108.Debug和Release状态的不同**

**109.dp是什么，sp呢，有什么区别**

**110.自定义View，ViewGroup注意那些回调？**

**111.界面卡顿的原因以及解决方法**

**112.android中的存储类型**

**113.service用过么，基本调用方法**

**114.Handler机制**

**115.LinearLayout、FrameLayout、RelativeLayout性能对比，为什么**

RelativeLayout会让子View调用2次onMeasure，LinearLayout 在有weight时，也会调用子View2次onMeasure

RelativeLayout的子View如果高度和RelativeLayout不同，则会引发效率问题，当子View很复杂时，这个问题会更加严重。如果可以，尽量使用padding代替margin。
 
在不影响层级深度的情况下,使用LinearLayout和FrameLayout而不是RelativeLayout。
最后再思考一下文章开头那个矛盾的问题，为什么Google给开发者默认新建了个RelativeLayout，而自己却在DecorView中用了个LinearLayout。因为DecorView的层级深度是已知而且固定的，上面一个标题栏，下面一个内容栏。采用RelativeLayout并不会降低层级深度，所以此时在根节点上用LinearLayout是效率最高的。而之所以给开发者默认新建了个RelativeLayout是希望开发者能采用尽量少的View层级来表达布局以实现性能最优，因为复杂的View嵌套对性能的影响会更大一些。

**116.Activity的生命周期，finish调用后其他生命周期还会走么？**

**117.内存泄漏如何排查，MAT分析方法以及原理，各种泄漏的原因是什么比如Handler为什么会泄漏**


**118.view的绘制熟悉么，介绍下**

**119.anr是因为什么产生的，怎么排查**

**120.界面上的话，有什么优化措施么？比如列表展示之类的，平时遇到过内存问题吗，怎么优化的？**

**121.线程之间怎么通信的？**


**122.有没有做过 apk 多渠道打包；**


**123.java线程，场景实现，多个线程如何同时请求，返回的结果如何等待所有线程数据完成后合成一个数据**

**124.Manifest.xml的里有什么和作用**

**125.什么是多进程，进程和线程的区别，如何给四大组件指定多进程。**

**126.如何选择第三方，从那些方面考虑**

**127.scheme跳转协议**

Android中的scheme是一种页面内跳转协议，通过定义自己的scheme协议，可以跳转到app中的各个页面

服务器可以定制化告诉app跳转哪个页面

App可以通过跳转到另一个App页面

可以通过H5页面跳转页面

**128.Service和Thread的区别**

Service是安卓中系统的组件，它运行在独立进程的主线程中，不可以执行耗时操作。Thread是程序执行的最小单元，分配CPU的基本单位，可以开启子线程执行耗时操作

Service在不同Activity中可以获取自身实例，可以方便的对Service进行操作。Thread在不同的Activity中难以获取自身实例，如果Activity被销毁，Thread实例就很难再获取得到

**129.Bitmap recycle**

在安卓3.0以前Bitmap是存放在堆中的，我们只要回收堆内存即可
    
在安卓3.0以后Bitmap是存放在内存中的，我们需要回收native层和Java层的内存
    
官方建议我们3.0以后使用recycle方法进行回收，该方法也可以不主动调用，因为垃圾回收器会自动收集不可用的Bitmap对象进行回收
    
recycle方法会判断Bitmap在不可用的情况下，将发送指令到垃圾回收器，让其回收native层和Java层的内存，则Bitmap进入dead状态
    
recycle方法是不可逆的，如果再次调用getPixels()等方法，则获取不到想要的结果

**120.手写一个简易的结合Retrofit + okhttp的网络请求的代码**

**121.JSON的结构？**

json是一种轻量级的数据交换格式，
json简单说就是对象和数组，所以这两种结构就是对象和数组两种结构，通过这两种结构可以表示各种复杂的结构

1、对象：对象表示为“{}”扩起来的内容，数据结构为 {key：value,key：value,...}的键值对的结构，在面向对象的语言中，key为对象的属性，value为对应的属性值，所以很容易理解，取值方法为 对象.key 获取属性值，这个属性值的类型可以是 数字、字符串、数组、对象几种。

2、数组：数组在json中是中括号“[]”扩起来的内容，数据结构为 ["java","javascript","vb",...]，取值方式和所有语言中一样，使用索引获取，字段值的类型可以是 数字、字符串、数组、对象几种。
经过对象、数组2种结构就可以组合成复杂的数据结构了。

**122.xml有几种解析方式、区别?**

基本的解析方式有三种: DOM,SAX,Pull
dom解析：解析器读入整个文档，然后构建一个驻留内存的树结构，然后就可以使用 DOM 接口来操作这个树结构。优点是对文档增删改查比较方便，缺点占用内存比较大。
sax解析：基于事件驱动型,优点占用内存少，解析速度快，缺点是只适合做文档的读取，不适合做文档的增删改，不能中途停止。
pull解析：同样基于事件驱动型,android 官方API提供,可随时终止,调用next() 方法提取它们（主动提取事件）

**123.json解析方式的两种区别**

1，SDK提供JSONArray，JSONObject

2，google提供的 Gson
通过fromJson()实现对象的反序列化（即将json串转换为对象类型）
通过toJson()实现对象的序列化 （即将对象类型转换为json串）

**124.通过google提供的Gson解析json时，定义JavaBean的规则是什么？**

1). 实现序列化 Serializable

2). 属性私有化，并提供get，set方法

3). 提供无参构造

4). 属性名必须与json串中属性名保持一致 （因为Gson解析json串底层用到了Java的反射原理）

**125.了解 aar 文件没，有没有遇到什么坎；**

**126.数据加载更多涉及到分页，你是怎么实现的；**

**127.Java 静态变量在 new 的对象中会不会更改；**

**128.equals 和 hashcode 的关系；**

**129.activity的startActivity和context的startActivity区别**

(1)从Activity中启动新的Activity时可以直接mContext.startActivity(intent)就好

(2)如果从其他Context中启动Activity则必须给intent设置Flag:

    intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK) ; 
    mContext.startActivity(intent);

怎么在Service中创建Dialog对话框

1.在我们取得Dialog对象后，需给它设置类型，即：

    dialog.getWindow().setType(WindowManager.LayoutPaams.TYPE_SYSTEM_ALERT)

2.在Manifest中加上权限:

    <uses-permissionandroid:name="android.permission.SYSTEM_ALERT_WINOW" />
    
**130.HandlerThread**

1、HandlerThread作用

当系统有多个耗时任务需要执行时，每个任务都会开启个新线程去执行耗时任务，这样会导致系统多次创建和毁线程，从而影响性能。为了解决这一问题，Google提了HandlerThread，HandlerThread是在线程中创建一个Loper循环器，让Looper轮询消息队列，当有耗时任务进队列时，则不需要开启新线程，在原有的线程中执行耗任务即可，否则线程阻塞。
    
2、HanlderThread的优缺点

- HandlerThread本质上是一个线程类，它继承了Thread；

- HandlerThread有自己的内部Looper对象，可以进行loopr循环；

- 通过获取HandlerThread的looper对象传递给Handler对，可以在handleMessage()方法中执行异步任务。

- 创建HandlerThread后必须先调用HandlerThread.start(方法，Thread会先调用run方法，创建Looper对象。

- HandlerThread优点是异步不会堵塞，减少对性能的消耗

- HandlerThread缺点是不能同时继续进行多任务处理，要等待进行处理，处理效率较低

- HandlerThread与线程池不同，HandlerThread是一个串队列，背后只有一个线程。

**131.IntentService**

https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fjavazejian%2Farticle%2Fdetails%2F52426425

**132.AsyncTask**

https://link.juejin.im/?target=http%3A%2F%2Fblog.csdn.net%2Fjavazejian%2Farticle%2Fdetails%2F52462830

**133.有遇到过哪些屏幕和资源适配问题？**

https://www.jianshu.com/p/46ce37b8553c

**134.项目中遇到哪些难题，最终你是如何解决的？**

https://www.jianshu.com/p/69d9444e2a9a

**135.如何将一个Activity设置成窗口的样式。**

<activity>中配置：

    android:theme="@android:style/Theme.Dialog"
    
另外 

    android:theme="@android:style/Theme.Translucnt"
是设置透明


**136.Android Application对象必须掌握的七点**

https://blog.csdn.net/lilu_leo/article/details/8649941

**137.listview图片加载错乱的原理和解决方案**

https://blog.csdn.net/guolin_blog/article/details/45586553

**138.数据库数据迁移，把大象装进新冰箱共分几步？**

- 将表名改成临时表ALTER TABLE Order RENAME TO _Order

- 创建新表 CREATETABLE Test(Id VARCHAR(32) PRIMARY KEY ,
  CustomName VARCHAR(32) NOTNULL , Country VARCHAR(16) NOTNULL)

- 导入数据 INSERTINTO Order SELECT id, “”, Age FROM _Order

- 删除临时表DROPTABLE _Order

**139.深入理解Java注解类型(@Annotation)**

https://blog.csdn.net/javazejian/article/details/71860633

**140.如何开启多进程?应用是否可以开启N个进程？**

**141.final修饰一个对象，能否调用对象修改属性的方法**

**142.注解如何获取，反射为何耗性能？**

**143.int,long的取值范围以及BigDecimal，数值越界了如何处理？**

**144.Android中如何查看一个对象的回收情况？**

**145.ContentProvider、ContentResolver、ContentObserver 之间的关系。**

**146.Android怎么加速启动Activity。**


**147.invalidate和requestLayout的区别及使用。**

**148.如何反编译，对代码逆向分析；**



**149.Intent传值有大小限制吗，为什么，如何处理；**

**151.广播中怎么进行网络请求**

**152.双线程通过线程同步的方式打印12121212.......**



**153.RemoteViews实现和使用场景**


**154.对服务器众多错误码的处理（错误码有好几万个）**


**155.adb常用命令行 **

**156.Android中跨进程通讯的几种方式**

1：访问其他应用程序的Activity
如调用系统通话应用

    Intent callIntent = new Intent(Intent.ACTION_CALL,Uri.parse("tel:12345678");
    startActivity(callIntent);

2：Content Provider
如访问系统相册

3：广播（Broadcast）
如显示系统时间

4：AIDL服务

**157.显示Intent与隐式Intent的区别**

对明确指出了目标组件名称的Intent，我们称之为“显式Intent”。

对于没有明确指出目标组件名称的Intent，则称之为“隐式 Intent”。

对于隐式意图，在定义Activity时，指定一个intent-filter，当一个隐式意图对象被一个意图过滤器进行匹配时，将有三个方面会被参考到：

动作(Action)

类别(Category ['kætɪg(ə)rɪ] )

数据(Data )

    <activity android:name=".MainActivity"  android:label="@string/app_name">
                <intent-filter>
                    <action android:name="com.wpc.test" />
                    <category android:name="android.intent.category.DEFAULT" />
                    <data android:mimeType="image/gif"/>
                </intent-filter>
    </activity>

**158.什么是线程池，线程池的作用是什么**

线程池的基本思想还是一种对象池的思想，开辟一块内存空间，里面存放了众多(未死亡)的线程，池中线程执行调度由池管理器来处理。当有线程任务时，从池中取一个，执行完成后线程对象归池，这样可以避免反复创建线程对象所带来的性能开销，节省了系统的资源。就好比原来去食堂打饭是每个人看谁抢的赢，谁先抢到谁先吃，有了线程池之后，就是排好队形，今天我跟你关系好，你先来吃饭。比如：一个应用要和网络打交道，有很多步骤需要访问网络，为了不阻塞主线程，每个步骤都创建个线程，在线程中和网络交互，用线程池就变的简单，线程池是对线程的一种封装，让线程用起来更加简便，只需要创一个线程池，把这些步骤像任务一样放进线程池，在程序销毁时只要调用线程池的销毁函数即可。

单个线程的弊端：a. 每次new Thread新建对象性能差b. 线程缺乏统一管理，可能无限制新建线程，相互之间竞争，及可能占用过多系统资源导致死机或者OOM,c. 缺乏更多功能，如定时执行、定期执行、线程中断。

java提供的四种线程池的好处在于：a. 重用存在的线程，减少对象创建、消亡的开销，性能佳。b. 可有效控制最大并发线程数，提高系统资源的使用率，同时避免过多资源竞争，避免堵塞。c. 提供定时执行、定期执行、单线程、并发数控制等功能。

Java 线程池

Java通过Executors提供四种线程池，分别为：

newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。

newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。

newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。

newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。

(1). newCachedThreadPool

创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。线程池为无限大，当执行第二个任务时第一个任务已经完成，会复用执行第一个任务的线程，而不用每次新建线程。

(2). newFixedThreadPool

创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。

(3) newScheduledThreadPool

创建一个定长线程池，支持定时及周期性任务执行。ScheduledExecutorService比Timer更安全，功能更强大

(4)、newSingleThreadExecutor

创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行

http://yukai.space/2017/05/08/java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E7%9A%84%E4%BD%BF%E7%94%A8/

https://juejin.im/post/5906b6e78d6d810058dab1bf

虚拟机如何实现的synchronized？

http://www.infoq.com/cn/articles/java-se-16-synchronized

**159.IntentService的用法**

一、IntentService简介 

IntentService是Service的子类，比普通的Service增加了额外的功能。先看Service本身存在两个问题：  

Service不会专门启动一条单独的进程，Service与它所在应用位于同一个进程中；  

Service也不是专门一条新线程，因此不应该在Service中直接处理耗时的任务；  

二、IntentService特征 

会创建独立的worker线程来处理所有的Intent请求；  

会创建独立的worker线程来处理onHandleIntent()方法实现的代码，无需处理多线程问题；  

所有请求处理完成后，IntentService会自动停止，无需调用stopSelf()方法停止Service；  

为Service的onBind()提供默认实现，返回null；  

为Service的onStartCommand提供默认实现，将请求Intent添加到队列中； 

IntentService不会阻塞UI线程，而普通Serveice会导致ANR异常
Intentservice若未执行完成上一次的任务，将不会新开一个线程，是等待之前的任务完成后，再执行新的任务，等任务完成后再次调用stopSelf

三、作用

生成一个默认的且与主线程互相独立的工作者线程来执行所有传送至onStartCommand() 方法的Intetnt。

生成一个工作队列来传送Intent对象给你的onHandleIntent()方法，同一时刻只传送一个Intent对象，这样一来，你就不必担心多线程的问题。在所有的请求(Intent)都被执行完以后会自动停止服务，所以，你不需要自己去调用stopSelf()方法来停止。

该服务提供了一个onBind()方法的默认实现，它返回null

提供了一个onStartCommand()方法的默认实现，它将Intent先传送至工作队列，然后从工作队列中每次取出一个传送至onHandleIntent()方法，在该方法中对Intent对相应的处理。

**160.Android Holo主题与MD主题的理念，以及你的看法**

Holo Theme

Holo Theme 是 Android Design 的最基础的呈现方式。因为是最为基础的 Android Design 呈现形式，每一台 Android 4.X 的手机系统内部都有集成 Holo Theme 需要的控件，即开发者不需要自己设计控件，而是直接从系统里调用相应的控件。在 UI 方面没有任何的亮点，和 Android4.X 的设置/电话的视觉效果极度统一。由此带来的好处显而易见，这个应用作为 Android 应用的辨识度极高，且完全不可能与系统风格产生冲突。

Material Design

Material design其实是单纯一种设计语言，它包含了系统界面风格、交互、UI,更加专注拟真,更加大胆丰富的用色,更加丰富的交互形式,更加灵活的布局形式

1.鲜明、形象的界面风格，

2.色彩搭配使得应用看起来非常的大胆、充满色彩感，凸显内容

3.Material design对于界面的排版非常的重视

4.Material design的交互设计上采用的是响应式交互，这样的交互设计能把一个应用从简单展现用户所请求的信息，提升至能与用户产生更强烈、更具体化交互的工具。

**161.String，StringBuffer，StringBuilder有哪些不同**

三者在执行速度方面的比较：StringBuilder >  StringBuffer  >  String

String每次变化一个值就会开辟一个新的内存空间

StringBuilder：线程非安全的

StringBuffer：线程安全的

对于三者使用的总结： 

1.如果要操作少量的数据用 = String

2.单线程操作字符串缓冲区 下操作大量数据 = StringBuilder

3.多线程操作字符串缓冲区 下操作大量数据 = StringBuffer

**162.浮点数的精准计算**

BigDecimal类进行商业计算，Float和Double只能用来做科学计算或者是工程计算

**163.Android系统提供了那些动画机制**

1.逐帧动画

  逐帧动画的工作原理很简单，其实就是将一个完整的动画拆分成一张张单独的图片，然后再将它们连贯起来进行播放，类似于动画片的工作原理

2.补间动画

  补间动画则是可以对View进行一系列的动画操作，包括淡入淡出、缩放、平移、旋转四种

  缺陷：补间动画是只能够作用在View上，它只能够实现移动、缩放、旋转和淡入淡出这四种动画操作，补间动画还有一个致命的缺陷，就是它只是改变了View的显示效果而已，而不会真正去改变View的属性

  逐帧动画和补间动画它们的技术已经比较老了，而且网上资料也非常多

3.属性动画

  ValueAnimator

  ValueAnimator是整个属性动画机制当中最核心的一个类

  ObjectAnimator

  相比于ValueAnimator，ObjectAnimator可能才是我们最常接触到的类，因为ValueAnimator只不过是对值进行了一个平滑的动画过渡，但我们实际使用到这种功能的场景好像并不多。而ObjectAnimator则就不同了，它是可以直接对任意对象的任意属性进行动画操作的，比如说View的alpha属性

**164.java为什么跨平台**

因为Java程序编译之后的代码不是能被硬件系统直接运行的代码，而是一种“中间码”——字节码。然后不同的硬件平台上安装有不同的Java虚拟机(JVM)，由JVM来把字节码再“翻译”成所对应的硬件平台能够执行的代码。因此对于Java编程者来说，不需要考虑硬件平台是什么。所以Java可以跨平台。


**165.[Activity正常和异常情况下的生命周期](http://blog.csdn.net/geekerhw/article/details/48749935)**

**166.[IntentService比Service好在哪](https://link.zhihu.com/?target=http%3A//blog.qiji.tech/archives/2693)**



**167.[Thread和HandlerThread区别](https://www.jianshu.com/p/5b6c71a7e8d7)**

**168.[关于< include >< merge >< stub >三者的使用场景](http://www.trinea.cn/android/layout-performance/)**


**169.[对Android消息机制的理解](https://zhuanlan.zhihu.com/p/25222485)**

**170.[哪些情况会导致OOM？](http://note.youdao.com/)**

**171.[用leak工具监测内存泄露的原理是什么？](https://www.jianshu.com/p/5ee6b471970e)**

**172.[ThreadLocal原理，实现及如何保证Local属性]()**

**173.[Android对HashMap做了优化后推出的新的容器类是什么？](http://blog.csdn.net/u010687392/article/details/47809295)**

**174.[线程池的实现机制](http://www.cnblogs.com/dolphin0520/p/3932921.html)**

**175.[Integer类对int的优化](http://denverj.iteye.com/blog/745422)**

**176.[通过静态内部类实现单例模式有哪些优点](http://blog.csdn.net/yingpaixiaochuan/article/details/63260514)**

**177.单例实现线程的同步的要求：**

1.单例类确保自己只有一个实例(构造函数私有:不被外部实例化,也不被继承)。

2.单例类必须自己创建自己的实例。

3.单例类必须为其他对象提供唯一的实例。

**178.[界面卡顿的原因有哪些？](https://www.jianshu.com/p/1fb065c806e6)**

**179.[造成OOM/ANR 的原因？](http://www.cnblogs.com/purediy/p/3276545.html)**

**180.[Activity与Fragment之间如何进行通信？](http://blog.csdn.net/u012702547/article/details/49786417)**

**181.[操作系统进程间通信有哪些方法](http://blog.csdn.net/shinehoo/article/details/5818843/)**

**182.如何保证Service不被杀死**

Android 进程不死从3个层面入手：

A.提供进程优先级，降低进程被杀死的概率

方法一：监控手机锁屏解锁事件，在屏幕锁屏时启动1个像素的 Activity，在用户解锁时将 Activity 销毁掉。

方法二：启动前台service。

方法三：提升service优先级：

在AndroidManifest.xml文件中对于intent-filter可以通过android:priority = "1000"这个属性设置最高优先级，1000是最高值，如果数字越小则优先级越低，同时适用于广播。

B. 在进程被杀死后，进行拉活

方法一：注册高频率广播接收器，唤起进程。如网络变化，解锁屏幕，开机等

方法二：双进程相互唤起。

方法三：依靠系统唤起。

方法四：onDestroy方法里重启service：service +broadcast 方式，就是当service走ondestory的时候，发送一个自定义的广播，当收到广播的时候，重新启动service；

C. 依靠第三方

根据终端不同，在小米手机（包括 MIUI）接入小米推送、华为手机接入华为推送；其他手机可以考虑接入腾讯信鸽或极光推送与小米推送做 A/B Test。

**183.引起内存泄漏的情况**

- 对于使用了BraodcastReceiver，ContentObserver，File，游标 Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销。
- 静态内部类持有外部成员变量（或context）:可以使用弱引用或使用ApplicationContext。
- 内部类持有外部类引用,异步任务中，持有外部成员变量。
- 集合中没用的对象没有及时remove。
- 不用的对象及时释放，如使用完Bitmap后掉用recycle（），再赋null。
- handler引起的内存泄漏，MessageQueue里的消息如果在activity销毁时没有处理完，就会引起内存的泄漏，可以使用弱引用解决。
- 设置过的监听不用时，及时移除。如在Destroy时及时remove。尤其以addListener开头的，在Destroy中都需要remove。
- activity泄漏可以使用LeakCanary。
- 在内存引用上做些处理，常用的有软引用、弱引用
- 在内存中加载图片时直接在内存中作处理，如：边界压缩
- 动态回收内存
- 优化Dalvik虚拟机的堆内存分配
- 自定义堆内存大小

**184.Handler机制** 

andriod提供了Handler 和 Looper 来满足线程间的通信。Handler先进先出原则。Looper类用来管理特定线程内对象之间的消息交换(MessageExchange)。

Looper: 一个线程可以产生一个Looper对象，由它来管理此线程里的MessageQueue(消息队列)。

Handler: 你可以构造Handler对象来与Looper沟通，以便push新消息到MessageQueue里;或者接收Looper从Message Queue取出)所送来的消息

Message Queue(消息队列):用来存放线程放入的消息。

线程：UIthread 通常就是main thread，而Android启动程序时会替它建立一个MessageQueue。
    
**185.ListView卡顿原因** 

Adapter的getView方法里面convertView没有使用setTag和getTag方式；

在getView方法里面ViewHolder初始化后的赋值或者是多个控件的显示状态和背景的显示没有优化好，抑或是里面含有复杂的计算和耗时操作；

在getView方法里面 inflate的row 嵌套太深（布局过于复杂）或者是布局里面有大图片或者背景所致；

Adapter多余或者不合理的notifySetDataChanged；

listview 被多层嵌套，多次的onMessure导致卡顿，如果多层嵌套无法避免，建议把listview的高和宽设置为fill_parent. 如果是代码继承的listview，那么也请你别忘记为你的继承类添加上LayoutPrams，注意高和宽都是fill_parent的；

**186.Json有什么优劣势**

1.JSON的速度要远远快于XML

2.JSON相对于XML来讲，数据的体积小

3.JSON对数据的描述性比XML较差

**187.AndroidManifest的作用与理解**

**188.Android中开启摄像头的主要步骤**

**189.AlertDialog,popupWindow,Activity区别**

**190.LaunchMode应用场景**

standard，创建一个新的Activity。

singleTop，栈顶不是该类型的Activity，创建一个新的Activity。否则，onNewIntent。

singleTask，回退栈中没有该类型的Activity，创建Activity，否则，onNewIntent+ClearTop。

注意:

设置了"singleTask"启动模式的Activity，它在启动的时候，会先在系统中查找属性值affinity等于它的属性值taskAffinity的Task存在；如果存在这样的Task，它就会在这个Task中启动，否则就会在新的任务栈中启动。因此， 如果我们想要设置了"singleTask"启动模式的Activity在新的任务中启动，就要为它设置一个独立的taskAffinity属性值。

如果设置了"singleTask"启动模式的Activity不是在新的任务中启动时，它会在已有的任务中查看是否已经存在相应的Activity实例， 如果存在，就会把位于这个Activity实例上面的Activity全部结束掉，即最终这个Activity 实例会位于任务的Stack顶端中。

在一个任务栈中只有一个”singleTask”启动模式的Activity存在。他的上面可以有其他的Activity。这点与singleInstance是有区别的。

singleInstance，回退栈中，只有这一个Activity，没有其他Activity。

singleTop适合接收通知启动的内容显示页面。

例如，某个新闻客户端的新闻内容页面，如果收到10个新闻推送，每次都打开一个新闻内容页面是很烦人的。

singleTask适合作为程序入口点。

例如浏览器的主界面。不管从多少个应用启动浏览器，只会启动主界面一次，其余情况都会走onNewIntent，并且会清空主界面上面的其他页面。

singleInstance应用场景：

闹铃的响铃界面。 你以前设置了一个闹铃：上午6点。在上午5点58分，你启动了闹铃设置界面，并按 Home 键回桌面；在上午5点59分时，你在微信和朋友聊天；在6点时，闹铃响了，并且弹出了一个对话框形式的 Activity(名为 AlarmAlertActivity) 提示你到6点了(这个 Activity 就是以 SingleInstance 加载模式打开的)，你按返回键，回到的是微信的聊天界面，这是因为 AlarmAlertActivity 所在的 Task 的栈只有他一个元素， 因此退出之后这个 Task 的栈空了。如果是以 SingleTask 打开 AlarmAlertActivity，那么当闹铃响了的时候，按返回键应该进入闹铃设置界面。

**191.AsyncTask 如何使用?**

**192.SpareArray原理**

**193.请介绍下ContentProvider 是如何实现数据共享的？**

**194.Android Service 与 Activity 之间通信的几种方式**

- 通过Binder对象

- 通过broadcast(广播)的形式

**195.IntentService原理及作用是什么？**

**196.说说Activity、Intent、Service 是什么关系**

**197.ApplicationContext和ActivityContext的区别**

**198.SP是进程同步的吗?有什么方法做到同步？**

**199.进程和 Application 的生命周期**

**200.封装View的时候怎么知道view的大小**

**201.AndroidManifest的作用与理解**

**202.Handler、Thread和HandlerThread的差别**

**203.handler发消息给子线程，looper怎么启动？**

**204.关于Handler，在任何地方new Handler 都是什么线程下?**

**205.ThreadLocal的原理**

ThreadLocal是一个关于创建线程局部变量的类。使用场景如下所示：

实现单个线程单例以及单个线程上下文信息存储，比如交易id等。

实现线程安全，非线程安全的对象使用ThreadLocal之后就会变得线程安全，因为每个线程都会有一个对应的实例。
承载一些线程相关的数据，避免在方法中来回传递参数。

**206.请解释下在单线程模型中Message、Handler、Message Queue、Looper之间的关系**

**207.ANR定位和修正**

如果开发机器上出现问题，我们可以通过查看/data/anr/traces.txt即可，最新的ANR信息在最开始部分。

主线程被IO操作（从4.0之后网络IO不允许在主线程中）阻塞。
主线程中存在耗时的计算
主线程中错误的操作，比如Thread.wait或者Thread.sleep等 Android系统会监控程序的响应状况，一旦出现下面两种情况，则弹出ANR对话框
应用在5秒内未响应用户的输入事件（如按键或者触摸）
BroadcastReceiver未在10秒内完成相关的处理
Service在特定的时间内无法处理完成 20秒

使用AsyncTask处理耗时IO操作。

使用Thread或者HandlerThread时，调用Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND)设置优先级，否则仍然会降低程序响应，因为默认Thread的优先级和主线程相同。

使用Handler处理工作线程结果，而不是使用Thread.wait()或者Thread.sleep()来阻塞主线程。

Activity的onCreate和onResume回调中尽量避免耗时的代码
BroadcastReceiver中onReceive代码也要尽量减少耗时，建议使用IntentService处理。

**208.oom是什么？什么情况导致oom？**

http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0920/3478.html

1）使用更加轻量的数据结构 

2）Android里面使用Enum

3）Bitmap对象的内存占用 

4）更大的图片

5）onDraw方法里面执行对象的创建

6）StringBuilder

**209.有什么解决方法可以避免OOM？**

**210.Oom 是否可以try catch？为什么？**

**211.什么情况导致内存泄漏？**

1.资源对象没关闭造成的内存泄漏

描述： 资源性对象比如(Cursor，File文件等)往往都用了一些缓冲，我们在不使用的时候，应该及时关闭它们，以便它们的缓冲及时回收内存。它们的缓冲不仅存在于 java虚拟机内，还存在于java虚拟机外。如果我们仅仅是把它的引用设置为null,而不关闭它们，往往会造成内存泄漏。因为有些资源性对象，比如SQLiteCursor(在析构函数finalize(),如果我们没有关闭它，它自己会调close()关闭)，如果我们没有关闭它，系统在回收它时也会关闭它，但是这样的效率太低了。因此对于资源性对象在不使用的时候，应该调用它的close()函数，将其关闭掉，然后才置为null.在我们的程序退出时一定要确保我们的资源性对象已经关闭。

程序中经常会进行查询数据库的操作，但是经常会有使用完毕Cursor后没有关闭的情况。如果我们的查询结果集比较小，对内存的消耗不容易被发现，只有在常时间大量操作的情况下才会复现内存问题，这样就会给以后的测试和问题排查带来困难和风险。

2.构造Adapter时，没有使用缓存的convertView

描述： 以构造ListView的BaseAdapter为例，在BaseAdapter中提供了方法： public View getView(int position, ViewconvertView, ViewGroup parent) 来向ListView提供每一个item所需要的view对象。初始时ListView会从BaseAdapter中根据当前的屏幕布局实例化一定数量的 view对象，同时ListView会将这些view对象缓存起来。当向上滚动ListView时，原先位于最上面的list item的view对象会被回收，然后被用来构造新出现的最下面的list item。这个构造过程就是由getView()方法完成的，getView()的第二个形参View convertView就是被缓存起来的list item的view对象(初始化时缓存中没有view对象则convertView是null)。由此可以看出，如果我们不去使用 convertView，而是每次都在getView()中重新实例化一个View对象的话，即浪费资源也浪费时间，也会使得内存占用越来越大。 ListView回收list item的view对象的过程可以查看: android.widget.AbsListView.java --> voidaddScrapView(View scrap) 方法。 示例代码：

    public View getView(int position, ViewconvertView, ViewGroup parent) {
    View view = new Xxx(...); 
    ... ... 
    return view; 
    }
    
修正示例代码：

    public View getView(int position, ViewconvertView, ViewGroup parent) {
    View view = null; 
    if (convertView != null) { 
    view = convertView; 
    populate(view, getItem(position)); 
    ... 
    } else { 
    view = new Xxx(...); 
    ... 
    } 
    return view; 
    }
    
3.Bitmap对象不在使用时调用recycle()释放内存

描述： 有时我们会手工的操作Bitmap对象，如果一个Bitmap对象比较占内存，当它不在被使用的时候，可以调用Bitmap.recycle()方法回收此对象的像素所占用的内存，但这不是必须的，视情况而定。可以看一下代码中的注释：

/* •Free up the memory associated with thisbitmap's pixels, and mark the •bitmap as "dead", meaning itwill throw an exception if getPixels() or •setPixels() is called, and will drawnothing. This operation cannot be •reversed, so it should only be called ifyou are sure there are no •further uses for the bitmap. This is anadvanced call, and normally need •not be called, since the normal GCprocess will free up this memory when •there are no more references to thisbitmap. /

4.试着使用关于application的context来替代和activity相关的context

这是一个很隐晦的内存泄漏的情况。有一种简单的方法来避免context相关的内存泄漏。最显著地一个是避免context逃出他自己的范围之外。使用Application context。这个context的生存周期和你的应用的生存周期一样长，而不是取决于activity的生存周期。如果你想保持一个长期生存的对象，并且这个对象需要一个context,记得使用application对象。你可以通过调用 Context.getApplicationContext() or Activity.getApplication()来获得。更多的请看这篇文章如何避免 Android内存泄漏。

5.注册没取消造成的内存泄漏

一些Android程序可能引用我们的Anroid程序的对象(比如注册机制)。即使我们的Android程序已经结束了，但是别的引用程序仍然还有对我们的Android程序的某个对象的引用，泄漏的内存依然不能被垃圾回收。调用registerReceiver后未调用unregisterReceiver。 比如:假设我们希望在锁屏界面(LockScreen)中，监听系统中的电话服务以获取一些信息(如信号强度等)，则可以在LockScreen中定义一个 PhoneStateListener的对象，同时将它注册到TelephonyManager服务中。对于LockScreen对象，当需要显示锁屏界面的时候就会创建一个LockScreen对象，而当锁屏界面消失的时候LockScreen对象就会被释放掉。 但是如果在释放 LockScreen对象的时候忘记取消我们之前注册的PhoneStateListener对象，则会导致LockScreen无法被垃圾回收。如果不断的使锁屏界面显示和消失，则最终会由于大量的LockScreen对象没有办法被回收而引起OutOfMemory,使得system_process 进程挂掉。 虽然有些系统程序，它本身好像是可以自动取消注册的(当然不及时)，但是我们还是应该在我们的程序中明确的取消注册，程序结束时应该把所有的注册都取消掉。

6.集合中对象没清理造成的内存泄漏

我们通常把一些对象的引用加入到了集合中，当我们不需要该对象时，并没有把它的引用从集合中清理掉，这样这个集合就会越来越大。如果这个集合是static的话，那情况就更严重了。

**212.ContentProvider的权限管理(解答：读写分离，权限控制-精确到表级，URL控制)**

**213.如何通过广播拦截和abort一条短信？**

**214.广播是否可以请求网络？**

**215.计算一个view的嵌套层级**

**216.Android线程有没有上限？**

**217.线程池有没有上限？**

**218.ListView重用的是什么？**

**219.Android为什么引入Parcelable？**

所谓序列化就是将对象变成二进制流，便于存储和传输。

Serializable是java实现的一套序列化方式，可能会触发频繁的IO操作，效率比较低，适合将对象存储到磁盘上的情况。
Parcelable是Android提供一套序列化机制，它将序列化后的字节流写入到一个共性内存中，其他对象可以从这块共享内存中读出字节流，并反序列化成对象。因此效率比较高，适合在对象间或者进程间传递信息。

**220.有没有尝试简化Parcelable的使用？**

**221.Android进程分类？**

Android的进程主要分为以下几种：

前台进程

用户当前操作所必需的进程。如果一个进程满足以下任一条件，即视为前台进程：

托管用户正在交互的 Activity（已调用 Activity 的 onResume() 方法）
托管某个 Service，后者绑定到用户正在交互的 Activity
托管正在“前台”运行的 Service（服务已调用 startForeground()）
托管正执行一个生命周期回调的 Service（onCreate()、onStart() 或 onDestroy()）
托管正执行其 onReceive() 方法的 BroadcastReceiver
通常，在任意给定时间前台进程都为数不多。只有在内存不足以支持它们同时继续运行这一万不得已的情况下，系统才会终止它们。 此时，设备往往已达到内存分页状态，因此需要终止一些前台进程来确保用户界面正常响应。

可见进程

没有任何前台组件、但仍会影响用户在屏幕上所见内容的进程。 如果一个进程满足以下任一条件，即视为可见进程：

托管不在前台、但仍对用户可见的 Activity（已调用其 onPause() 方法）。例如，如果前台 Activity 启动了一个对话框，允许在其后显示上一 Activity，则有可能会发生这种情况。
托管绑定到可见（或前台）Activity 的 Service。
可见进程被视为是极其重要的进程，除非为了维持所有前台进程同时运行而必须终止，否则系统不会终止这些进程。

服务进程

正在运行已使用 startService() 方法启动的服务且不属于上述两个更高类别进程的进程。尽管服务进程与用户所见内容没有直接关联，但是它们通常在执行一些用户关 心的操作（例如，在后台播放音乐或从网络下载数据）。因此，除非内存不足以维持所有前台进程和可见进程同时运行，否则系统会让服务进程保持运行状态。

后台进程

包含目前对用户不可见的 Activity 的进程（已调用 Activity 的 onStop() 方法）。这些进程对用户体验没有直接影响，系统可能随时终止它们，以回收内存供前台进程、可见进程或服务进程使用。 通常会有很多后台进程在运行，因此它们会保存在 LRU （最近最少使用）列表中，以确保包含用户最近查看的 Activity 的进程最后一个被终止。如果某个 Activity 正确实现了生命周期方法，并保存了其当前状态，则终止其进程不会对用户体验产生明显影响，因为当用户导航回该 Activity 时，Activity 会恢复其所有可见状态。

空进程

不含任何活动应用组件的进程。保留这种进程的的唯一目的是用作缓存，以缩短下次在其中运行组件所需的启动时间。 为使总体系统资源在进程缓存和底层内核缓存之间保持平衡，系统往往会终止这些进程。

ActivityManagerService负责根据各种策略算法计算进程的adj值，然后交由系统内核进行进程的管理。

SharePreference性能优化，可以做进程同步吗？
在Android中, SharePreferences是一个轻量级的存储类，特别适合用于保存软件配置参数。使用SharedPreferences保存数据，其背后是用xml文件存放数据，文件 存放在/data/data/ < package name > /shared_prefs目录下.

之所以说SharedPreference是一种轻量级的存储方式，是因为它在创建的时候会把整个文件全部加载进内存，如果SharedPreference文件比较大，会带来以下问题：

第一次从sp中获取值的时候，有可能阻塞主线程，使界面卡顿、掉帧。
解析sp的时候会产生大量的临时对象，导致频繁GC，引起界面卡顿。
这些key和value会永远存在于内存之中，占用大量内存。
优化建议

不要存放大的key和value，会引起界面卡、频繁GC、占用内存等等。
毫不相关的配置项就不要放在在一起，文件越大读取越慢。
读取频繁的key和不易变动的key尽量不要放在一起，影响速度，如果整个文件很小，那么忽略吧，为了这点性能添加维护成本得不偿失。
不要乱edit和apply，尽量批量修改一次提交，多次apply会阻塞主线程。
尽量不要存放JSON和HTML，这种场景请直接使用JSON。
SharedPreference无法进行跨进程通信，MODE_MULTI_PROCESS只是保证了在API 11以前的系统上，如果sp已经被读取进内存，再次获取这个SharedPreference的时候，如果有这个flag，会重新读一遍文件，仅此而已。

**222.Fragment的replace和add的区别？**

**223.MVP，MVVM，MVC解释和实践**

**224.项目之外的，对技术的见解，拓展知识**

**225.SharedPrefrences的apply和commit有什么区别**

**226.MD5是加密方法么，Base64呢**

**227.有博客和github，主要是写的什么？有哪些关注**

**228.HttpClient和HttpConnection的区别**

**229.除了日常开发，其他有做过什么工作？比如持续化集成，自动化测试等等**

**230.ActivityA跳转ActivityB然后B按back返回A，各自的生命周期顺序，A与B均不透明。**

**231.Android中main方法入口在哪里**

**232.写出你认为最优的懒汉式单例模式**

**233.activity意外退出时信息的储存与恢复，onCreate正常进入时的判断。**

**234.抽象类能否实例化，理论依据是什么？**

**235.如何通过Gradle配置差异较大(20%差异)的多渠道包
class文件如何转化成dex**

**236.Service先start再bind如何关闭service，为什么bindService可以跟Activity生命周期联动？**

**237.直接在Activity中创建一个thread跟在service中创建一个thread之间恩区别**

**238.ListView针对多种item的缓存是如何实现的**

**239.Android绘制二维跟三维的View的区别**

**240.是否了解硬件加速**

**241.ListView是如何实现对不同type的item的管理的**

**242.Android为什么要设计两种classloader，为什么不用一种，通过type来区分**

**243.Bundle传递数据为什么需要序列化**

**244.广播传输的数据是否有限制，是多少，为什么要限制？**

**245.关于反射混淆，耗性能的解决方式**

**246.RecyclerView的itemdecoration如何处理点击事件**

**247.编译期注解跟运行时注解**

**248.Canvas.save()跟Canvas.restore()的调用时机**




