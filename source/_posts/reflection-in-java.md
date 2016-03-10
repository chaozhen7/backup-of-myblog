layout: post
title: Java 中的反射机制
date: 2015-12-6 16:40:22
tags: 
	- Java
comments: true
toc: true
---

## 1. 什么是反射
Java 反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

Java反射机制主要提供了以下功能： 在运行时判断任意一个对象所属的类；在运行时构造任意一个类的对象；在运行时判断任意一个类所具有的成员变量和方法；在运行时调用任意一个对象的方法；生成动态代理。
<!--more-->

## 2. Class 类的介绍
java 是一门面向对象的语言，在面向对象的世界里一切事物都是对象。那么问题来了，一个类是不是对象呢，如果是，是谁的对象呢。答案是肯定的，类也是一个对象，类是 ***java.lang.Class*** 类的一个实例对象。那么怎么去表示呢，下面举个案例来说明。
我们首先创建一个普通的类，假设为类名为 Foo:
```java
package com.cighao.reflect;
class Foo{

}
```
正常情况下我们可以用 <code>Foo foo1 = new Foo();</code> 来表示Foo的一个实例对象。
上面说了，类也是一个实例对象，是 *Class* 的实例对象，那我我们如何来表示呢，有三种方法可以表示。
**第一种：**
```java
Class c1 = Foo.class;
```
通过这种方式我们可以知道任何一个类都有一个隐含的静态成员变量 ***class***。
**第二种：**
```java
Class c2 = foo1.getClass();
```
已经知道该类的对象时，可以通过 ***getClass*** 方法得到。
上面两种方法得到的 c1,c2 表示了类的类类型(class type)。上面说了类也是对象，是class类的实例对象，这个对象我们称之为该类的类类型。
**第三种：**
```java
c3 = Class.forName("com.cighao.reflect.Foo");  //Foo前需要添加完整包名，该语句还需要添加异常处理
```
上面通过三种方法获得了类的类类型，这里的 c1,c2,c3 是完全相等的，即 c1=c2=c3，因为一个类只可能是 Class 类的一个实例对象。
获得了一个类的类类型后我们就可以根据这个类类型来创建该类的对象实例了。方法如下：
``` java
Foo foo = (Foo)c1.newInstance();   //需要有无参数的构造方法
```
当然上面的 c1 也可以替换成 c2 或者 c3，通过这种方式创建一个对象实例需要该类有一个无参的构造方法。


## 3. 类的反射
上面我们介绍了类的类类型，给定一个类我们都可以通过上面介绍的三种方法中的某一种去得到该类的类类型(class类的一个实例对象)。例如：
```java
Class c1 = int.class;     //int 的类类型
Class c2 = String.class;   //String类的类类型
Class c3 = double.class;
Class c4 = Double.class; //与上面的double.class不同
Class c5 = void.class;
```
关于 Class 这个类，他也有一些方法，下面简单介绍下Class类的一些方法：

| 方法 | 描述 |
|----|----|
|getName()|以 String 的形式返回此 Class 对象所表示的实体（类、接口、数组类、基本类型或 void）名称，包含包名|
|getSimpleName()| 返回源代码中给出的底层类的简称，不包含包名|
|getMethods()|返回一个 Method 对象的数组，class对象所表示的类的所有public的函数，包括父类继承而来的|
|getDeclaredMethods()|返回一个 Method 对象的数组，class对象所表示的类的自己所声明得方法|
|getFields()|返回一个 Field 对象的数组，Class 对象所表示的类的所有可访问公共字段|
|getDeclaredFields()|返回一个 Field 对象的数组，Class 对象所表示的类自己所声明的所有字段。|
|getConstructors()|返回一个 Constructor 对象的数组，Class 对象所表示的类的所有公共构造方法|
|getDeclaredConstructors()|返回一个 Constructor 对象的数组，Class 对象表示的类自己所声明的所有构造方法。|

关于 Class 类的API上面只是介绍了很少很少的一部分，具体的可以去查阅JDK API。

## 4. 方法的反射
java 中有个 Method 类，这个类表示的是方法对象。下面举个例子来说说方法的反射吧。
首先我们来定义一个类，在这个类里声明几个方法。
```java
class A{
	public void print(){
		System.out.println("helloworld");
	}
	public void print(int a,int b){
		System.out.println(a+b);
	}
	public void print(String a,String b){
		System.out.println(a.toUpperCase()+","+b.toLowerCase());
	}
}
```
接下来我们要获取上面代码中的 print(int,int)方法。
首先，要获取一个方法就要获取类的信息，获取类的信息首先要获取类的类类型：
```java
A a1 = new A();
Class c = a1.getClass();
```
其次，可以通过方法名称和参数列表来选取具体方法：

```java
//Method m =  c.getMethod("print", new Class[]{int.class,int.class});
Method m = c.getMethod("print", int.class,int.class);
```
上面两种写法都可以，注意: getMethod 获取的是public的方法, getDelcaredMethod 获取的是自己声明的方法。

获取到方法后就可以调用了：
```java
//Object o = m.invoke(a1,new Object[]{10,20});
Object o = m.invoke(a1, 10,20);
```
上面的两种调用方法都可以。
如果要获取 print(String,String) 方法，也可以用同样的方式，代码如下:
```java

Method m1 = c.getMethod("print",String.class,String.class);
//用方法进行反射操作
o = m1.invoke(a1, "hello","WORLD");  //完全等同 a1.print("hello", "WORLD");
	
//  Method m2 = c.getMethod("print", new Class[]{});
Method m2 = c.getMethod("print");
// m2.invoke(a1, new Object[]{});
m2.invoke(a1);
```