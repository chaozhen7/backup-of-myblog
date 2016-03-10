layout: post
title: Java Web 之监听器
date: 2015-12-05 18:35:22
tags: 
	- Java Web
	- Java
comments: ture
toc: true
---
 
## 1. 简介

__监听器__： 指专门用于对其他对象身上*发生的事件或状态改变*进行监听和相应处理的对象，当被监视的对象发生变化时，立即采取相应的行动。

Web监听器既可以监听客户端的请求也可以监听服务器端的操作，主要监听的是__*ServletContext* , *ServletRequest* , *HttpSession*__ 这三个对象。

当一个Web应用程序中有多个监听器时，监听器的启动顺序是按照其在web.xml注册的顺序启动的。当一个web应用程序中既有监听器，又有过滤器，还有Servlet时，启动的优先级是: __监听器 > 过滤器 > *Servlet*__ 。


<!--more-->

## 2. 监听器的分类

按监听的对象划分：
(1) 用于监听应用程序环境对象 (ServletContext) 的事件监听器。
(2) 用于监听用户会话对象 (HttpSession) 的事件监听器。
(3) 用于监听请求消息对象 (ServletRequest) 的事件监听器。

按监听的事件划分：
(1) 监听域对象自身的创建和销毁的事件监听器。
(2) 监听域对象中的属性的增加和删除的事件监听器。
(3) 监听绑定到 HttpSession 域中的某个对象的状态的事件监听器。

## 3. 监听器的分类介绍

下面按照监听的事件的不同来分别介绍三类监听器

### 3.1 监听域对象自身的创建和销毁的事件监听器

这类监听器主要是监听 *ServletContext* , *HttpSession* , *ServletRequest* 这三个对像的创建和销毁的。创建时需要分别实现 *ServletContextListener, HttpSessionListener, ServletRequestListener* 这三个接口。

以 __ServletContextListener__ 为例

*ServletContextListener* 主要是监听 *ServletContext* 对象的创建和销毁的。我们在新建一个 Class 的时候需要在在 Interfaces 选项中选择新建的类实现<code>ServletContextListener</code>接口。类的代码代码如下：
```java
// import 的相关部分已省略
public class MyServletContextListener implements ServletContextListener {

	@Override
	public void contextDestroyed(ServletContextEvent arg0) {
		// 当ServletContext对象销毁时调用
	}

	@Override
	public void contextInitialized(ServletContextEvent arg0) {
		// 当ServletContext对象创建时调用
	}
}

```

当然我们还要在 *web.xml* 中对该监听器进行注册，代码如下：
```html
<listener>
	<!-- 下面的listener-class包名也要包含在内 -->
	<listener-class>MyServletContextListener<listener-class>
</listener>
```
</br>
其余的两个也是类似。创建监听 *HttpSession* 和 *ServletRequest* 对象的创建和销毁的监听器，分别需要器实现<code>HttpSessionListener</code> 和 <code>ServletRequestListener</code> 这两个接口。示例的代码如下，*web.xml* 中注册的部分就省略了，仅一行代码而已。

```java
public class MyHttpSessionListener implements HttpSessionListener {
	public void sessionCreated(HttpSessionEvent httpsessionevent) {
		// 当HttpSession对象创建时调用
	}
	public void sessionDestroyed(HttpSessionEvent httpsessionevent) {
		// 当HttpSession对象销毁时调用
	}
}

```
```java
public class MyServletRequestListener implements ServletRequestListener {
	public void requestDestroyed(ServletRequestEvent servletrequestevent) {
		// 当ServletRequest对象销毁时调用
	}
	public void requestInitialized(ServletRequestEvent servletrequestevent) {
		// 当ServletRequest对象创建时调用
	}
}

```

### 3.2 监听域对象中的属性的增加和删除的事件监听器

这类监听器主要是监听 *ServletContext* , *HttpSession* , *ServletRequest* 这三个对像中的属性的增加和删除事件。

