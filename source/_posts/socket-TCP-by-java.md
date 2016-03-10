layout: post
title: Java Socket 实现 TCP 编程
date: 2016-2-03 16:20:22
tags: 
	- Java
comments: true
---

### **1. Socket 实现 TCP 的通信模型** ###

TCP 的通信分为服务器端和客户端，整个通信过程可以分为三个步骤：1.建立连接；2.开始通信；3.结束通信。

首先服务器端建立监听 socket，等待并接受连接请求，然后客户端创建连接 socket 向服务器发送请求，服务器接受请求后创建连接 socket，接着服务器端和客户端的 socket 通过 InputStream 和 Outputstream 进行通信。最后通信结束时，关闭 socket 及相关资源。

<!--more-->

Socket 通信实现步骤：
1. 创建 ServerSocket 和 Socket
2. 打开连接到 Socket 的输入/输出流
3. 按照协议对 Socket 进行读/写操作
4. 关闭输入输出流，关闭 Socket

### **2. 例子** ###

关于 ServerSocket 和 Socket 类的相关信息可以参见 API，这里我们用一个登录界面的例子来介绍一下 Socket 通信。

服务器端：
1. 创建 ServerSocket 对象，绑定监听端口
2. 通过 accept() 方法监听客户端请求
3. 连接建立后，通过输入流读取客户端发送的请求信息
4. 通过输出流向客户端发送响应信息
5. 关闭相应资源

代码：
```java
public class Server {
    public static void main(String[] args) {
        try {
            //1.创建一个服务器端Socket，即ServerSocket，指定绑定的端口，并监听此端口
            ServerSocket serverSocket=new ServerSocket(8888);
            //2.调用accept()方法开始监听，等待客户端的连接
            Socket socket=serverSocket.accept();
            //3.获取输入流，并读取客户端信息
            InputStream is = socket.getInputStream();//字节输入流
            InputStreamReader isr = new InputStreamReader(is); //将字节输入流装换位字符输入流
            BufferedReader br = new BufferedReader(isr);//为输入流添加缓冲
            String info=null;
            while((info=br.readLine())!=null){//循环读取客户端的信息
                System.out.println(info);
            }
            socket.shutdownInput();//关闭输入流
            //4.获取输出流，响应客户端的请求
            OutputStream os = socket.getOutputStream();
            PrintWriter pw = new PrintWriter(os);//包装为打印流
            pw.write("欢迎您！");
            pw.flush();//调用flush()方法将缓冲输出
            socket.shutdownOutput();//关闭输出流
            //5.关闭资源
            is.close();
            pw.close();
            os.close();
            br.close();
            isr.close();
            socket.close();
            serverSocket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

客户端：
1. 创建 Socket 对象，指明需要连接的服务器的地址和端口号
2. 连接建立后，通过输出流向服务器发送请求
3. 通过输入流获取服务器响应信息
4. 关闭资源

代码：
```java
public class Client {
    public static void main(String[] args) {
        try {
            //1.创建客户端Socket，指定服务器地址和端口
            Socket socket=new Socket("localhost", 8888);
            //2.获取输出流，向服务器端发送信息
            OutputStream os=socket.getOutputStream();//字节输出流
            PrintWriter pw=new PrintWriter(os);//将输出流包装为打印流
            pw.write("用户名：admin;密码：admin");
            pw.flush();
            socket.shutdownOutput();//关闭输出流
            //3.获取输入流，并读取服务器端的响应信息
            InputStream is=socket.getInputStream();
            BufferedReader br=new BufferedReader(new InputStreamReader(is));
            String info=null;
            while((info=br.readLine())!=null){
                System.out.println(info);
            }
            socket.shutdownInput();//关闭输入流
            //4.关闭资源
            br.close();
            is.close();
            pw.close();
            os.close();
            socket.close();
        } catch (UnknownHostException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### **3. 多线程的实现** ###

上面的例子虽然已经实现了服务器和客户端的通信，但是却只能实现一个客户端到一个服务器的通信，不能实现多个客户端到一个服务器的通信，下面我们可以用多线程的方式来实现。客户端的代码不用变，只要改变服务器的实现方式就行了。

基本步骤：
1. 服务器端创建 ServerSocket，循环调用 accept() 等待客户端连接
2. 客户端创建一个 socket 并请求和服务器端连接
3. 服务器端接受客户端请求，创建 socket 与该客户建立专线连接
4. 建立连接的两个 socket 在一个单独的线程上对话

代码：
服务器端的线程类
```java
public class ServerThread extends Thread {
    // 和本线程相关的Socket
    Socket socket = null;
    public ServerThread(Socket socket) {
        this.socket = socket;
    }
    //线程执行的操作，响应客户端的请求
    public void run(){
        InputStream is=null;
        InputStreamReader isr=null;
        BufferedReader br=null;
        OutputStream os=null;
        PrintWriter pw=null;
        try {
            //获取输入流，并读取客户端信息
            is = socket.getInputStream();
            isr = new InputStreamReader(is);
            br = new BufferedReader(isr);
            String info=null;
            while((info=br.readLine())!=null){//循环读取客户端的信息
                System.out.println(info);
            }
            socket.shutdownInput();//关闭输入流
            //获取输出流，响应客户端的请求
            os = socket.getOutputStream();
            pw = new PrintWriter(os);
            pw.write("欢迎您！");
            pw.flush();//调用flush()方法将缓冲输出
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }finally{
            //关闭资源
            try {
                if(pw!=null)
                    pw.close();
                if(os!=null)
                    os.close();
                if(br!=null)
                    br.close();
                if(isr!=null)
                    isr.close();
                if(is!=null)
                    is.close();
                if(socket!=null)
                    socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

服务器 Socket:
```java
public class Server {
    public static void main(String[] args) {
        try {
            //1.创建一个服务器端Socket，即ServerSocket，指定绑定的端口，并监听此端口
            ServerSocket serverSocket=new ServerSocket(8888);
            Socket socket=null;
            //循环监听等待客户端的连接
            while(true){
                //调用accept()方法开始监听，等待客户端的连接
                socket=serverSocket.accept();
                //创建一个新的线程
                ServerThread serverThread=new ServerThread(socket);
                //启动线程
                serverThread.start();          
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```