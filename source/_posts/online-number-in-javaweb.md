layout: post
title: Java Web 中基于监听器实现的在线人数统计
date: 2015-12-5 20:40:22
tags: 
	- Java Web
comments: true
---
关于监听器的基本介绍前面已经介绍过了，如果不清楚请[点击这里](../../../../2015/12/05/listener-of-javaweb/)

## 1. 基本功能的实现
在线人数的统计可以通过统计当前 session 对象的个数来得到，因为每当一个用户登录的时候都会创建一个 session。前面我们已经介绍过了，继承了 *HttpSessionListener* 这个接口的监听器可以监听到 session 对象的创建和销毁的。当监听到有 session 对象创建时在线人数加一，监听到有 session 对象销毁时在线人数减一。代码如下：
```java
@WebListener
public class MyHttpSessionListener implements HttpSessionListener {
	
	private int userNumber = 0;    //在线用户总人数
	
	@Override
	public void sessionCreated(HttpSessionEvent arg0) {
		userNumber++;
		arg0.getSession().getServletContext().setAttribute("userNumber", userNumber);
	}

	@Override
	public void sessionDestroyed(HttpSessionEvent arg0) {
		userNumber--;
		arg0.getSession().getServletContext().setAttribute("userNumber", userNumber);
	}

}

```
<!--more-->

## 2. 功能拓展

上面的代码已经基本实现了在线人数的统计了，但是如果我们要获取更多的关于在线用户的信息(如ip)，前面的代码就没法实现了，下面做进一步的拓展：
关于 ip 地址的信息是存在 ServletRequest 对象中的，那么我们可以通过实现 *ServletRequestListener* 接口来创建一个监听器用来监听每一个request请求，如果这个用户是个新用户，则实例化一个用户对象保存相关的信息，然后将这个用户对象加入到在线用户列表中。代码如下：
``` java
//import部分已省略
@WebListener
public class MyServletRequestListener implements ServletRequestListener {

	private ArrayList<User> userList;   //在线用户的列表
	
	@Override
	public void requestDestroyed(ServletRequestEvent arg0) {

	}

	@Override
	public void requestInitialized(ServletRequestEvent arg0) {
		//获取在线用户列表
		userList  = (ArrayList<User>)arg0.getServletContext().getAttribute("userList");
		
		if(userList==null)   // 如果当前在线用户列表为空，则新建一个
			userList = new ArrayList<User>();
		
		HttpServletRequest request = (HttpServletRequest) arg0.getServletRequest();
		String sessionIdString = request.getSession().getId();
		
		//判断这个 request 请求是否是新的用户发起的
		if(SessionUtil.getUserBySessionId(userList,sessionIdString)==null){
			User user = new User();
			user.setSessionIdString(sessionIdString);
			user.setFirstTimeString(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date()));
			user.setIpString(request.getRemoteAddr());
			userList.add(user);
		}
		arg0.getServletContext().setAttribute("userList", userList); //更新在线用户列表
	}
}
```

上面用到了，user 对象和 getUserBySessionId() 方法。解释如下：
关于 user 对象，每有一个用户登录时，会实例化一个 user 对象保存用户的信息，并加入的在线用户列表中。user()对象的代码如下：
```java
//import部分已省略
public class User {
	private String sessionIdString; // 该用户的sessionId
	private String ipString;    // 该用户的IP
	private String firstTimeString;  // 首次登陆时间
	public String getSessionIdString() {
		return sessionIdString;
	}
	public void setSessionIdString(String sessionIdString) {
		this.sessionIdString = sessionIdString;
	}
	public String getIpString() {
		return ipString;
	}
	public void setIpString(String ipString) {
		this.ipString = ipString;
	}
	public String getFirstTimeString() {
		return firstTimeString;
	}
	public void setFirstTimeString(String firstTimeString) {
		this.firstTimeString = firstTimeString;
	}
}
```
关于 getUserBySessionId() 这个方法有两个传入参数是一个是在线用户列表userList，一个是String 类型的sessionId，该方法用以查找在当前在线用户列表中是否存在该 sessionId 为指定值的用户，有就返回该用户对象，否则返回null。这里此方法主要用来判断该 request 请求是新用户的请求还是已经在线的用户的请求。      
关于 getUserBySessionId() 方法我们把它放到一个新的类中去实现，代码如下:
```java
//import部分已省略
public class SessionUtil {

	public static Object getUserBySessionId(ArrayList<User> userList, String sessionIdString) {
		for (int i = 0; i < userList.size(); i++) {
			User user = userList.get(i);
			if (user.getSessionIdString().equals(sessionIdString)) {
				return user;
			}
		}
		return null;
	}
}
```
</br>
最后，关于 HttpSessionListener 监听器中的部分还需要做相应的修改，修改后代码如下:
```java
//import部分已省略
@WebListener
public class MyHttpSessionListener implements HttpSessionListener {
	
	private int userNumber = 0;  //在线用户总人数
	
	@Override
	public void sessionCreated(HttpSessionEvent arg0) {
		userNumber++;
		arg0.getSession().getServletContext().setAttribute("userNumber", userNumber);
	}

	@Override
	public void sessionDestroyed(HttpSessionEvent arg0) {
		userNumber--;
		arg0.getSession().getServletContext().setAttribute("userNumber", userNumber);
		
		ArrayList<User> userList = null;  //在线用户列表
		userList = (ArrayList<User>)arg0.getSession().getServletContext().getAttribute("userList");
		
		// 当有 session 被销毁时从在线用户列表中删除该 session 所对应的用户对象
		if(SessionUtil.getUserBySessionId(userList, arg0.getSession().getId())!=null){
			userList.remove(SessionUtil.getUserBySessionId(userList, arg0.getSession().getId()));
		}
		
	}

}
```

到这里功能就已经全部实现了。不仅记录了在线人数，还记录了在线用户的相关信息。