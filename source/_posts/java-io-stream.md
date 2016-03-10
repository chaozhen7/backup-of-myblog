layout: post
title: Java 输入/输出流
date: 2016-2-03 11:20:22
tags: 
	- Java
comments: true
toc: true
---

### **1. 编码问题** ###

在介绍输入输出之前我们先介绍下关于编码的一些基本知识点，当一个文件中既有中文字符又有英文字符时，他们在不同的编码方式下会占据不同的内存：
1. ANSI 中文占据 2 个字节的内存空间，英文占据 1 个字节的内存空间。
2. GBK 中文占据 2 个字节的内存空间，英文占据 1 个字节的内存空间。
3. UTF-8 中文占据 3 个字节的内存空间，英文占据 1 个字节的内存空间。
4. UTF-16Be 中文占据 2 个字节的内存空间，英文占据 2 字节的内存空间。

<!--more-->

JAVA 内置使用的是双字节编码，也就是 UTF-16BE。

### **2. File 类** ###

java.io.File 类用于表示文件（或目录）

File 类只用于表示文件（或目录）的信息（名称、大小等），不能用于文件内容的访问。

File 类的构造方法：
```java
File(File parent, String child);//根据 parent 抽象路径名和 child 路径名字符串创建一个新 File 实例。 
File(String pathname); //通过将给定路径名字符串转换为抽象路径名来创建一个新 File 实例。 
File(String parent, String child);//根据 parent 路径名字符串和 child 路径名字符串创建一个新 File 实例。 
File(URI uri);//通过将给定的 file: URI 转换为一个抽象路径名来创建一个新的 File 实例。 
```

关于 File 类的一些其他方法我们可查阅 API 文档。

### **3. RandomAccessFile 类** ###

RandomAccessFile 是 java 提供的对文件内容的访问，既可以读文件，也可以写文件。

RandomAccessFile 类的实例支持对随机访问文件的读取和写入，可以访问文件的任意位置。随机访问文件的行为类似存储在文件系统中的一个大型 byte 数组。存在指向该隐含数组的光标或索引，称为文件指针；输入操作从文件指针开始读取字节，并随着对字节的读取而前移此文件指针。如果随机访问文件以读取/写入模式创建，则输出操作也可用；输出操作从文件指针开始写入字节，并随着对字节的写入而前移此文件指针。写入隐含数组的当前末尾之后的输出操作导致该数组扩展。该文件指针可以通过 getFilePointer 方法读取，并通过 seek 方法设置。 
通常，如果此类中的所有读取例程在读取所需数量的字节之前已到达文件末尾，则抛出 EOFException（是一种 IOException）。如果由于某些原因无法读取任何字节，而不是在读取所需数量的字节之前已到达文件末尾，则抛出 IOException，而不是 EOFException。需要特别指出的是，如果流已被关闭，则可能抛出 IOException。 

构造方法：
```java
RandomAccessFile(File file, String mode)；//创建从中读取和向其中写入（可选）的随机访问文件流，该文件由 File 参数指定。 
RandomAccessFile(String name, String mode);//创建从中读取和向其中写入（可选）的随机访问文件流，该文件具有指定名称。 
```
其中 mode 主要有两种模式，"rw"(读写) 和 "r"(只读)。

该类还分别提供了一个写入方法和读方法：
write(int)： 只写一个字节（后8位),同时指针指向下一个位置，准备再次写入
raf.read()： 读一个字节

### **4. 字节流** ###

提供字节流读取方式的类主要有 InputStream 类和 OutputStream 类及其子类。
InputStream 抽象类是表示字节输入流的所有类的超类，OutputStream 抽象类是表示字节输出流的所有类的超类。
InputStream 的子类主要有:
- FileInputStream
- FilterInputStream (其下有 BufferedInputStream、DataInputStream 和 PushbackInputStream 三个子类)
- ObjectInputStream
- PipedInputStream
- SequenceInputStream
- StringBufferInputStream
- ByteArrayInputStream

OutputStream 的子类主要有:
- FileOutputStream
- FilterOutputStream (其下有 BufferedOutputStream、DataOutputStream 和 PrintStream 三个子类)
- ObjectOutputStream
- PipedOutputStream
- ByteArrayOutputStream

主要子类的介绍：
FileInputStream： 具体实现了在文件上读取数据
FileOutputStream： 实现了向文件中写出byte数据的方法
DataOutputStream/DataInputStream： 对"流"功能的扩展，可以更加方面的读取int,long，字符等类型数据
BufferedInputStream/BufferedOutputStream： 这两个流类位IO提供了带缓冲区的操作，一般打开文件进行写入或读取操作时，都会加上缓冲，这种流模式提高了IO的性能
如果从应用程序中把输入放入文件，相当于将一缸水倒入到另一个缸中，那么：
FileOutputStream ---> write() 方法相当于一滴一滴地把水“转移”过去
DataOutputStream --> writeXxx() 方法会方便一些，相当于一瓢一瓢把水“转移”过去
BufferedOutputStream ---> write 方法更方便，相当于一飘一瓢先放入桶中，再从桶中倒入到另一个缸中，性能提高了
   
