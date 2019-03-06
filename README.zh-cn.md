
# CanalSharp

## 一.CanalSharp是什么?

CanalSharp 是阿里巴巴开源项目 Canal 的 .NET 客户端。为 .NET 开发者提供一个更友好的使用 Canal 的方式。Canal 是mysql数据库binlog的增量订阅&消费组件。

基于日志增量订阅&消费支持的业务：

1. 数据库镜像
2. 数据库实时备份
3. 多级索引 (卖家和买家各自分库索引)
4. search build
5. 业务cache刷新
6. 价格变化等重要业务消息

关于 Canal 的更多信息请访问 https://github.com/alibaba/canal/wiki

## 二.应用场景

CanalSharp作为Canal的客户端，其应用场景就是Canal的应用场景。关于应用场景在Canal介绍一节已有概述。举一些实际的使用例子：

1.代替使用轮询数据库方式来监控数据库变更，有效改善轮询耗费数据库资源。

2.根据数据库的变更实时更新搜索引擎，比如电商场景下商品信息发生变更，实时同步到商品搜索引擎 Elasticsearch、solr等

3.根据数据库的变更实时更新缓存，比如电商场景下商品价格、库存发生变更实时同步到redis

4.数据库异地备份、数据同步

5.根据数据库变更触发某种业务，比如电商场景下，创建订单超过xx时间未支付被自动取消，我们获取到这条订单数据的状态变更即可向用户推送消息。

6.将数据库变更整理成自己的数据格式发送到kafka等消息队列，供消息队列的消费者进行消费。

## 三.工作原理

CanalSharp 是 Canal 的 .NET 客户端，它与 Canal 是采用的Socket来进行通信的，传输协议是TCP，交互协议采用的是 Google Protocol Buffer 3.0。

## 四.工作流程

1.Canal连接到mysql数据库，模拟slave

2.CanalSharp与Canal建立连接

2.数据库发生变更写入到binlog

5.Canal向数据库发送dump请求，获取binlog并解析

4.CanalSharp向Canal请求数据库变更

4.Canal发送解析后的数据给CanalSharp

5.CanalSharp收到数据，消费成功，发送回执。（可选）

6.Canal记录消费位置。

以一张图来表示：

![1537860226808](assets/668104-20180925182816462-2110152563.png)

## 五.快速入门

### 1.安装Canal

Canal的安装以及配置使用请查看 https://github.com/alibaba/canal/wiki/QuickStart

### 2.建立一个.NET Core 控制台项目

### 3.为该项目从 Nuget 安装 CanalSharp

````shell
Install-Package CanalSharp.Client
````

### 4.建立与Canal的连接

````csharp
//canal 配置的 destination，默认为 example
var destination = "example";
//创建一个简单CanalClient连接对象（此对象不支持集群）传入参数分别为 canal地址、端口、destination、用户名、密码
var connector = CanalConnectors.NewSingleConnector("127.0.0.1", 11111, destination, "", "");
//连接 Canal
connector.Connect();
//订阅，同时传入Filter，如果不传则以Canal的Filter为准。Filter是一种过滤规则，通过该规则的表数据变更才会传递过来
connector.Subscribe(".*\\..*");
//获取数据但是不需要发送Ack来表示消费成功
connector.Get(batchSize);
//获取数据并且需要发送Ack表示消费成功
connector.GetWithoutAck(batchSize);
````

更多详情请查看 [Sample](https://github.com/CanalSharp/CanalSharp/tree/master/sample/CanalSharp.SimpleClient)

## 六.通过docker方式快速运行CanalSharp

### 1.执行命令通过docker方式运行 mysql与canal

````shell
git clone https://github.com/CanalSharp/CanalSharp.git
cd CanalSharp
cd docker
docker-compose up -d
````

### 2.使用navicat等数据库管理工具连接mysql

ip：运行docker的服务器ip

mysql用户：root

mysql密码：000000

mysql端口：4406

默认提供了一个test数据库，然后有一张名为test的表。

![1537866852816](assets/668104-20180925182815646-1209020640.png)

### 3.运行Sample项目

### 4.测试

执行下列sql:

````sql
insert into test values(1000,'111');
update test set name='222' where id=1000;
delete from test where id=1000;
````

![](assets/ys.gif)

可以看见我们分别执行 insert、update、delete 语句，我们的CanalSharp都获取到了数据库变更。

## 七.接下来的工作

CanalSharp集群支持

## 八.贡献代码

1.fork本项目

2.做出你的更改

3.提交 pull request
