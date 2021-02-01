---
title: Netty（一）：netty-hello-world
date: 2018-09-15 18:16:40
tags: ["Netty"]

categories: ["Netty"]
---

### 前言

&emsp;&emsp;作为一个 java 后台的程序员，在传统的简单的 java web 项目中用到 Netty 的时候并不多。但是，当你的项目成规模，访问用户量增大，B/S 和 C/S 架构同时出现的时候，就会考虑一系列的分布式框架，如 Dubbo,Dubbo 底层的 RPC 协议即用 Netty 实现，基于网络 NIO 模型，说到 Netty，有这样的一个小故事，在 java7 出现后，Netty 曾有两个小版本使用 java7 Fork/Join 框架，命名为 Netty5.0，但是对于性能的提升并不明显，反而提升了编程的复杂度，所以后来被废弃了，Netty 最新的版本是

```java
<dependency>
     <groupId>io.netty</groupId>
     <artifactId>netty-all</artifactId>
     <version>4.1.29.Final</version>
   </dependency>
```

<!--more-->

### netty 中比较重要的几个对象

参考 [Netty Overview](http://tutorials.jenkov.com/netty/overview.html)

1.  Bootstrap、ServerBootstrap 是一个启动 NIO 服务的辅助启动类 Bootstrap 客户端 ServerBootStrap 服务端
2.  EventLoopGroop LoopGroop 是 netty 线程组，server 端启动两个线程组，一个是用于接收 Client 端连接的，另一个用来处理业务逻辑
3.  Channel(SocketChannel) 与网络上的的其他机器建立 TCP 连接
4.  ChannelInitializer 建立 Socket 对象，初始化 Socket 的信息
5.  ChannelPipeline 每一个 SocketChannel 都有一个 ChannelPipeline，ChannelPipeline 对象包含多个 ChannelHandler,
    内部的 ChannelHandler 依次调用
6.  ChannelHandler 处理来自 SocketChannel 的数据，并可以写入 SocketCannel
    ![](Netty模型.PNG)

### Netty 版 hello world

#### 服务端代码

##### NettyServer

```java
package top.alertcode.trainhigh.netty;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

/**
 *
 * @author alertcode
 * @date 2018-09-15
 * @copyright alertcode.top
 */
public class NettyServer {


  public static void main(String[] args) throws Exception {
    //1 第一个线程组 是用于接收Client端连接的
    EventLoopGroup bossGroup = new NioEventLoopGroup();
    //2 第二个线程组 是用于实际的业务处理操作的
    EventLoopGroup workerGroup = new NioEventLoopGroup();
    ServerBootstrap b;

    try {
      //3 创建一个辅助类Bootstrap，就是对我们的Server进行一系列的配置
      b = new ServerBootstrap();
      //把俩个工作线程组加入进来
      b.group(bossGroup, workerGroup)
          //我要指定使用NioServerSocketChannel这种类型的通道
          .channel(NioServerSocketChannel.class).option(ChannelOption.SO_BACKLOG, 1024)
          //一定要使用 childHandler 去绑定具体的 事件处理器
          .childHandler(new ChannelInitializer<SocketChannel>() {
            @Override
            protected void initChannel(SocketChannel channel) throws Exception {
              ChannelPipeline pipeline = channel.pipeline();
              pipeline.addLast(new ServerHandler());
            }
          });

      //绑定指定的端口 进行监听
      ChannelFuture f = b.bind(8765).sync();
      //等待服务监听端口关闭
      f.channel().closeFuture().sync();
    } finally {
      bossGroup.shutdownGracefully();
      workerGroup.shutdownGracefully();
    }
  }
}
```

##### ServerHandler

```java
package top.alertcode.trainhigh.netty;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

public class ServerHandler extends ChannelInboundHandlerAdapter {

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

    //do something msg
    ByteBuf buf = (ByteBuf) msg;
    byte[] data = new byte[buf.readableBytes()];
    buf.readBytes(data);
    String request = new String(data, "utf-8");
    System.out.println("Server: " + request);
    //写给客户端
    String response = "我是反馈的信息";
    ctx.writeAndFlush(Unpooled.copiedBuffer(response.getBytes()));
    //.addListener(ChannelFutureListener.CLOSE);

  }

  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    cause.printStackTrace();
    ctx.close();
  }

}
```

#### 客户端代码

##### NettyClient

```java
package top.alertcode.trainhigh.netty;

import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelPipeline;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

/**
 * @author alertcode
 * @date 2018-09-15
 * @copyright alertcode.top
 */
public class NettyClient {

  public static void main(String[] args) throws Exception {

    EventLoopGroup workgroup = new NioEventLoopGroup();
    Bootstrap b = new Bootstrap();
    ChannelFuture cf1;
    try {
      b.group(workgroup)
          .channel(NioSocketChannel.class)
          .handler(new ChannelInitializer<SocketChannel>() {
            @Override
            protected void initChannel(SocketChannel sc) throws Exception {
              //ChannelPipeline 负责安排handler的执行顺序de
              ChannelPipeline pipeline = sc.pipeline();
              pipeline.addLast(new ClientHandler());
            }
          });

      cf1 = b.connect("127.0.0.1", 8765).sync();

      //buf
      int count = 0;
      while (true) {
        cf1.channel().writeAndFlush(Unpooled.copiedBuffer("来自客户端消息！".getBytes()));
        Thread.sleep(1000);
        count++;
        if (count == 10) {
          cf1.channel().closeFuture().sync();
          return;
        }
      }

    } finally {
      workgroup.shutdownGracefully();
    }
  }
}

```

##### ClientHandler

```java
package top.alertcode.trainhigh.netty;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.util.ReferenceCountUtil;

public class ClientHandler extends ChannelInboundHandlerAdapter {

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    try {
      //do something msg
      ByteBuf buf = (ByteBuf) msg;
      byte[] data = new byte[buf.readableBytes()];
      buf.readBytes(data);
      String request = new String(data, "utf-8");
      System.out.println("Client: " + request);


    } finally {
      ReferenceCountUtil.release(msg);
    }
  }

  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    cause.printStackTrace();
    ctx.close();
  }
}
```

#### 运行结果

##### 服务端接受请求响应

```
20:36:02.709 [main] DEBUG io.netty.buffer.ByteBufUtil - -Dio.netty.maxThreadLocalCharBufferSize: 16384
服务器启动，监听请求。。。。。。
20:36:11.954 [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler - -Dio.netty.recycler.maxCapacityPerThread: 4096
20:36:11.954 [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler - -Dio.netty.recycler.maxSharedCapacityFactor: 2
20:36:11.954 [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler - -Dio.netty.recycler.linkCapacity: 16
20:36:11.954 [nioEventLoopGroup-3-1] DEBUG io.netty.util.Recycler - -Dio.netty.recycler.ratio: 8
20:36:11.964 [nioEventLoopGroup-3-1] DEBUG io.netty.buffer.AbstractByteBuf - -Dio.netty.buffer.bytebuf.checkAccessible: true
20:36:11.966 [nioEventLoopGroup-3-1] DEBUG io.netty.util.ResourceLeakDetectorFactory - Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@17f4d087
Server: 来自客户端消息！
Server: 来自客户端消息！
Server: 来自客户端消息！
...
```

##### 客户端接受结果

```
20:36:11.873 [main] DEBUG io.netty.buffer.ByteBufUtil - -Dio.netty.maxThreadLocalCharBufferSize: 16384
客户端启动，发送消息。。。。。。
20:36:11.925 [main] DEBUG io.netty.buffer.AbstractByteBuf - -Dio.netty.buffer.bytebuf.checkAccessible: true
20:36:11.928 [main] DEBUG io.netty.util.ResourceLeakDetectorFactory - Loaded default ResourceLeakDetector: io.netty.util.ResourceLeakDetector@30a3107a
20:36:11.933 [main] DEBUG io.netty.util.Recycler - -Dio.netty.recycler.maxCapacityPerThread: 4096
20:36:11.933 [main] DEBUG io.netty.util.Recycler - -Dio.netty.recycler.maxSharedCapacityFactor: 2
20:36:11.933 [main] DEBUG io.netty.util.Recycler - -Dio.netty.recycler.linkCapacity: 16
20:36:11.933 [main] DEBUG io.netty.util.Recycler - -Dio.netty.recycler.ratio: 8
Client: 我是反馈的信息
Client: 我是反馈的信息
Client: 我是反馈的信息
...
```
