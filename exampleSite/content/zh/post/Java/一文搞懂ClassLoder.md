---
title: 一文搞懂ClassLoder
date: 2020-03-30 11:53:09
image: 
categories:
 - Java
draft: true
---

# 1. 何为类加载器
顾名思义，类加载器（`Class Loader`）用来把Class文件加载Java类到JVM。在Java世界里，任意一个类，都必须由加载它的加载器与类自身一起共同确定在JVM中唯一性，每个一个类加载器，都拥有一个独立的类命名空间。JDK9之前的类加载器组结构如下：

![](_v_images/20200330235713911_15784.png)
- 引导类加载器（Bootstrap Class Loader）：它主要用于加载Java核心类库的，C++语言实现的，并不是继承`java.lang.ClassLoader`；
- 拓展类加载器（Ext Calss Loader）: 它主要用于加载Java拓展类库；
- 应用类加载器（App Class Loader）：它主要是根据指定CLASSPATH路径来完成对类的加载，通常可以通过`java.lang.ClassLoader#getSystemClassLoader`获取。通常情况下，我们若没有自定义类加载器，则使用该加载器加载类文件。


在Java中加载一个类加载JVM中需要经历一下过程：


![](_v_images/20200330233539224_30710.png)

# 2. 双亲委派机制


# 3. JDK9类加载器模型

