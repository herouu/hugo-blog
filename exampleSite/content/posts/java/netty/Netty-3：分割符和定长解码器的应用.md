---
title: Netty（三）：分割符和定长解码器的应用
date: 2018-10-07 22:10:40
tags: ["Netty"]

categories: ["Netty"]
---

&emsp;&emsp;主要介绍两种实用的解码器 DelimiterBasedFrameDecoder 和 FixedLengthFrameDecoder，前者可以自动完成一分割符做结束标志的消息的解码，后者可以自动完成对定长消息的解码，他们都能解决 TCP 粘包/拆包导致的读半包问题。

<!--more-->

### DelimiterBasedFrameDecoder 应用开发

&emsp;&emsp;通过对 DelimiterBasedFrameDecoder 的使用，我们可以自动完成一分割符作为码流结束标识的解码，下面通过一个演示程序来学习下如何使用 DelimiterBasedFrameDecoder 进行开发。EchoServer 接受到 EchoClient 的请求消息后，将其大衣出来，然后将原始消息返回给客户端，消息以`$_`作为分割符。

#### 服务端代码

```java
package top.alertcode.trainhigh.netty.delimiter;

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
import io.netty.handler.codec.DelimiterBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.extern.slf4j.Slf4j;

/**
 * @author alertcode
 * @date 2018-10-07
 * @copyright alertcode.top
 */
public class EchoServer {

  public static void main(String[] args) {
    new EchoServer().bind(8080);
  }

  void bind(int port) {
    NioEventLoopGroup worker = new NioEventLoopGroup();
    NioEventLoopGroup base = new NioEventLoopGroup();
    ServerBootstrap b = new ServerBootstrap();
    b.group(base, worker).channel(NioServerSocketChannel.class).option(ChannelOption.TCP_NODELAY, true)
        .handler(new LoggingHandler(LogLevel.INFO)).childHandler(new ChannelInitializer<SocketChannel>() {
      @Override
      protected void initChannel(SocketChannel socketChannel) throws Exception {
        ByteBuf byteBuf = Unpooled.copiedBuffer("$_".getBytes());
        socketChannel.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, byteBuf))
            .addLast(new StringDecoder()).addLast(new EchoServerHandler());
      }
    });
    try {
      ChannelFuture f = b.bind(port).sync();
      f.channel().closeFuture().sync();
    } catch (InterruptedException e) {
      e.printStackTrace();
      worker.shutdownGracefully();
      base.shutdownGracefully();
    }
  }
}

@Slf4j
class EchoServerHandler extends ChannelInboundHandlerAdapter {

  private int counter;

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    String body = (String) msg;
    log.info("this is counter:{}, times receive client:{}", ++counter, body);
    body += "$_";
    ByteBuf byteBuf = Unpooled.copiedBuffer(body.getBytes());
    ctx.writeAndFlush(byteBuf);

  }

  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    ctx.close();
  }
}

```

#### 客户端代码

```java
package top.alertcode.trainhigh.netty.delimiter;

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
import io.netty.handler.codec.DelimiterBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;
import lombok.extern.slf4j.Slf4j;

/**
 * @author alertcode
 * @date 2018-10-07
 * @copyright alertcode.top
 */
public class EchoClient {

  public static void main(String[] args) {
    new EchoClient().connect(8080, "localhost");
  }

  void connect(int port, String host) {
    NioEventLoopGroup client = new NioEventLoopGroup();
    Bootstrap bootstrap = new Bootstrap();
    bootstrap.group(client).channel(NioSocketChannel.class).option(ChannelOption.TCP_NODELAY, true)
        .handler(
            new ChannelInitializer<SocketChannel>() {
              @Override
              protected void initChannel(SocketChannel socketChannel) throws Exception {
                ByteBuf byteBuf = Unpooled.copiedBuffer("$_".getBytes());
                socketChannel.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, byteBuf))
                    .addLast(new StringDecoder()).addLast(new EchoClientHandler());

              }
            });
    try {
      ChannelFuture f = bootstrap.connect(host, port).sync();
      f.channel().closeFuture().sync();
    } catch (InterruptedException e) {
      client.shutdownGracefully();
    }
  }
}

@Slf4j
class EchoClientHandler extends ChannelInboundHandlerAdapter {

  private int counter;

  static final String ECHO_MESSAGE = "Hi alertcode,this is Netty.." + "$_";

  @Override
  public void channelActive(ChannelHandlerContext ctx) throws Exception {
    for (int i = 0; i < 10; i++) {
      ctx.writeAndFlush(Unpooled.copiedBuffer(ECHO_MESSAGE.getBytes()));
    }
  }

  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    ctx.close();
  }

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    log.info("this is counter:{} time receive server:{}", ++counter, msg);
  }
}
```

#### 运行结果

##### 客户端

