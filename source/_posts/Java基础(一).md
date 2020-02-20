---
title: Java基础(一)
date: 2019-03-12 16:39:47
tags:
  - Java
  - 面试
toc: true
categories:
  - 面试
comments: true
---

<!--more-->

#### 基础问题

##### 1.hashcode相等两个类一定相等吗？equals呢？相反呢？

重写过 hashcode 和 equals 么，为什么重写equals时必须重写hashCode方法？equals和hashCode都是Object对象中的非final方法，它们设计的目的就是被用来覆盖(override)的，所以在程序设计中还是经常需要处理这两个方法。



##### 2.集合框架

Java集合框架下大致可以分为五个部分：List列表、Set集合、Map映射、迭代器（Iterator、Enumeration）、工具类（Arrays、Collections）。

Java集合类的整体框架如下：![img](Java基础(一)/4676229-f093fafe4fc63713.png)

集合类主要分为两大类：Collection和Map。

##### Collection接口

Collection是List、Set等集合高度抽象出来的接口，它包含了这些集合的基本操作，它主要又分为两大部分：List和Set。

##### List接口

List是有序的Collection，使用此接口能够精确的控制每个元素插入的位置。用户能够使用索引（元素在List中的位置，类似于数组下标）来访问List中的元素，这类似于Java的数组。和下面要提到的Set不同，List允许有相同的元素。

除了具有Collection接口必备的iterator()方法外，List还提供一个listIterator()方法，返回一个ListIterator接口，和标准的Iterator接口相比，ListIterator多了一些add()之类的方法，允许添加，删除，设定元素，还能向前或向后遍历。

实现List接口的常用类有LinkedList，ArrayList，Vector和Stack。

##### Set接口

Set是一种不包含重复的元素的Collection，即任意的两个元素e1和e2都有e1.equals(e2)=false，Set最多有一个null元素。
很明显，Set的构造函数有一个约束条件，传入的Collection参数不能包含重复的元素。请注意：必须小心操作可变对象（Mutable Object）。如果一个Set中的可变元素改变了自身状态导致Object.equals(Object)=true将导致一些问题。

Set接口通常表示一个集合，其中的元素不允许重复（通过hashcode和equals函数保证），常用实现类有HashSet和TreeSet，HashSet是通过Map中的HashMap实现的，而TreeSet是通过Map中的TreeMap实现的。另外，TreeSet还实现了SortedSet接口，因此是有序的集合（集合中的元素要实现Comparable接口，并覆写Compartor函数才行）。

------

我们看到，抽象类AbstractCollection、AbstractList和AbstractSet分别实现了Collection、List和Set接口，这就是在Java集合框架中用的很多的适配器设计模式，用这些抽象类去实现接口，在抽象类中实现接口中的若干或全部方法，这样下面的一些类只需直接继承该抽象类，并实现自己需要的方法即可，而不用实现接口中的全部抽象方法。

- ##### Map接口

请注意，Map没有继承Collection接口，Map提供key到value的映射。一个Map中不能包含相同的key，每个key只能映射一个value。Map接口提供3种集合的视图，Map的内容可以被当作一组key集合，一组value集合，或者一组key-value映射。同样抽象类AbstractMap通过适配器模式实现了Map接口中的大部分函数，TreeMap、HashMap、WeakHashMap等实现类都通过继承AbstractMap来实现，另外，不常用的HashTable直接实现了Map接口，它和Vector都是JDK1.0就引入的集合类。

- ##### Iterator接口

Iterator是遍历集合的迭代器（不能遍历Map，只用来遍历Collection），Collection的实现类都实现了iterator()函数，它返回一个Iterator对象，用来遍历集合，ListIterator则专门用来遍历List。而Enumeration则是JDK1.0时引入的，作用与Iterator相同，但它的功能比Iterator要少，它只能再Hashtable、Vector和Stack中使用。

- ##### Arrays和Collections工具类

Arrays和Collections是用来操作数组、集合的两个工具类，例如在ArrayList和Vector中大量调用了Arrays.Copyof()方法，而Collections中有很多静态方法可以返回各集合类的synchronized版本，即线程安全的版本，当然了，如果要用线程安全的结合类，首选Concurrent并发包下的对应的集合类。

### Collection

#### 1. List

- **Arraylist：** Object数组
- **Vector：** Object数组
- **LinkedList：** 双向循环链表

#### 2. Set

