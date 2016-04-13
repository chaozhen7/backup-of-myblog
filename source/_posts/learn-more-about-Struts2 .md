layout: post
title: 深入了解 Struts2
date: 2015-12-15 12:40:22
tags: 
	- Java
	- Java Web
	- Struts2
comments: true
toc: true
copyright: true
---

## 1. 访问 Servlet ##
Struts2 提供了三种方式 让 action 访问 Servlet：
(1) ActionContext
(2) 实现 \*\*\*Aware接口
(3) ServletActionContext

## 2. 动态方法调用 ##
动态方法调用是为了解决一个 Action 对应多个请求的处理，以免 Action 太多。
这里用前面的  [Struts2 的第一个小示例](../../../../2015/12/14/first-project-of-struts2/) 继续讨论。

<!--more-->

假设我们要在一个 Action 中去实现多个功能，那要怎么去写，怎么去实现呢。在前面的 Action 类中我们添加一个方法，修改后如下：
```java
public class HelloStruts extends ActionSupport {
	public String add(){
		return "success";
	}
	@Override
	public String execute() throws Exception{
		//System.out.println("start action");
		return "success";
	}
}
```
接下来怎么去配置相关的信息，这里有三种方法：
**指定 method 属性**
这种方式只要在 package 包下新增一个 action 即可。如下:
```
<action name="hello" class="com.cighao.action.HelloStruts" >
	<result name="success">/index.jsp</result>
</action>
<action name="add" method="add"  class="com.cighao.action.HelloStruts" >
	<result name="success">/add.jsp</result>
</action>
```    
**感叹号方式**
使用该方式首先要做如下配置：
```html
<constant name="struts.enable.DynamicMethodInvocation" value="true"></constant>
```    
这时我们再把 add() 函数的返回值从 "success" 改为 **```"add"```**。在把配置信息修改如下:
```html
<action name="hello" class="com.cighao.action.HelloStruts" >
	<result name="success">/index.jsp</result>
	<result name="add">/add.jsp</result>
</action>
```    
这时如果要访问 add() 方法 可以用该路径：localhost:8080/hellostruts/hello!add.action。

**通配符方式**
首先把上面的 constant 配置去掉，add 的返回值依然是 "add"。
在配置文件中做如下修改：
```html
<action name="hello_*" method="{1}" class="com.cighao.action.HelloStruts" >
	<result name="success">/index.jsp</result>
	<result name="add">/{1}.jsp</result>
</action>
```    
如果要访问 add() 方法可以使用该路径： localhost:8080/hellostruts/hello\_add.action 。
其实这里的 \* 的主要作用就是起一个通配符的作用。

## 3. 默认 Action ##
当用户请求一个不存在的 action 时，程序就会报错，我们这个时候可以设置一个默认的 Action，当用户访问的 Action 不存在时我们就去执行这个默认的 Action。
首先我们要写一个 Action 的类。然后像普通的 action 的配置一样在 struts.xml 中去配置，最后用 ```<default-action-ref>``` 标签将这个 action 设置为默认的 action。配置方法如下：
```html
<default-action-ref name="index"> </default-action-ref>
<action name="index" class="com.cighao.action.DefaultAction" >
	<result name="success">/index.jsp</result>             
</action>
```    

## 4. 修改 Struts2 后缀  ##
前面我们访问 Action 时都要在后面添加 .action 后缀，比如： ../../hello.action，但是如果我们想修改这个后缀怎么办，如果我们想用其他的后缀去访问这个 Action, 比如 ../../hello.do 或者 ../../hello.html，这要怎么办呢？
这时只要添加下面的配置就行了：
```html
<constant name="struts.action.extension" value="html"></constant>
```    
如果不想用后缀可以令 ```value=""```, 如果要配置多个后缀可以在 value 中写入多个值用逗号隔开，如 ```value="html,do"```。
切记：修改完后缀后记得看 ```web.xml``` 过滤器的过滤准则是否包含该后缀。

