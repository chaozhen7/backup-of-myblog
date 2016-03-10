layout: post
title: 通过 Java 的反射了解集合泛型的本质
date: 2015-12-06 21:20:22
tags: 
	- Java
comments: true
---

关于 Java 反射机制可以[点击这里](../../../../2015/12/06/reflection-in-java/) 。

下面开始正题吧，以集合中的 ArrayList 为例来介绍吧。
当我们要新建一个 ArrayList 对象时可以用下面的语句：
```java
ArrayList list = new ArrayList()
```
还可以在新建这个对象时给他规定 ArrayList 中对象的类型，比如：
```java
ArrayList<String> list1 = new ArrayList<String>();
```
<!--more-->
那么问题来了，如果我们这时候执行：
```java
list1.add("hello"); 
```
上面的语句是可以正常执行没有问题的，但是如果要执行：
```java
list1.add(20);  //这是错误的
```
这个时候就要报错了。因为20不是String类型，集合的泛型就是为了防止用户来错误输入的。

通过前面我们对[反射](../../../../2015/12/06/reflection-in-java/)的学习，我们来做下面的实验：
``` java
Class c1 = list.getClass();
Class c2 = list1.getClass();
System.out.println(c1==c2);
```
这个时候我们会惊奇的发现，在控制台输出的结果是 true。
我们知道反射的操作是编译之后的操作。
那么通过上面的小例子我们就可以得到这个结论，编译之后集合的泛型是去泛型化的，Java 集合中的泛型是防止错误输入的，只在编译阶段有效，绕过编译阶段就无效了。
下面我们可以通过方法的反射来绕过编译，来验证上面的结论，继续在上面的代码后面添加新的代码：
```java
try{
	Method m = c2.getMethod("add",object.class);
	m.invoke(list1,100);  //绕过编译操作，就绕过了泛型
}catch(Exception){
	e.printStackTrace();
}
```
这个时候程序就会正常运行不会报错，那我们接着来打印下 list1 中的内容：
```java
System.out.println(list1);
```
这个时候我们惊讶的发现，list1 的输出内容中居然包含100。
但是如果我们这样去输出呢：
```java
for(String s:list1)
	System.out.println(s);
```
这样就会抛异常了，因为这个时候他会把每个对象都当成String类型的了。


</br>
全文完