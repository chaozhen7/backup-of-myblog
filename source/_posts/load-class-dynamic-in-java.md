layout: post
title: Java 动态加载类
date: 2015-12-6 19:40:22
tags: 
	- Java
comments: true
---
## 1. 什么是动态加载类 ##

在编译时刻加载类是静态加载类；运行时刻加载类是动态加载类。

在前面的介绍 [Java反射机制](../../../../2015/12/06/reflection-in-java/) 时提到了 ***Class.forName()*** 函数，它不仅表示了类的类类型，还表示了动态加载类。

## 2. 动态加载类具体分析 ##
首先我们用一个案例来引入动态加载类的机制。
下面我们先编写一个类：
<!--more-->
```java
class Office{
	public static void main(String[] args){
		if("Word".equals(args[0])){
			Word w = new Word();
			w.start();
		}
		if("Excel".equals(args[0])){
			Excel e = new Word();
			e.start();
		}		
	}
}
```
当我们在命令行下输入 <code>javac Office.java</code> 命令时，很明显时会报错的，因为 Word , Eecle 这些类都不存在，而且 start() 方法也不存在。
***思考：*** 上面代码两个 if 分支有可能都不成立，也就是 Word , Excel 类可能都用不到，但是这个代码却不能编译通过。

好了，现在我们就按照要求写一个 Word 类吧。
```java
class Word{
	public static void start(){
		System.out.println("word...start...");
	}
}
```
这时我们编译完 Word.java 后，在命令行下输入 <code>javac Office.java</code> 命令时，还是会报错，因为虽然有了 Word 类但是还没有 Excel 类。这个时候有没有发现真的很不友好，我们可能只需要用到 Word, 但是我们还要必须要有 Excel 才能让程序通过编译。假设这个程序有一百种功能，那么只要有一个功能有问题程序就不能编译，很明显我们并希望这样。

***可是为什么会出现这种问题呢？***
new 创建对象，是静态加载类，在编译时刻就需要加载所有可能使用到的类。

***那怎么解决呢？***
之所以出现上面的问题是因为静态加载类，如果改为动态加载类就好多了，那我们重新写个新的更好的 Office 类来感受下。
``` java
class OfficeBetter{
	public static void main(String[] args){
		try{
			Class c = Class.forName(args[0]); //动态加载类  
		}catch(Exception e){
			e.printStackTrace();		
		}
	}
}
```
这个时候我们利用 <code>javac OfficeBetter.java</code> 去编译时就不会出现任何问题了。但是当我们运行呢，比如输入 <code>java OfficeBeter Excel</code> ，这个时候就会报错了，因为没有 Excel 类，这样就可以做到了在运行时再检查错误了。

按照上面的方法动态加载类完之后，我们就可以实例化一个对象了，在 [Java反射机制](../../../../2015/12/06/reflection-in-java/) 中我们提到了 newInstance() 方法，那么问题来，这里利用 c.newInstance() 获取一个对象后，该把它强制类型转换给谁呢，Word 还是 Excel 呢？

好了，为了解决这个问题，我们就给 Word 和 Excel 来统一一个标准，我们下面来新建一个接口。
```java
interface OfficeAble{
	public void start();
}
```
然后就可以来利用 newInstance() 来实例化一个对象了。修改了 OfficeBetter代码如下：
``` java
class OfficeBetter{
	public static void main(String[] args){
		try{
			Class c = Class.forName(args[0]); //动态加载类  
			OfficeAble oa = (OfficeAble)c.newInstance();
			oa.start();
		}catch(Exception e){
			e.printStackTrace();		
		}
	}
}
```

现在如果我们要用 Word，只要让它实现 OfficAble 这个接口就行了，代码如下：
```java
class Word implements OfficeAble{
	public static void start(){
		System.out.println("word...start...");
	}
}
```
好了，这个时候我们顺利编译完，就可以运行程序了，例如输入 <code>java OfficeBetter Word</code>程序就会打印出相应的结果。

这个时候如果我们要加一个Excel，只要新写一个Excel类去实现 OfficeAble 的接口就好了，原来的代码完全不用改。
比如我们新加一个 Excel：
```java
class Excel implements OfficeAble{
	public static void start(){
		System.out.println("excel...start...");
	}
}
```
这个时候我们都不用重新编译原来的程序就可以直接使用 <code>java OfficeBetter Excel</code> 来运行程序了。后面我们如果还需要添加什么功能只要直接新建一个类去实现 OfficeAble 接口就好了，其他的都不用修改。有木有觉得很方便。

</br>
全文完！