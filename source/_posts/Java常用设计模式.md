---
title: Java常用设计模式
date: 2019-03-21 10:53:45
tags:
  - Java
  - 面试
toc: true
categories:
  - Java
comments: true
---

### 前言：

在Java中，传说有23中模式，总共分为三大类，分别是：

- **创建型**模式(5种)：**工厂方法模式**、**抽象工厂模式**、**建造者模式**、**单例模式**、原型模式；
- 结构型模式(7种)：**适配器模式**、**装饰器模式**、代理模式、外观模式、桥接模式、组合模式、享元模式；
- 行为型模式(11种)：**策略模式**、**模板方法模式**、**观察者模式**、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。

<!--more-->

### 一、工厂方法模式：

工厂是干嘛的，就是用来生产的嘛，这里说的工厂也是用来生产的，它是用来生产对象的。也就是说，有些对象我们可以在工厂里面生产，需要用时直接从工厂里面拿出来即可，而不用每次需要用的时候都去new对象。工厂方法模式又分为以下三种：

-  **普通工厂模式：**就是建立一个工厂类，对实现了同一接口的一些类进行实例的创建。
-  **多个工厂方法模式：**是对普通工厂方法模式的改进，在普通工厂方法模式中，如果传递的字符串出错，则不能正确创建对象，而多个工厂方法模式是提供多个工厂方法，分别创建对象。
-  **静态工厂方法模式：**将上面的多个工厂方法模式里的方法置为静态的，不需要创建实例，直接调用即可。

上面三个模式中，后一个都是对前一个的改良。下面分别看看这三个模式的具体案例。

**情景：**有一个发送消息的接口，有两个实现类，一个是发送短信，一个是发送邮件。代码如下：

```javascript
// 接口
public interface Sender {
  public void Send();
} 
//实现一
public class MailSender implements Sender { 
  @Override
  public void Send() {
     System.out.println("this is mail sender!");
  } 
} 
//实现二
public class SmsSender implements Sender { 
  @Override
  public void Send() { 
     System.out.println("this is sms sender!"); 
  }
}
```

下面就看看用这三种工厂方法模式分别要怎么做。

- **普通工厂模式：**

```javascript
 public class SendFactory { 
   public Sender produce(String type) { 
       if ("mail".equals(type)) { 
           return new MailSender();
       } else if ("sms".equals(type)) { 
           return new SmsSender(); 
       } else { 
           System.out.println("请输入正确的类型!"); 
           return null;
       } 
   } 
} 
```

使用的时候就创建这个工厂的对象，调用produce方法，需要发邮件就传入"mail"，需要发短信就传入"sms"，如果传入的是别的内容，就不会创建任何对象。

- **多个工厂方法模式：**

```javascript
public class SendFactory { 
   public Sender produceMail(){
      return new MailSender();
   }
   public Sender produceSms(){
      return new SmsSender();
   } 
}
```

普通工厂模式生产对象都是在一个方法内完成，多个工厂方法模式是提供多个方法，分别生产对应的对象。使用时先创建工厂的实例，要发短信就调用生产短信实例的方法，要发邮件就调用生产邮件实例的方法。

- **静态工厂方法模式：**

```javascript
public class SendFactory {
  public static Sender produceMail(){   
       return new MailSender(); 
  } 
  public static Sender produceSms(){  
       return new SmsSender();
  } 
} 
```

就是多个工厂方法模式里面的方法设为静态的，这样可以直接通过工厂类的类名调用，更加方便。

### 二、抽象工厂模式：

上面的工厂方法模式有一个缺点，就是类的创建依赖工厂类，比如现在还可以用微信发消息，那么就得在工厂类中新增一个创建WeChat实体的方法。这样就违背了开闭原则。如何解决？就用到抽象工厂模式，创建多个工厂类，这样一旦需要增加新的功能，直接增加新的工厂类就可以了，就不需要修改之前的工厂类代码。 上面的案例代码就可以修改成下面这样：

```javascript
// 工厂类接口
public interface Provider {
    public Sender produce();
}
//生产SmsSender对象的工厂
public class SendSmsFactory implements Provider {
  @Override
   public Sender produce() {
       return new SmsSender();
   }
}
//生产MailSender对象的工厂
public class SendMailFactory implements Provider {
   @Override
   public Sender produce() {
      return new MailSender();
   }
}
// 测试
public class Test {
    public static void main(String[] args) {
       Provider provider = new SendMailFactory();
       Sender sender = provider.produce(); 
       sender.send();
    }
} 
```

这样即使要新增生产WechatSender实例的方法，也不需要修改现有代码，只需实现工厂接口，重写生产方法即可。从工厂方法模式到抽象工厂模式，后者更加体现了Java的封装、抽象等思想。

### 三、建造者模式：

工厂模式提供的是创建单个类实例的模式，而建造者模式可以理解为是批量生产。还是使用工厂方法模式中的情景，看看用建造者模式怎么实现：