```
23:33:19.678 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoClientHandler - this is counter:1 time receive server:Hi alertcode,this is Netty..
23:33:19.678 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoClientHandler - this is counter:2 time receive server:Hi alertcode,this is Netty..
23:33:19.679 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoClientHandler - this is counter:3 time receive server:Hi alertcode,this is Netty..
23:33:19.679 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoClientHandler - this is counter:4 time receive server:Hi alertcode,this is Netty..
23:33:19.679 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoClientHandler - this is counter:5 time receive server:Hi alertcode,this is Netty..
23:33:19.679 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoClientHandler - this is counter:6 time receive server:Hi alertcode,this is Netty..
23:33:19.679 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoClientHandler - this is counter:7 time receive server:Hi alertcode,this is Netty..
23:33:19.679 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoClientHandler - this is counter:8 time receive server:Hi alertcode,this is Netty..
23:33:19.680 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoClientHandler - this is counter:9 time receive server:Hi alertcode,this is Netty..
23:33:19.682 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoClientHandler - this is counter:10 time receive server:Hi alertcode,this is Netty..
```

##### 服务端

```
23:33:19.664 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoServerHandler - this is counter:1, times receive client:Hi alertcode,this is Netty..
23:33:19.668 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoServerHandler - this is counter:2, times receive client:Hi alertcode,this is Netty..
23:33:19.669 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoServerHandler - this is counter:3, times receive client:Hi alertcode,this is Netty..
23:33:19.669 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoServerHandler - this is counter:4, times receive client:Hi alertcode,this is Netty..
23:33:19.670 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoServerHandler - this is counter:5, times receive client:Hi alertcode,this is Netty..
23:33:19.670 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoServerHandler - this is counter:6, times receive client:Hi alertcode,this is Netty..
23:33:19.671 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoServerHandler - this is counter:7, times receive client:Hi alertcode,this is Netty..
23:33:19.671 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoServerHandler - this is counter:8, times receive client:Hi alertcode,this is Netty..
23:33:19.672 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoServerHandler - this is counter:9, times receive client:Hi alertcode,this is Netty..
23:33:19.681 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.delimiter.EchoServerHandler - this is counter:10, times receive client:Hi alertcode,this is Netty..
```

### FixedLengthFrameDecoder 应用开发

&emsp;&emsp;FixedLengthFrameDecoder 是固定长度解码器，它能够按照指定的长度对消息进行自动解码，开发者不需要考虑 TCP 的粘包、拆包问题，非常实用。下面我们通过一个应用实例将对其用法进行讲解。

#### 服务端代码

```java
package top.alertcode.trainhigh.netty.fixedLength;

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
import io.netty.handler.codec.FixedLengthFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;
import io.netty.handler.logging.LogLevel;
import io.netty.handler.logging.LoggingHandler;
import lombok.extern.slf4j.Slf4j;

/**
 * @author alertcode
 * @date 2018-10-07
 * @copyright alertcode.top
 */
public class EchoServer {

  public static void main(String[] args) {
    new EchoServer().bind(8080);
  }

  void bind(int port) {
    NioEventLoopGroup worker = new NioEventLoopGroup();
    NioEventLoopGroup base = new NioEventLoopGroup();
    try {
      ServerBootstrap b = new ServerBootstrap();
      b.group(base, worker).channel(NioServerSocketChannel.class).option(ChannelOption.TCP_NODELAY, true)
          .handler(new LoggingHandler(LogLevel.INFO)).childHandler(new ChannelInitializer<SocketChannel>() {
        @Override
        protected void initChannel(SocketChannel socketChannel) throws Exception {
          socketChannel.pipeline().addLast(new FixedLengthFrameDecoder(20))
              .addLast(new StringDecoder()).addLast(new EchoServerHandler());
        }
      });

      ChannelFuture f = b.bind(port).sync();
      f.channel().closeFuture().sync();
    } catch (InterruptedException e) {
      e.printStackTrace();
      worker.shutdownGracefully();
      base.shutdownGracefully();
    }
  }
}

@Slf4j
class EchoServerHandler extends ChannelInboundHandlerAdapter {


  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    String body = (String) msg;
    log.info("this is server,receive client:{}", body);
    ByteBuf byteBuf = Unpooled.copiedBuffer(body.getBytes());
    ctx.writeAndFlush(byteBuf);

  }

  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    ctx.close();
  }
}

```

#### 利用 telnet 命令行测试 EchoServer 服务器

&emsp;&emsp;测试场景：开启 win10 下 telnet 客户端，使用 telnet 命令行连接服务端，在控制台输入如下内容。
this is message.
![telnet模拟客户端发送消息](360截图1672040290138123.png)
注：telnet `ctrl+]`开启回显功能，如图片所示。

- 服务端结果：

```
23:47:32.645 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.fixedLength.EchoServerHandler - this is server,receive client:this is message.this
23:47:51.334 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.fixedLength.EchoServerHandler - this is server,receive client: is message.this is
```

&emsp;&emsp;因为`this is message.`消息未达到 20 字节，不进行发包，待消息长度满足 20 字节的要求后，才进行发包，所以有如上所示的日志。

### 总结：

&emsp;&emsp;DelimiterBasedFrameDecoder 用于对使用分割符结尾的消息进行自动解码，FixedLengthFrameDecoder 用于对固定长度的消息进行自动解码。有了上述两种解码器再结合其他的解码器，如字符串解码器等，可以轻松的完成的很多消息的自动解码，而且不需要考虑 TCP 粘包/拆拆包导致的读半包问题，极大的提升了开发效率。
