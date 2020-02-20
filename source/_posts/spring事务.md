---
title: spring事务
date: 2019-03-20 11:01:20
tags:
  - Java
  - Spring
toc: true
categories:
  - Spring
comments: true
---

<!--more-->

## Spring事务的用法与原理

#### 介绍

Spring事务的本质其实就是数据库对事务的支持，没有数据库的事务支持，spring是无法提供事务功能的。对于纯JDBC操作数据库，想要用到事务，可以按照以下步骤进行：

1. 获取连接 Connection con = DriverManager.getConnection()
2. 开启事务con.setAutoCommit(true/false);
3. 执行CRUD
4. 提交事务/回滚事务 con.commit() / con.rollback();
5. 关闭连接 conn.close();

使用Spring的事务管理功能后，我们可以不再写步骤 2 和 4 的代码，而是由Spirng 自动完成。Spring事务处理模块是通过AOP功能来实现声明式事务处理的，具体操作（比如事务实行的配置和读取，事务对象的抽象），用TransactionProxyFactoryBean接口来使用AOP功能，生成proxy代理对象，通过TransactionInterceptor完成对代理方法的拦截，将事务处理的功能编织到拦截的方法中。说得更详细一点：

（1）Spring事务处理模块是通过AOP功能为没有编写事务代码但加上了@Transactional注解的类生成代理。

（2）生成代理的过程中会读取@Transactional注解中的配置，比如传播行为、隔离级别、事务超时等。

（3）生成的代理会拦截目标对象的外部方法调用，自动开启事务、自动提交事务或回滚。

#### 事务的特性

Atomicity原子性：一个事务要么全部执行,要么不执行；

Consistency一致性：事务的运行并不改变数据库中数据的一致性，例如检查约束、非空约束、主键约束、外键约束；

Isolation隔离性：两个以上的事务不会出现交错执行的状态；

Durability持久性：事务执行成功以后,该事务对数据库所作的更改便是持久的保存在数据库之中，不会无缘无故的回滚；

#### 核心接口

![img](spring事务/20180202232828622.png)

Spring并不直接管理事务，而是提供了多种事务管理器，他们将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。 Spring 框架中，涉及到事务管理的 API 大约有100个左右，其中最重要的有三个：TransactionDefinition、PlatformTransactionManager、TransactionStatus。

##### PlatformTransactionManager

Spring事务管理器的接口是org.springframework.transaction.PlatformTransactionManager，通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，但是具体的实现就是各个平台自己的事情了。主要包含三个方法：

（1）getTransaction获取事务状态；

（2）commit提交；

（3）rollback回滚；

```java
import org.springframework.transaction.TransactionDefinition;
import org.springframework.transaction.TransactionException;
import org.springframework.transaction.TransactionStatus;

public interface PlatformTransactionManager {
    // 由TransactionDefinition得到TransactionStatus对象
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;
	// 提交
	Void commit(TransactionStatus status) throws TransactionException;

	// 回滚
	Void rollback(TransactionStatus status) throws TransactionException;
}
```

根据底层所使用的不同的持久化 API 或框架，使用如下：

DataSourceTransactionManager：适用于使用JDBC和iBatis进行数据持久化操作的情况，在定义时需要提供底层的数据源作为其属性，也就是 DataSource。
HibernateTransactionManager：适用于使用Hibernate进行数据持久化操作的情况，与 HibernateTransactionManager 对应的是 SessionFactory。

JpaTransactionManager：适用于使用JPA进行数据持久化操作的情况，与 JpaTransactionManager 对应的是 EntityManagerFactory。

##### TransactionStatus
PlatformTransactionManager.getTransaction(…) 方法返回一个 TransactionStatus 对象。返回的TransactionStatus 对象可能代表一个新的或已经存在的事务（如果在当前调用堆栈有一个符合条件的事务）。TransactionStatus 接口提供了一个简单的控制事务执行和查询事务状态的方法：

```java
public interface TransactionStatus{
    boolean isNewTransaction(); // 是否是新的事物
    boolean hasSavepoint(); // 是否有恢复点
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
} 
```


##### TransactionDefinition
org.springframework.transaction.TransactionDefinition，它用于定义一个事务。它包含了事务的静态属性，比如：事务传播行为、隔离级别、超时时间等等。