```javascript
// 建造类
public class Builder { 
   private List<Sender> list = new ArrayList<Sender>();
   public void produceMailSender(int count) { // 生产count个MailSender
      for (int i = 0; i < count; i++) {
           list.add(new MailSender());
      }
   }
   public void produceSmsSender(int count) { // 生产count个SmsSender
      for (int i = 0; i < count; i++) {
           list.add(new SmsSender());
      }
   }
} 
/* =========================== 使用 ============================*/
public class TestBuilder {
   public static void main(String[] args) {
      Builder builder = new Builder();
      builder.produceMailSender(10); // 生产10个MailSender
   }
}
```

### 四、单例模式：

什么叫单例，就是保证一个类在内存中只有一个对象。Runtime()方法就是单例设计模式进行设计的。如何保证内存中只有一个对象呢？  **设计思路：**

- 不让其他程序创建该类对象。
- 在本类中创建一个本类对象。
- 对外提供方法，让其他程序获取这个对象。

**实现步骤：**

- 私有化构造函数；
- 创建私有并静态的本类对象；
- 定义公有并静态的方法，返回该对象。

**代码实现：**

- 懒汉式：延迟加载

```javascript
class Single{
    private Single(){} // 将构造方法私有化
    private static Single s = null; // 私有静态的本类对象
    public static synchronized Single getInstance(){ // 静态公共的返回对象的方法
        if(s==null)
            s = new Single();
        return s;
    }
}
```

- 饿汉式：

```javascript
class Single{
    private Single(){} //私有化构造函数。
    private static Single s = new Single(); //创建私有并静态的本类对象。
    public static Single getInstance(){ //定义公有并静态的方法，返回该对象。
        return s;
    }
}
```

### 五、适配器模式：

适配器模式将某个类的接口转换成客户端期望的另一个接口表示，目的是消除由于接口不匹配所造成的类的兼容性问题。主要分为以下三类：

- 类的适配器模式：比如一个类有一个方法method1，但是客户端使用的时候还需要一个method2方法，那就可以将method1和method2方法写进接口中，然后新建一个适配器类继承原来的类并实现这个接口。
- 对象的适配器模式：与类适配器相比，不需再继承source类，而是将source的对象传过去。
- 接口的适配器模式

看一下具体用代码怎么体现：

- 类的适配器模式：

```javascript
public class Source {
   public void method1() {
     System.out.println("这是method1方法"); 
   } 
}
// 新建接口
public interface Targetable { 
   /* 与原类中的方法相同 */ 
   public void method1();  
   /* 新类的方法 */
   public void method2(); 
} 
// 适配类
public class Adapter extends Source implements Targetable {
   @Override
   public void method2() { 
      System.out.println("这是method2方法"); 
   }
} 
/* ===================== 使用 ======================*/
public class AdapterTest {
   public static void main(String[] args) {
      Targetable target = new Adapter(); 
      target.method1(); 
      target.method2(); 
   }
}  
```

- 对象的适配器模式：

```javascript
public class Source {
   public void method1() {
     System.out.println("这是method1方法"); 
   } 
}
public interface Targetable {
   /* 与原类中的方法相同 */
   public void method1();
   /* 新类的方法 */
   public void method2();
}
// 适配类
public class Wrapper implements Targetable { 
   private Source source;
   public Wrapper(Source source) { 
      super(); 
      this.source = source;
   } 
   @Override
   public void method2() {
      System.out.println("this is the targetable method!");
   }
   @Override
   public void method1() {
       source.method1();
   } 
}
/* ======================= 使用 =========================*/
public class AdapterTest {
   public static void main(String[] args) {
      Source source = new Source();
      Targetable target = new Wrapper(source);
      target.method1();
      target.method2();
   }
}    
```

与类的适配器模式不同的是，对象的适配器模式不再继承source类，而是直接将source对象传到Wrapper类就可以了。   

- 接口的适配器模式

接口的适配器是这样的：有时我们写的一个接口中有多个抽象方法，当我们写该接口的实现类时，必须实现该接口的所有方法，这明显有时比较浪费，因为并不是所有的方法都是我们需要的，有时只需要某一些，此处为了解决这个问题，我们引入了接口的适配器模式，借助于一个抽象类，该抽象类实现了该接口，实现了所有的方法，而我们不和原始的接口打交道，只和该抽象类取得联系，所以我们写一个类，继承该抽象类，重写我们需要的方法就行。

### 六、装饰器模式：

装饰模式就是给一个对象动态的增加一些新的功能。要求装饰对象和被装饰对象实现同一个接口，装饰对象持有被装饰对象的实例。 这样说得也很抽象，看看具体的案例：

```javascript
// 接口
public interface Sourceable {
    public void method();
}
// 被装饰的类，实现Sourceable接口
public class Source implements Sourceable {
    @Override
    public void method() {
        System.out.println("the original method!");
    }
}
// 装饰的类，也要实现Sourceable 接口
public class Decorator implements Sourceable {
    private Sourceable source; // 持有被装饰类的对象
    public Decorator(Sourceable source) {
         super();
         this.source = source;
    }
    @Override
    public void method() {
         System.out.println("before decorator!"); // 装饰
         source.method(); 
         System.out.println("after decorator!"); // 装饰
    }
} 
// 测试
public class DecoratorTest {
    public static void main(String[] args) {
         Sourceable source = new Source();
         Sourceable obj = new Decorator(source);
         obj.method();
    } 
} 
```

