title: MyBatis小结
date: 2015-06-15 20:28:55
categories: 技术
tags: [MyBatis,ORM]
---

数据持久化层，公司的框架是封装的JDBC，几乎用法是一样的，现在应该流行用ORM框架吧，今天看了下MyBatis，相比较与Hibernate那样无法写SQL语句，更喜欢MyBatis这样可调整的方式，而且还能通过优化SQL来优化数据库访问，还能练习SQL语法，一举多得啊。
<!--more-->
MyBatis消除了几乎所有的JDBC代码和参数的手工设置以及对结果集的检索封装。MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old Java Objects，普通的Java对象）映射成数据库中的记录。

至于配置文件和sql映射文件等步骤可以参考[这里](http://www.cnblogs.com/xdp-gacl/p/4261895.html)。

