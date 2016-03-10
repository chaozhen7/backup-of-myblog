layout: post
title: 设计模式之抽象工厂模式
date: 2015-12-18 19:20:22
tags: 
	- Java
	- 设计模式
comments: true
---

前面介绍了[工厂模式](../../../../2015/12/17/introduction-of-Factory/)，现在我们介绍下抽象工厂模式。

**定义** 

为创建一组相关或相互依赖的对象提供一个接口，而且无需指定它们的具体类。

**抽象工厂模式与工厂模式的区别**

(1) 工厂模式是一种极端情况的抽象工厂模式，而抽象工厂模式可以看成是工厂模式的推广。
(2) 工厂模式用来创建一个产品的等级结构，而抽象工厂模式是用来创建多个产品的等级结构。
(3) 工厂模式只有一个抽象产品类，而抽象工厂模式有多个抽象产品类。
<!--more-->

**下面举个案例来分析**

假设我们要生产一个汽车，汽车可以分为**美系**的汽车和**日系**的汽车，美系和日系汽车又分为**小面包车**和**货车**。

首先来分别建一个**小面包车和货车的接口类**：
```java
public interface Minibus {
	public void create();
}
```

```java
public interface Truck {
	public void create();
}
```
然后我们要分别新建一个**汽车的生产类**去生产日系小面包车，日系货车，美系小面包车，美系货车：

```java
public class JapaneseMinibus implements Minibus{
	@Override
	public void create(){
		System.out.println("日系小面包车");
	}
}
```

```java
public class JapaneseTruck implements Truck {
	@Override
	public void create() {
		System.out.println("日系货车");
	}
}
```

```java
public class AmericanMinibus implements Minibus {
	@Override
	public void create() {
		System.out.println("美系小面包车");
	}
}
```

```java
public class AmericanMinibus implements Minibus {
	@Override
	public void create() {
		System.out.println("美系小面包车");
	}
}
```

接下来再新建一个**生产汽车的工厂接口类**：
```java
public interface VehicleFactory {
	public Minibus createMinibus();   // 生产小面包车
	public Truck createTruck();     //生产货车
}
```

接着我们需要新建一个类去实现上面的生产汽车的工厂接口，因为生产的汽车分为日系和美系，所以要新建两个这样的类，一个是生产日系，一个是生产美系的。

生产日系车的工厂：
```java
public class JapaneseFactory implements VehicleFactory {
	@Override
	public Minibus createMinibus() {
		return new JapaneseMinibus();  //日系小面包车
	}
	@Override
	public Truck createTruck() {
		return new JapaneseTruck();   //日系货车
	}
}
```

生产美系车的工厂：

```java
public class AmericanFactory implements VehicleFactory {
	@Override
	public Minibus createMinibus() {
		return new AmericanMinibus();  //美系小面包车
	}
	@Override
	public Truck createTruck() {
		return new AmericanTruck();   //美系货车
	}
}
```

最后我们就可以用这种方式去生产我们的汽车了：
```java
VehicleFactory facoty = new AmericanFactory();
Minibus minibus =  facoty.createMinibus();
minibus.create();
```