创建这类监听器需要分别实现 *ServletContextAttributeListener* , *HttpSessionAttributeListener* , *ServletRequestAttributeListener*  这三个接口，在这个三个接口中分别有 <code>attributeAdded()</code> , <code>attributeRemoved()</code> , <code>attributeReplaced()</code> 这三个方法。

以实现 *ServletContextAttributeListener* 这个接口的监听器为例，代码如下：(当然也是需要在 *web.xml* 中注册的)
```java
public class MyServletContextAttributeListener implements ServletContextAttributeListener {

	public void attributeAdded(ServletContextAttributeEvent servletcontextattributeevent) {
		// 添加属性时调用
	}

	public void attributeRemoved(ServletContextAttributeEvent servletcontextattributeevent) {		
		// 删除属性时调用
	}

	public void attributeReplaced(ServletContextAttributeEvent servletcontextattributeevent) {
		// 替代属性时调用
	}
}
```


### 3.3 监听绑定到 HttpSession 域中的某个对象的状态的事件监听器

*补充知识:* 
对于 Session 中的对象，主要有两组状态：(1)绑定和解除绑定，(2)钝化和活化。

Session的钝化机制: 本质就在于把服务器中不经常使用的 Session 对象暂时序列化到系统文件系统或是数据库系统中，当被使用时反序列化到内存中，整个过程由服务器自动完成。

关于Session中钝化和活化的补充知识，[请点击这里](../../../../2015/12/05/states-of-session/)

对于创建这类监听器有两个接口可供实现：
(1) HttpSessionBindingListener
- 绑定： valueBound 方法
- 解除绑定： valueUnbound 方法

(2) HttpSessionActivationListener
- 钝化： sessionWillPassivate 方法
- 活化： sessionDidActivate 方法

注意：这两个监听器不需要到 *web.xml* 中去注册。

我们创建一个类去实现这个接口时，并不是像前面那样直接去创建一个监听器出来，而是要先创建一个 JavaBean ，然后让这个 JavaBean 实现相应的接口，因为我们这里监听的是 session 中的对象的状态。

下面举例说明。
我们首先建立一个普通的 JavaBean 然后让这个类实现 *HttpSessionBindingListener，HttpSessionActivationListener* 这个两个接口，当然只实现一个接口也可以，但是这时只具备一个监听器的功能。
``` java
public class User implements HttpSessionBindingListener,HttpSessionActivationListener,Serializable {

	private String username;
	private String password;
	
	//绑定
	public void valueBound(HttpSessionBindingEvent httpsessionbindingevent) {
		       //当这个对象被存入到session中时调用
	}

	//解除绑定
	public void valueUnbound(HttpSessionBindingEvent httpsessionbindingevent) {
		       //当这个对象从session中移除时调用
	}

	//钝化
	public void sessionWillPassivate(HttpSessionEvent httpsessionevent) {
		      //当这个对象被钝化时调用
	}
	//活化
	public void sessionDidActivate(HttpSessionEvent httpsessionevent) {
		      //当这个对象被活化时执调用
	}
	
	public String getUsername() {
		return username;
	}

	public void setUsername(String username) {
		this.username = username;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}
}

```

注意：上面的代码中还实现了第三个接口 *Serializable*,这个是在序列化时用到的。


## 4. 在Servlet3.0中使用监听器

在Servlet3.0中使用监听器可以使用注解的方式来注册监听器，从而代替在 *web.xml* 中写入注册信息。
例如:
```java
@WebListener("this is only a demo listener")
public class SimpleListener implements ServletContextListener(...)
``` 
@WebListener 的常用属性只有一个，那就是value，用于表示该监听器的描述信息。

补充关于使用Servlet3.0的前提条件：
- 使用servlet3.0新标准jar包；
- JDK必须在1.6以上版本；
- 编译器的编译级别为6.0；
- 在web.xml文件中，使用3.0规范；
- 使用支持servlet3.0特性的web容器，如tomcat7。


</br>
</br>关于使用监听器实现的在线人数统计的例子，[请点击这里](../../../../2015/12/05/online-number-in-javaweb/)