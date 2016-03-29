layout: post
title: 全面解析 Java 注解
date: 2015-12-12 19:20:22
tags: 
	- Java
comments: true
toc: true
copyright: true
---

## 1. 什么是注解 ##
Java 提供了一种源程序中的元素关联任何信息和任何元数据的途径和方法。

## 2. Java 中的常见注解 ##
**(1) JDK 自带的注解**

> @Override
@Deprecated
@Suppvisewarnings

<!--more-->
**(2)常见的第三方注解**
Spring:


> @Autowired
@Service
@Repository

Mybatis

> @InsertProvider
@UpdateProvider
@Options


## 3. 注解分类 ##

按运行机制分：
（1）**源码注解**：注解只在源码中存在，编译成 .class 文件就不存在了。
（2）**编译时注解**： 注解在源码和 .class 文件中都存在。例如 <code>@Override</code>, <code>@Deprecated</code>, <code>@Suppvisewarnings</code>。
（3）**运行时注解**： 在运行阶段还起作用，甚至会影响运行逻辑的注解。例如，<code>@Autowired</code> 。

按来源分：
（1）来自 JDK 的注解
（2）来自第三方的注解
（3）自定义的注解

最后补充一类特别的注解，叫做**元注解**，这是注解的注解。


## 4. 自定义注解 ##

下面重点介绍下自定义注解的使用。

### 自定义注解的语法要求 ###
下面给出一段具体的代码来分析下。
```java
@Target({ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Description{

	String desc();

	String author();

	int age() default 18;

}
```
下面对其中的语法做一些解释。
前四行表示的是元注解，也就对我们自定义的这个注解的注解。
<code>**@Target({ElementType.METHOD,ElementType.TYPE})**</code>
其中 ElementType.METHOD 中的 METHOD 表示该注解可声明的位置，METHOD 表示该注解在方法处声明。取值范围和含义如下：
- CONSTRUCTOR：构造方法声明
- FIELD：字段声明（包括枚举常量）
- LOCAL_VARIABLE：局部变量声明
- METHOD：局部方法声明
- PACKAGE：包声明
- PARAMETER：参数声明
- TYPE：类接口

<code>**@Retention(RetentionPolicy.RUNTIME)**</code>
其中 RetentionPolicy.RUNTIME 中的 RUNTIME 表示该注解的类型。取值范围和含义如下：
- SOURCE：只在源码显示，编译时丢弃
- CLASS：编译时会记录到 class 中，运行时忽略
- RUNTIME：运行时存在，可通过反射读取

<code>**Inherited**</code> 表示允许子类继承。

<code>**Documented**</code> 生成 javadoc 时会包含注解。

下面开始介绍自定义注解的主体部分，即
```java
public @interface Description{
	String desc();
	String author();
	int age() default 18;
}
```

<code>**@interface**</code> 是自定义注解时必须使用的关键字
<code>**Description**</code> 是你自定义注解的名称
<code>**desc**</code>, <code>**author**</code>,<code>**age**</code> 这三个均是注解的成员:
- 成员的类型是受限制的，合法的类型包括原始类型，String, Class，Annotation, Enumeration。
- 成员以无参无异常方式声明。
- 如果注解只有一个成员，则成员名必须为 value()，使用时可忽略赋值号( = )。
- 注解可以没有成员，没有成员的注解为标识注解。
- 可以使用 default 为成员制定一个默认值。

### 注解的使用 ###

注解使用的格式： <code>**@注解名(<成员名1>=<成员值1>, <成员名2>=<成员值2>, ...)**</code>
下面给个示例演示下：
注解的定义：
```
@Target({ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Description{

	String value();
}
```

注解的使用：
```java
@Description("I am class annotation")
public class HelloAnn{
	@Description("I am method annotation")
	public String name(){
		return null;
	}
}
```

### 注解的解析 ###

注解的解析：通过反射获取类、函数或成员上的**运行时**注解信息，从而实现动态控制程序运行的逻辑。

不了解反射的请[点击这里](../../../../2015/12/06/reflection-in-java/)。

注解的解析的主要步骤如下：
step1：使用类加载器加载类；
step2：找到类上面的注解；
step3：拿到注解实例；
step4：找到方法上的注解。

对于上面使用的注解我们来做一个解析的示例，
```java
public class ParseAnn{
	public static void main(String[] args){
		// step1：使用类加载器加载类
		try{
			Class c = Class.forName("HelloAnn");  //注意这里需要把 HelloAnn 的包名加上
			// step2：找到类上面的注解
			boolean isExist = c.isAnnotationPresent(Description.class);
			if(isExist){
				// step3：拿到注解实例
				Description d = (Description)c.getAnnotation(Description.class);
				System.out.println(d.value());
			}
			// step4：找到方法上的注解
			Method[] ms = c.getMethods();
			for(Method m:ms){
				isExist = m.isAnnotationPresent(Description.class);
				if(isExist){
					Description d = (Description)m.getAnnotation(Description.class);
					System.out.println(d.value());
				}
			}
		}catch(ClassNotFoundException e){
			e.printStackTrace();
		}
	}
}

```

输出的结果是：
> I am class annotation
> I am method annotation

关于获取方法中的注解还可以用下面方法：

```java
for(Method m:ms){
	Annotation[] as = m.getAnnotations();
	for(Annotation a:as){
		if(a instanceof Description){
			Description d = (Description)a;
			System.out.println(d.value());
		}	
	}
}
```