title: MVC与前后端分离
date: 2015-06-7 19:53:50
categories: 技术
tags: [SpingMVC,前后端分离,REST]
---

SpringMVC除了众所周知的好处，有个问题也是要说的：渲染视图的过程是在服务端完成的，最终呈现给浏览器的是带有模型的视图页面，性能上是要打一点折扣的。前后端分离正好可以解决这个问题：浏览器端使用Ajax发起请求，服务器端根据请求返回JSON数据，由浏览器负责渲染界面。

<!--more-->
也就是说，我们输入的是AJAX请求，输出的是JSON数据，市面上有这样的技术来实现这个功能吗？答案是REST。

REST本质上是使用URL来访问资源种方式。其中包括两部分：请求方式与请求路径，比较常见的请求方式是GET与POST，但在REST中又提出了几种其它类型的请求方式，汇总起来有六种：GET、POST、PUT、DELETE、HEAD、OPTIONS。尤其是前四种，正好与CRUD（Create-Retrieve-Update-Delete，增删改查）四种操作相对应，例如，GET（查）、POST（增）、PUT（改）、DELETE（删），这正是REST与CRUD的异曲同工之妙！需要强调的是，REST是“面向资源”的，这里提到的资源，实际上就是我们常说的领域对象，在系统设计过程中，我们经常通过领域对象来进行数据建模。

REST是一个“无状态”的架构模式，因为在任何时候都可以由客户端发出请求到服务端，最终返回自己想要的数据，当前请求不会受到上次请求的影响。也就是说，服务端将内部资源发布REST服务，客户端通过URL来访问这些资源，这不就是SOA所提倡的“面向服务”的思想吗？所以，REST也被人们看做是一种“轻量级”的SOA实现技术，因此在企业级应用与互联网应用中都得到了广泛应用。

例如：
```java
REST请求         描述
-----------------------------
GET:/cars       获取所有的车
GET:/car/1      获取id为1的车
POST:/car       新增1个车
PUT:/car/1      更新id为1的车
DELETE:/car/1   删除id为1的车
```

基于REST实现前后端分离有很多好处。最大好处是前后端可以实现较好的人员分工和并行开发。双方只需要约定数据接口，而不必在代码层面有任何的耦合。传统的JSP或者后端模板方式都存在着较为严重的前后端耦合，前后端人员不仅需要掌握自身所需知识，还需要对对方的领域有一定了解；开发时进度会因为双方的大量沟通而被拖慢，且项目存在缺陷时难以分清是前后端哪一方的责任，调试困难。目前较为流行的解决方案是后端提供纯数据接口，前端使用MVC/MVVM等框架实现数据绑定、界面路由等功能，且未来的发展趋势是后端负责的任务将逐步减少，进入“大前端”时代。◑‿◑

其他的比如 处理异常、参数验证、跨域问题、安全机制等问题可参考[这里](http://www.csdn.net/article/2015-10-25/2826033)。
