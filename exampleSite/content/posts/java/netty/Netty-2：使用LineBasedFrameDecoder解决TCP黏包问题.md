---
title: Netty（二）：使用LineBasedFrameDecoder解决TCP黏包问题
date: 2018-10-07 00:43:10
tags: ["Netty"]

categories: ["Netty"]
---

### TCP 粘包/拆包问题说明

&emsp;&emsp;假设客户端分别发送了两个数据包 D1、D2 给服务端，由于服务端一次读取的字节数是不确定的，故可能存在以下 4 种情况：

- 服务端分两次读取到了两个独立的数据包，分别是 D1 和 D2,没有粘包和拆包。
- 服务端一次接收到了两个数据包，D1 和 D2 粘合在一起，被称为 TCP 粘包
- 服务端分两次读取到了两个数据包，第一次读取到了完整的 D1 包的和 D2 包的部分内容，地为此读取到了 D2 包的剩余内容，这被称为 TCP 拆包。
- 服务端分两次读取到了两个数据包，第一次读取到了 D1 包的部分内容 D1-1，第二次读取到了 D1 包剩余内容 D1-2 和 D2 包整包。

### TCP 粘包/拆包问题发生的原因

- 应用程序 write 写入的自己大小大于套接口发送缓冲区的大小
- 进行 MSS 大小的 TCP 分段
- 以太网帧的 payload 大于 MTU 进行 IP 分片

### 解决策略

- 消息定长，例如每个报文的大小为固定长度的 200 字节，如果不够，空位补空格
- 在包尾增加回车换行符进行分割，例如 FTP 协议
- 将消息分为消息头和消息体，消息头中包含表示消息总长度（或消息体长度）的字段，通常设计思路为消息头的第一个字段使用 int32 来表示消息的总长度。
- 更复杂的应用层协议。
  <!--more-->

### 未考虑拆包/粘包问题 demo

#### 服务端代码

```java
package top.alertcode.trainhigh.netty.tcppackage;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import java.util.Date;
import lombok.extern.slf4j.Slf4j;

/**
 * @author alertcode
 * @date 2018-10-06
 * @copyright alertcode.top
 */
@Slf4j
public class TimeServer {

  public static void main(String[] args) {
    int port = 8080;
    new TimeServer().bind(port);
  }

  void bind(int port) {
    NioEventLoopGroup bossGroup = new NioEventLoopGroup();
    NioEventLoopGroup workerGroup = new NioEventLoopGroup();
    ServerBootstrap serverBootstrap = new ServerBootstrap();
    serverBootstrap.group(bossGroup, workerGroup)
        .channel(NioServerSocketChannel.class)
        .option(ChannelOption.SO_BACKLOG, 1024).childHandler(new ChannelInitializer<SocketChannel>() {

      @Override
      protected void initChannel(SocketChannel socketChannel) throws Exception {

        socketChannel.pipeline().addLast(new TimeServerHandler());

      }
    });
    try {
      ChannelFuture sync = serverBootstrap.bind(port).sync();
      sync.channel().closeFuture().sync();
    } catch (InterruptedException e) {
      bossGroup.shutdownGracefully();
      workerGroup.shutdownGracefully();
    }
  }

}

@Slf4j
class TimeServerHandler extends ChannelInboundHandlerAdapter {

  private int counter;

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

    //tcp 拆包粘包问题
    ByteBuf buf = (ByteBuf) msg;
    byte[] bytes = new byte[buf.readableBytes()];
    buf.readBytes(bytes);
    String body = new String(bytes, "UTF-8").substring(0, bytes.length);


    String rep =
        "client message".equalsIgnoreCase(body) ? new Date(System.currentTimeMillis()).toString()
            : "bad message";
    rep = rep + System.getProperty("line.separator");

    log.info("the time server receive order:{} counter:{}", body, ++counter);
    ByteBuf buf1 = Unpooled.copiedBuffer(rep.getBytes());
    ctx.writeAndFlush(buf1);
  }


  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    ctx.close();
  }

}
```

#### 客户端代码