## 5. 接受参数 ##
这里介绍三种接受参数的方法
**使用 Action 的属性接收参数**
前台的输入界面:
```html
<form action="login.action" method="post">
	<input type="text" name="username">
	<input type="submit" value="提交">
</form>
```    
Action类
```java
public class Login extends ActionSupport {
	String username;  // 必须跟接受参数的input的name一致
	public String login(){
		//System.out.println(username);
		return "success";
	}
	public String getUsername() {  // 每个成员变量都应该有set和get方法
		return username;
	}
	public void setUsername(String username) {
		this.username = username;
	}
	
}
```   
form 表单提交到 Action 后，action 会根据表单的 name 值自动执行 action 中的 set 方法给同名参数赋值。可以直接在 login() 使用 username 变量来访问提交的数据。
注意：成员变量的名称必须跟接受参数的 input 的 name 一致。

**使用 DomainModel 接收参数**
这种方法可以利用面向对象的思想，将需要接受的参数封装到一个对象中。例如我们可以先创建一个 User 对象。
```java
public class User {
	private String username;
	private String password;
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
然后我们可以在 Action 中可以来引用这个对象。
```java
public class Login extends ActionSupport {
	private User user;
	public String login(){
		//System.out.println(user.getUsername());
		return "success";
	}
	public User getUser() {
		return user;
	}
	public void setUser(User user) {
		this.user = user;
	}
}
```    
当然这个时候表单提交界面需要略做修改，需要把 input 的 name 属性从 username 修改为 user.username 这样程序就知道该把这个表单的值赋给哪个对象中的 username 了，因为可能不只一个对象中有 username 字段。

**使用 ModelDriven 接收参数**
这种方式需要使 action 去实现 ModelDriven 这个接口
```java
public class Login extends ActionSupport implements ModelDriven<User>{
	private User user = new User();  // 这种方式下必须要先实例化
	public String login(){
		//System.out.println(user.getUsername());
		return "success";
	}
	@Override
	public User getModel() {
		// TODO Auto-generated method stub
		return user;
	}
}
```   
这个时候成员变量中的 user 就不需要 get 和 set 方法了，但是必须要在声明的时候实例化。
还有，提交页面中 input 的 name 值这个时候就不需要前面的 user. 了直接写成 username 就行了。

**如何接收数组呢**
假如我们在 User 对象里面添加一个字段。
```java   
private List<String> books;
```    
并添加了相应的 get 和 set 方法，那么如何给这它赋值呢。
只要在提交页面修改相应 input 的 name 属性值即可，例如： 
```html   
<input type="text" name="books[0]">
<input type="text" name="books[1]">
```    
在 action 中使用这个值只需要这要操作就行，以打印输出为例：
```java   
System.out.println(user.getBooks().get(0));
System.out.println(user.getBooks().get(1));
```   
如果这里的 book 不是一个字符串而是一个对象怎么办呢，假设 books 为：
```java   
private List<Book> books; //
```    
假设 Book 有两个字段分别为 name 和 price 那么可以下面的方式传递参数,只要修改提交页面 input 的 name 属性即可。例如：
```html
<input type="text" name="books[0].name">
<input type="text" name="books[0].price">
```    

## 6. 处理结果类型 ##

Struts 中 Action 的返回值是一个字符串，内置的返回类型有：
**SUCCESS**： Action 正确执行完成，返回相应的视图，success 是 name 属性的默认值。
**NONE**： 表示 Action 正确执行完成，但不返回任何视图。
**ERROR**： 表示 Action 执行失败，返回到错误处理视图。
**LOGIN**： Action 因为用户没有登录的原因没有正确执行，将返回该登录视图，要求用户进行登录。
**INPUT**： Action 的执行，需要从前端界面获取参数，INPUT 就是代表这个参数输入的界面，一般在应用中，会对这些参数进行验证，如果验证没有通过，将自动返回到该视图。

处理结果是通过在 struts.xml 使用 &lt;result &gt; 标签来配置的。根据位置的不同分为两种结果：
局部结果： 将 result 作为 &lt;action &gt; 元素的子元素配置
全局结果： 将 result 作为 &lt;global-result &gt; 元素的子元素配置