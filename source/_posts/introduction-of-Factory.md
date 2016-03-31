layout: post
title: 设计模式之工厂模式
date: 2015-12-17 19:20:22
tags: 
	- Java
	- 设计模式
comments: true
copyright: true
---

**工厂模式：** 定义一个用于创建对象的接口，让子类决定实例化哪一个类，工厂方法使一个类的实例化延迟到其子类。

**类图：**
![工厂模式类图](/img/articles/factory1.png)

<!--more-->

**优点：**
(1) 可以使代码结构清晰，有效地封装变化。在编程中，产品类的实例化有时候是比较复杂和多变的，通过工厂模式，将产品的实例化封装起来，使得调用者根本无需关心产品的实例化过程，只需依赖工厂即可得到自己想要的产品。

(2) 对调用者屏蔽具体的产品类。如果使用工厂模式，调用者只关心产品的接口就可以了，至于具体的实现，调用者根本无需关心。即使变更了具体的实现，对调用者来说没有任何影响。

(3) 降低耦合度。产品类的实例化通常来说是很复杂的，它需要依赖很多的类，而这些类对于调用者来说根本无需知道，如果使用了工厂方法，我们需要做的仅仅是实例化好产品类，然后交给调用者使用。对调用者来说，产品所依赖的类都是透明的。

**下面来举个例子具体说明**
假设我们有个汽车工厂要生产各种汽车，比如小面包车，货车等。
我们先声明一个一个**汽车接口**：

```java
package com.cighao.factory;
public interface VehicleInterface {
	public void create();
}
```

我们让**小面包车**和**货车**分别来实现这个接口：

```java
package com.cighao.factory;
public class Minibus implements VehicleInterface {
	@Override
	public void create(){
		System.out.println("这是一个面包车");
	}
}
```
```java
package com.cighao.factory;
public class Truck implements VehicleInterface {
	@Override
	public void create() {
		System.out.println("这是一个小货车");
	}
}
```

其实这个时候只要用 `VehicleInterface minbus = new Minibus();` `minibus.create();` 这两个语句就可以来生产一个小面包车或者汽车了，但是为了更友好，我们可以把汽车的生产放到一个汽车生产厂的类中去实现。如下是一个**汽车生产厂类**：

```java
package com.cighao.factory;
public class VehicleFactory {
	public VehicleInterface getVehicle(String key){
		if("minibus".equals(key)){
			return new Minibus();			
		}else if("truck".equals(key)){
			return new Truck();
		}
		return null;
	}
}

```

那么现在现在如何去生产一个小面包车或者货车呢，下面给出一个示例：

```java
VehicleFactory factory = new VehicleFactory ();
VehicleInterface minibus = factory.getVehicle("minibus");
minibus.create();
VehicleInterface truck = factory.getVehicle("truck");
truck.create();
```

按照上面的方式我们就可以生产小面包车和货车了。但是我们会发现在汽车生产厂里的 getVehicle() 这个生产方法有很多 `if else` 语句去判断生产的车的类型，这样很繁琐，所以我们可以利用[反射机制](../../../../2015/12/06/reflection-in-java/)来对**汽车生产类 VehicleFactory 进行优化**，优化后的代码如下：

```java
package com.cighao.factory;
public class VehicleFactory {
	public VehicleInterface getVehicle(String className){
		try {
			VehicleInterface vehicle = (VehicleInterface) Class.forName(className).newInstance();
			return vehicle;
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}
}
```

这种方式有个小小的缺陷就是，getVehicle() 这个方法传入的参数不再是一个关键字了，而是一个**完整的类名(包括包名)**，像下面这样：

```java
VehicleFactory factory = new VehicleFactory ();
VehicleInterface minibus = factory.getVehicle("com.cighao.factory.Minibus");
minibus.create();
```


</br>
题外话
-----------------------------
完整工厂模式方法介绍完了，下面我们来说点题外的话，如何去**优化**一下，以至于不需要在 getVehicle() 方法中完整类名只要输入关键字也可以被识别。有兴趣的可以往下看，**下面的跟设计模式并无关系**。 
首先建立一个 `type.properties` 文件，输入如下内容： (放在与上面的类同一个路径下)

```
minibus=com.cighao.factory.Minibus
truck=com.cighao.factory.Truck
```

这个时候再写一个解析这个 `type.properties` 文件的类 **PropertiesReader**：

```java
public class PropertiesReader {
	public Map<String, String> getProperties() {
		Properties props = new Properties();
		Map<String, String> map = new HashMap<String, String>();
		try {
			InputStream in = getClass().getResourceAsStream("type.properties"); //注意文件路径
			props.load(in);
			Enumeration en = props.propertyNames();
			while (en.hasMoreElements()) {
				String key = (String) en.nextElement();
				String property = props.getProperty(key);
				map.put(key, property);
			    //System.out.println(key + "  " + property);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		return map;
	}
}
```

这个时候再在 **getVehicle**上做些微弱的修改：
```java
public class VehicleFactory {
	public VehicleInterface getVehicle(String key){
		try {
			Map<String, String> map = new PropertiesReader().getProperties();
			VehicleInterface vehicle = (VehicleInterface) Class.forName(map.get(key)).newInstance();
			return vehicle;
		} catch (Exception e) {
			e.printStackTrace();
		}
		return null;
	}
}
```

现在就可以按照下面的这种简洁方式调用了：
```java
VehicleFactory factory = new VehicleFactory ();
VehicleInterface minibus = factory.getVehicle("minibus");
minibus.create();
```