title: Nginx小结
date: 2016-05-20 17:58:11
categories: 技术
tags: [反向代理,负载均衡]
---

最近工作需要用到了Nginx，以前没怎么用过，找到一个不错的[文档](http://tengine.taobao.org/book/)。

主要的用法就是反向代理和负载均衡。3台海外服务器通过Nginx进行代理，分别部署了3个Tomcat容器，用法上最主要就是Nginx的配置文件怎么配置的，基本不需要其他配置。简单好用，估计以后也需要加SSL模块实现https的访问。
