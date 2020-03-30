---
title: Java基础
date: 2019-03-13 10:52:45
tags:
  - Java
  - 面试
toc: true
categories:
  - 面试
comments: true
---

<!--more-->

##### 1.hashcode相等的两个类一定相等吗？equals呢？相反呢？

1.equal()相等的两个对象他们的hashCode()肯定相等，也就是用equal()对比是绝对可靠的。 
2.hashCode()相等的两个对象他们的equal()不一定相等，也就是hashCode()不是绝对可靠的。 
对于需要大量并且快速的对比的话如果都用equal()去做显然效率太低，所以解决方式是，每当需要对比的时候，首先用hashCode()去对比，如果hashCode()不一样，则表示这两个对象肯定不相等（也就是不必再用equal()去再对比了）,如果hashCode()相同，此时再对比他们的equal()，如果equal()也相同，则表示这两个对象是真的相同了，这样既能大大提高了效率也保证了对比的绝对正确性。

##### 2.介绍一下集合框架

分为三块：List列表、Set集合、Map映射

**Set、List和Map可以看做集合的三大类：**
**List集合是有序集合，集合中的元素可以重复，访问集合中的元素可以根据元素的索引来访问。**
**Set集合是无序集合，集合中的元素不可以重复，访问集合中的元素只能根据元素本身来访问（也是集合里元素不允许重复的原因）。**
**Map集合中保存Key-value对形式的元素，访问时只能根据每项元素的key来访问其value。** 

**1.List接口**
List接口继承于Collection接口，它可以定义一个**允许重复**的**有序集合**。因为List中的元素是有序的，所以我们可以通过使用索引（元素在List中的位置，类似于数组下标）来访问List中的元素，这类似于Java的数组。

List接口为Collection直接接口。List所代表的是**有序的Collection**，即它用某种特定的插入顺序来维护元素顺序。用户可以对列表中每个元素的插入位置进行精确地控制，同时可以根据元素的整数索引（在列表中的位置）访问元素，并搜索列表中的元素。实现List接口的集合主要有：ArrayList、LinkedList、Vector、Stack。

**Map一种映射，用于储存关系型数据，保存着两种值，一组用于保存key,另一组用来保存value。并且key不允许重复。** 
**HashMap底层就是一个数组，数组中根据存入的Key的HashCode决定存放位置，其Entry单元中有四个属性，分别为HashCode，Key，Vaule，和下一个Entry**，这样就形成了一个链表，当HashMap中的另一个拥有相同的HashCode值的不同的Key存入时，会将原来的Entry赋到新Entry的属性中，然后形成Entry链，查询的时候先比较HashCode，如果相同且Key值相同则直接取出，如果HashCode相同Key值不同则继续顺着链表寻找直到寻找到相同的Key值。 
TreeMap与HashMap的不同：表象上时TreeMap可以对Key进行排序，原因时TreeMap使用的时“红黑树”的二叉树结构储存Entry，也就是排序二叉树，左边恒放比此值小的数右边恒放比此值大的树，按照当前节点值与传入查询值的比较进行判断决定其存放位置/查询其数值； 
Set集合：Set与Map可以手动的互相转换 Set转换Map只需要新建一个对象，对象中又key和value两个属性，新建一个类继承Set存储新建的对象即可实现。Map转换为Set只需要将Map的Value固定，只使用Key存储数据即可实现； 
Table:Map的线程安全型号；

##### 3.hashmap，hashtable底层实现的区别，hashtable和concurrenthashtable呢？

hashtable简单的理解就是hashmap的线程安全类 其方法大部分都相同只不过家了synchronize关键字保证其线程安全。其他的区别也有继承的接口不同这点。 
concurrenthashtable则是改进了hashtable的效率，hashtable虽然安全但是不能多线程同时操作，concurrenthashtable使用了分块的模式支持多线程操作，且使用了lock替换synchronize来提高了效率。

##### 4.hashmap与treemap的区别，底层数据结构是什么样的？

HashMap：数组方式存储key/value，线程非安全，允许null作为key和value，key不可以重复，value允许重复，不保证元素迭代顺序是按照插入时的顺序，key的hash值是先计算key的hashcode值，然后再进行计算，每次容量扩容会重新计算所以key的hash值，会消耗资源，要求key必须重写equals和hashcode方法

默认初始容量16，加载因子0.75，扩容为旧容量乘2，查找元素快，如果key一样则比较value，如果value不一样，则按照链表结构存储value，就是一个key后面有多个value；

TreeMap：基于红黑二叉树的NavigableMap的实现，线程非安全，不允许null，key不可以重复，value允许重复，存入TreeMap的元素应当实现Comparable接口或者实现Comparator接口，会按照排序后的顺序迭代元素，两个相比较的key不得抛出classCastException。主要用于存入元素的时候对元素进行自动排序，迭代输出的时候就按排序顺序输出  