- **HashSet（无序，唯一）:** 基于 HashMap 实现的，底层采用 HashMap 来保存元素。HashMap底层数据结构见下。
- **LinkedHashSet：** LinkedHashSet 继承与 HashSet，并且其内部是通过 LinkedHashMap 来实现的。有点类似于我们之前说的LinkedHashMap 其内部是基于 Hashmap 实现一样，不过还是有一点点区别的。
- **TreeSet（有序，唯一）：** 红黑树(自平衡的排序二叉树。)

### Map

- **HashMap：** JDK1.8之前HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）.JDK1.8以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间
- **LinkedHashMap:** LinkedHashMap 继承自 HashMap，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，LinkedHashMap 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。
- **HashTable:** 数组+链表组成的，数组是 HashMap 的主体，链表则是主要为了解决哈希冲突而存在的
- **TreeMap:** 红黑树（自平衡的排序二叉树）

##### 3.hashmap和hashtable,ConcurrentHashMap底层实现有什么区别？

- hashtable简单的理解就是hashmap的线程安全类 其方法大部分都相同只不过家了synchronize关键字保证其线程安全。其他的区别也有继承的接口不同这点。 

  concurrenthashtable则是改进了hashtable的效率，hashtable虽然安全但是不能多线程同时操作，concurrenthashtable使用了分块的模式支持多线程操作，且使用了lock替换synchronize来提高了效率。

4.hashmap和treemap有什么区别？底层数据结构是什么？

HashMap底层就是一个数组，数组中根据存入的Key的HashCode决定存放位置，其Entry单元中有四个属性，分别为HashCode，Key，Vaule，和下一个Entry，这样就形成了一个链表，当HashMap中的另一个拥有相同的HashCode值的不同的Key存入时，会将原来的Entry赋到新Entry的属性中，然后形成Entry链，查询的时候先比较HashCode，如果相同且Key值相同则直接取出，如果HashCode相同Key值不同则继续顺着链表寻找直到寻找到相同的Key值。 

TreeMap与HashMap的不同：表象上时TreeMap可以对Key进行排序，原因时TreeMap使用的时“红黑树”的二叉树结构储存Entry，也就是排序二叉树，左边恒放比此值小的数右边恒放比此值大的树，按照当前节点值与传入查询值的比较进行判断决定其存放位置/查询其数值；

5.线程池用过吗?都有什么参数？底层如何实现的？
线程池： 
1、线程是稀缺资源，使用线程池可以减少创建和销毁线程的次数，每个工作线程都可以重复使用。 
2、可以根据系统的承受能力，调整线程池中工作线程的数量，防止因为消耗过多内存导致服务器崩溃。 
参数： 
corePoolSize：线程池核心线程数量 
maximumPoolSize:线程池最大线程数量 
keepAliverTime：当活跃线程数大于核心线程数时，空闲的多余线程最大存活时间 
unit：存活时间的单位 
workQueue：存放任务的队列 
handler：超出线程范围和队列容量的任务的处理程序 
底层核心实现为封装一层线程类work，在运行的时候再执行完自己的线程后主动去队列中拿取下一条线程去执行。

6.sychnized和Lock的区别、sychnized什么情况是对象锁，什么情况是全局锁，为什么？

synchronized是java中的一个关键字，用于线程同步。 
1. 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象； 
2. 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象； 
3. 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象； 
4. 修改一个类，其作用的范围是synchronized后面括号括起来的部分，作用的对象是这个类的所有对象。

synchronized修饰不加static的方法，锁是加在单个对象上，不同的对象没有竞争关系；修饰加了static的方法，锁是加载类上，这个类所有的对象竞争一把锁。

Lock是一个java接口 里面有一些实现类，也用于实现线程同步，但是相比较于synchronized，无论功能还是性能都有很大提升，但是要注意需要手动释放。 
性能上由于synchronized在编译时会对代码进行修正，最终由cpu通过调度线程的方式处理线程同步问题，开销很大。而lock使用了Java的unsafe类中的CAS方法实现对线程同步的控制，减小了消耗。 

功能上：synchronized由线程控制，只有等待执行结束和异常释放，本身是可重入的，非公平锁，书写时无需手动释放。lock实现多样，除了最基础的可重入锁ReentrantLock，还有使用读写锁（ReadWriteLock）来实现读写分离，进一步提高效率，ReentrantLock默认使用的是非公平锁（竞争锁，可以设置）。