```java
package top.alertcode.trainhigh.netty.tcppackage;

import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import lombok.extern.slf4j.Slf4j;

/**
 * @author alertcode
 * @date 2018-10-06
 * @copyright alertcode.top
 */
public class TimeClient {

  public static void main(String[] args) {
    new TimeClient().connect(8080, "localhost");
  }

  void connect(int port, String host) {
    NioEventLoopGroup eventExecutors = new NioEventLoopGroup();
    Bootstrap bootstrap = new Bootstrap();
    bootstrap.group(eventExecutors).channel(NioSocketChannel.class)
        .option(ChannelOption.TCP_NODELAY, true).handler(

        new ChannelInitializer<SocketChannel>() {
          @Override
          protected void initChannel(SocketChannel socketChannel) throws Exception {
            socketChannel.pipeline().addLast(new TimeClientHandler());
          }
        });
    try {
      ChannelFuture f = bootstrap.connect(host, port).sync();
      f.channel().closeFuture().sync();
    } catch (InterruptedException e) {
      eventExecutors.shutdownGracefully();
    }
  }
}

@Slf4j
class TimeClientHandler extends ChannelInboundHandlerAdapter {

  private int counter;

  private byte[] req;

  public TimeClientHandler() {
    // System.getProperty("line.separator")) 换行符
    req = ("client message" + System.getProperty("line.separator")).getBytes();

  }

  @Override
  public void channelActive(ChannelHandlerContext ctx) throws Exception {
    ByteBuf message = null;
    for (int i = 0; i < 100; i++) {
      message = Unpooled.buffer(req.length);
      message.writeBytes(req);
      ctx.writeAndFlush(message);
    }
    ctx.writeAndFlush(req);
  }

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    ByteBuf buf = (ByteBuf) msg;
    byte[] bytes = new byte[buf.readableBytes()];
    buf.readBytes(bytes);
    String body = new String(bytes, "UTF-8");
    log.info("now is :{},the counter is :{}", body, ++counter);
  }

  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    log.info("client is exception", cause.getMessage());
    ctx.close();
  }
}

```

#### 运行结果

- 服务端发生粘包，按道理应该 counter 为 100 次

```shell
01:19:53.114 [nioEventLoopGroup-3-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeServerHandler - the time server receive order:client message
client message
...省略64次
counter:1
01:19:53.124 [nioEventLoopGroup-3-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeServerHandler - the time server receive order:client message
client message
...省略34次
counter:2
```

- 由于服务端接受客户端的内容不匹配，服务端应该返回 2 条 bad message,正常情况 counter 应该为 2,有两条日志。日志为 1 条，发生客户端粘包。

```
01:19:53.126 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeClientHandler - now is :bad message
bad message
,the counter is :1
```

### 利用 LineBasedFrameDecoder 解决 TCP 粘包问题

#### 服务端代码

```java
package top.alertcode.trainhigh.netty.tcppackage;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;
import java.util.Date;
import lombok.extern.slf4j.Slf4j;

/**
 * @author alertcode
 * @date 2018-10-06
 * @copyright alertcode.top
 */
@Slf4j
public class TimeServer {

  public static void main(String[] args) {
    int port = 8080;
    new TimeServer().bind(port);
  }

  void bind(int port) {
    NioEventLoopGroup bossGroup = new NioEventLoopGroup();
    NioEventLoopGroup workerGroup = new NioEventLoopGroup();
    ServerBootstrap serverBootstrap = new ServerBootstrap();
    serverBootstrap.group(bossGroup, workerGroup)
        .channel(NioServerSocketChannel.class)
        .option(ChannelOption.SO_BACKLOG, 1024).childHandler(new ChannelInitializer<SocketChannel>() {

      @Override
      protected void initChannel(SocketChannel socketChannel) throws Exception {
        //使用LineBasedFrameDecoder和StringDecoder解决半包问题
        socketChannel.pipeline().addLast(new LineBasedFrameDecoder(1024));
        socketChannel.pipeline().addLast(new StringDecoder());
        socketChannel.pipeline().addLast(new TimeServerHandler());

      }
    });
    try {
      ChannelFuture sync = serverBootstrap.bind(port).sync();
      sync.channel().closeFuture().sync();
    } catch (InterruptedException e) {
      bossGroup.shutdownGracefully();
      workerGroup.shutdownGracefully();
    }
  }

}

@Slf4j
class TimeServerHandler extends ChannelInboundHandlerAdapter {

  private int counter;

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

    String body = (String) msg;
    String rep =
        "client message".equalsIgnoreCase(body) ? new Date(System.currentTimeMillis()).toString()
            : "bad message";
    //使用LineBasedFrameDecoder和StringDecoder解决半包问题,根据换行符进行分割
    rep = rep + System.getProperty("line.separator");
    log.info("the time server receive order:{} counter:{}", body, ++counter);
    ByteBuf buf1 = Unpooled.copiedBuffer(rep.getBytes());
    ctx.writeAndFlush(buf1);
  }


  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    ctx.close();
  }

}

```