##### 5.线程池用过么，都有什么参数？底层如何实现的？

1. newSingleThreadExecutor

创建一个单线程的线程池。这个线程池只有一个线程在工作，也就是相当于单线程串行执行所有任务。如果这个唯一的线程因为异常结束，那么会有一个新的线程来替代它。此线程池保证所有任务的执行顺序按照任务的提交顺序执行。

2.newFixedThreadPool

创建固定大小的线程池。每次提交一个任务就创建一个线程，直到线程达到线程池的最大大小。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。

3. newCachedThreadPool

创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲（60秒不执行任务）的线程，当任务数增加时，此线程池又可以智能的添加新线程来处理任务。此线程池不会对线程池大小做限制，线程池大小完全依赖于操作系统（或者说JVM）能够创建的最大线程大小。

4.newScheduledThreadPool

创建一个大小无限的线程池。此线程池支持定时以及周期性执行任务的需求。

线程池： 
1、线程是稀缺资源，使用线程池可以减少创建和销毁线程的次数，每个工作线程都可以重复使用。 
2、可以根据系统的承受能力，调整线程池中工作线程的数量，防止因为消耗过多内存导致服务器崩溃。 
参数： 

一般来说，非CPU密集型的业务（加解密、压缩解压缩、搜索排序等业务是CPU密集型的业务），瓶颈都在后端数据库，本地CPU计算的时间很少，所以设置几十或者几百个工作线程也都是可能的。

N核服务器，通过执行业务的单线程分析出本地计算时间为x，等待时间为y，则工作线程数（线程池线程数）设置为 N*(x+y)/x，能让CPU的利用率最大化。

corePoolSize - 池中所保存的线程数，包括空闲线程。

maximumPoolSize-池中允许的最大线程数。

keepAliveTime - 当线程数大于核心时，此为终止前多余的空闲线程等待新任务的最长时间。

unit - keepAliveTime 参数的时间单位。

workQueue - 执行前用于保持任务的队列。此队列仅保持由 execute方法提交的 Runnable任务。

threadFactory - 执行程序创建新线程时使用的工厂。

handler - 由于超出线程范围和队列容量而使执行被阻塞时所使用的处理程序。

底层核心实现为封装一层线程类work，在运行的时候再执行完自己的线程后主动去队列中拿取下一条线程去执行。

#####  Thread 类中的start() 和 run() 方法有什么区别？

这个问题经常被问到，但还是能从此区分出面试者对Java线程模型的理解程度。start()方法被用来启动新创建的线程，而且start()内部调用了run()方法，这和直接调用run()方法的效果不一样。当你调用run()方法的时候，只会是在原来的线程中调用，没有新的线程启动，start()方法才会启动新线程。

##### RPC

RPC就是从一台机器（客户端）上通过参数传递的方式调用另一台机器（服务器）上的一个函数或方法（可以统称为服务）并得到返回的结果。

RPC会隐藏底层的通讯细节（不需要直接处理Socket通讯或Http通讯）。

客户端发起请求，服务器返回响应（类似于Http的工作方式）RPC在使用形式上像调用本地函数（或方法）一样去调用远程的函数（或方法）。

客户端把参数先转成一个字节流，传给服务端后，再把字节流转成自己能读取的格式。这个过程叫序列化和反序列化。

RPC 调用分以下两种：

1. 同步调用
   客户方等待调用执行完成并返回结果。
2. 异步调用
   客户方调用后不用等待执行结果返回，但依然可以通过回调通知等方式获取返回结果。 若客户方不关心调用返回结果，则变成单向异步调用，单向调用不用返回结果。

异步和同步的区分在于是否等待服务端执行完成并返回结果。

##### java读取xml文件的四种方法

第一种 DOM 实现方法：

第二种 DOM4J实现方法

第三种 JDOM实现方法：

第四种 SAX实现方法：

##### Tomcat内部工作原理

Connector负责接受客户的请求并向客户返回响应，是Tomcat接收请求的入口，每个Connector有自己专属的监听端口, 在同一个Service中，多个Connector共享一个Engine。同一个Engine有多个Host,Engine负责处理Service内的所有请求。它接收来自Connector的请求，并决定传给哪个Host来处理，Host处理完请求后，将结果返回给Engine，Engine再将结果返回给Connector，同一个Host有多个Context



#### Redis

##### Redis有哪些数据结构

字符串String、字典Hash、列表List、集合Set、有序集合SortedSet。

##### 3.使用redis有哪些好处？

(1) **速度快**，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)；
(2) **支持丰富数据类型**，支持string，list，set，sorted set，hash；
(3) **支持事务**，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行；
(4) **丰富的特性**：可用于缓存，消息，按key设置过期时间，过期后将会自动删除；

