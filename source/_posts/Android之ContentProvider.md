title: Android之ContentProvider
date: 2014-11-17 16:39:27
categories: 技术
tags: [ContentProvider]
---
## 一、ContentProvider
Android官方指出的数据存储方式总共有五种，分别是：Shared Preferences、网络存储、文件存储、外储存储、SQLite。当应用继承ContentProvider类，并重写该类用于提供数据和存储数据的方法，就可以向其他应用共享其数据。虽然使用其他方法也可以对外共享数据，但数据访问方式会因数据存储的方式而不同，如：采用文件方式对外共享数据，需要进行文件操作读写数据；采用sharedpreferences共享数据，需要使用sharedpreferences API读写数据。而使用ContentProvider共享数据的好处是统一了数据访问方式。
<!--more--> 

回顾一下Android中数据存储的方式有很多种：
(1) SharePreferences通过api进行get、put操作   ----进程内部使用，可以实现资源共享，局限性比较大
可以参考：[SharedPreference实现不同进程间的数据共享](http://blog.csdn.net/shift_wwx/article/details/9243109)
(2) 通过file进行一些输入输出流控制   ----能实现进程间的共享，但是使用比较麻烦，个人不太喜欢用
(3) 通过SQLite进行数据库的读写   ----进程内部使用，可以实现资源共享，局限性比较大
(4) ContentProvider 进行数据的存储   ----可以实现进程间的共享，可以和SQLite配合使用(当然也可以用其他几个存储方式，稍后补充上实例)，一般实现的是大型数据操作
(5) 网络存储数据  ----比较麻烦，有局限性
(6) system.prop 也可以进行简单的数据存储    ----有局限性，有default值，可以临时修改，重启后就会恢复默认值或重新设置

**使用ContentProvider共享数据的好处是统一了数据访问方式，可以存储一些图片等的数据。而其他的访问方式是有局限性的。**
[参考这里](http://blog.csdn.net/shift_wwx/article/details/24350781)
## 二、Uri类简介
Uri代表了要操作的数据，Uri主要包含了两部分信息：

1.需要操作的ContentProvider
2.对ContentProvider中的什么数据进行操作

一个Uri由以下几部分组成：

     1.scheme：ContentProvider（内容提供者）的scheme已经由Android所规定为：content://。
     2.主机名（或Authority）：用于唯一标识这个ContentProvider，外部调用者可以根据这个标识来找到它。
     3.路径（path）：可以用来表示我们要操作的数据，路径的构建应根据业务而定，如下：
       • 要操作contact表中id为10的记录，可以构建这样的路径:/contact/10
       • 要操作contact表中id为10的记录的name字段， contact/10/name
       • 要操作contact表中的所有记录，可以构建这样的路径:/contact
       要操作的数据不一定来自数据库，也可以是文件等其他存储方式，如下:
         要操作xml文件中contact节点下的name节点，可以构建这样的路径：/contact/name
如果要把一个字符串转换成Uri，可以使用Uri类中的parse()方法，如下：

    Uri uri = Uri.parse("content://com.changcheng.provider.contactprovider/contact")

## 三、UriMatcher、ContentUrist和ContentResolver简介
因为Uri代表了要操作的数据，所以我们经常需要解析Uri，并从Uri中获取数据。Android系统提供了两个用于操作Uri的工具类，分别为UriMatcher和ContentUris。掌握它们的使用，会便于我们的开发工作。
UriMatcher：用于匹配Uri，它的用法如下：

       1.首先把你需要匹配Uri路径全部给注册上，如下：
       //常量UriMatcher.NO_MATCH表示不匹配任何路径的返回码(-1)。
       UriMatcher  uriMatcher = new UriMatcher(UriMatcher.NO_MATCH);

       //如果match()方法匹配content://com.changcheng.sqlite.provider.contactprovider/contact路径，返回匹配码为1
       uriMatcher.addURI(“com.changcheng.sqlite.provider.contactprovider”, “contact”, 1);//添加需要匹配uri，如果匹配就会返回匹配码

       //如果match()方法匹配content://com.changcheng.sqlite.provider.contactprovider/contact/230路径，返回匹配码为2
       uriMatcher.addURI(“com.changcheng.sqlite.provider.contactprovider”, “contact/#”, 2);//#号为通配符
      
       2.注册完需要匹配的Uri后，就可以使用uriMatcher.match(uri)方法对输入的Uri进行匹配，如果匹配就返回匹配码，匹配码是调用addURI()方法传入的第三个参数，
       假设匹配content://com.changcheng.sqlite.provider.contactprovider/contact路径，返回的匹配码为1。

ContentUris：用于获取Uri路径后面的ID部分，它有两个比较实用的方法：
* withAppendedId(uri, id)用于为路径加上ID部分
* parseId(uri)方法用于从路径中获取ID部分

ContentResolver：当外部应用需要对ContentProvider中的数据进行添加、删除、修改和查询操作时，可以使用ContentResolver类来完成，要获取ContentResolver对象，可以使用Activity提供的getContentResolver()方法。 ContentResolver使用insert、delete、update、query方法来操作数据。

## 四、详细图解

 ~ ~这下你能看懂了吧！~ ~

1、![](http://jackhai.qiniudn.com/1.png)

2、![](http://jackhai.qiniudn.com/2.png)

3、![](http://jackhai.qiniudn.com/3.png)

4、![](http://jackhai.qiniudn.com/4_总结.png)




