### Java IO

主要可以分成4类

1. 基于字节操作的IO接口：InputStream和 OutputStream
2. 基于字符操作的IO接口：Writer和Reader
3. 基于磁盘才做的IO接口：FIle
4. 基于网络操作的IO接口：Socker

前两者从数据格式的角度出发，后两者从传输方式的角度出发，也就是要把数据传到哪里去。

#### 基于字节操作的IO接口

需要注意的有两点：

- 操作数据的方式可以组合使用

  ```java
  OutputStream out = new BufferedOutputStream(new ObjectOutputStream(new FileOutputStream("fileName"))；
  ```

  这些数据流之间没有继承关系，只是所有的接口都继承自OutputStream

- 流最终写到什么地方必须指定，比如本地磁盘的文件或者网络文件。

#### 基于字符操作的IO接口

注意点

- 文件是以字节的形式存在，想要以字符的方式读取，必须经过编码

```java
try { 
            StringBuffer str = new StringBuffer(); 
            char[] buf = new char[1024]; 
            FileReader f = new FileReader("file"); 
            while(f.read(buf)>0){ 
                str.append(buf); 
            } 
            str.toString(); 
 } catch (IOException e) {}
```



#### 磁盘的IO工作机制

文件是操作系统和磁盘驱动器交互的最小单元。

首先会创建一个文件对象File，这个时候并不关心文件是否真的存在，而是关心文件到底如何操作。

然后根据具体操作，比如读取文件数据，则需要创建对象FIleInputStream，这里会利用FileDescrior去看文件是否真的存在，然后利用转码对象StreamDecoder读的文件。

#### Java Socket的工作机制

- 网络传输的7层协议：

  1. 物理层：光纤
  2. 链路层：以太网
  3. 网络层：IP
  4. 传输层：TCP
  5. 会话层：Scoket
  6. 表示层：AFP
  7. 应用层：http

  由此可以明白http协议和TCP/IP协议之间的区别。可以说IP协议铺好了网络之间的高速公路，TCP协议作为卡车将信息进行运输，http协议负责对货物进行解释

- Socket机制

  1. 建立通信链路

     客户端：建立Socket实例，分配端口，创建一个包含本地和远程地址以及端口号的数据结构

     服务端：创建ServerSocket实例（保证端口没有被占用*这里两个端口可以不同，需要发送时指定* ），然后创建底层数据结构，包含监听端口和监听地址的通配符。之后调用accept() 方法以后，进入阻塞状态，等待客户端的请求。

  2. 数据传输

     客户端和服务端都有一个Socket实例，每个实例都有InputStream和OutputStream。系统为流分配一定大小的缓冲区，数据的写入和读取都是通过这个缓存区完成的。



### Java HttpURLConnection Class （官方API）

使用这个类遵循的模式：

1. 通过调用url.openConnection()去获得一个http链接并将结果赋给一个HttpURLConnection；
2. 准备请求。请求的基本要素是URL。请求头还可以包含元数据，如凭据，cookie等；
3. 有选择的上传一个请求体（request body）。后面大概提了一下可以通过getOutputStream获取写入数据（*是因为我传的不一定是我想传的所以我还要看一下我到底传了啥？*）
4. 读取响应。通过getInputStream获取返回数据。
5. 断开连接。

Example：

```Java
 URL url = new URL("http://www.android.com/");
   HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
   try {
     InputStream in = new BufferedInputStream(urlConnection.getInputStream());
     readStream(in);
   } finally {
     urlConnection.disconnect();
   }
 
```

 **这里readStream需要注意一下**：

[参考博客](http://blog.isming.me/2014/05/11/use-network-in-android/)说这个是自己实现的

#### 安全连接

没看，啥玩意？

#### 返回值处理

如果getInputStream（）抛出了一个错误，可以使用getErrorStream（）获取错误信息。headers可以使用getHeaderFields（）获取。

#### 传输内容

为了给网页服务端上传数据，使用setDoPoutput(true)来配置输出连接。

如果数据长度已知，可以使用setFixedLengthStreamingMode(int)

如果数据长度位置，可以使用`setChunkedStreamingMode(int)` 

不然的话 `HttpURLConnection` 会将请求体 强制缓存。

Example：

```
HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection();
   try {
     urlConnection.setDoOutput(true);
     urlConnection.setChunkedStreamingMode(0);

     OutputStream out = new BufferedOutputStream(urlConnection.getOutputStream());
     writeStream(out);

     InputStream in = new BufferedInputStream(urlConnection.getInputStream());
     readStream(in);
   } finally {
     urlConnection.disconnect();
   }
```

这里的问题还是  **writeStream(out);到底咋写的？** 也是自己写的。

后面还解释了一下关于传输表现和细节问题，如何处理登录和授权问题以及cookie怎么写。



### 对于输出流写内容

1. 如果写不成功，问题应当抛出
2. 获取数据
3. out.write(result.getBytes());