2.3 TransactionDefinition
org.springframework.transaction.TransactionDefinition，它用于定义一个事务。它包含了事务的静态属性，比如：事务传播行为、隔离级别、超时时间等等。

```java
public interface TransactionDefinition {
    int getPropagationBehavior(); // 返回事务的传播行为
    int getIsolationLevel(); // 返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据
    int getTimeout();  // 返回事务必须在多少秒内完成
    boolean isReadOnly(); // 事务是否只读，事务管理器能够根据这个返回值进行优化，确保事务是只读的
} 
```

### 事务的属性

#### 传播行为

##### support(存在与否)

PROPAGATION_SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行。

##### required

PROPAGATION_REQUIRED：支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。

使用spring声明式事务，spring使用AOP来支持声明式事务，会根据事务属性，自动在方法调用之前决定是否开启一个事务，并在方法执行之后决定事务提交或回滚事务。例如单独调用一个PROPAGATION_REQUIRED的 methodB相当于：

```java
Main{ 
    Connection con=null; 
    try{ 
        con = getConnection(); 
        con.setAutoCommit(false); 
        //方法调用
        methodB(); 
        //提交事务
        con.commit(); 
	} catch(RuntimeException ex) { 
        //回滚事务
        con.rollback();   
    } finally { 
        //释放资源
        closeCon(); 
    } 
} 
```

如果methodB是在一个事务方法methodA中调用，那么对于methodB的调用会加入到methodA的事务当中。

##### required_new

PROPAGATION_REQUIRED_NEW：表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。

##### mandatory强制的
PROPAGATION_MANDATORY：支持当前事务，如果当前没有事务，就抛出异常。

```java
throw new IllegalTransactionStateException(“Transaction propagation ‘mandatory’ but no existing transaction found”)
```

##### not_supported

PROPAGATION_NOT_SUPPORTED: 表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。

##### never
PROPAGATION_NEVER: 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常。

##### nested
PROPAGATION_NESTED: 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。注意各厂商对这种传播行为的支持是有所差异的。可以参考资源管理器的文档来确认它们是否支持嵌套事务

> 1.嵌套事务一个非常重要的概念就是内层事务依赖于外层事务。
> 2.外层事务失败时，会回滚内层事务所做的动作。
> 3.而内层事务操作失败并不会引起外层事务的回滚。嵌套事务是外部事务的一部分，只有当外部事务成功之后嵌套事务才会被提交。

PROPAGATION_NESTED 与PROPAGATION_REQUIRES_NEW的区别:它们非常类似,都像一个嵌套事务，如果不存在一个活动的事务，都会开启一个新的事务。使用 PROPAGATION_REQUIRES_NEW时，内层事务与外层事务就像两个独立的事务一样，一旦内层事务进行了提交后，外层事务不能对其进行回滚。两个事务互不影响。两个事务不是一个真正的嵌套事务。

#### 隔离级别

隔离级别定义了一个事务可能受其他并发事务影响的程度。

##### 事务并发问题
（1）脏读 Dirty reads

脏读发生在一个事务读取了另一个事务改写但尚未提交的数据时。如果改写在稍后被回滚了，那么第一个事务获取的数据就是无效的。

（2）不可重复读 Nonrepeatable read

不可重复读发生在一个事务执行相同的查询两次或两次以上，但是每次都得到不同的数据时。这通常是因为另一个并发事务在两次查询期间进行了更新。

（3）幻读 Phantom read

幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录。

从总的结果来看, 似乎不可重复读和幻读都表现为两次读取的结果不一致。但如果你从控制的角度来看, 两者的区别就比较大：

对于前者，只需要锁住满足条件的记录；
对于后者，要锁住满足条件及其相近的记录；
√: 可能出现 ×: 不会出现

脏读	                                   不可重复读	幻读
Read Uncommitted	               √	              √
Read Committed	                   ×	              √
Repeatable Read	                  ×	               ×
Serializable	                           ×	               ×

MySQL的默认事务隔离级别是：Repeatable Read

##### Read Committed
在提交读(READ COMMITTED)级别中，基于锁机制并发控制的DBMS需要对选定对象的