IO流体系中很多类就用到了这种设计模式。

### 七、策略模式：

策略模式是对算法的包装，是把使用算法的责任和算法本身分割开来，委派给不同的对象管理。策略模式通常把一个系列的算法包装到一系列的策略类里面，作为一个抽象策略类的子类。看看具体的代码：

```javascript
// 策略接口
public interface Strategy {
    public void strategyInterface();
}
// 具体策略类A
public class ConcreteStrategyA implements Strategy {
    @Override
    public void strategyInterface() {
        // 相关的业务
    }
}
// 具体策略类B
public class ConcreteStrategyB implements Strategy {
    @Override
    public void strategyInterface() {
        //相关的业务
    }
}
// 使用策略的类
public class Context {
    public Context(Strategy strategy){ // 构造函数，传入一个具体策略对象
        this.strategy = strategy;
    }
    public void contextInterface(){ // 策略方法
        strategy.strategyInterface();
    }
}
```

在创建Context类对象的时候，需要使用哪个策略就传入该策略，然后就可以使用。

### 八、模板方法模式：

定义一个操作中的算法的骨架，而将一些步骤延迟到子类中。模板方法使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。简单的说就是很多相同的步骤，只是在某一些地方有差别，那么就可以使用这种模式。看例子：

- 获取一段程序运行时间的模板：

```javascript
public abstract class GetTime{
      public long getTime(){
         long  start = System.currentTimeMillis;
         //表示要计算运行时间的代码
         code();
         long  end = System.currentTimeMillis;
         return end-start;
      }
      public abstract void code(); 
}
```

- 使用该模板：

```javascript
public class forDemo extends GetTime{
     //重写抽象方法
     public void code(){
          for(int x=0;x<1000;x++){
                 System.out.println(x);
          }
     }
}
```

- 测试：

```javascript
public class test{
     GetTime gt=new forDemo();
     gt.getTime();
}
```

这样就可以计算那个for循环运行的时间了。

### 九、观察者模式：

在对象之间定义了一对多的依赖，这样一来，当一个对象改变状态，依赖它的对象会收到通知并自动更新，这就是观察者模式。  **情景：**有一个微信公众号服务，不定时发布一些消息，关注公众号就可以收到推送消息，取消关注就收不到推送消息。  示例代码：

- 定义一个被观察者接口：

```javascript
public interface BeObserverd {
    public void registerObserver(Observer o);// 添加观察者
    public void removeObserver(Observer o);// 删除观察者
    public void notifyObserver();// 通知观察者
}
```

- 定义一个观察者接口：

```javascript
public interface Observer {
    public void update(String message);// 当被观察者发出通知时，这个方法就会执行
}
```

- 定义被观察者，实现了BeObserverd接口，对BeObserverd接口的三个方法进行了具体实现，同时有一个List集合，用以保存注册的观察者，等需要通知观察者时，遍历该集合即可。

```javascript
// 被观察者，也就是微信公众号服务实现了BeObserverd接口，对BeObserverd接口的三个方法进行了具体实现
public class WechatServer implements BeObserverd {
    private List<Observer> list;
    private String message;
    public WechatServer() {
        list = new ArrayList<Observer>(); // 观察者的集合
    }
    // 新增观察者
    @Override
    public void registerObserver(Observer o) {
        list.add(o);
    }
    // 删除观察者
    @Override
    public void removeObserver(Observer o) {
        if(!list.isEmpty())
            list.remove(o);
    }
    // 给观察者发通知
    @Override
    public void notifyObserver() {
        for(int i = 0; i < list.size(); i++) {
            Observer observer = list.get(i);
            observer.update(message);
        }
    }
    // 模拟公众号推送消息的方法
    public void setInfomation(String s) {
        this.message = s;
        System.out.println("微信服务更新消息： " + s);
        //消息更新，通知所有观察者
        notifyObserver();
    }
}
```

- 定义具体观察者，微信公众号的具体观察者为用户User。

```javascript
public class User implements Observer {
    private String name;
    private String message;
    public User(String name) {
        this.name = name;
    }
    @Override
    public void update(String message) {
        this.message = message;
        read();
    }
        public void read() {
        System.out.println(name + " 收到推送消息： " + message);
    }
}
```

假如现有三个人关注公众号，当微信公众号调用setInfomation方法发送推文时，通过调用notifyObserver方法，通知每个观察者，在notifyObserver方法里调用update方法，update里面调用read方法，三个人都能收到推送；加入现在有人取消关注了，那么就会调用removeObserver方法，下次推送这个人就收不到了。(本案例参考[罗汉果](https://www.cnblogs.com/luohanguo/p/7825656.html)的博文，感谢大神整理！)