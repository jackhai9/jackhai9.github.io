title: Dubbo分布式服务
date: 2016-11-17 21:10:04
categories: 技术
tags: [Dubbo,分布式服务架构]
---

对Gaea服务进行重构，使用Dubbo实现分布式服务架构，记录下过程。

用到了4台主机，分别是：2台部署服务提供者，做主从；1台Zookeeper服务器，同时部署dubbo-admin管控台；1台部署Redis，如果有必要，Redis也可增加主机，扩展为分布式。

<!--more-->
具体步骤：
1. Gaea服务进行向上提取，引入Dubbo，部署两份分别到2台服务器，作为服务提供者。注意注册方式使用Zookeeper，服务暴露地址等等配置事项。
2. Zookeeper的搭建，注意地址，Dubbo管控台搭建，方便起见与Zookeeper搭在了一起。
3. 使用Redis作缓存，这里是单机，可以做伪集群或者是集群。注意配置项。
4. 注意测试：高并发和高可用性。集群某台宕机测试。
5. 服务消费者分批改为调用Dubbo服务提供者。

重构后服务调运不受影响，具有了高并发和高可用性。通过这次重构学到了不少。中间遇到了不少问题，基本都是搭建过程和配置的原因，具体参考文档如下：

[参考文档](http://blog.csdn.net/xvshu/article/details/47667235)。
[参考文档](http://www.cnblogs.com/ASPNET2008/p/5622005.html)。
[参考文档](http://blog.csdn.net/congcong68/article/details/41113239)。
[Dubbo文档](http://dubbo.io/)。

