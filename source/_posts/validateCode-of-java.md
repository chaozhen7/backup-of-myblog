layout: post
title:  基于 Java 实现的图片验证码
date: 2015-08-21 17:19:22
tags: 
	- Java Web
	- Java
comments: true
---
这里主要用到了三个文件，两个servlet类，一个JSP页面。两个servlet分别为<code>drawCode.java</code>和<code>ValidateServlet.java</code>，前者用来产生验证码，后者用来检验验证码是否正确，JSP页面为<code>login.jsp</code>，作为前台的交互。
**drawCode 类**
step1： 定义 BufferImage 对象
step2： 获得 Graphics 对象
step3： 通过 Random 产生随机验证码信息
step4： 使用 Graphics 绘制图片
step5： 记录验证码信息到 session 中
step6： 使用 ImageIO 输出图片

<!--more-->

**ValidateServlet 类**
step1： 获取页面验证码
step2： 获取 session 保存的验证码
step3： 比较验证码
step4： 返回校验结果

**代码如下：**

### login.jsp
```html
<%@ page language="java" pageEncoding="utf-8"%>
<!-- 引入JSTL标签 -->
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %> 
<html>
	<body>
		<script type="text/javascript">		
			function refresh(){				
				a = parseInt(100*Math.random());//随机生成一个整数
				//给url加上随机参数，使图片链接每次都不同，从而解决因为浏览器缓存而不更新的问题。
				// 也可以使用这种方式添加参数
				// var time = new Date().getTime();
				url = "/Prj27/servlet/drawCode?" + a;  
				document.getElementById("imgValidate").src = url;
			}	
		</script>
		<BR>
		<form name="loginForm" action="/Prj27/servlet/ValidateServlet">
			请您输入账号:
			<input type="text" name="account" style="height:25px"/>
			<br/><br/>
			请您输入密码:
			<input type="password" name="password" style="height:25px" />
			<br/><br/>
			请输入验证码:
			<input type="text" name="code" size="10" style="height:25px;vertical-align:middle">
			<img id="imgValidate" src="/Prj27/servlet/drawCode" style="vertical-align:middle">
			<img src="/Prj27/image/refresh.jpg" width="25" onclick="refresh()" style="vertical-align:middle">
			<br/><br/>
			<input type="submit" value="登录">
		</form>
		<!-- 判断验证码是否正确  JSTL-->
		<c:set var="codeErro" value="${param.codeErro}"/>
	 	<c:if test="${codeErro=='yes'}">
			<p><span style="color:red;">验证码错误</span></p>
	     </c:if>
	     <c:if test="${codeErro=='no'}">
			<p><span style="color:green;">验证码正确</span></p>
	     </c:if>
	</body>
</html>
```

### drawCode.java
``` java
package servlet;

import java.awt.*;
import java.awt.image.BufferedImage;
import java.io.IOException;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.util.*;

import javax.imageio.ImageIO;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

public class drawCode extends HttpServlet {

	public drawCode() {
		super();
	}

	public void destroy() {
		super.destroy(); 
	}
	
	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		this.doPost(request, response);
		
	}
	public void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
        
		  //设置页面不缓存    
		response.setHeader("Pragma", "No-cache");    
		response.setHeader("Cache-Control", "no-cache"); 
		response.setDateHeader("Expires", 0);     
		// 在内存中创建图象 
		int width = 60, height = 20;
		BufferedImage image = new BufferedImage(width, height,BufferedImage.TYPE_INT_RGB);
		//获取画笔
		Graphics g = image.getGraphics();
		//设定背景色 
		g.setColor(new Color(200, 200, 200));
		g.fillRect(0, 0, width, height);
		//取随机产生的验证码(4位数字) 
		Random rnd = new Random();
		int randNum = rnd.nextInt(8999) + 1000;
		String randStr = String.valueOf(randNum);
		//将验证码存入session
		request.getSession().setAttribute("randStr", randStr);
		//将验证码显示到图象中 
		g.setColor(Color.black);
		g.setFont(new Font("", Font.PLAIN, 20));
		g.drawString(randStr, 10, 17);
		// 随机产生100个干扰点，使图象中的验证码不易被其它程序探测到 
		for (int i = 0; i < 100; i++){
			int x = rnd.nextInt(width);
			int y = rnd.nextInt(height);
			g.drawOval(x, y, 1, 1);
		}  
      
		// 输出图象到页面 
		ImageIO.write(image, "JPEG", response.getOutputStream());
	}

	public void init() throws ServletException {
		// Put your code here
	}

}
```

### ValidateServlet.java
``` java
package servlet;
import java.io.IOException;
import java.io.PrintWriter;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
public class ValidateServlet extends HttpServlet {
	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		
		request.setCharacterEncoding("utf-8");		
		//判断验证码是否正确
		String code = request.getParameter("code");
		String randStr = request.getSession().getAttribute("randStr").toString();
		System.out.println(randStr);
		if(!code.equals(randStr)){
			response.sendRedirect("/Prj27/login.jsp?codeErro=yes");//验证码错误
			return ;
		}	
		else{
			response.sendRedirect("/Prj27/login.jsp?codeErro=no");//验证码正确
			return ;
		}
	}
	public void doPost(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {
		this.doGet(request, response);		
	}
}
```