---
title: 并发编程（七）：一个基于线程池技术的简单web服务器
date: 2018-10-26 07:11:23
tags: ["多线程"]
categories: ["多线程"]

---

目前浏览器都是支持多线程进行访问，比如说在请求一个 HTML 页面的时候，页面中包含的图片资源，样式资源会被浏览器发起并发的获取，这样用户就不会遇到一直等到一个图片完全下载完成才能继续查看文字内容的尴尬情况。

如果 web 服务器是单线程的，多线程的浏览器也没有用武之地，因为服务端还是一个请求一个请求的顺序处理。因此，大部分 web 服务器都是支持并发访问的。常用的 java web 服务器，如 Tomcat、Jetty,在器处理请求的过程中都使用到了线程池技术。

下面通过使用前面创建的线程池构造一个简单的 web 服务器，这个 web 服务器用来处理 Http 请求，目前只能处理简单的文本和 JPG 图片内容。这个 web 服务器使用 main 线程不断地接受客户端 Socket 的连接，将连接以及请求提交给线程池处理，这样使得 web 服务器能够同时处理多个客户端请求。

<!--more-->

### 代码

```java
package top.alertcode.trainhigh.concurrent;

import java.io.BufferedReader;
import java.io.ByteArrayOutputStream;
import java.io.Closeable;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

public class SimpleHttpServer {

  //处理HttpRequest的线程池
  static ThreadPool<HttpRequestHandler> threadPool = new DefaultThreadPool<HttpRequestHandler>(1);
  static String basePath;
  static ServerSocket serverSocket;
  //服务监听端口
  static int port = 8181;

  public static void setPort(int port) {
    if (port > 0) {
      SimpleHttpServer.port = port;
    }
  }

  public static void setBasePath(String basePath) {
    if (basePath != null && new File(basePath).exists() && new File(basePath).isDirectory()) {
      SimpleHttpServer.basePath = basePath;
    }
  }

  //启动SimpleHttpServer
  public static void start() throws Exception {
    serverSocket = new ServerSocket(port);
    Socket socket = null;
    while ((socket = serverSocket.accept()) != null) {
      //接收一个客户端Socket，生成一个HttpRequestHandler，放入线程池中执行
      threadPool.execute(new HttpRequestHandler(socket));
    }
    serverSocket.close();
  }

  static class HttpRequestHandler implements Runnable {

    private Socket socket;

    public HttpRequestHandler(Socket socket) {
      this.socket = socket;
    }

    //关闭流或者socket
    private static void close(Closeable... closeables) {
      if (closeables != null) {
        for (Closeable closeable : closeables) {
          try {
            if (closeable != null) {
              closeable.close();
            }
          } catch (IOException e) {
            e.printStackTrace();
          }
        }
      }
    }

    @Override
    public void run() {
      String line = null;
      BufferedReader br = null;
      BufferedReader reader = null;//读socket的输入
      PrintWriter out = null;//读socket的输出
      InputStream in = null;//读图片文件
      try {
        reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        String header = reader.readLine();
        System.out.println("收到header=" + header);
        //由相对路径计算出绝对路径
        String filePath = basePath + header.split(" ")[1];
        out = new PrintWriter(socket.getOutputStream());
        if (!new File(filePath).exists()) {
          out.flush();
          return;
        }
        //如果有请求资源的后缀是.jpg或者.ico，则读取资源并输出
        if (filePath.endsWith(".jpg")) {
          in = new FileInputStream(filePath);
          ByteArrayOutputStream baos = new ByteArrayOutputStream();
          int i = 0;
          while ((i = in.read()) != -1) {
            baos.write(i);
          }
          byte[] array = baos.toByteArray();
          out.println("HTTP/1.1 200 OK");
          out.println("Content-Type:image/jpeg");
          out.println("Content-Length:" + array.length);
          out.println("");
          socket.getOutputStream().write(array, 0, array.length);
        } else {
          br = new BufferedReader(new InputStreamReader(new FileInputStream(filePath)));
          out.println("HTTP/1.1 200 OK");
          out.println("Server: Molly");
          out.println("Content-Type:text/html;charset=UTF-8");
          out.println("");
          while ((line = br.readLine()) != null) {
            out.println(line);
          }
        }
        out.flush();
      } catch (Exception e) {
        e.printStackTrace();
        out.println("HTTP/1.1 500");
        out.println("");
        out.flush();
      } finally {
        close(br, in, reader, out, socket);
      }
    }
  }
}

```

```java
package top.alertcode.trainhigh.concurrent;

public class HttpServerMain {

    public static void main(String[] args) {
        SimpleHttpServer shs = new SimpleHttpServer();
        shs.setBasePath("D:/basedir"); //将根目录定义到html所在的目录
        try {
            shs.start();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

```html
<html>
  <head>
    <title>测试页面</title>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
  </head>
  <body align="center">
    <h1>第一张图片</h1>
    <img src="1.jpg" alt="" align="middle" />
    <h1>第二张图片</h1>
    <img src="2.jpg" alt="" align="middle" />
    <h1>第三张图片</h1>
    <img src="3.jpg" alt="" align="middle" />
  </body>
</html>
```

### 运行效果

&emsp;&emsp;将图片和 test.html 放在 D://basedir 目录下，启动服务，使用火狐浏览器访问`http://192.168.0.105:8181/test.html`
![图片以及test.html路径](360截图17891226112143101.jpg)
![运行结果](360截图182805066265104.jpg)

### 使用压力测试工具 ab 进行简单压测

&emsp;&emsp;ab -c 10 -n 5000 `http://192.168.0.105:8181/test.html/`

|  线程池数量  |    1    |    5    |   10    |
| :----------: | :-----: | :-----: | :-----: |
|  响应时间 s  |  0.763  |  0.879  |  0.678  |
| 每秒查询数量 | 1310.39 | 1137.26 | 1476.01 |
| 测试完成时间 |  3.816  |  4.397  |  3.388  |