#### 客户端代码

```java
package top.alertcode.trainhigh.netty.tcppackage;

import io.netty.bootstrap.Bootstrap;
import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;
import lombok.extern.slf4j.Slf4j;

/**
 * @author alertcode
 * @date 2018-10-06
 * @copyright alertcode.top
 */
public class TimeClient {

  public static void main(String[] args) {
    new TimeClient().connect(8080, "localhost");
  }

  void connect(int port, String host) {
    NioEventLoopGroup eventExecutors = new NioEventLoopGroup();
    Bootstrap bootstrap = new Bootstrap();
    bootstrap.group(eventExecutors).channel(NioSocketChannel.class)
        .option(ChannelOption.TCP_NODELAY, true).handler(

        new ChannelInitializer<SocketChannel>() {
          @Override
          protected void initChannel(SocketChannel socketChannel) throws Exception {
            //使用LineBasedFrameDecoder和StringDecoder解决读半包问题
            socketChannel.pipeline().addLast(new LineBasedFrameDecoder(1024));
            socketChannel.pipeline().addLast(new StringDecoder());
            socketChannel.pipeline().addLast(new TimeClientHandler());
          }
        });
    try {
      ChannelFuture f = bootstrap.connect(host, port).sync();
      f.channel().closeFuture().sync();
    } catch (InterruptedException e) {
      eventExecutors.shutdownGracefully();
    }
  }
}

@Slf4j
class TimeClientHandler extends ChannelInboundHandlerAdapter {

  private int counter;

  private byte[] req;

  public TimeClientHandler() {
    // System.getProperty("line.separator")) 换行符
    req = ("client message" + System.getProperty("line.separator")).getBytes();
  }

  @Override
  public void channelActive(ChannelHandlerContext ctx) throws Exception {
    ByteBuf message = null;
    for (int i = 0; i < 100; i++) {
      message = Unpooled.buffer(req.length);
      message.writeBytes(req);
      ctx.writeAndFlush(message);
    }
    ctx.writeAndFlush(req);
  }

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    String body = (String) msg;
    log.info("now is :{},the counter is :{}", body, ++counter);
  }

  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    log.info("client is exception", cause.getMessage());
    ctx.close();
  }
}

```

#### 运行结果

- 服务端

```shell
01:33:23.745 [nioEventLoopGroup-3-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeServerHandler - the time server receive order:client message counter:1
01:33:23.746 [nioEventLoopGroup-3-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeServerHandler - the time server receive order:client message counter:2
01:33:23.746 [nioEventLoopGroup-3-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeServerHandler - the time server receive order:client message counter:3
...
01:33:23.747 [nioEventLoopGroup-3-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeServerHandler - the time server receive order:client message counter:100
```

- 客户端

```shell
01:33:23.715 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeClientHandler - now is :Sun Oct 07 01:33:23 CST 2018,the counter is :1
01:33:23.715 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeClientHandler - now is :Sun Oct 07 01:33:23 CST 2018,the counter is :2
01:33:23.715 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeClientHandler - now is :Sun Oct 07 01:33:23 CST 2018,the counter is :3
01:33:23.715 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeClientHandler - now is :Sun Oct 07 01:33:23 CST 2018,the counter is :4
01:33:23.716 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeClientHandler - now is :Sun Oct 07 01:33:23 CST 2018,the counter is :5
...
01:33:23.716 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.tcppackage.TimeClientHandler - now is :Sun Oct 07 01:33:23 CST 2018,the counter is :100
```

### LineBasedFrameDecoder 和 StringDecoder 的原理分析

&emsp;&emsp;LineBasedFrameDecoder 的工作原理是它依次遍历 ByteBuf 中的可读字节，判断看是否有"\n"或者"\r\n",如果有，就以此位置为结束位置，从可读索引到结束位置区间字节就组成了一行。它是以换行符为结束标志的解码器，支持携带结束符或者不携带结束符两种解码方式，同事至此配置单行的最大长度。如果连续读取到最大长度后仍然没有出现换行符，就会抛出异常，同时忽略掉之前读取到的异常码流。
&emsp;&emsp;StringDecoder 的功能非常简单，就是将接收到的对象转换成字符串，然后继续调用后面的 Handler.LineBasedFrameDecoder+StringDecoder 组合就是按行切换的文本解码器，它被设计用来支持 TCP 的粘包和拆包。