7.ThreadLocal是什么？底层如何实现？写个例子呗？
是一个解决线程并发问题的一个类，底层实现主要是存有一个map，以线程作为key，范型作为value。可以理解为线程级别的缓存。 
使用起来比较简单 假设我们要实现一个线程级别的缓存。

```java
private static final ThreadLocal<Map> testInt=new ThreadLocal<Map>();
    public static Map getMap(){
    Map s = (Map) testInt.get();
            if(s==null){
                s=new HashMap();
                testInt.set(s);
            }
            return s;
    }
}
```
这样我们就能实现在调用这个方法的时候 获得的map是线程级别的。每一个线程都会获得一个单独的map。


8.volitile的工作原理？
定义： 
java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致的更新，线程应该确保通过排他锁单独获得这个变量。Java语言提供了volatile，在某些情况下比锁更加方便。如果一个字段被声明成volatile，java线程内存模型确保所有线程看到这个变量的值是一致的。 
在多个线程之间能够被共享的变量被称为共享变量。共享变量包括所有的实例变量，静态变量和数组元素。他们都被存放在堆内存中，volatile只作用于共享变量。 
工作原理： 
在汇编时 有volatile修饰符的变量将会被lock，这个lock做两件事：

1.将当前处理器缓存行的数据会写回到系统内存。
2.这个写回内存的操作会引起在其他CPU里缓存了该内存地址的数据无效。
你也可以理解为：volatile修饰的变量在修改时直接修改内存（越过缓存）

### 1.equals方法的几个特性：

equals方法必须满足自反性、对称性、传递性和一致性。

### 2. 为什么函数不能根据返回类型来区分重载？

因为调用时不能指定类型信息，编译器不知道你要调用哪个函数。例如：

```javascript
float max(int a, int b);
int max(int a, int b);
```

当调用 max(1, 2);时无法确定调用的是哪个。

### 3. 抽象方法(abstract)是否可同时是静态的(static)？是否可同时是本地方法(native)？是否可同时被 synchronized修饰？

都不能。抽象方法需要子类重写，而静态的方法是无法被重写的，因此二者是矛盾的；本地方法是由本地代码实现的方法，而抽象方法是没有实现的，也是矛盾的；synchronized 和方法的实现细节有关， 抽象方法不涉及实现细节，因此也是相互矛盾的。

### 4. Math.round(11.5)等于多少？Math.round(- 11.5) 又等于多少?

第一个等于12，第二个等于-11。四舍五入的原理是在参数上加 0.5然后进行取整。

### 5. 关于生成随机数问题：

- Math.random()：令系统随机选取大于等于 0.0 且小于 1.0 的伪随机 double 值，比如要生成1到10之间的随机数：

```javascript
Math.random()*9+1; //  这样生成的有十多为小数
```

写一个生成任意两个数之间的随机整数的方法：

```javascript
public static int getRandom(int start,int end){
    return (int)(Math.random()*(end-start+1))+start;  
 }
```

- Random类：用法如下：

```javascript
Random random = new Random();
//Integer a =random.nextInt(90) + 10;//生成两位随机数
Integer number =random.nextInt(900000) + 100000;//生成六位随机数
```

### 6. 如何获取年月日小时分钟秒？

java 1.8之前：

```javascript
Calendar calendar = Calendar.getInstance();
System.out.println(calendar.get(Calendar.MONTH));// 0-11，所以获取当前月份要加1
```

- java 1.8新增：

```javascript
LocalDateTime localTime = LocalDateTime.now();
System.out.println(localTime.getMonthValue());// 1-12
```

### 7.  如何取得从 1970 年 1 月 1 日 0 时 0 分 0 秒到现在的毫秒数？

```javascript
Calendar.getInstance().getTimeInMillis();  //第一种方式 
System.currentTimeMillis();  //第二种方式
```

### 8. 格式化日期：

```javascript
SimpleDateFormat sdf = new SimpleDateFormat("yyyy/MM/dd"); 
Date date1 = new Date();
System.out.println(sdf.format(date1)); 
// Java 8
DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyy/MM/dd");      
LocalDate date2 = LocalDate.now(); 
System.out.println(date2.format(dtf)); 
```

### 9. 关于Integer的问题：下面这段代码输出结果如何?

```javascript
Integer i1 = 100,i2 = 100,i3 = 150,i4 = 150;
System.out.println(i1 == i2);
System.out.println(i3 == i4);
```

结果是：

```javascript
true
false
```

这是因为Integer在设计的时候用了一个IntegerCache，如果数值在 -128 ~ 127之间，那么就直接从IntegerCache里拿出来用，如果在这个范围之外，就会new新的Integer对象。

