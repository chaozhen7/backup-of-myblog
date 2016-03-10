layout: post
title: 第一个 Struts2 程序
date: 2015-12-14 17:20:22
tags: 
	- Java
	- Java Web
	- Struts2

comments: true
---


下面主要介绍如何在 myEclipse 下使用 Struts2 搭建一个简单的 hello world 程序

## 1. Struts2 的环境需求 ##
Servlet API 2.4
JSP API 2.0
Java 5

## 2. 第一个 struts2 项目的搭建 ##

step1. 首先我们在 myeclipse 下新建一个 web 工程，假设命名为 hellostruts

<!--more-->

step2. 鼠标放在项目名称上右键依次选择 **MyEclipse --> Project Facets --> Install Apache Struts(2.x) Facet**

step3. 这个时候已经配置好 struts2 了，下面我们来看看项目的变化。
（1）项目的 lib-INF 的 lib 目录下多了许多和 struts 相关的 jar 包。
（2）在 web.xml 下多了如下代码：
```html
 <filter>
    <filter-name>struts2</filter-name>
    <filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
  </filter>
  <filter-mapping>
    <filter-name>struts2</filter-name>
    <url-pattern>*.action</url-pattern>
  </filter-mapping>
```
(3) 在 <code>src</code> 目录下多了个 <code>struts.xml</code> 的文件，其初始内容如下：
```html
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC "-//Apache Software Foundation//DTD Struts Configuration 2.1//EN" "http://struts.apache.org/dtds/struts-2.1.dtd">

<struts>

</struts>    
```
step4. 编写第一个 action 类，注意要继承 <code>ActionSupport</code> 这个类，代码如下：
```java
package com.cighao.action;
import com.opensymphony.xwork2.ActionSupport;
public class HelloStruts extends ActionSupport {
	
	@Override
	public String execute() throws Exception{
		//System.out.println("start action");
		return "success";
	}
}
```

step5. 在 <code>struts.xml</code> 中写入配置信息。代码如下：
```html
<struts>

	<package name="default" namespace="" extends="struts-default">
		<action name="hello" class="com.cighao.action.HelloStruts" >
			<result name="success">/index.jsp</result>
		</action>
	</package>
  
</struts>    
```

step6. 添加视图层的内容，我们在 index.jsp 中加入 *hello struts2* 这段文字。

## 3. 项目演示   ##

我们部署项目后，在浏览器中输入 ```localhost:8080/hellostruts/hello.action``` 就可以看到 ```index.jsp``` 中的内容，这里显示的是:
> hello struts2 


</br>
想更多的了解 Struts2 的其他内容可以参见本博客的其他关于 Struts 的文章。