写锁(write locks)一直保持到事务结束；
读锁(read locks)在SELECT操作完成后马上释放；
不要求“范围锁(range-locks)”
所以READ COMMITTED只能保证读到的数据都是提交之后的数据，但是不能保证“可重复读”。在SELECT操作结束后就释放了共享锁，其他事务就可以对这个数据进行修改了，所以在本事务中再次读取这个数据，有可能已经改变。

##### Repeatable Read
在可重复读(REPEATABLE READS)隔离级别中，基于锁机制并发控制的DBMS需要对选定对象的

读锁(read locks)和写锁(write locks)一直保持到事务结束；
但不要求“范围锁(range-locks)”；
读锁也保持到事务结束，自然就杜绝了一次事务过程中两次读取的数据不一致的问题。但是只锁住了SELECT相关的记录，还是可以往表中插入新的记录的。所以，幻读的问题没法解决。

##### Serializable

将事务的执行变成了串行的，自然就没有并发问题了。

#### 其他

##### 只读

事务的第三个特性是它是否为只读事务。如果事务只对后端的数据库进行该操作，数据库可以利用事务的只读特性来进行一些特定的优化。通过将事务设置为只读，你就可以给数据库一个机会，让它应用它认为合适的优化措施。

##### 事务超时

为了使应用程序很好地运行，事务不能运行太长的时间。因为事务可能涉及对后端数据库的锁定，所以长时间的事务会不必要的占用数据库资源。事务超时就是事务的一个定时器，在特定时间内事务如果没有执行完毕，那么就会自动回滚，而不是一直等待其结束。

### 用法示例

**@Transactional 可以作用于接口、接口方法、类以及类方法上。当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性。**同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。

虽然 @Transactional 注解可以作用于接口、接口方法、类以及类方法上，但是 **Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。另外， @Transactional 注解应该只被应用到 public 方法上**，这是由 Spring AOP 的本质决定的。如果你在 protected、private 或者默认可见性的方法上使用 @Transactional 注解，这将被忽略，也不会抛出任何异常。

默认情况下，**只有来自外部的方法调用才会被AOP代理捕获**，也就是，类内部方法调用本类内部的其他方法并不会引起事务行为，即使被调用方法使用@Transactional注解进行修饰。

#### 异常触发回滚的示例

```java
@Autowired  
private MyBatisDao dao;

@Transactional  
@Override  
public void insert(Test test) {  
    dao.insert(test);  
    throw new RuntimeException("test");//抛出unchecked异常，触发事物，回滚  
} 
```

#### RuntimeException时不回滚的事务

```java
@Transactional(noRollbackFor=RuntimeException.class)  
@Override  
public void insert(Test test) {  
    dao.insert(test);  
    //抛出unchecked异常，触发事物，noRollbackFor=RuntimeException.class,不回滚  
    throw new RuntimeException("test");  
}  
```

#### 作用于类上


```java
@Transactional  
public class MyBatisServiceImpl implements MyBatisService {  
    @Autowired  
    private MyBatisDao dao; 

    @Override  
    public void insert(Test test) {  
        dao.insert(test);  
        //抛出unchecked异常，触发事物，回滚  
        throw new RuntimeException("test");  
    } 
}
```
#### 设置以非事务方式运行

```java
@Transactional(propagation=Propagation.NOT_SUPPORTED)  
@Override  
public void insert(Test test) {  
    //事物传播行为是PROPAGATION_NOT_SUPPORTED，以非事务方式运行，不会存入数据库  
    dao.insert(test);  
}
```

#### 设置传播行为和隔离级别

```java
@Service("businessSerivce")  
public class BusinessServiceImpl implements IBaseService {  
    @Autowired  
    IStudentDao studentDao;  

    @Autowired  
    IBaseServiceB baseServiceb;  

    @Transactional(propagation = Propagation.REQUIRED, isolation = Isolation.DEFAULT, 	rollbackFor = Exception.class)  
    public String doA() throws Exception {  
        Student st = new Student();  
        st.setId(1);  
        st.setSex("girl");  
        st.setUsername("zx");  
        studentDao.insertStudent(st);  
    }
}       
```