##### Redis分布式锁

分布式中的锁，通常不会是一个对象，而是一个唯一的数据，可以是商户的订单号，商户的编码，或者是一条数据的唯一主键等等。

synchronized主要对于单个应用中，多线程的同步；

而分布式锁对应的是多个应用，每个应用中都可能会处理相同的数据，所以需要对对个应用的数据进行同步，保证数据的一致性；

分布式锁一般有三种实现方式：1. 数据库乐观锁；2. 基于Redis的分布式锁；3. 基于ZooKeeper的分布式锁。

- 基于 DB 的唯一索引。
- 基于 ZK 的临时有序节点。
- 基于 Redis 的 `NX EX` 参数。

向redis中添加一个key，添加的操作是原子性操作，key不存在才能添加成功；

在redis实现的分布式锁中，我们需要强调以下几点，只有保证了以下几点，才可说是确保了锁的实现：

- 高性能(加、解锁时高性能)
- 可以使用阻塞锁与非阻塞锁。
- 不能出现死锁。
- 可用性(不能出现节点 down 掉后加锁失败)。

利用 `Redis set key` 时的一个 NX 参数可以保证在这个 key 不存在的情况下写入成功。并且再加上 EX 参数可以让该 key 在超时之后自动删除。

#### Maven

Maven主要服务于**基于Java平台的项目构建、依赖管理和项目信息管理**...

##### Maven常见的依赖范围有哪些?

compile:编译依赖，默认的依赖方式，在编译,测试,运行三个阶段都有效，典型地有spring-core等jar。 
test:测试依赖，只在编译测试用例和运行测试用例有效，典型地有JUnit。 
provided:对于编译和测试有效，不会打包进发布包中，典型的例子为servlet-api,一般的web工程运行时都使用容器的servlet-api。 
runtime:只在运行测试用例和实际运行时有效，典型地是jdbc驱动jar包。 
system: 不从maven仓库获取该jar,而是通过systemPath指定该jar的路径。 
import: 用于一个dependencyManagement对另一个dependencyManagement的继承。

##### 如何设置本地仓库和远程仓库

4.1 配置本地仓库

​        用文本编辑器工具打开setting.xml文件，然后配置自定义地址：

```
<localRepository>
    F:\DevInstall\yuxx-meven-libs
</localRepository>
```

4.2 配置远程仓库

​        Maven默认的远程地址是：<http://my.repository.com/repo/path>，这个地址是国外网站，下载速度很慢，这里推荐我国阿里云的地址。

用文本编辑器工具打开setting.xml文件：

```
<mirrors>
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror> 
  </mirrors>
```

排除依赖

 将不需要依赖的传递依赖排除掉

  在<dependency>标签中添加子标签<exclusions>和<exclusion>将某传递依赖的jar包排除掉

 版本锁定（推荐使用）

  直接指定所依赖的jar包版本。

  使用<dependencyManagement>标签，在改标签中明确指定所依赖的jar包的版本。

##### java 常见的几种运行时异常RuntimeException

常见的几种如下：

NullPointerException - 空指针引用异常
ClassCastException - 类型强制转换异常。
IllegalArgumentException - 传递非法参数异常。
ArithmeticException - 算术运算异常
ArrayStoreException - 向数组中存放与声明类型不兼容对象异常
IndexOutOfBoundsException - 下标越界异常
NegativeArraySizeException - 创建一个大小为负数的数组错误异常
NumberFormatException - 数字格式异常
SecurityException - 安全异常
UnsupportedOperationException - 不支持的操作异常

##### 重载与重写的区别

方法重载是指同一个类中的多个方法具有相同的名字,但这些方法具有不同的参数列表,即参数的数量或参数类型不能完全相同    在方法的参数不同的情况下，方法的返回类型可以不相同。

方法重写是存在子父类之间的,子类定义的方法与父类中的方法要求返回值、方法名和参数都相同。

##### ArrayList、LinkedList、Vector的区别

**List的三个子类的特点**

**ArrayList:**

- 底层数据结构是数组，查询快，增删慢。
- 线程不安全，效率高。

**Vector:**

- 底层数据结构是数组，查询快，增删慢。
- 线程安全，效率低。
- Vector相对ArrayList查询慢(线程安全的)。
- Vector相对LinkedList增删慢(数组结构)。

**LinkedList**

- 底层数据结构是链表，查询慢，增删快。
- 线程不安全，效率高。

**Vector和ArrayList的区别**

- Vector是线程安全的,效率低。
- ArrayList是线程不安全的,效率高。
- 共同点:底层数据结构都是数组实现的,查询快,增删慢。

**ArrayList和LinkedList的区别**

