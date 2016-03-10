layout: post
title: Java 序列化与反序列化
date: 2016-2-1 17:20:22
tags: 
	- Java
comments: true
toc: true
---

Java序列化是指把Java对象转换为字节序列的过程；而Java反序列化是指把字节序列恢复为Java对象的过程。

### **1. 为什么需要序列化与反序列化** ###

我们知道，当两个进程进行远程通信时，可以相互发送各种类型的数据，包括文本、图片、音频、视频等， 而这些数据都会以二进制序列的形式在网络上传送。那么当两个Java进程进行通信时，能否实现进程间的对象传送呢？答案是可以的。如何做到呢？这就需要Java序列化与反序列化了。换句话说，一方面，发送方需要把这个Java对象转换为字节序列，然后在网络上传送；另一方面，接收方需要从字节序列中恢复出Java对象。<!--more-->

当我们明晰了为什么需要Java序列化和反序列化后，我们很自然地会想Java序列化的好处。其好处一是实现了数据的持久化，通过序列化可以把数据永久地保存到硬盘上（通常存放在文件里），二是，利用序列化实现远程通信，即在网络上传送对象的字节序列。(该部分内容引用自[这里](http://blog.csdn.net/wangloveall/article/details/7992448))

### **2. 如何序列化与反序列化** ###

序列化与反序列化分别用到了这两个流类 ObjectOutputStream 和 ObjectInputStream。

如果要序列话一个对象的话这个对象还必须实现 Serializable 接口。下面举例来说明。

```java
public class Student implements Serializable{
	private String stuno;
	private String stuname;
	private int stuage;  
	public Student(String stuno, String stuname, int stuage) {
		super();
		this.stuno = stuno;
		this.stuname = stuname;
		this.stuage = stuage;
	}
}
```

上面已经写好了一个类，并且这个类已经实现了 Serializable 接口，如果我们要序列化这个类的一个对象只要用下面的代码即可：

```java
String file = "demo/obj.dat";
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
Student stu = new Student("10001", "张三", 20);
oos.writeObject(stu);
oos.flush();
oos.close();
```

上面的操作先实例化了一个 Student 对象，然后将这个对象序列化到了 demo 文件下的 obj.dat 文件中。

接下来，如果我们要从刚才序列化的 obj.dat 文件中反序列化出一个对象来，用下面的代码即可实现：
```java
String file = "demo/obj.dat";
ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
Student stu = (Student)ois.readObject();
ois.close();
```

### **3. transient 关键字** ###

上面的例子中，我们在序列化 Student 对象时，会把 Student 对象中的所有属性都序列化，反序列化时也是把所有属性都反序列化，但是如果对于某些属性我们不想让他序列化该怎么办呢，这个时候可以利用 transient 关键字去修饰这个属性，那么在默认的情况下，序列化的时候是不会去序列化这个属性的。比如上面的例子中，如果我们不需要序列化 stuage 属性时我们只要这样来修饰它就行了：
```java
private transient int stuage;  
```

### **4. 如何手动序列化** ###

对于用 transient 关键字修饰的属性我们是不可以用 ObjectOutputStream 对象的 writeObject() 方法去自动序列化的，但是我们可以手动去序列话。还是用上面的例子来继续讨论。
```java
public class Student implements Serializable{
	private String stuno;
	private String stuname;
	private transient int stuage;  //该元素不会进行jvm默认的序列化
	public Student(String stuno, String stuname, int stuage) {
		super();
		this.stuno = stuno;
		this.stuname = stuname;
		this.stuage = stuage;
	}
	private void writeObject(java.io.ObjectOutputStream s)
		        throws java.io.IOException{
		 s.defaultWriteObject();//把jvm能默认序列化的元素进行序列化操作
		 s.writeInt(stuage);//自己完成stuage的序列化
	}
	private void readObject(java.io.ObjectInputStream s)
		        throws java.io.IOException, ClassNotFoundException{
		  s.defaultReadObject();//把jvm能默认反序列化的元素进行反序列化操作
		  this.stuage = s.readInt();//自己完成stuage的反序列化操作
	}
}
```

这个时候序列化与反序列化的操作跟之前的还是一样：
```java
String file = "demo/obj.dat";
//序列化
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(file));
Student stu = new Student("10001", "张三", 20);
oos.writeObject(stu);
oos.flush();
oos.close();
//反序列化
ObjectInputStream ois = new ObjectInputStream(new FileInputStream(file));
Student stu1 = (Student)ois.readObject();
System.out.println(stu1);
ois.close();
```

### **5. 序列化中子类和父类构造函数的调用问题** ###

假设有三个类如下：
```java
class Foo implements Serializable{	
	public Foo(){
		System.out.println("foo...");
	}
}
class Foo1 extends Foo{
	public Foo1(){
		System.out.println("foo1...");
	}
}
class Foo2 extends Foo1{
	public Foo2(){
		System.out.println("foo2...");
	}
}
```

序列化与反序列化的操作如下：

```java
//序列化
System.out.println("----下面是序列化----");
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("demo/obj1.dat"));
Foo2 foo2 = new Foo2();
oos.writeObject(foo2);
oos.flush();
oos.close();
//反序列化
System.out.println("----下面是反序列化----");
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("demo/obj1.dat"));
Foo2 foo21 = (Foo2)ois.readObject();
ois.close();
```
控制台的输出如下：
> ----下面是序列化----
foo...
foo1...
foo2...
----下面是反序列化----

我们会发现一个有趣的现象就是在反序列化子类的过程中并没有调用父类的构造函数。

其中的缘由我们再举个例子来探讨，假设有如下的三个类：
```java
class Bar{
	public Bar(){
		System.out.println("bar");
	}
}
class Bar1 extends Bar implements Serializable{
	public Bar1(){
		System.out.println("bar1..");
	}
}
class Bar2 extends Bar1{
	public Bar2(){
		System.out.println("bar2...");
	}
}
```

序列化与反序列化的操作如下：

```java
//序列化
System.out.println("----下面是序列化----");
ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("demo/obj1.dat"));
Bar2 bar2 = new Bar2();
oos.writeObject(bar2);
oos.flush();
oos.close();
//反序列化
System.out.println("----下面是反序列化----");
ObjectInputStream ois = new ObjectInputStream(new FileInputStream("demo/obj1.dat"));
Bar2 bar21 = (Bar2)ois.readObject();
ois.close();
```

控制台的输出如下：
> bar
bar1..
bar2...
----下面是反序列化----
bar

这次的结果跟上次的又不太一样，这是为什么呢，其实是这样的：** 对子类对象进行反序列化操作时，如果其父类没有实现序列化接口那么其父类的构造函数会被调用**。