layout: post
title: Struts2 拦截器浅析
date: 2015-12-16 17:20:22
tags: 
	- Java Web
	- Java
	- Struts2
comments: true
toc: true
copyright: true
---
## 1. Struts2 框架执行流程 ##
下面这个是 Struts2 框架的一个流程图
![Struts2执行流程](/img/articles/struts2.png)

<!--more-->

首先客户端会通过 HttpServletRequest 向Servlert 容器（也就是 tomcat）提交一个请求，这个请求会通过一系列的过滤器(包括Struts2的核心过滤器 StrutsPrepareAndExecuteFilter ), 被核心过滤器过滤到后，核心过滤器会访问到 ActionMapper 来判断是否需要访问某个 Action。   

如果需要访问某个 Action, StrutsPrepareAndExecuteFilter 核心控制器会将控制权委派给 Action 代理，即 ActionProxy。这时候 ActionProxy 会通过 ConfigurationManager 来加载 struts.xml。   

如果在 struts.xml 找到了需要调用的 Action，ActionProxy 会创建 ActionInvocation 的一个实例。 
  
ActionInvocation 里面包括了需要创建的 Action 实例，同时还包括了一些拦截器。在调用 Action 之前会依次调用所配置的拦截器，执行完拦截器后会执行 Action，执行完 Action 会返回一个字符串的结果，然后可以根据这个字符串去调用相应的视图，但是返回这个视图以后并没有立刻去响应用户的请求，而是会把之前执行过的拦截器反过来执行一遍，最后才会通过 HttpServletResponse 去响应用户的请求。

这里的拦截器的执行其实很像前面说到的[Java Web 中的过滤器](../../../../2015/12/03/filter-of-javaweb/)很像。

## 2. Struts2 的内置拦截器 ##

我们在 Struts2 的配置文件中经常会使用到下面的配置:
```html
<package name="default" namespace="/" extends="struts-default">
<!--  
	...
-->
<package>
```    
这里的 ```extents="struts-default"``` 就表示这个 package 继承了 ```struts-default``` 这个 package。 我们可以在 ```struts2-core-X.X.X.jar``` (x.x.x表示版本号) 下找到一个  ```struts-default``` 这个配置文件，我们可以发现这个配置文件下面注册了很多拦截器，并还引用了一个拦截器栈：
```html
<default-interceptor-ref name="defaultStack" />
```    
当我们在配置文件中去继承这个默认的 package 的时候也就默认使用了这些拦截器组成的拦截器栈了。
Struts2 中一些主要的拦截器如下：
params 拦截器：负责将请求参数设置为 Action 属性。
staticParams 拦截器： 将配置文件中 action 元素的子元素 param 参数设置为 Action 属性。
servletConfig 拦截器： 将源于 Servlet API 的各种对象注入到 Action, 必须实现对应接口。
fileUpload 拦截器： 对文件上传提供支持，将文件和元数据设置到对应的 Action 属性。
exception 拦截器： 捕获异常，并且将异常映射到用户自定义的错误页面。
validation 拦截器： 调用验证框架进行数据验证。

## 3. 自定义拦截器 ##
在 Struts2 中自定义拦截器的两种方式：
**实现 Interceptor 接口**
- void init()： 初始化拦截器所需资源
- void destroy()： 释放在 init() 中分配的资源
- String intercep(ActionInvocation ai) throws Exception
	- 实现拦截器功能
	- 利用 ActionInvocation 参数获取 Action 状态
	- 返回 result 字符串作为逻辑视图

**继承 AbstractInterceptor 类**
- 提供了 init() 和 destroy() 方法的实现
- 只需要实现 intercept() 方法即可


## 4. 自定义拦截器示例 ##
这里自定义一个拦截器来计算 Action 的执行时间。
step1： 首先新建一个 Action 类
```java
package com.cighao.action;
import com.opensymphony.xwork2.ActionSupport;
public class HelloStruts extends ActionSupport {

	@Override
	public String execute() throws Exception{
		for(int i=0;i<100000;i++)	
			;	
		return SUCCESS;
	}
}
```    

step2： 新建拦截器的类
```java
package com.cighao.interceptor;
import com.opensymphony.xwork2.ActionInvocation;
public class TimerInterceptor extends AbstractInterceptor {

	@Override
	public String intercept(ActionInvocation invocation) throws Exception {
		// 1. action 执行之前
		long start = System.currentTimeMillis();
		// 2. 执行下一个拦截器,如果已经是最后一个拦截器，则执行目标 action
		String result = invocation.invoke(); //result 为 action 返回的视图
		// 3. action 执行之后
		long end = System.currentTimeMillis();
		System.out.println("action执行时间为："+(end-start)+"ms");
		return result;
	}
}
```   

step3： 注册并配置拦截器和 Action
```html
<package name="default" namespace="/" extends="struts-default">
	<!-- 注册拦截器 -->
	<interceptors>
		<interceptor name="mytimer" class="com.cighao.interceptor.TimerInterceptor"></interceptor>
	</interceptors>
            
	<action name="hello"  class="com.cighao.action.HelloStruts" >
		<result name="success">/index.jsp</result>
		<!-- 引用拦截器 -->
		<interceptor-ref name="mytimer"></interceptor-ref>
	</action>               
</package>
```    

访问该 hello.action后会在控制台看到如下信息：
> action执行时间为：199ms

注意：当为包中的某个 action 显示指定某个拦截器，则默认拦截器不会起作用。需要显示调用。