FileInputStream 和 FileOutputStream 类的主要方法：
```java
//FileInputStream 类
int read() ;// 从此输入流中读取一个数据字节 
int read(byte[] b);  //从此输入流中将最多 b.length 个字节的数据读入一个 byte 数组中，返回实际读取的字节的个数 
int read(byte[] b, int off, int len);  //从此输入流中将最多 len 个字节的数据读入一个 byte 数组中，返回实际读取的字节的个数 
long skip(long n); //从输入流中跳过并丢弃 n 个字节的数据 
//FileOutputStream 类
void write(byte[] b); //将 b.length 个字节从指定 byte 数组写入此文件输出流中 
void write(byte[] b, int off, int len);  //将指定 byte 数组中从偏移量 off 开始的 len 个字节写入此文件输出流
void write(int b);  //将指定字节写入此文件输出流。 
```

DataInputStream 和 DataOutputStream 类的主要方法：
```java
//DataInputStream 类
int read(byte[] b);  //从包含的输入流中读取一定数量的字节，并将它们存储到缓冲区数组 b 中
int read(byte[] b, int off, int len); //从包含的输入流中将最多 len 个字节读入一个 byte 数组中
boolean readBoolean(); //读取一个输入字节，如果该字节不是零，则返回 true，如果是零，则返回 false 
char readChar();  //读取两个输入字节并返回一个 char 值
double readDouble(); //读取八个输入字节并返回一个 double 值
void readFully(byte[] b); //从输入流中读取一些字节，并将它们存储在缓冲区数组 b 中
void readFully(byte[] b, int off, int len);  //从输入流中读取 len 个字节
//DataOutputStream 类
write(byte[] b, int off, int len); //将指定 byte 数组中从偏移量 off 开始的 len 个字节写入基础输出流
void write(int b);  //将指定字节（参数 b 的八个低位）写入基础输出流
void writeBoolean(boolean v); //将一个 boolean 值以 1-byte 值形式写入基础输出流
void writeChars(String s);   //将字符串按字符顺序写入基础输出流
void writeDouble(double v); //使用 Double 类中的 doubleToLongBits 方法将 double 参数转换为一个 long 值，然后将该 long 值以 8-byte 值形式写入基础输出流中，先写入高字节
void writeInt(int v);  //将一个 int 值以 4-byte 值形式写入基础输出流中，先写入高字节  
void writeUTF(String str); /以与机器无关方式使用 UTF-8 修改版编码将一个字符串写入基础输出流
//更多方法可参见API
```

BufferedInputStream 和 BufferedOutputStream 类的主要方法： 
```java
//BufferedInputStream类
void mark(int readlimit);  //在此输入流中标记当前的位置
boolean markSupported();   //测试此输入流是否支持 mark 和 reset 方法
int read(byte[] b); //从输入流中读取一定数量的字节，并将其存储在缓冲区数组 b 中
int read(byte[] b, int off, int len);   //将输入流中最多 len 个数据字节读入 byte 数组 
void reset();    //将此流重新定位到最后一次对此输入流调用 mark 方法时的位置
long skip(long n);  //跳过和丢弃此输入流中数据的 n 个字节
//BufferedOutputStream 类
void flush();  //刷新此缓冲的输出流
void write(byte[] b, int off, int len);  //将指定 byte 数组中从偏移量 off 开始的 len 个字节写入此缓冲的输出流
void write(int b);  //将指定的字节写入此缓冲的输出流
```

### **5. 字符流** ###

字符流(Reader Writer)操作的是文本文本文件，一次处理一个字符，字符的底层任然是基本的字节序列。

提供字符流读取方式的类主要有 Reader 类和 Writer 类及其子类。

Reader 的子类主要有：
- BufferedReader
- InputStreamReader(及其还包含 FileReader 子类)
- StringReader
- PipedReader
- ByteArrayReader
- FilterReader(及其还包含 PushbackReader 子类)

Writer 的子类主要有：
- BufferedWriter
- OutputStreamWriter(及其还包含 FileWriter 子类)
- StringWriter
- PipedWriter
- CharArrayWriter
- FilterWriter
- PrinterWriter

BufferedReader 和 BufferedWriter 类的主要方法：
```java
//BufferedReader
int read(); //读取单个字符
int read(char[] cbuf, int off, int len); //将字符读入数组的某一部分 
String readLine(); //读取一个文本行
long skip(long n); //跳过字符
//BufferedWriter
void flush();   //刷新该流的缓冲
void newLine();  //写入一个行分隔符
void write(char[] cbuf, int off, int len); //写入字符数组的某一部分
void write(int c);  //写入单个字符
void write(String s, int off, int len);  //写入字符串的某一部分 
```

FileReader 和 FileWriter 类的主要方法：
```java
//FileReader
int read(); 读取单个字符
int read(char[] cbuf, int offset, int length); 将字符读入数组中的某一部分
//FileWriter
void flush();  //刷新该流的缓冲
String getEncoding(); //返回此流使用的字符编码的名称
void write(char[] cbuf, int off, int len);  //写入字符数组的某一部分 
void write(int c);   //写入单个字符
```