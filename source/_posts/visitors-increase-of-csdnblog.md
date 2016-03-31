layout: post
title: 利用 HttpClient 刷 CSDN 博客文章浏览量
date: 2016-2-04 14:20:22
tags: 
	- Java
comments: true
copyright: true
---

### **1. HttpClient 的相关解释** ###

Apache HttpComponents 包含了两大部分：

**HttpComponents Core ==>> HttpCore**

实现了一系列的底层传输的功能，这些底层功能，可以用来去建立自己的client和server。

支持两种I/O模式：阻塞型Blocking，基于典型的Java的I/O模型；非阻塞型Non-Blocking，基于Java的NIO，事件驱动型。

在线文档： [HttpCore Tutorial](http://hc.apache.org/httpcomponents-core-ga/tutorial/html/)；[中文版](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/translation/HttpCore-Simplified%20Chinese.pdf)

示例代码：[HttpCore Examples](http://hc.apache.org/httpcomponents-core-ga/examples.html)

<!--more-->

**HttpComponents Client ==>> HttpClient**

兼容HTTP 1.1。基于HttpCore，意味着导入对应的库 HttpClient 库时，也要导入相关的 HttpCore 的库。

同时提供了其他功能：客户端认证功能；HTTP状态管理；HTTP连接管理。

HttpClient是之前常用的那个：Commons HttpClient 3.x的继承者，之前的HttpClient 3.x，现已废弃。

如果还要用之前的HttpClient 3.x，也强烈推荐你换用最新的HttpClient 4.1（或更新版本的）

在线文档：[HttpClient Tutorial](http://hc.apache.org/httpcomponents-client-4.5.x/tutorial/html/index.html)；[中文版](https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/translation/HttpClient-Simplified%20Chinese.pdf)

示例代码：[HttpClient Examples](http://hc.apache.org/httpcomponents-client-4.5.x/examples.html)

HttpClient还有个异步的版本： Asynch HttpClient

(说明：该部分内容转载自[这里](http://www.crifan.com/java_http_related_lib_httpclient_httpcore/))

### **2. 在项目中导入 HttpClient** ###

**下载**

打开站点[http://hc.apache.org/](http://hc.apache.org/)，在网站右侧点击 download，这里展示了最新版的 HttpClient，我们这里选择下载 HttpClient 的二进制文件(Binary)，例如： 4.5.1.zip。

**导入**

下载后得到 `httpcomponents-client-4.5.1-bin.zip` 这个文件，解压得到 `httpcomponents-client-4.5.1` 这个文件夹。将该文件下的 lib 下的 jar 包都导入到 java 项目中即可。导入方法：Eclipse(或myEclipse)中，右击项目文件夹->Build Path->Add External Archive->把上面的那些jar包都加进去，即可。

### **3. 如何来刷浏览量** ###

原理很简单，就是用 HttpClient 来不停的模拟浏览器访问你的博文就行了。代码如下：

```java
public class test {
    public static void main(String[] args) throws ClientProtocolException, IOException {
        String url = "http://blog.csdn.net/cighao/article/details/50629266"; //需要刷的博文的url
        // 创建默认的客户端实例  
        CloseableHttpClient httpclient = HttpClients.createDefault();
        // 创建get请求实例 
        HttpGet httpget = new HttpGet(url);
        // 设置头部信息
        httpget.setHeader("User-Agent", "Mozilla/5.0 (Windows NT 6.1; rv:6.0.2) Gecko/20100101 Firefox/6.0.2");
        // 客户端执行get请求 返回响应实体 
        HttpResponse response = httpclient.execute(httpget);
        // 获取响应消息实体  
        HttpEntity entity = response.getEntity();
        //打印输出网页的内容(可删除)
        if (entity != null) {
            InputStream in = entity.getContent();
            InputStreamReader isr = new InputStreamReader(in);
            int s;
            s = isr.read() ;
            while (s!= -1) {
                System.out.print((char)s);
                s = isr.read() ;
            }
        }
    }
}
```
上面的程序每运行一次，对应博文的浏览量就会加1。
