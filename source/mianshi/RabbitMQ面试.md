---
title: RabbitMQ面试
date: 2019-03-12 16:39:47
tags:
  - 基础
toc: true
categories:
  - 基础
comments: true
---

<!--more-->

##### 使用RabbitMQ有什么好处？

- 应用解耦（系统拆分）
- 异步处理（预约挂号业务处理成功后，异步发送短信、推送消息、日志记录等）
- 消息分发
- 流量削峰
- 消息缓冲

##### 使用了消息队列会有什么缺点

- 系统可用性降低：你想呀，本来其他系统只要运行好好的，那你的系统就是正常的。现在你非要加入个消息队列进去，那消息队列挂了，你的系统不是呵呵了。因此，系统可用性会降低
- 系统复杂性增加：加入了消息队列，要多考虑很多方面的问题，比如：一致性问题、如何保证消息不被重复消费、如何保证消息可靠性传输等。因此，需要考虑的东西更多，刺痛复杂性增大。

但是，我们该用的还是要用的。

##### 如何保证消费的可靠性传输？

分析：我们在使用消息队列的过程中，应该做到消息不能多消费，也不能少消费。如果无法做到可靠性传输，可能给公司带来千万级别的财产损失。同样的，如果可靠性传输在使用过程中，没有考虑到，这不是给公司挖坑麽，你可以拍拍屁股走人，公司损失的钱，谁承担。还是那句话，认真对待每一个项目，不要给公司挖坑。

回答：其实这个可靠性传输，每种MQ都要从三个角度来分析：

- 生产者弄丢数据
- 消息队列弄丢数据
- 消费者弄丢数据

（1）生产者丢数据
从生产者弄丢数据这个角度来看，RabbitMQ提供transaction和confirm模式来确保生产者不丢消息。

transaction机制就是说，发送消息前，开启事务（channel.txSelect()）,然后发送消息，如果发送过程中出现什么异常，事务就会回滚（channel.txRollback()）,如果发送成功则提交事务（channel.txCommit()）。

然而，这种方式有个缺点：吞吐量下降。因为，按照经验，生产上用confirm模式的居多。一旦channel进入confirm模式，所有在该信道上发布的消息都将会被指派一个唯一的ID（从1开始），一旦消息被投递到所有匹配的队列之后，rabbitMQ就会发送一个ACK给生产者（包含消息的唯一ID），这就使得生产者知道消息已经正确到达目的队列了。如果rabbitMQ没能处理该消息，则会发送一个Nack消息给你，你可以进行重试操作。处理Ack和Nack的代码如下所示：

```
channel.addConfirmListener(new ConfirmListener() {  
                @Override  
                public void handleNack(long deliveryTag, boolean multiple) throws IOException {  
                    System.out.println("nack: deliveryTag = "+deliveryTag+" multiple: "+multiple);  
                }  
                @Override  
                public void handleAck(long deliveryTag, boolean multiple) throws IOException {  
                    System.out.println("ack: deliveryTag = "+deliveryTag+" multiple: "+multiple);  
                }  
            });  
```

(2)消息队列丢数据

处理消息队列丢数据的情况，一般是开启持久化磁盘的配置。这个持久化配置可以和confirm机制配合使用，你可以在消息持久化磁盘后，再给生产者发送一个Ack信号。这样，如果消息持久化磁盘之前，rabbitMQ阵亡了，那么生产者收不到Ack信号，生产者会自动重发。

那么如何持久化呢，这里顺便说一下吧，其实也很容易，就下面两步

1. 将queue的持久化标识durable设置为true,则代表是一个持久的队列
2. 发送消息的时候将deliveryMode=2

这样设置以后，即使rabbitMQ挂了，重启后也能恢复数据

（3）消费者丢数据

消费者丢数据一般是因为采用了自动确认消息模式。这种模式下，消费者会自动确认收到信息。这时rabbitMQ会立即将消息删除，这种情况下，如果消费者出现异常而未能处理消息，就会丢失该消息。

至于解决方案，采用手动确认消息即可。