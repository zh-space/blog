---
title: RabbitMQ的学习(持续学习中)
date: 2022-06-11 
categories:
 - RabbitMQ
tags:
 - 消息队列
 - RabbitMQ
sticky: 1
---
## 首先安装RabbitMQ
刚刚学完docker，如果使用docker安装RabbitMQ的话就很快，一个命令就可以了，然后需要自己配置一下允许远程访问就可以了。但是我们刚开始学习的话还是要自己手动安装一下。  
```sh
docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3.10-management
```
1. 首先我们需要安装Erlang 环境。注意Erlang 和 rabbitmq的版本对应关系喔。还要安装socat软件包。最后安装rabbitmq就好。我的安装包都是从老师哪里拿来的，大家可以自己去官网下载。
以下是软件的安装地址，一定记得版本需要对应喔:  
```sh
【erlang下载地址】：https://hub.fastgit.org/rabbitmq/erlang-rpm/releases  

【socat下载地址】：http://www.rpmfind.net/linux/rpm2html/search.php?query=socat(x86-64)  

【rabbitmq下载地址】：https://github.com/rabbitmq/rabbitmq-server/releases  
```
2. 然后将这三个安装包都上传到linux服务器上，要记得上传到那个目录了哟。  
三个安装包的安装命令,需要按照顺序执行:
```sh
rpm -ivh erlang-21.3-1.el7.x86_64.rpm  
yum install socat -y  
rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm  
```
3. 安装好后就可以启动啦，RabbitMQ其实就是一个中间件，和大家经常使用的redis，nginx，nacos都差不多，有过这些中间件的使用经验就上手很快。  
添加开机启动 RabbitMQ 服务  
```sh
chkconfig rabbitmq-server on  
启动服务  
/sbin/service rabbitmq-server start   
查看服务状态  
/sbin/service rabbitmq-server status  
停止服务(选择执行)  
/sbin/service rabbitmq-server stop   
开启 web 管理插件  
rabbitmq-plugins enable rabbitmq_management  
```
4. 添加配置文件，解决只能localhost访问的问题
```sh
进入【/etc/rabbitmq】文件夹下  
cd /etc/rabbitmq  
编辑【rabbitmq.config】文件  
vim rabbitmq.config  
在新建的配置文件下添加如下，不要忘记后面的点:
[{rabbit,[{loopback_users,[]}]}].
```
5. 开放两个端口，一个是图形化界面需要的端口，一个是RabbitMQ服务的端口
```sh
开放5672端口命令  
/sbin/iptables -I INPUT -p tcp --dport 5672 -j ACCEPT

开放15672端口命令
/sbin/iptables -I INPUT -p tcp --dport 15672 -j ACCEPT
```
## 新增用户
```sh
创建账号  
rabbitmqctl add_user admin 123  
设置用户角色  
rabbitmqctl set_user_tags admin administrator  
设置用户权限  
set_permissions [-p <vhostpath>] <user> <conf> <write> <read>  

rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"  
用户 user_admin 具有/vhost1 这个 virtual host 中所有资源的配置、写、读权限  
当前用户和角色  
rabbitmqctl list_users 
``` 
## 学东西脑图是很重要的，要知道rabbitmq的组成。
![Image text](http://101.42.150.127:8081/blog/rabbitmq.PNG)
![Image text](http://101.42.150.127:8081/blog/rabbitmq1.PNG)
## 永远的hello world
现在编写一个java程序，实现消息的发布和接收。  
```xml
<!--指定 jdk 编译版本--> <build> <plugins> <plugin> <groupId>org.apache.maven.plugins</groupId> <artifactId>maven-compiler-plugin</artifactId> <configuration> <source>8</source> <target>8</target>
</configuration>
</plugin>
</plugins>
</build> <dependencies>
<!--rabbitmq 依赖客户端--> <dependency> <groupId>com.rabbitmq</groupId> <artifactId>amqp-client</artifactId> <version>5.8.0</version>
</dependency>
<!--操作文件流的一个依赖--> <dependency> <groupId>commons-io</groupId> <artifactId>commons-io</artifactId> <version>2.6</version>
</dependency>
</dependencies>
```
生产者  
```java
public class Producer {
private final static String QUEUE_NAME = "hello";
public static void main(String[] args) throws Exception {
//创建一个连接工厂
ConnectionFactory factory = new ConnectionFactory();
factory.setHost("182.92.234.71");
factory.setUsername("admin");
factory.setPassword("123");
//channel 实现了自动 close 接口 自动关闭 不需要显示关闭
try(Connection connection = factory.newConnection();Channel
connection.createChannel()) {
/**
* 生成一个队列
* 1.队列名称
* 2.队列里面的消息是否持久化 默认消息存储在内存中
* 3.该队列是否只供一个消费者进行消费 是否进行共享 true 可以多个消费者消费
* 4.是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true 自动删除
* 5.其他参数
*/
channel.queueDeclare(QUEUE_NAME,false,false,false,null);
String message="hello world";
/**
* 发送一个消息
* 1.发送到那个交换机
* 2.路由的 key 是哪个
* 3.其他的参数信息
* 4.发送消息的消息体
*/
channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
System.out.println("消息发送完毕");
} } }
```
消费者  
```java
public class Consumer {
private final static String QUEUE_NAME = "hello";
public static void main(String[] args) throws Exception 
{ConnectionFactory factory = new ConnectionFactory();
factory.setHost("182.92.234.71");
factory.setUsername("admin");
factory.setPassword("123");
Connection connection = factory.newConnection();
Channel channel = connection.createChannel();
System.out.println("等待接收消息.........");
//推送的消息如何进行消费的接口回调
DeliverCallback deliverCallback=(consumerTag,delivery)-
>{String message= new String(delivery.getBody());
System.out.println(message);
};
//取消消费的一个回调接口 如在消费的时候队列被删除掉了
CancelCallback cancelCallback=(consumerTag)-
>{System.out.println("消息消费被中断");
};
/**
* 消费者消费消息
* 1.消费哪个队列
* 2.消费成功之后是否要自动应答 true 代表自动应答 false 手动应答
* 3.消费者未成功消费的回调
*/
channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
}
}
```
## rabbitmq是轮训接收消息的，比如说有两个消费者关注了同一个队列，那么他们会交替从队列中获取信息
工作队列(又称任务队列)的主要思想是避免立即执行资源密集型任务，而不得不等待它完成。
相反我们安排任务在之后执行。我们把任务封装为消息并将其发送到队列。在后台运行的工作进
程将弹出任务并最终执行作业。当有多个工作线程时，这些工作线程将一起处理这些任务。  
我们使用两个线程同时从同一个队列中取消息，看看是否是轮训  
每次都要建立连接获取信道，干脆抽取出来当做工具类，还有消费者获取信息成功和取消获取信息需要执行的函数也一并写到工具类中。  
```java
package com.zh.rabbitmq.util;

import com.rabbitmq.client.*;

public class RabbitmaUtils {
    //得到一个连接的 channel
    public static Channel getChannel() throws Exception {
        //创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //工厂ip 连接rabbitmq的队列
        factory.setHost("120.48.26.57");
        //用户名
        factory.setUsername("zh");
        //密码
        factory.setPassword("zh1505075162");
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        return channel;
    }

    //声明 接收消息
    public static DeliverCallback getDeliverCallback() {
        DeliverCallback deliverCallback = (consumerTag, message) -> {
            System.out.println("消费成功区消息:" + new String(message.getBody()));
        };
        return deliverCallback;
    }

    //声明 取消消息时的回调
    public static CancelCallback getCancelCallback() {
        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("消息被中断" + consumerTag);
        };
        return cancelCallback;
    }
}

```
创建多个消费者，允许以下代码多个实例存在即可启动多次，创建多个消费者。  
```java
package com.zh.rabbitmq.two;

import com.rabbitmq.client.Channel;
import com.zh.rabbitmq.util.RabbitmaUtils;

/**
 * @Description 工作线程
 */
public class Work1 {
    //队列名称
    public static final String QUEUE_NAME = "hello rabbitmq";

    //接收消息
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitmaUtils.getChannel();
        //消息的接收
        System.out.println("第2个工作线程等待接收消息");
        channel.basicConsume(QUEUE_NAME, true, RabbitmaUtils.getDeliverCallback(), RabbitmaUtils.getCancelCallback());
    }
}

```
创建生产者发布信息  
```java
package com.zh.rabbitmq.two;

import com.rabbitmq.client.Channel;
import com.zh.rabbitmq.util.RabbitmaUtils;

import java.util.Scanner;

/**
 * @Description 生产者
 */
public class Task1 {
    //队列名称
    public static final String QUEUE_NAME = "hello rabbitmq";

    //发送大量消息
    public static void main(String[] args) throws Exception {
        Channel channel = RabbitmaUtils.getChannel();
        //队列的声明
        /**
         * 生成一个队列
         * 1.队列名称
         * 2.队列里面的消息是否持久化（磁盘）  默认情况消息存储在内存中
         * 3.该队列是否只供一个消费者进行消费 是否进行消息共享，true可以多个消费者消费
         * 4.是否自动删除 最后一个消费者断开连接后 该队列是否自动删除
         * 5.其他参数
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //从控制台接收然后发送消息
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String message = scanner.next();
            /**
             * 发送一个消息
             * 1.发送到哪里一个交换机
             * 2.路由的key值是哪个 本次是队列的名称
             * 3.表示其他参数信息
             * 4.发送消息的消息体
             */
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println("消息发送完毕");
        }
    }
}
```
结果：  
第一个消费者：  
```java
第1个工作线程等待接收消息
消费成功区消息:d
消费成功区消息:f
消费成功区消息:h
```
第二个消费者：    
```java
第2个工作线程等待接收消息
消费成功区消息:c
消费成功区消息:e
消费成功区消息:g
```
生产者：  
```java
c
消息发送完毕
d
消息发送完毕
e
消息发送完毕
f
消息发送完毕
g
消息发送完毕
h
消息发送完毕
```
结论：rabbitmq就是轮训获取队列里面的信息的，但是如果第一个队列处理消息时间过长，导致消息没有处理，那么就会造成消息的丢失，怎么解决这个问题呢？  
## 消息应答机制
消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成
了部分突然它挂掉了，会发生什么情况。RabbitMQ 一旦向消费者传递了一条消息，便立即将该消
息标记为删除。在这种情况下，突然有个消费者挂掉了，我们将丢失正在处理的消息。以及后续
发送给该消费这的消息，因为它无法接收到。  
为了保证消息在发送过程中不丢失，rabbitmq 引入消息应答机制，消息应答就是:消费者在接收
到消息并且处理该消息之后，告诉 rabbitmq 它已经处理了，rabbitmq 可以把该消息删除了。    
应答又分为自动应答和手动应答：  
自动应答：    
消息发送后立即被认为已经传送成功，这种模式需要在高吞吐量和数据传输安全性方面做权
衡,因为这种模式如果消息在接收到之前，消费者那边出现连接或者 channel 关闭，那么消息就丢失 了,当然另一方面这种模式消费者那边可以传递过载的消息，没有对传递的消息数量进行限制，当
然这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终使
得内存耗尽，最终这些消费者线程被操作系统杀死，所以这种模式仅适用在消费者可以高效并以
某种速率能够处理这些消息的情况下使用。  
简单来说，自动应答就是当消费者接收到消息就返回接收成功的信息，后续代码是否正确执行就不得而知。  
手动应答：  
```java
A.Channel.basicAck(用于肯定确认) 
 RabbitMQ 已知道该消息并且成功的处理消息，可以将其丢弃了
B.Channel.basicNack(用于否定确认) 
C.Channel.basicReject(用于否定确认) 
与 Channel.basicNack 相比少一个参数-Multiple
 不处理该消息了直接拒绝，可以将其丢弃了
```
手动应答的好处是可以批量应答并且减少网络拥堵 
multiple 的 true 和 false 代表不同意思
true 代表批量应答 channel 上未应答的消息
 比如说 channel 上有传送 tag 的消息 5,6,7,8 当前 tag 是8 那么此时
 5-8 的这些还未应答的消息都会被确认收到消息应答
false 同上面相比
 只会应答 tag=8 的消息 5,6,7 这三个消息依然不会被确认收到消息应答  
 消息自动重新入队  
 如果消费者由于某些原因失去连接(其通道已关闭，连接已关闭或 TCP 连接丢失)，导致消息
未发送 ACK 确认，RabbitMQ 将了解到消息未完全处理，并将对其重新排队。如果此时其他消费者
可以处理，它将很快将其重新分发给另一个消费者。这样，即使某个消费者偶尔死亡，也可以确
保不会丢失任何消息。
## 手动应答 自动应答最好不用，就不多介绍了。
睡眠工具类：  
```java
    //线程沉睡
    public static void sleep(int a){
        try {
            Thread.sleep(1000*a);
        } catch (InterruptedException e) {
            Thread.currentThread().isInterrupted();
            e.printStackTrace();
        }
    }
```
生产者：  
```java
package com.zh.rabbitmq.three;

import com.rabbitmq.client.Channel;
import com.zh.rabbitmq.util.RabbitmaUtils;
import java.util.Scanner;

/**
 * @Description 生产者
 * 消息在手动应答的时候是不会丢失，放回队列中重新消费
 */
public class Task2 {
    //队列名称
    public static final String TASK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitmaUtils.getChannel();
        //创建队列
        channel.queueDeclare(TASK_QUEUE_NAME, false, false, false, null);
        //从控制台输入信息
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String message = scanner.next();
            //发送消息
            channel.basicPublish("", TASK_QUEUE_NAME, null, message.getBytes());
            System.out.println("生产者发出消息：" + message);
        }
    }
}

```
消费者（应答时间短（1s））：  
```java
package com.zh.rabbitmq.three;

import com.rabbitmq.client.CancelCallback;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DeliverCallback;
import com.zh.rabbitmq.util.RabbitmaUtils;

/**
 * @Description 消费者
 * 消息在手动应答的时候是不会丢失，放回队列中重新消费
 */
public class Work2 {
    //队列名称
    public static final String TASK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitmaUtils.getChannel();
        System.out.println("第一个消费者等待接收信息，处理信息短（1s）");

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            //沉睡一秒
            RabbitmaUtils.sleep(1);

            System.out.println("消费成功区消息:" + new String(message.getBody()));
            //手动应答
            /**
             * 手动应答
             * 1.消息的标记 tag
             * 2.是否批量应答
             */
            channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
            System.out.println("手动应答完成！");
        };

        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("消息被中断" + consumerTag);
        };

        //手动应答,第二个参数为false
        channel.basicConsume(TASK_QUEUE_NAME, false, deliverCallback, cancelCallback);
    }
}

```
消费者（应答时间长（30s））：  
```java
package com.zh.rabbitmq.three;

import com.rabbitmq.client.CancelCallback;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.DeliverCallback;
import com.zh.rabbitmq.util.RabbitmaUtils;

/**
 * @Description 消费者
 * 消息在手动应答的时候是不会丢失，放回队列中重新消费
 */
public class Work2 {
    //队列名称
    public static final String TASK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitmaUtils.getChannel();
        System.out.println("第一个消费者等待接收信息，处理信息短（1s）");

        DeliverCallback deliverCallback = (consumerTag, message) -> {
            //沉睡三十秒
            RabbitmaUtils.sleep(30);

            System.out.println("消费成功区消息:" + new String(message.getBody()));
            //手动应答
            /**
             * 手动应答
             * 1.消息的标记 tag
             * 2.是否批量应答
             */
            channel.basicAck(message.getEnvelope().getDeliveryTag(),false);
            System.out.println("手动应答完成！");
        };

        CancelCallback cancelCallback = (consumerTag) -> {
            System.out.println("消息被中断" + consumerTag);
        };

        //手动应答,第二个参数为false
        channel.basicConsume(TASK_QUEUE_NAME, false, deliverCallback, cancelCallback);
    }
}

```
当生产者发出两个消息，第一个处理快的消费者先处理完消息，另一个消息轮训给处理慢的消费者，在处理慢的消费者还没处理完的时候就结束进程，结果：  
生产者：  
```
aa
生产者发出消息：aa
bb
生产者发出消息：bb
```
处理慢的消费者  
```
第一个消费者等待接收信息，处理信息短（30s）

进程已结束，退出代码为 -1
```
处理快的消费者  
```
第一个消费者等待接收信息，处理信息短（1s）
消费成功区消息:aa
手动应答完成！
消费成功区消息:bb
手动应答完成！
```
可以看到当处理慢的消费者停止进程后，属于他的未处理的消息就重新入队，然后分配给处理快的消费者了。

## 持久化
刚刚我们已经看到了如何处理任务不丢失的情况，但是如何保障当 RabbitMQ 服务停掉以后消
息生产者发送过来的消息不丢失。默认情况下 RabbitMQ 退出或由于某种原因崩溃时，它忽视队列
和消息，除非告知它不要这样做。确保消息不会丢失需要做两件事：我们需要将队列和消息都标
记为持久化。  
1. 队列如何实现持久化
之前我们创建的队列都是非持久化的，rabbitmq 如果重启的化，该队列就会被删除掉，如果
要队列实现持久化 需要在声明队列的时候把 durable 参数设置为持久化  
但是需要注意的就是如果之前声明的队列不是持久化的，需要把原先队列先删除，或者重新
创建一个持久化的队列，不然就会出现错误  
声明持久化后，在rabbitmq的操作页面，queues的队列Features会有一个大写字母D。
![Image text](http://101.42.150.127:8081/blog/rabbitmq2.PNG)
2. 消息如何实现持久化
在将消息发送给队列的时候就要告诉队列这个消息需要持久化。  
第三个参数，将消息设置为持久化，设置预取值。  
```java
/**
             * 发送一个消息
             * 1.发送到哪里一个交换机
             * 2.路由的key值是哪个 本次是队列的名称
             * 3.表示其他参数信息
             * 4.发送消息的消息体
             */
            channel.basicPublish("", QUEUE_NAME, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
```
将消息标记为持久化并不能完全保证不会丢失消息。尽管它告诉 RabbitMQ 将消息保存到磁盘，但是
这里依然存在当消息刚准备存储在磁盘的时候 但是还没有存储完，消息还在缓存的一个间隔点。此时并没
有真正写入磁盘。持久性保证并不强，但是对于我们的简单任务队列而言，这已经绰绰有余了。如果需要
更强有力的持久化策略，参考后边课件发布确认章节。在从队列里面取消息的时候，可以通过手动确认来确保消息不会丢失，那么在生产者发送消息到队列的时候也可以通过收到确认来保证消息在传到队列的过程中不会丢失。  
## 不公平分发
在最开始的时候我们学习到 RabbitMQ 分发消息采用的轮训分发，但是在某种场景下这种策略并不是
很好，比方说有两个消费者在处理任务，其中有个消费者 1 处理任务的速度非常快，而另外一个消费者 2
处理速度却很慢，这个时候我们还是采用轮训分发的化就会到这处理速度快的这个消费者很大一部分时间
处于空闲状态，而处理慢的那个消费者一直在干活，这种分配方式在这种情况下其实就不太好，但是
RabbitMQ 并不知道这种情况它依然很公平的进行分发。
为了避免这种情况，我们可以设置参数 channel.basicQos(1);  

意思就是如果这个任务我还没有处理完或者我还没有应答你，你先别分配给我，我目前只能处理一个
任务，然后 rabbitmq 就会把该任务分配给没有那么忙的那个空闲消费者，当然如果所有的消费者都没有完
成手上任务，队列还在不停的添加新任务，队列有可能就会遇到队列被撑满的情况，这个时候就只能添加
新的 worker 或者改变其他存储任务的策略。  
## 发布确认
生产者将信道设置成 confirm 模式，一旦信道进入 confirm 模式，所有在该信道上面发布的消
息都将会被指派一个唯一的 ID(从 1 开始)，一旦消息被投递到所有匹配的队列之后，broker 就会
发送一个确认给生产者(包含消息的唯一 ID)，这就使得生产者知道消息已经正确到达目的队列了，
如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出，broker 回传给生产
者的确认消息中 delivery-tag 域包含了确认消息的序列号，此外 broker 也可以设置basic.ack 的
multiple 域，表示到这个序列号之前的所有消息都已经得到了处理。
confirm 模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等信道
返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方
法来处理该确认消息，如果 RabbitMQ 因为自身内部错误导致消息丢失，就会发送一条 nack 消息，
生产者应用程序同样可以在回调方法中处理该 nack 消息。  
## 开启发布确认的方法
```java
//开启发布确认
        channel.confirmSelect();
```
1. 单个确认发布  发布：1000个单独确认消息，耗时136553ms
这是一种简单的确认方式，它是一种同步确认发布的方式，也就是发布一个消息之后只有它
被确认发布，后续的消息才能继续发布,waitForConfirmsOrDie(long)这个方法只有在消息被确认
的时候才返回，如果在指定时间范围内这个消息没有被确认那么它将抛出异常。
这种确认方式有一个最大的缺点就是:发布速度特别的慢，因为如果没有确认发布的消息就会
阻塞所有后续消息的发布，这种方式最多提供每秒不超过数百条发布消息的吞吐量。当然对于某
些应用程序来说这可能已经足够了。  
```java  
public static void publishMessageIndividually() throws Exception {
try (Channel channel = RabbitMqUtils.getChannel()) 
{String queueName = UUID.randomUUID().toString();
channel.queueDeclare(queueName, false, false, false, null);
//开启发布确认
 channel.confirmSelect();
long begin = System.currentTimeMillis();
for (int i = 0; i < MESSAGE_COUNT; i++) 
{String message = i + "";
channel.basicPublish("", queueName, null, message.getBytes());
//服务端返回 false 或超时时间内未返回，生产者可以消息重发
 boolean flag = channel.waitForConfirms();
if(flag){
System.out.println("消息发送成功");
} }
long end = System.currentTimeMillis();
System.out.println("发布" + MESSAGE_COUNT + "个单独确认消息,耗时" + (end - begin) +
"ms");
} 
```
2. 批量确认发布  发布：1000个批量确认消息，耗时7076ms
上面那种方式非常慢，与单个等待确认消息相比，先发布一批消息然后一起确认可以极大地
提高吞吐量，当然这种方式的缺点就是:当发生故障导致发布出现问题时，不知道是哪个消息出现
问题了，我们必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息。当然这种
方案仍然是同步的，也一样阻塞消息的发布。  
```java
public static void publishMessageBatch() throws Exception {
try (Channel channel = RabbitMqUtils.getChannel()) 
{String queueName = UUID.randomUUID().toString();
channel.queueDeclare(queueName, false, false, false, null);
//开启发布确认
 channel.confirmSelect();
//批量确认消息大小
 int batchSize = 100;
//未确认消息个数
 int outstandingMessageCount = 0;
long begin = System.currentTimeMillis();
for (int i = 0; i < MESSAGE_COUNT; i++) 
{String message = i + "";
channel.basicPublish("", queueName, null, message.getBytes());
outstandingMessageCount++;
if (outstandingMessageCount == batchSize) 
{channel.waitForConfirms();
outstandingMessageCount = 0; } }
//为了确保还有剩余没有确认消息 再次确认
 if (outstandingMessageCount > 0) 
{channel.waitForConfirms();
}
long end = System.currentTimeMillis();
System.out.println("发布" + MESSAGE_COUNT + "个批量确认消息,耗时" + (end - begin) +
"ms");
} 
```
3. 异步确认发布  发布：1000个异步确认消息，耗时1000ms
```java
//异步发布确认
    public static void publishMessageSyn() throws Exception {
        Channel channel = RabbitmaUtils.getChannel();
        //队列的声明
        String queueName = UUID.randomUUID().toString();
        channel.queueDeclare(queueName, true, false, false, null);
        //开启确认模式
        channel.confirmSelect();
        //开始时间
        long begin = System.currentTimeMillis();
        //消息确认成功的回调
        /**
         * 1.消息的标记
         * 2.是否为批量确认
         */
        ConfirmCallback confirmCallback=(var1,var3)->{
            System.out.println("已确认的消息:"+var1);
        };
        //消息确认失败的回调
        ConfirmCallback confirmCallback1=(var1,var3)->{
            System.out.println("未确认的消息:"+var1);
        };
        //消息的监听器  监听那些消息成功了 那些消息失败了
        /**
         * 1.监听那些消息成功
         * 2.监听那些消息失败
         */
        channel.addConfirmListener(confirmCallback,confirmCallback1);//异步通知
        //批量发送消息
        for (int i = 0; i < MESSAGE_COUNT; i++) {
            String message=String.valueOf(i);
            channel.basicPublish("", queueName, MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());

        }
        //结束时间
        long end = System.currentTimeMillis();
        System.out.println("发布：" + MESSAGE_COUNT + "个异步确认消息，耗时" + (end - begin) + "ms");
    }
````
## 卸载RabbitMQ
```sh
卸载前先停止rabbitmq服务
systemctl stop rabbitmq-server
查看rabbitmq安装的相关列表
yum list | grep rabbitmq
卸载rabbitmq-server.noarch
yum -y remove rabbitmq-server.noarch
```
## 卸载erlang
```sh
查看erlang安装的相关列表
yum list | grep erlang
卸载erlang已安装的相关内容
yum -y remove erlang-*
删除有关的所有文件
rm -rf /usr/lib64/erlang 
rm -rf /var/lib/rabbitmq
rm -rf /usr/local/erlang
rm -rf /usr/local/rabbitmq
```
