---
title: Netty（六）：一切皆socket
date: 2018-10-18 22:52:17
tags: ["Netty"]

categories: ["Netty"]
---

&emsp;&emsp;基于 socket 通信模型实现的最简单服务器，不考虑健壮性，稳定性。
&emsp;&emsp;网络编程考虑需要考虑的 3 个方面，ip、端口、协议，而协议是网络编程最重要的一块。

<!--  more -->

```java
package top.alertcode.trainhigh.netty.http;

import java.io.BufferedReader;
import java.io.File;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.net.URLDecoder;
import java.util.StringTokenizer;

/**
 * @author alertcode
 * @date 2018-10-18
 * @copyright alertcode.top
 */
public class TSocket implements Runnable {

  private final static int PORT = 8080;
  private ServerSocket server = null;

  public static void main(String[] args) {
    new TSocket();
  }

  public TSocket() {
    try {
      server = new ServerSocket(PORT);
      if (server == null) {
        System.exit(1);
      }
      new Thread(this).start();
    } catch (IOException e) {
      e.printStackTrace();
    }
  }

  @Override
  public void run() {
    while (true) {
      try {
        Socket client = null;
        client = server.accept();
        if (client != null) {
          try {
            System.out.println("连接服务器成功！！...");

            BufferedReader reader = new BufferedReader(
                new InputStreamReader(client.getInputStream()));

            // GET /test.jpg /HTTP1.1
            String line = reader.readLine();

            System.out.println("line: " + line);

            String resource = line.substring(line.indexOf('/'),
                line.lastIndexOf('/') - 5);

            System.out.println("the resource you request is: "
                + resource);

            resource = URLDecoder.decode(resource, "UTF-8");

            String method = new StringTokenizer(line).nextElement()
                .toString();

            System.out.println("the request method you send is: "
                + method);

            while ((line = reader.readLine()) != null) {
              if (line.equals("")) {
                break;
              }
              System.out.println("the Http Header is : " + line);
            }

            if ("post".equals(method.toLowerCase())) {
              System.out.println("the post request body is: "
                  + reader.readLine());
            }

            if (resource.endsWith(".mkv")) {

              transferFileHandle("E:\\ideaworkspace\\train-high\\src\\main\\resources\\videos\\test.mkv",
                  client);
              closeSocket(client);
            } else if (resource.endsWith(".jpg")) {

              transferFileHandle("images/test.jpg", client);
              closeSocket(client);
              continue;

            } else if (resource.endsWith(".rmvb")) {

              transferFileHandle("videos/test.rmvb", client);
              closeSocket(client);
            } else if (resource.endsWith("test")) {
              PrintStream w=new PrintStream(client.getOutputStream(),true);
              w.println("HTTP/1.0 200 OK");
              w.println("Content-Type: text/html; charset=utf-8");
              w.println();
              w.println("<h1>这是响应报文</h1>");
              w.println("<h1>这是响应报文1</h1>");
              w.close();
              closeSocket(client);
            } else {
              PrintStream writer = new PrintStream(
                  client.getOutputStream(), true);
              writer.println("HTTP/1.0 404 Not found");// 返回应答消息,并结束应答
              writer.println();// 根据 HTTP 协议, 空行将结束头信息
              writer.close();
              closeSocket(client);
            }
          } catch (Exception e) {
            System.out.println("HTTP服务器错误:"
                + e.getLocalizedMessage());
          }
        }
      } catch (Exception e) {
        e.printStackTrace();
      }
    }
  }

  private void closeSocket(Socket socket) {
    try {
      socket.close();
    } catch (IOException ex) {
      ex.printStackTrace();
    }
    System.out.println(socket + "离开了HTTP服务器");
  }


  private void transferFileHandle(String path, Socket client) {

    File fileToSend = new File(path);

    if (fileToSend.exists() && !fileToSend.isDirectory()) {
      try {
        PrintStream writer = new PrintStream(client.getOutputStream());
        writer.println("HTTP/1.0 200 OK");// 返回应答消息,并结束应答
        writer.println("Content-Type:application/binary");
        writer.println("Content-Length:" + fileToSend.length());// 返回内容字节数
        writer.println();// 根据 HTTP 协议, 空行将结束头信息

        FileInputStream fis = new FileInputStream(fileToSend);
        byte[] buf = new byte[fis.available()];
        fis.read(buf);
        writer.write(buf);
        writer.close();
        fis.close();
      } catch (IOException e) {
        e.printStackTrace();
      }
    }
  }
}

```
