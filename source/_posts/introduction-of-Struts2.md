layout: post
title: 初识 Struts2
date: 2015-12-14 21:40:22
tags: 
	- Java
	- Java Web
	- Struts2
comments: true
toc: true
copyright: true
---
## 1. Struts2 的体系结构

从一个高水平看，Struts 2 是一个 pull-MVC（或 MVC2）框架。Struts 2 的模型-视图-控制器模式由下面的
五个核心部件实现：**动作，拦截器，值栈/OGNL，结果/结果类型，视图技术。**
Struts 2 与传统的 MVC 框架稍有不同，其中动作担任模型的角色，而不是控制器的角色，虽然有一些重叠。
下图是关于 Struts2 体系结构的一个描绘
![Struts2 体系结构](/img/articles/struts.jpg)

<!--more-->

上面的图描绘 模型，视图和控制器到 Struts 2 高级架构。控制器是由 Struts 2 调度 servlet 过滤器和拦截器实现的，模型是由动作实现的，视图是由结果类型和结果结合而成的。值栈和 OGNL 提供共同主线，连接和集成其他组件。除了上面的组件，还有很多与配置相关的信息。不仅要配置 web 应用程序，也要配置动作，拦截器，结果 等等。

## 2. Struts2 的核心文件 ##

***web.xml***
任何 MVC 框架都需要与 web 应用整合，这就不得不借助 web.xml 文件，只有配置在 web.xml 文件中 Servlet 才会被应用加载。
通常，所有的 MVC 框架都需要 web 应用加载一个核心控制器，对于 Struts2 框架而言，需要加载 <code>StrutsPrepareAndExecuteFilter</code>, 只要 web 应用负责加载 <code>StrutsPrepareAndExecuteFilter</code> , <code>StrutsPrepareAndExecuteFilter</code> 将会加载 Struts2 框架。 

***struts.xml***
struts2 的核心配置文件。
该文件主要负责管理应用中的 Action 映射，以及该 Action 包含的 Result 定义等。
struts.xml中包含的内容：
- 全局属性
- 用户请求和相应 Action 之间的对应关系
- Action 可能用到的参数和返回结果
- 各种拦截器的配置

***struts.properties***

struts2 框架的全局属性文件，自动加载，和 struts.xml 放在同一个目录下。
该文件包含很多 key-value 键值对。
该文件完全可以配置在 struts.xml 文件中，使用 constant 元素。

## 3. 详解 struts.xml ##
前面我们介绍了关于 [Struts2 的第一个小示例](../../../../2015/12/14/first-project-of-struts2/) 这里我们结合其中的 struts.xml 文件来还谈谈 struts.xml 文件的作用。下面给出该示例中的 struts.xml 文件中的内容：
``` html
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.1//EN" "http://struts.apache.org/dtds/struts-2.1.dtd">

<struts>
	<package name="default" namespace="" extends="struts-default">
		<action name="hello" class="com.cighao.action.HelloStruts" >
			<result name="success">/index.jsp</result>
		</action>
	</package>
</struts>

```
**package 标签**
package 提供了将多个 ```Action```组织为一个模块的一种方式。package 的名字必须是唯一的。package 可以扩展，当一个 package 扩展自另一个 package 时，该 package 会在本身配置的基础上加入扩展的 package 的配置，父配置必须在子配置前配置。
package 中的各属性的含义：
 ```name```： 表示 package 的名称
```extends```： 表示继承的父 package 名称
```abstract```： 设置 package 的属性为抽象的 package，不能定义 action，值为 ture 或 false
```namespace```： 定义 package 命名空间，该命名空间影响到 url 的地址，例如，如果命名空间为 /test 那么访问的地址为 <code>localhost:8080/struts2/test/XX.action</code>

**action 标签**
每个 package 包里可以定义多个 action。一个 action 可以多次被映射（只要 action 配置中的 name 不同）。
```name```： action 名称，用来访问 action，访问路径为 localhost:8080/项目名称/package的namespace/actionName.action
```class```： 对应的类的路径(包括包名)
```method```： 调用 Action 中的方法名

action 中的 result 标签:
```result```： result 的名称，和 Action 类中的返回值相同。
```type```： result 的名称，默认为 superpackage 的 type。

**constant 标签**
constant 标签应在包外定义，前面说过 struts.properties 里面的内容可以利用 constant 在 struts.xml 中定义。
如在 struts.properties 中配置以下信息：
```  
struts.i18n.encoding=GB2312
struts.i18n.reload=true
```  
等同在 struts.xml 中这么定义：   
```html
<constant name="struts.i18n.encoding" value="GB2312"></constant>
<constant name="struts.i18n.reload" value="true"></constant>
```   

**其他标签**
```include```： 可以将每个功能模块独立到一个 xml 配置文件中，然后用 include 节点引用，例如:
``` html
<include file="struts-default.xml"></include>
```    

```interceptors```： 主要用来定义拦截器。属性有：
```name```： 拦截器的名称
```classs```： 拦截器类路径
格式：
```html
<interceptor name=" " class=" "></interceptor>
``` 
