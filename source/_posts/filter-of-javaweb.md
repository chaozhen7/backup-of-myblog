layout: post
title: Java Web 之过滤器
date: 2015-12-3 21:50:22
tags: 
	- Java Web
	- Java
comments: true  
toc: true
copyright: true
---
## 1. 什么是过滤器
一个服务器端的组件，可以截取用户端的请求用与响应信息，并对这些信息过滤。
![过滤器原理](/img/articles/filter1.png) 
<!--more-->

## 2. 过滤器的生命周期
![过滤器原理](/img/articles/filter2.png) 

在web容器开启时，会实例化过滤器；在web容器关闭时，销毁过滤器。相关函数的解释如下：

<code>init()</code> ：这是过滤器的初始化方法，Web容器创建过滤器实例后将调用这个方法，这个方法中可以读取web.xml文件中过滤器的参数。
<code>doFilter()</code> 这个方法完成实际的过滤操作，这个地方是过滤器的核心方法，当用户请求访问与过滤器关联的url时，Web容器将先调用过滤器的doFilter方法，FilterChain参数可以调用chain.doFilter方法，将请求传给下一个过滤器(或目标资源)，或利用转发，重定向将请求转发到其他资源。 
<code>destroy()</code> Web容器在销毁过滤器实例前调用该方法，在这个方法中可以释放过滤器占用的资源。(大多情况用不到)


## 3. 第一个过滤器程序

step1： 首先在myEclipse下新建一个项目

step2： 在src下新建一个类Class，在新建类的时候在 Interfaces 选项中选择新建的类实现<code>javax.servlet.Filter</code>接口。这个时候发现新建的类中已经默认存在了<code>destroy()</code>, <code>doFilter()</code>,<code>init()</code>这三个方法。

step3： 新建好的过滤器类需要在<code>web.xml</code>下配置相关的信息，格式如下

``` html
<filter>
	<filter-name> filter的名字 </filter-name>
	<filter-class> filter的类名 </filter-class>
	<init-param>
		<description>描述信息，可省略 </description>
		<param-name> 参数名称  </param-name>
		<param-value> 参数的值 </param-value>
	</init-param>
	<init-param>
		<param-name> 参数名称 </param-name>
		<param-value> 参数的值  </param-value>
	</init-param>
</filter>

<filter-mapping>
	<filter-name> filter的名字   </filter-name>
	<url-pattern> URL   </url-pattern>
	<dispatcher>       </dispatcher>
</filter-mapping>
```

<code>&lt;filter-class&gt;</code>： 过滤器类完整的名字，包括包名。
<code>&lt;init-param&gt;</code>： 初始化参数，可是零对或者多对。
<code>&lt;url-pattern&gt;</code>： 当用户请求的url和指定url匹配时将触发过滤器工作  。 
<code>&lt;dispatcher&gt;</code>： 可以是零对或者多对，值为：REQUEST|INCLUDE|FORWARD|ERROR，未设定时，默认值为REQUEST。


## 4. 过滤器链
当一个页面注册了多个过滤器时，在请求该页面时会依次通过每一个过滤器，通过的顺序与在在web.xml中注册的顺序一致。如下图所示，
![过滤器原理](/img/articles/filter3.png) 
我们假设index.jsp页面有两个过滤器，分别为filter1和filter2，那么当我请求index.jsp页面时，会按如下方式执行：filter1过滤前的代码 --> filter1放行代码 --> filter2过滤前的代码 --> filter2放行代码 --> filter2放行后的代码 --> filter1放行后的代码-->返回信息。


## 5. 过滤器的分类
在Servlet2.5中将过滤器分为4类，在Servlet3.0中新增了一个<code>ASYNC</code>类。
Servlet2.5中的四类过滤器：
- REQUEST： 用户直接访问页面时，web容器将会调用过滤器。
- ERROR： 目标资源是通过声明式异常处理机制调用时，过滤器将被调用。
- FORWARD： 目标资源是通过 RequestDispatchaer 的 forward 方法调用时，过滤器将被调用。
- INCLUDE： 目标资源是通过 RequestDispatchaer 的 include 方法调用时，过滤器将被调用。

Servlet3.0新增的一类过滤器：
- ASYNC： 目标资源是异步处理时，过滤器将被调用。



## 6. Servlet3.0中的 @WebFilter 注解
按照上面的方法，我们在新建一个过滤器类的时候需要在web.xml中写入相应的配置信息，在但是在servlet3.0中我们可以直接在过滤器类中通过注解来代替在web.xml中写入配置信息。下面的代码就是一个示例。
``` java
// 上面的import部分的代码已省略

// 下面通过注解来配置相关信息
@WebFilter(filterName="MyFilter",asyncSupported=true,value={"*"},dispatcherTypes={DispatcherType.REQUEST,DispatcherType.ASYNC})
public class MyFilter implements Filter {

	@Override
	public void destroy() {

	}

	@Override
	public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2) throws IOException, ServletException {
		System.out.println("start.....AsynFilter");  // 过滤前的代码
		arg2.doFilter(arg0, arg1);              // 放行代码
		System.out.println("end.....AsynFilter"); //放行后的代码
	}
	@Override
	public void init(FilterConfig arg0) throws ServletException {

	}
}

```
关于 *@WebFilter* 的常用属性如下：

| 属性名|类型 |描述|
|----|----|----|
|filterName|String|指定过滤器的name属性，等价于 &lt;filter-name&gt;|
|value|String[]|该属性等价于urlPatterns属性，但两者不可同时使用|
|urlPatterns|String[]|指定一组过滤器的URL匹配模式，等价于 &lt;url-pattern&gt;|
|servletNames|String[]|指定过滤器将应用于哪些Servlet。取值是@WebServlet中的name属性的取值，或者是web.xml中 &lt;servlet-name&gt;的取值|
|dispatcherTypes|DispatcherTypes|指定过滤器的转发模式，具体取值包括：ASYNC，ERROR，FORWARD，INCLUDE，REQUEST|
|initParams|WebInitParam[]|指定一组过滤器初始化参数，等价于 &lt;init-param&gt;|
|asyncSupported|boolean|声明过滤器是否支持异步操作模式，等价于 &lt;async-supported&gt;|
|description|String|该过滤器的描述信息，等价于 &lt;description &gt;|
|displayName|String|该过滤器的显示名，通常配合工具使用，等价于 &lt;display-name&gt;|
   
</br>

关于 Java Web 过滤器的相关示例可以[点击这里](../../../../2015/12/04/filter-demos-of-javaweb/)