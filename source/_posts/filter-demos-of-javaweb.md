layout: post
title: Java Web 中过滤器的几个小例子
date: 2015-12-4 15:35:22
tags: 
	- Java Web
	- Java
comments: true
---

前面已经介绍了 Java Web 中过滤器的一些基本知识，如果没看过请[点击这里](../../../../2015/12/03/filter-of-javaweb/)

## 1. 过滤器的应用
- 对用户请求进行统一认证
- 编码转换
- 对用户发送的数据进行过滤替换
- 转换图像格式
- 对响应的内容进行压缩

<!--more-->

## 2. 登陆验证中的过滤器
在web应用程序中，某些页面只能对登录的用户可见，对于这个问题我们采取的措施通常是，在用户登录时创建一个session来保存用户信息，然后通过session来判断是否有用户登录。在这里我们可以创建一个过滤器对所有有权限限制的页面进行过滤，在过滤方法中把session中是否有用户登录信息作为过滤条件。
对于部分不需要进行权限验证过滤的页面，我们可以将他们的路径信息作为参数，在过滤器初始化的时候加载到过滤器中，在进行过滤操作时，读取参数，对于不需要过滤的页面依次放行。

过滤器的代码如下：
``` java
// import 部分的代码已省略
public class LoginFilter implements Filter {

	private FilterConfig config; //过滤器的参数
	
	@Override
	public void destroy() {
		
	}

	@Override
	public void doFilter(ServletRequest arg0, ServletResponse arg1,
			FilterChain arg2) throws IOException, ServletException {
		HttpServletRequest request = (HttpServletRequest) arg0;
		HttpServletResponse response = (HttpServletResponse) arg1;
		HttpSession session = request.getSession();
		
		String noLoginPaths = config.getInitParameter("noLoginPaths"); //获取不需要过滤的路径
		if(noLoginPaths!=null){
			String[] strArray = noLoginPaths.split(";");  
			for (int i = 0; i < strArray.length; i++) {
				if(strArray[i]==null || "".equals(strArray[i]))
					continue;
				if(request.getRequestURI().indexOf(strArray[i])!=-1 ){
					arg2.doFilter(arg0, arg1);  //对于不需要过滤的路径依次放行
					return;
				}
			}
		}
		//下面是过滤操作
		if(session.getAttribute("username")!=null){
			arg2.doFilter(arg0, arg1);   //如果用户已经登录了则放行
		}else{
			response.sendRedirect("login.jsp"); //用户没有登录则返回登录页面
		}
	}

	@Override
	public void init(FilterConfig arg0) throws ServletException {
		config = arg0;   //加载参数
	}
}

```

web.xml中的配置信息如下：
``` html
<filter>
	<filter-name>LoginFilter</filter-name>
	<filter-class>filter.LoginFilter</filter-class>
	<init-param>
		<param-name>noLoginPaths</param-name>   
		<param-value> </param-value>    <!-- 不需要过滤的页面的路径的关键词写在这里，分号隔开多个关键词 -->
	</init-param>
</filter>
<filter-mapping>
	<filter-name>LoginFilter</filter-name>
	<url-pattern>/admin/*</url-pattern>
</filter-mapping>

```

## 3. 编码转换
在web开发过程中，如果在每个页面上都写一条语句进行编码转换会很麻烦，这里可以用过滤器来进行操作。代码如下：
``` java
public class Charset implements Filter {
	
	private FilterConfig config;
	
	@Override
	public void destroy() {
		// TODO Auto-generated method stub

	}

	@Override
	public void doFilter(ServletRequest arg0, ServletResponse arg1,
			FilterChain arg2) throws IOException, ServletException {
		
		HttpServletRequest request = (HttpServletRequest) arg0;
		String charset = config.getInitParameter("charset"); 
		if(charset==null){
			charset = "UTF-8";  //若没有指定字符集，默认utf-8
		}
		request.setCharacterEncoding(charset);
		arg2.doFilter(arg0, arg1);
	}

	@Override
	public void init(FilterConfig arg0) throws ServletException {
		config = arg0;
	}
}
		
```

``` html
<filter>
	<filter-name>CharsetFilter</filter-name>
	<filter-class>filter.Charset</filter-class>
	<init-param>
		<param-name>charset</param-name>
		<param-value>UTF-8</param-value>
	</init-param>
</filter>
<filter-mapping>
	<filter-name>CharsetFilter</filter-name>
	<url-pattern>*</url-pattern>
</filter-mapping>

```