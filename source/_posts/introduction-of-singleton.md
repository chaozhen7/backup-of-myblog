layout: post
title: 设计模式之单例模式
date: 2015-12-17 17:20:22
tags: 
	- Java
	- 设计模式
comments: true
toc: true
copyright: true
---
## 1. 了解设计模式 ##
设计模式： 是一套反复使用，多数人知晓得的，经过分类编目的，代码设计经验的总结。
目的： 使用设计模式是为了可重用代码、让代码更容易被他人理解，保证代码可靠性。

## 2. 单例模式的介绍 ##

定义：确保一个类只有一个实例，而且自行实例化并向整个系统提供这个实例。

<!--more-->

分类
- 饿汉式单例： 在单例类被加载时候，就实例化一个对象交给自己的引用。
- 懒汉式单例： 在调用取得实例方法的时候才会实例化对象。

优点：
- 在内存中只有一个对象，节省内存空间。
- 避免频繁的创建销毁对象，可以提高性能。
- 避免对共享资源的多重占用。
- 可以全局访问。

适用场景：
- 需要频繁实例化然后销毁的对象。
- 创建对象时耗时过多或者耗资源过多，但又经常用到的对象。
- 有状态的工具类对象。
- 频繁访问数据库或文件的对象。
- 以及其他所有要求只有一个对象的场景。

## 3. 饿汉式单例 ##
正常情况下我们声明一个类是这样子：
```java
public class Singleton {
	// 不写构造方法也会有个默认的构造方法 Singleton(){ }
}
```
我们示例化这个对象时可以这样子 ``new Singleton()`` 但是我们可以执行多次这个语句去实例化多个对象，现在我们的要求是只能有一实例。所以我们可以更改把该对象的构造方法改为私有的然后在外部就不能去通过这个构造方法创建该对象的实例了。改为 ``private Singleton(){ }`` 后新的问题又来了，那么我们要怎么去获取一个唯一的实例呢，这个时候我们可以把这个唯一的实例作为该对象的一个属性封装起来，我们可以给该对象添加一个属性：
```java
public static Singleton instance=new Singleton();
```
这样我们在外部就可以直接使用 ``Singleton.instance`` 来访问这个实例了，而且该实例还是唯一的，为了体现面向对象的封装性，我们还可以把 static Singleton instance 这个属性改为私有的，然后在里面声明一个静态的 get 方法去获取这个实例。
完整的饿汉模式单例代码如下:
```java
public class Singleton {
	//1.将构造方法私有化，不允许外部直接创建对象
	private Singleton(){		
	}
	
	//2.创建类的唯一实例，使用private static修饰
	private static Singleton instance=new Singleton();
	
	//3.提供一个用于获取实例的方法，使用public static修饰
	public static Singleton getInstance(){
		return instance;
	}
}
```
通过上面的方法定义好一个单例类后，我们就可以通过下面的方式来使用那个单例。我们给出下面的测试代码：
```java
Singleton s1 = Singleton.getInstance();
Singleton s2 = Singleton.getInstance();
System.out.println(s1==s2);
```
这个时候我们看到的输出结果是 true 。这就说明了这个实例的唯一性。


## 4. 懒汉式单例 ##

前面说到的饿汉式单例中单例类被加载的时候，就实例化一个对象交给自己的引用。下面我们介绍的懒汉式只有在调用取得实例方法的时候才会实例化对象。代码如下：
```java
public class Singleton {
	private Singleton(){
	}
	
	private static Singleton instance;   //注意这里没有像上面一样实例化
	
	public static Singleton getInstance(){
		if(instance==null){
			instance=new Singleton();   //调用时才实例化
		}
		return instance;
	}
}
```

## 5. 饿汉式与懒汉式的区别 ##

饿汉模式的特点是加载类时比较慢，但运行时获取对象的速度比较快，线程安全
懒汉模式的特点是加载类时比较快，但运行时获取对象的速度比较慢，线程不安全(有争议)

## 6. 单例模式注意事项 ##
(1) 只能使用单例类提供的方法得到单例对象，不要使用反射，否则将会实例化一个新对象。
(2) 不要做断开单例类对象与类中静态引用的危险操作。
(3) 多线程使用单例使用共享资源时，注意线程安全问题。