---
title: 反射 study
date: 2021-12-17 
tags:
 - java
---
## 反射的概述
JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。
要想解剖一个类,必须先要获取到该类的字节码文件对象。而解剖使用的就是Class类中的方法.所以先要获取到每一个字节码文件对应的Class类型的对象.

以上的总结就是什么是反射
反射就是把java类中的各种成分映射成一个个的Java对象
例如：一个类有：成员变量、方法、构造方法、包等等信息，利用反射技术可以对一个类进行解剖，把个个组成部分映射成一个个对象。（其实：一个类中这些成员方法、构造方法、在加入类中都有一个类来描述）
如图是类的正常加载过程：反射的原理在与class对象。
熟悉一下加载的时候：Class对象的由来是将class文件读入内存，并为之创建一个Class对象。
![反射很简单的啦](https://s2.loli.net/2021/12/17/4McNIFgwPq8aKYE.png)
## java中为什么需要反射？反射要解决什么问题？
Java中编译类型有两种：

静态编译：在编译时确定类型，绑定对象即通过。
动态编译：运行时确定类型，绑定对象。动态编译最大限度地发挥了Java的灵活性，体现了多态的应用，可以减低类之间的耦合性。
Java反射是Java被视为动态（或准动态）语言的一个关键性质。这个机制允许程序在运行时透过Reflection APIs取得任何一个已知名称的class的内部信息，包括其modifiers（诸如public、static等）、superclass（例如Object）、实现之interfaces（例如Cloneable），也包括fields和methods的所有信息，并可于运行时改变fields内容或唤起methods。

Reflection可以在运行时加载、探知、使用编译期间完全未知的classes。即Java程序可以加载一个运行时才得知名称的class，获取其完整构造，并生成其对象实体、或对其fields设值、或唤起其methods。

反射（reflection）允许静态语言在运行时（runtime）检查、修改程序的结构与行为。
在静态语言中，使用一个变量时，必须知道它的类型。在Java中，变量的类型信息在编译时都保存到了class文件中，这样在运行时才能保证准确无误；换句话说，程序在运行时的行为都是固定的。如果想在运行时改变，就需要反射这东西了。

实现Java反射机制的类都位于java.lang.reflect包中：

Class类：代表一个类
Field类：代表类的成员变量（类的属性）
Method类：代表类的方法
Constructor类：代表类的构造方法
Array类：提供了动态创建数组，以及访问数组的元素的静态方法
一句话概括就是使用反射可以赋予jvm动态编译的能力，否则类的元数据信息只能用静态编译的方式实现，例如热加载，Tomcat的classloader等等都没法支持
## 使用
共有64个方法，用到那个就详解哪一个
1、获取Class对象的三种方式

1.1 Object ——> getClass();
1.2 任何数据类型（包括基本数据类型）都有一个“静态”的class属性
1.3 通过Class类的静态方法：forName（String  className）(常用)
```

/**
 * 获取Class对象的三种方式
 * 1 Object ——> getClass();
 * 2 任何数据类型（包括基本数据类型）都有一个“静态”的class属性
 * 3 通过Class类的静态方法：forName（String  className）(常用)
 *
 */
        //第一种方式获取Class对象
        User user=new User();
        Class aClass1 = user.getClass();//获取class对象
        //System.out.println(aClass1);
        //第二种方式获取Class对象
        Class aClass2 = User.class;
         //判断第一种方式获取的class对象是否和第二获取的是同一个
        //System.out.println(aClass1 == aClass2);
        //第三种方式获取Class对象
        Class aClass3=null;
        try {
          //注意此字符串必须是真实路径，就是带包名的类路径，包名.类名
            aClass3 = Class.forName("com.example.User");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        //System.out.println(aClass3==aClass2);
        System.out.println("*****************获取公有、无参的构造方法*************");
        //1>、因为是无参的构造方法所以类型是一个null,不写也可以：这里需要的是一个参数的类型，切记是类型
        //2>、返回的是描述这个无参构造函数的类对象。
        Constructor constructor=null;
        try {
            constructor = aClass1.getConstructor(null);
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }
        System.out.println("获取公有、无参的构造方法"+constructor);
        System.out.println("*************获取公有字段**并调用*****************");
        //private修饰的不可以访问
        Field name = null;
        try {
            name = aClass1.getField("giao");
        } catch (NoSuchFieldException e) {
            e.printStackTrace();
        }
        System.out.println("获取公有字段**并调用"+name);
        System.out.println("***************获取私有的show4()方法******************");
        //需要两个参数，一个是要调用的对象，一个是实参
        Method giao=null;
        try {
            giao = aClass1.getDeclaredMethod("giao", String.class, String.class);
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        }
        System.out.println("获取了user里面的私有方法"+giao);
        //解除私有限定
        giao.setAccessible(true);
        Object invoke = null;
        try {//需要两个参数，一个是要调用的对象（获取有反射），一个是实参数
            invoke = giao.invoke(user, "z", "h");
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
        System.out.println("调用了user里面的私有方法"+invoke);
```