### 10. 字符串如何转成基本类型？

使用对应的包装类调用valueOf或者parseXxx方法即可，例如要将字符串转成int类型，方法如下：

```javascript
 String s = "1234";
 int x = Integer.parseInt(s);// 方式一
 int y = Integer.valueOf(s);// 方式二
```

### 11. 动静态代理的区别，什么场景使用？

静态代理通常只代理一个类，动态代理是代理一个接口下的多个实现类。静态代理事先知道要代理的是什么，而动态代理不知道要代理什么东西，只有在运行时才知道。动态代理是实现JDK里的 InvocationHandler接口的 invoke方法，但注意的是代理的是接口，也就是你的业务类必须要实现接口，通过Proxy里的newProxyInstance得到代理对象。  AOP编程就是基于动态代理实现的，比如著名的Spring框架、Hibernate框架等等都是动态代理的使用例子。

### 12.heap(堆)和stack(栈)有什么区别？

有以下几个方面的区别：

- 申请方式 ：stack由系统自动分配。例如声明一个局部变量 int b，系统自动在栈中为b开辟空间 ；heap需要程序员自己以new Object的形式申请，并指明大小。
- 申请后系统的响应 ：只要栈的剩余空间大于所申请空间，系统将为程序提供内存，否则将报异常提示栈溢出；而对于堆而言，操作系统有一个记录空闲内存地址的链表，当系统收到程序的申请时，会遍历该链表，寻找第一个空间大于所申请空间的堆结点，然后将该结点从空闲结点链表中删除，并将该结点的空间分配给程序。另外，由于找到的堆结点的大小不一定正好等于申请的大小，系统会自动的将多余的那部分重新放入空闲链表中。
- 申请大小的限制 ：能从栈获得的空间较小；堆获得的空间比较灵活，也比较大。
- 申请效率的比较： stack由系统自动分配，速度较快，但程序员是无法控制的。 heap由new分配的内存，一般速度比较慢，而且容易产生内存碎片,不过用起来最方便。
- 存储的内容不同：栈中存储引用、局部变量等；堆中存储对象、成员变量等内容。

### 13. Java的类加载器有哪些？

- 根类加载器(Bootstrap)：C++写的 ，看不到源码
- 扩展类加载器(Extension)：加载位置 ：jre\lib\ext中
- 系统(应用)类加载器(System\App)：加载位置 ：classpath中
- 自定义加载器(必须继承ClassLoader)

## 二、JavaWeb基础：

### 1. 说一下原生的JDBC操作数据库的流程。

总共有5个步骤，如下：

- 第一步加载数据库连接驱动：Class.forName()
- 第二步获取数据连接对象：DriverManager.getConnection()
- 第三步根据SQL获取sql会话对象，可以使用 Statement或者PreparedStatement
- 第四步执行SQL处理结果集，执行SQL前如果有参数值就设置参数值setXXX()
- 第五步关闭结果集、关闭会话、关闭连接

### 2. 说说PreparedStatement和Statement的区别。

PreparedStatement接口继承Statement， PreparedStatement 实例包含已编译的 SQL 语句，所以其执行速度要快于 Statement 对象。Statement需要不断地拼接sql语句，而PreparedStatement不用，所以可以防止SQL注入。

### 3. 谈谈关系数据库连接池的机制。

连接池其实就是为数据库建立了一个缓冲池，连接池中的连接数量一直保持一个不少于最小连接数的数量，使用时从连接池拿出连接，用完还给连接池。当数量不够时，数据库会创建一些连接，直到一个最大连接数。

### 4. 什么叫http协议？

HTTP(Hypertext transfer protocol)：超文本传输协议，详细的制定了万维网服务器与客户端间的数据传输的通信规则。HTTP是基于TCP/IP通信协议来传递数据的，属于应用层的面向对象的协议，由于其简捷、快速的方式，适用于分布式超媒体信息系统。

### 5. http常见的状态码有哪些？

常见状态码如下：

- 400 Bad Request  //客户端请求有语法错误，不能被服务器所理解
- 403 Forbidden   //服务器收到请求，但是拒绝提供服务
- 404 Not Found  //请求资源不存在
- 500 Internal Server Error //服务器发生不可预期的错误

### 6. Get请求和Post请求的区别是什么？

