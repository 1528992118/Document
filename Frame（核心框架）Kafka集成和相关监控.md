Frame（核心框架）Kafka集成和相关监控
================
[![Release](https://img.shields.io/badge/build-springboot-green.svg)]()&nbsp;[![Build](https://img.shields.io/badge/release-2.0.0-blue.svg)]()&nbsp;[![Author](https://img.shields.io/badge/author-Xiaolong.Cao-yellow.svg)]()&nbsp;[![Time](https://img.shields.io/badge/time-2018.8.20-red.svg)]()&nbsp;

# Preface
### 本次文档针对Frame中KafaKa集成和其监控平台，完整产品介绍请参见 [Enjoyor-Springboot 框架](https://www.zybuluo.com/1528992118/note/1180138)，上期更新见[Frame-Update-Date-8.4](https://www.zybuluo.com/1528992118/note/1238219)，本次更新特性请参见 [Features 特性](#Features)，开发使用请参见 [Development](#Development)

<span id="Features"></span>
# Features
* ***Kafka集成***
  * 基本集成架构
      - com.enjoyor.soa.traffic.frame.support.kafka
         - callback   
         - message   
         - send 
         - template
         
  * 架构阐述       
   ***callback***: *send方法中用于消息回调，分别提供 `CallbackFailure`和 `CallbackSuccess`*
   ***message***:*封装的 `KafkaMessage`,该类由统一消息头 `KafkaMessageHeader`和JSON格式消息体 `jsonString`构成，目的在于一个系统内复用 `Topic`*
   ***send***:*具体 `Kafka`生产者消息发送类*
   ***template***：*提供测试示例，分别为一个接收者和发送者*
   
  * *发送*，通过**IKafkaSendTool**，实现对消息的发送，默认提供以下三种发送方法：
   `send(String topic, KafkaMessage msg)` 
   `send(String topic, Integer partition, KafkaMessage msg)`
   `send(String topic, KafkaMessage data, CallbackSuccess callbackSuccess, CallbackFailure callbackFailure)`
  * *接收*，`@KafkaListener`注解用于接受特定`Topic`
  * 类位置：`com.enjoyor.soa.traffic.frame.support.kafka.send.IKafkaSendTool`

* **Kafka监控**

  * Kafaka-Eagle监控平台，版本为`1.2.3`，提供Kafka集群监控，`Kafka-Sql`查询等
 
  * Kafak监控访问地址，http://192.168.**.**:8048/ke/，账号：`**`，密码：`**`

<span id="Development"></span>
# Development
## 1 Kafka集成
###1.1 使用准备
1.1.1 **配置文件(kafka.properties)**

    bootstrap.servers=192.168.**.**:9092
    kafka.group-id=default-group-id
    kafka.auto-offset-reset=earliest

1.1.1.1 配置文件说明

*bootstrap.servers*   &nbsp;&nbsp;Kafka服务器地址
*kafka.group-id*  &nbsp;&nbsp;Consumer监听Group-id
*kafka.auto-offset-reset*&nbsp;&nbsp;消息队列分区偏移量(详述见备注)

*备注：*
**kafka.auto-offset-reset**有以下三个偏移量：

**earliest**：*表示当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，从头开始消费* 

***latest***：*表示当各分区下有已提交的offset时，从提交的offset开始消费；无提交的offset时，消费新产生的该分区下的数据* 

***none***：*表示topic各分区都存在已提交的offset时，从offset后开始消费；只要有一个分区不存在已提交的offset，则抛出异常*

**kafka.group-id**  ：***某Topic下的消息想实现广播，必须指定不同kafka.group-id***

1.1.2 **启动项注入配置类**

    @SpringBootApplication
    @Import({ KafkaCustomConfig.class })
    public class RestApplication 

### 1.2 使用

1.2.1 `com.enjoyor.soa.traffic.frame.support.kafka.template`下提供的发送案例

    @Component
    public class KafkaSenderTemplate {

      @Autowired
      private IKafkaSendTool kafkaSendTool;

      public void send() {
        KafkaMessage kafkaMessage = new KafkaMessage(new       
        KafkaMessageHeader(UUID.randomUUID().toString(), "Template",new Date(),
        MessageType.TEMPLATE.toString()), "Template Sender Msg");
        kafkaSendTool.send("template", kafkaMessage);
      }
    }
    
1.2.2 `com.enjoyor.soa.traffic.frame.support.kafka.template`下提供的接收案例
    
    @Component
    public class KafkaReceiverTemplate {

      @KafkaListener(topics = { "template" }, containerFactory = "kafkaListenerContainerFactory")
      public void listen(ConsumerRecord<?, ?> record) {
        System.out.println("########################## KafkaReceiver 
        start################################");
        System.out.println(String.format("{%s}|{%s}|{%s}|{%s}", record.topic(), 
        record.partition(), record.offset(),
                record.value()));
        System.out.println("########################## KafkaReceiver 
        end################################");
      }
    }
    
  1.2.3 具体调用测试

    
    @RestController
    @RequestMapping({ "/api/template" })
    @Api("TemplateController相关api")
    @CrossOrigin
    public class TemplateController{
    
        @Autowired
        private KafkaSenderTemplate kafkaSender;

        @GetMapping(value = { "/kafkaTest" }, produces = { "application/json; charset=UTF-8" })
        @ResponseBody
        @ApiOperation(value = "kafkaTest", notes = "kafkaTest")
        public ResultPojo kafkaTest() {
           kafkaSender.send();
           return new ResultPojo();
        }
    
    }

    
### 1.3  **使用说明**
**1）** 以上使用的是`com.enjoyor.soa.traffic.frame.support.kafka.template`包下的发送和接收模板案
例，并结合Swagger2-UI页面进行测试。

**2）** `IKafkaSendTool`类一共提供了三种发送方式，一二两种不过多阐述，比较简单，重点第三种提供了发送失败和成功的回调，其实第一二种也有简单的回调，主要为日志记录，具体可参见详细代码，第三种允许自定义自己的成功或失败回调。
 
**3）** 消息的接收主要依赖于 `@KafkaListener`注解，需指定**containerFactory**连接为`kafkaListenerContainerFactory`

## 2 Kafka监控平台
### 2.1 产品概述
本次产品为开源项目，GitHub地址：https://github.com/smartloli/kafka-eagle，
版本为`1.2.3`，其中Kafka服务器版本为`2.11-2.0.0`，需依赖`JDK-1.8`

### 2.2 Download地址
 Kafka下载地址：http://kafka.apache.org/downloads.html
 Kafka-Eagle下载地址：http://download.smartloli.org
 Quickstart：https://ke.smartloli.org/2.Install/2.Installing.html
 

### 2.3 Usage
Kafak监控访问地址，http://192.168.91.168:8048/ke/，账号：`admin`，密码：`123456`

**2.3.1 消费者GroupId列表和分布图**
![Kafka-eagle-consumers.png-25kB][1]
           
![kafka-eagle-actives.png-21.5kB][2]

**2.3.2 Topics 列表图及相关详情**
![kafa-eagle-topics.png-40.6kB][3]

**2.3.3 KafKa-Sql 消息查询**

*Kafka-Sql-Learn：* https://ke.smartloli.org/3.Manuals/9.KafkaSQL.html

*示例Kafka-Sql：* `select * from "template" where "partition" IN (0)`

*查询结果如下图：*

![Kafa-eagle-query.png-9.8kB][4]
![kafka-eagle-result.png-61.5kB][5]

[1]: http://static.zybuluo.com/1528992118/8ccqz49gwvwlwqmsk9aoeh5r/Kafka-eagle-consumers.png
[2]: http://static.zybuluo.com/1528992118/nsdwyki2ok39o8xw2ueze6av/kafka-eagle-actives.png
[3]: http://static.zybuluo.com/1528992118/r0ldwglzg0gm01jb9x20zgid/kafa-eagle-topics.png
[4]: http://static.zybuluo.com/1528992118/agm5wbjhwv1nbvu15erdhak1/Kafa-eagle-query.png
[5]: http://static.zybuluo.com/1528992118/seciqbkhh5f40qx5zuzu8txf/kafka-eagle-result.png