- ArrayList底层是数组结果,查询和修改快。
- LinkedList底层是链表结构的,增和删比较快,查询和修改比较慢。

**共同点:都是线程不安全的**

**List有三个子类使用**

- 查询多用ArrayList。
- 增删多用LinkedList。
- 如果都多ArrayList。

##### String、StringBuffer与StringBuilder的区别

String：适用于少量的字符串操作的情况。 StringBuilder：适用于单线程下在字符缓冲区进行大量操作的情况。 StringBuffer：适用多线程下在字符缓冲区进行大量操作的情况。 StringBuilder：是线程不安全的，而StringBuffer是线程安全的。

这三个类之间的区别主要是在两个方面，**即运行速度和线程安全**这两方面。 首先说运行速度，或者说是执行速度，在这方面运行速度快慢为：**StringBuilder > StringBuffer > String**。

String最慢的原因

String为字符串常量，而StringBuilder和StringBuffer均为字符串变量，即String对象一旦创建之后该对象是不可更改的，但后两者的对象是变量，是可以更改的。

再来说线程安全

在线程安全上，**StringBuilder是线程不安全的，而StringBuffer是线程安全的**。

如果一个StringBuffer对象在字符串缓冲区被多个线程使用时，StringBuffer中很多方法可以带有synchronized关键字，所以可以保证线程是安全的，但StringBuilder的方法则没有该关键字，所以不能保证线程安全，有可能会出现一些错误的操作。所以如果要进行的操作是多线程的，那么就要使用StringBuffer，但是在单线程的情况下，还是建议使用速度比较快的StringBuilder。

##### Java内存分配策略

堆内存、虚拟机栈、方法区、本地方法栈、程序计数器。

静态分配---静态存储区（方法区）：主要存放静态数据，全局static

栈式分配---栈区：当方法执行时，方法内部的局部变量都建立在栈内存中，并在方法结束后自动释放分配的内存。因为栈内存分配是在处理器的指令集当中所以效率很高，但是分配的内存容量有限。

**在方法体内定义的（局部变量）一些基本类型的变量和对象的引用变量都在方法的栈内存中分配。**当在一段方法块中定义一个变量时，Java就会在栈中为其分配内存，当超出变量作用域时，该变量也就无效了，此时占用的内存就会释放，然后会被重新利用。

堆内存用来存放所有new出来的对象(包括该对象内的所有成员变量)和数组。

堆式分配---堆区：Java堆是被所有线程所共享的一块内存区域，在虚拟机启动时创建，目的就是用来存放对象实例，几乎所有的对象都是在这里分配（但随着技术发展，“所有”不是那么绝对了）Java堆是垃圾收集器的主要管理区域，也叫“GC”堆。从内存回收的角度可以将Java堆分为“新生代”，“老年代”

本地方法栈。本地方法栈为虚拟机使用到的Native方法服务。而JAVA的栈内存是为了保存我们编写的临时变量与对象引用。

程序计数器。JAVA其实说到底也就是一条一条的指令。程序计数器个人理解也就是用来指示执行哪条指令的。每个线程都需要有自己独立的程序计数器，并且不能互相被干扰，否则就会影响到程序的正常执行次序。因此，可以这么说，程序计数器是每个线程所私有的。

##### ThreadLocal与Synchronized的区别

​        ThreadLocal和Synchonized都用于解决多线程并发访问。但是ThreadLocal与synchronized有本质的区别。**synchronized是利用锁的机制，使变量或代码块在某一时该只能被一个线程访问。而ThreadLocal为每一个线程都提供了变量的副本，这样就隔离了多个线程对数据的数据共享。而Synchronized却正好相反，它用于在多个线程间通信时能够获得数据共享。**

**一句话，ThreadLocal用于数据隔离，而Synchronized用于数据共享。**

**Synchronized用于线程间的数据共享（使变量或代码块在某一时该只能被一个线程访问），是一种以延长访问时间来换取线程安全性的策略；**

**而ThreadLocal则用于线程间的数据隔离（为每一个线程都提供了变量的副本），是一种以空间来换取线程安全性的策略。**

##### 在java方法中改变传递的参数的值

1、对于基本类型参数，在方法体内对参数进行重新赋值，并不会改变原有变量的值。

2、对于引用类型参数，在方法体内对参数进行重新赋予引用，并不会改变原有变量所持有的引用。 

3、方法体内对参数进行运算，不影响原有变量的值。 

4、方法体内对参数所指向对象的属性进行操作，将改变原有变量所指向对象的属性值。 

#### IO

IO是指对数据流的输入和输出，也称为IO流，IO流主要分为两大类，字节流和字符流。字节流可以处理任何类型的数据，如图片，视频等，字符流只能处理字符类型的数据。

IO流的本质是数据传输，并且流是单向的。

1字符=2字节