Get会把请求时的数据暴露在url中，Post则把提交的数据放置在HTTP包的包体中。所以就这点而言，Post方式更加安全。由于浏览器及服务器的限制，Get方式提交的数据最多只能是1024字节，Post理论上没有限制，可传较大量的数据。

### 7. 请求转发(forward)和重定向(redirect)有什么区别？

区别如下：

- 请求重定向：客户端行为，从本质上讲相当于请求两次，地址栏URL会改变，前一次的请求对象不会保存。举例：A去B局办事，B局说这个事是C管的，然后A就自己去了C局
- 请求转发：服务器行为，是一次请求，地址栏URL不变，会保存转发后的请求对象。 举例：A去B局办事，B局知道这个事是C局管的，B局就联系了C局的人办好了这件事。

### 8. 说说cookie和session的区别。

Cookie 是 web 服务器发送给浏览器的信息，浏览器会在本地一个文件中给每个 web 服务器存储 cookie。以后浏览器再给特定的web服务器发送请求时，同时会发送所有为该服务器存储的cookie。 Session 是存储在 web 服务器端的信息。session [对象存储](https://cloud.tencent.com/product/cos)特定用户会话所需的属性及配置信息。当用户在应用程序的 Web 页之间跳转时，存储在 Session 对象中的变量将不会丢失，而是在整个用户会话中一直存在下去。无论客户端做怎样的设置，session都能够正常工作；当客户端禁用cookie时将无法使用cookie。 在存储的数据量方面：session能够存储任意的java对象，cookie只能存储String类型的对象。

### 9. 什么是jsp，什么是Servlet？jsp和Servlet有什么区别？

Servlet是由 Java提供用于开发 web服务器应用程序的一个组件，运行在服务端，由servlet 容器管理，用来生成动态内容。一个 servlet 实例是实现了Servlet接口的 Java 类，所有自定义的 servlet 必须实现 Servlet 接口。jsp本质上就是一个Servlet，它是Servlet的一种特殊形式（由SUN公司推出），每个jsp页面都是一个servlet实例。 特殊在哪？就是特殊在jsp是html页面中内嵌的Java代码，侧重页面显示。

### 10. 你知道JSP的四大域对象和九大内置对象吗？

四大域对象是：

- pageContext：  page域，指当前页面，在当前jsp页面有效，跳到其它页面失效
- request： request域，指一次请求范围内有效，从http请求到服务器处理结束，返回响应的整个过程  在这个过程中使用forward（请求转发）方式跳转多个jsp，在这些页面里你都可以使用这个变量
- session： session域，指当前会话有效范围，浏览器从打开到关闭过程中，转发、重定向均可以使用
- application： context域，指只能在同一个web中使用，服务器未关闭或者重启，数据就有效

九大内置对象是：  out、request、response、session、application、page、pageContext、exception和config。

### 一、JavaWeb高级：

### 1. 什么叫监听器(listener)?

监听器主要是用来监听特定对象的创建或销毁、属性的变化的，是一个实现特定接口的普通java类。具体实现哪个接口，要看你监听什么内容，比如要监听Request对象的创建或销毁，就实现ServletRequestListener 接口。它是随web应用的启动而启动，只初始化一次，随web应用的停止而销毁。

### 2. 什么叫过滤器(filter)？

就是对servlet请求起到过滤的作用，它在监听器之后，作用在servlet之前。比如编码过滤器，就是经过了该过滤器的请求都会设置成过滤器中指定的编码。过滤器是随web应用启动而启动，只初始化一次，只有当web应用停止或重新部署的时候才销毁。

### 3. 什么叫拦截器(interceptor)？

拦截器类似于fileter ，也是拦截用户的请求。不同的是，它不需要在web.xml中配置，不随WEB应用的启动而启动，是基于JAVA的反射机制和动态代理实现的。只有调用相应的方法时才会调用,在面向切面编程中应用的。

### 4. servlet请求的执行过程是怎样的？

过程是这样的：context-param(初始化配置) --> listener --> filter --> servlet --> interceptor --> 页面。

### 5. 谈谈你对Ajax的认识。

Asynchronous JavaScript and XML的缩写，是一种创建交互式网页应用的的网页开发技术。通过异步提交的方式，可以实现局部刷新，在不更新整个页面的前提下维护数据，提升用户体验度。

### 二、数据库：

### 1. select语句的执行顺序怎样的？

SQL语言不同于其他编程语言的最明显特征是处理代码的执行顺序。在大多数据库语言中，代码按顺序执行，但是SQL语言执行顺序如下：  from --> where --> group by分组 --> 聚合函数 --> having筛选分组 --> 计算所有的表达式 --> select的字段 --> order by排序。

### 2. 你知道聚合函数吗？

聚合函数是对一组值进行计算并返回单一的值的函数，它经常与select 语句中的 group by 子句一同使用。 比如求平均值的聚合函数是avg()。

### 3. 你知道连接查询吗？

连接查询分为内连接和外连接，内连接显示表之间有连接匹配的所有行。外连接又分为左外连接、右外连接和全连接。左外连接就是以左表作为基准进行查询，左表数据会全部显示出来，右表如果和左表匹配的数据则显示相应字段的数据，如果不匹配则显示为null。 右连接是以右表作为基准进行查询，右表数据会全部显示出来，左表如果和右表匹配的数据则显示相应字段的数据，如果不匹配则显示为null。 全连接是先以左表进行左外连接，再以右表进行右外连接。

### 4. 事务有几大特性？分别是什么？

事务有四大特性，ACID。

- 原子性(A)：整个事务中的所有操作，要么全部完成，要么全部不完成。
- 一致性(C)：在事务开始之前和事务结束以后，数据库的完整性约束没有被破坏。
- 隔离性(I)：如果有两个事务，运行在相同的时间内，执行 相同的功能，事务的隔离性将确保每一事务在系统中认为只有该事务在使用系统。
- 持久性(D)：在事务完成以后，该事务所对数据库所作的更改便持久的保存在数据库之中，并不会被回滚。

### 5.mysql中的四种隔离级别是什么？

四种隔离级别如下：

- 读未提交(READ UNCOMMITTED)：未提交读也叫读脏，就是事务可以读取其它事务未提交的数据。
- 读已提交(READ COMMITTED)：读已提交就是在事务未提交之前所做的修改其它事务是不可见的。 在其它数据库系统比如[SQL Server](https://cloud.tencent.com/product/sqlserver)默认的隔离级别就是读已提交。
- 可重复读(REPEATABLE READ)：保证同一个事务中的多次相同的查询的结果是一致的，比如一个事务一开始查询了一条记录然后过了几秒钟又执行了相同的查询，保证两次查询的结果是相同的，可重复读也是mysql的默认隔离级别。
- 可串行化(SERIALIZABLE)：可串行化就是保证读取的范围内没有新的数据插入，比如事务第一次查询得到某个范围的数据，第二次查询也同样得到了相同范围的数据，中间没有新的数据插入到该范围中。

### 6. 对于MySQL性能优化，你知道多少？

我知道的有以下几点：

- 当只要一行数据时使用limit 1 。查询时如果已知会得到一条数据，这种情况下加上 limit 1 会增加性能。因为 [mysql 数据库](https://cloud.tencent.com/product/cdb)引擎会在找到一条结果停止搜索，而不是继续查询下一条是否符合标准直到所有记录查询完毕。
- 选择正确的数据库引擎 。Mysql中有两个引擎，MyISAM和InnoDB，每个引擎有利有弊。 MyISAM 适用于一些大量查询的应用，但对于有大量写功能的应用不是很好。InnoDB是一个非常复杂的存储引擎，对于一些小的应用会比MyISAM还慢，但是在写操作比较多的时候会比较优秀。并且，它支持很多的高级应用，例如：事物。
- 用not exists代替not in 。Not exists可以使用索引，not in不能使用索引。在数据量比较大的操作中不建议使用not in 这种方式。
- 对操作符的优化。尽量不采用不利于索引的操作符 ，如：in    not in    is null    is not null    <>  等 。
- limit 的基数比较大时使用 between 。  例如：select * from admin order by admin_id limit 100000,10  优化为：select * from admin where admin_id between 100000 and 100010 order by admin_id。
- 尽量避免在列上做运算，这样导致索引失效 。  例如：select * from admin where year(admin_time)>2014  优化为： select * from admin where admin_time> '2014-01-01′

# 三、框架篇：

#### (一)、Spring

**1. 谈谈你对spring的理解。**  **答：**Spring是一个轻量的开源框架，为简化企业级应用开发而生，它的核心如下：

- 控制反转(IOC)：传统的java开发模式中，当需要一个对象时，我们会自己使用new或者getInstance等直接或者间接调用构造方法创建一个对象。而在spring开发模式中，spring容器使用了工厂模式为我们创建了所需要的对象，不需要我们自己创建了，直接调用spring提供的对象就可以了，这就是控制反转的思想。 控制反转是一种思想，而不是一种技术。
- 依赖注入(DI)：，spring 使用 javaBean 对象的 set 方法或者带参数的构造方法为我们在创建所需对象时将其属性自动设置所需要的值，这就是依赖注入的思想。依赖注入就是使用了控制反转这种思想的一种技术。
- 面向切面编程(AOP)：在面向对象编程（oop）思想中，我们将事物纵向抽成一个个的对象。而在面向切面编程中，我们将一个个的对象某些类似的方面横向抽成一个切面，对这个切面进行一些如权限控制、事物管理，记录日志等公用操作处理，这就是面向切面编程的思想。AOP底层是动态代理，如果是接口采用的是JDK动态代理，如果是类采用的是CGLIB方式实现动态代理。

**2. 你知道spring框架中使用了哪些设计模式吗？**  **答：**spring中使用到的部分设计模式如下：

- 单例模式：在spring的配置文件中设置bean默认为单例模式。
- 模板方法模式：用来解决重复代码，JpaTemplate 、RedisTemplate等。
- 前端控制器模式：spring提供了前端控制器DispatherServlet来对请求进行分发。
- 工厂模式：Spring中使用beanFactory来创建对象的实例，就是用的工厂模式。

**3. 介绍一下spring bean的生命周期。**  **答：**bean的生命周期为 创建 --> 初始化 --> 调用 --> 销毁。

**4. 说一说spring有哪些核心模块？**  **答：**主要有以下七大核心模块：  

![img](Java基础(一)/menjqm3dgw.png)

七大核心模块

- Core模块：封装了框架依赖的最底层部分，包括资源访问、类型转换及一些常用工具类。
- Context模块：以Core和Beans为基础，集成Beans模块功能并添加资源绑定、数据验证、国际化、Java EE支持、容器生命周期、事件传播等，核心接口是ApplicationContext。
- AOP模块：面向切面编程，通过配置管理特性，spring AOP直接将面向切面的编程集中到了框架中，所以可以很容易使spring管理的对象支持AOP。
- ORM模块：Spring的ORM模块提供了对常用ORM框架如Hibernate，Mybaties等的辅助和支持，他本身更并不实现ORM，仅仅对常见的ORM框架进行封装并对其进行管理。
- DAO模块：通常编写数据库代码时总要写一些样板似的内容，如获取连接，创建语句，释放连接等 ，Dao模块将这些模板抽象出来，使得数据库代码变得简单明了，也可以避免因为释放数据库资源失败而导致的问题。
- WEB模块：提供了基本的面向web的集成功能，例如多个文件的上传功能、使用servlet监听器、面向web应用程序的上下文来初始化IOC容器，还实现了springMVC。
- WEB MVC模块：该模块为spring提供了一套轻量级的mvc实现，他还可以支持和管理其他的mvc框架，如struts。相对于struts，spring自己的mvc框架更加简洁和方便。

**5. 请描述一下spring的事务。**  **答：**Spring既支持编程式事务管理(也称编码式事务)，也支持声明式的事务管理。编程式事务就是把事务写在业务逻辑代码中，声明式事务是将事务管理代码从业务方法中分离出来，以声明的方式来实现事务管理。大多数情况下比编程式事务管理更好用。声明式事务可以在配置文件中用<tx >标签来实现，也可以在需要使用事务的方法上加@Transaction注解。

**6. 如何理解spring的 IOC容器？**  **答：**IOC就是控制反转，这是一种思想而不是一种技术。通常创建对象是由程序员来new的，而"控制反转"是指new实例的工作不由程序员来做，而是交给Spring容器来做。在Spring中BeanFactory是IOC容器的实际代表者。

**7. ApplicationContext是干嘛的？有哪些实现类？**  **答：**ApplicationContext是“应用容器”，继承自BeanFactory。Spring把Bean放在这个容器中，在需要的时候，用getBean方法取出。它的实现有以下三个：

- FileSystemXmlApplicationContext ：从指定的文件系统路径中寻找指定的XML配置文件，找到并装载完成ApplicationContext的实例化工作。
- ClassPathXmlApplicationContext：从类路径ClassPath中寻找指定的XML配置文件，找到并装载完成ApplicationContext的实例化工作。
- XmlWebApplicationContext：从Web应用中寻找指定的XML配置文件，找到并装载完成ApplicationContext的实例化工作。

**8. BeanFactory与AppliacationContext有什么区别？**  **答：**BeanFactory 是基础类型的IOC容器，提供完整的IOC服务支持。ApplicationContext 是在 BeanFactory 的基础上构建，是相对比较高级的容器，除了 BeanFactory 的所有支持外，ApplicationContext还提供了事件发布、国际化支持等功能。

**9. 依赖注入有哪些实现方式？**  **答：**spring提供了以下四种依赖注入的方式：

- 使用Set方法注入
- 使用构造方法注入
- 使用静态工厂注入
- 使用实例工厂注入

**10. 什么是spring beans？**  **答：**spring beans 就是被spring容器初始化、配置和管理的对象。

**11. 一个spring beans 的定义需要包含什么？**  **答：**一个 Spring Bean 的定义包含容器必知的所有配置元数据，包括如何创建一个 bean，它的生命周期详情及它的依赖。

**12. spring支持几种类的作用域？**  **答：**spring在配置bean的时候，可以通过scope属性来定义作用域，scope属性有以下5个值：

- singleton : bean在每个Spring ioc 容器中只有一个实例。 默认就是singleton，它是线程不安全的。
- prototype：每一次请求都会产生一个新的bean实例。
- request：request表示针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP request内有效。
- session：作用域表示针对每一次HTTP请求都会产生一个新的bean，同时该bean仅在当前HTTP session内有效。
- global session：作用域类似于标准的HTTP Session作用域，不过它仅仅在基于portlet的web应用中才有意义。

**13. 怎么开启spring的注解装配？**  **答：**spring默认是没有开启注解的，要使用注解，我们必须在 Spring 配置文件中配置 <context:annotation-config/>元素。

**14. 简单的说一下AOP编程中的相关概念。**  **答：**主要有如下概念：

- Joinpoint(连接点) : 类里面可以被增强的方法，这些方法称为连接点 。
- Pointcut(切入点)：在哪些类的哪些方法上切入(where)。
- Advice(通知/增强) ：拦截到Joinpoint之后所要做的事情就是通知。通知分为前置通知、后置通知、异常通知、最终通知、环绕通知。
- Aspect(切面) : 切面 = 切入点 + 通知，通俗点就是：在什么时机，什么地方，做什么增强。
- Weaving(织入) ：把切面加入到对象，并创建出代理对象的过程。（由 Spring 来完成）。
- Target(目标对象) ：需要增强的类就是目标对象。
- Proxy(代理) ：一个类被AOP织入增强后，就产生一个代理类。

#### (二)、SpringMVC

**1. 什么是springMVC?**  **答：**Spring MVC是一种基于Java的实现了Web MVC设计模式的请求驱动类型的轻量级Web框架。使用了MVC架构模式的思想，将web层进行职责解耦，基于请求驱动指的就是使用请求-响应模型。框架的目的就是帮助我们简化开发。

**2. 简单描述一下SpringMVC的工作原理。**  **答：**工作原理如下：

DispatcherServlet是前置控制器来对相应的规则分发到目标Controller来处理，是配置spring MVC的第一步。

- 用户向服务器发送请求，请求被springMVC前端控制器DispatcherServlet捕获；
- 由DispatcherServlet控制器找到处理请求的Controller；
- DispatcherServlet将请求提交到Controller；
- Controller调用业务逻辑处理后，返回ModelAndView给DispatcherServlet；
- DispatcherServlet查询一个或多个ViewResoler视图解析器，找到ModelAndView指定的视图；
- ViewResoler 解析出 ModelAndView()中的参数，将视图返回给客户端; 。

**3. springMVC有什么优点？**  **答：**它是基于组件技术的，全部的应用对象,无论控制器和视图,还是业务对象之类的都是 java组件；可以任意使用各种视图技术,而不仅仅局限于JSP；支持各种请求资源的映射策略；它应是易于扩展的。

**4. springMVC和struts2有什么区别？**  **答：**区别如下：

- springmvc的入口是一个servlet即前端控制器（DispatcherServlet），而struts2入口是一个filter过虑器（StrutsPrepareAndExecuteFilter）。
- springmvc是基于方法开发(一个url对应一个方法)，请求参数传递到方法的形参，可以设计为单例或多例(建议单例)，struts2是基于类开发，传递参数是通过类的属性，只能设计为多例。
- Struts采用值栈存储请求和响应的数据，通过OGNL存取数据，springmvc通过参数解析器是将request请求内容解析，并给方法形参赋值，将数据和视图封装成ModelAndView对象，最后又将ModelAndView中的模型数据通过reques域传输到页面。