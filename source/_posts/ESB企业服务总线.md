title: ESB企业服务总线
date: 2015-08-08 11:11:38
categories: 技术
tags: [ESB,ActiveMQ]
---

最近用到了ActiveMQ，感觉这种消息队列非常好用，也接触到了ESB这种高级点的技术，有时间很值得多研究。

公司内部用的是封装过的esb，是基于ActiveMQ的，router就是封装的Camel，估计当初进行封装的人拿这个来练手的。基于这个esb，我对寄送信息推送这一块进行了重构，由原来的使用http方式直接推送改为了使用消息队列，好处就是消息队列里的消息可以异步推送，比之前那种依赖于http连接的方式效率和数据完整性上要好多了。

后期也可以把邮件发送的功能也集成到esb这边来，可以摆脱对C#的Gaea服务的依赖。

Apache还有个ServiceMix项目，对ActiveMQ和Camel进行了集成，也可以关注下。

[ActiveMQ官网](http://activemq.apache.org/)。
