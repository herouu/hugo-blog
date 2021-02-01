---
title: Netty（五）：Google_Protobuf编解码
date: 2018-10-12 16:54:24
tags: ["Netty"]

categories: ["Netty"]
---

&emsp;&emsp;Google 的 Protobuf 在业界非常流行，很多商业项目选择 Protobuf 作为编解码框架。优点:

- 在 Google 内部长期使用，产品成熟度高
- 跨语言、支持多种语言，包括 C++、Java 和 Python
- 编码后消息更小，更加有利于存储和运输
- 编解码性能非常高
- 支持不同协议版本向前兼容
- 支持定义可选和必选字段

&emsp;&emsp;下面是关于 Protobuf 的语法教程：

- [Protocol Buffer Java 官方指南](https://developers.google.com/protocol-buffers/docs/javatutorial)
- [Protocol Buffers 简明教程](https://zhuanlan.zhihu.com/p/25174418)
- [【译】Protobuf 语法指南 ](https://colobu.com/2015/01/07/Protobuf-language-guide/)
  <!--more-->

### Protobuf 开发环境搭建

#### 编写.proto 文件

&emsp;&emsp;使用 idea 的插件 Protobuf support,支持语法高亮，，编写 SubscribeReq.proto 和 SubscriteResp.proto，使用 maven 的方式编译.proto 文件，生成 SubscribeReqProto.java 文件和 SubscribeRespProto.java。

- 参考[Intellij IDEA 中使用 Protobuf 的正确姿势](https://blog.csdn.net/u010674648/article/details/80673208)
  ![maven方式编译.proto文件](20181012213839.jpg)

##### 编写 SubscribeReq.proto

```
syntax = "proto3";
option java_package = "top.alertcode.trainhigh.netty.protobuf";
option java_outer_classname = "SubscribeReqProto";
message SubscribeReq {
  int32 subReqId = 1;
  string userName = 2;
  string productName = 3;
  string address = 4;
}
```

##### 编写 SubscriteResp.proto

```
syntax = "proto3";
option java_package = "top.alertcode.trainhigh.netty.protobuf";
option java_outer_classname = "SubscribeRespProto";
message SubscribeResp {
  int32 subReqId = 1;
  int32 respCode = 2;
  string desc = 3;
}
```

#### Protobuf 编解码开发

```java
package top.alertcode.trainhigh.netty.protobuf;

import com.google.protobuf.InvalidProtocolBufferException;
import top.alertcode.trainhigh.netty.protobuf.SubscribeReqProto.SubscribeReq;

/**
 * @author alertcode
 * @date 2018-10-12
 * @copyright alertcode.top
 */
public class TestSubscribeReqProto {

  public static void main(String[] args) throws InvalidProtocolBufferException {
    SubscribeReq subscribeReq = createSubscribeReq();
    System.out.println(" before encode:" + subscribeReq.toString());
    SubscribeReq decode = decode(encode(subscribeReq));
    System.out.println(" after decode:" + decode.toString());
    System.out.println("Assert:" + decode.equals(subscribeReq));
  }

  private static byte[] encode(SubscribeReqProto.SubscribeReq req) {
    return req.toByteArray();
  }


  private static SubscribeReq decode(byte[] body) throws InvalidProtocolBufferException {
    return SubscribeReqProto.SubscribeReq.parseFrom(body);
  }


  private static SubscribeReqProto.SubscribeReq createSubscribeReq() {
    SubscribeReq.Builder builder = SubscribeReq.newBuilder();
    builder.setSubReqId(1);
    builder.setUserName("alertcode");
    builder.setProductName("book for Netty");
    builder.setAddress("dalian");
    return builder.build();
  }

}
```

#### 运行 Protobuf 例程

```
before encode:subReqId: 1
userName: "alertcode"
productName: "book for Netty"
address: "dalian"

after decode:subReqId: 1
userName: "alertcode"
productName: "book for Netty"
address: "dalian"

Assert:true
```

### Netty 的 Protobuf 服务端开发

```java
package top.alertcode.trainhigh.netty.protobuf;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.protobuf.ProtobufDecoder;
import io.netty.handler.codec.protobuf.ProtobufEncoder;
import io.netty.handler.codec.protobuf.ProtobufVarint32FrameDecoder;
import io.netty.handler.codec.protobuf.ProtobufVarint32LengthFieldPrepender;
import io.netty.handler.logging.LoggingHandler;
import lombok.extern.slf4j.Slf4j;
import top.alertcode.trainhigh.netty.protobuf.SubscribeReqProto.SubscribeReq;
import top.alertcode.trainhigh.netty.protobuf.SubscribeRespProto.SubscribeResp;
import top.alertcode.trainhigh.netty.protobuf.SubscribeRespProto.SubscribeResp.Builder;

/**
 * @author alertcode
 * @date 2018-10-12
 * @copyright alertcode.top
 */
public class SubReqServer {

  void bind(int port) {
    NioEventLoopGroup baseGroup = new NioEventLoopGroup();
    NioEventLoopGroup workGroup = new NioEventLoopGroup();
    try {
      ServerBootstrap b = new ServerBootstrap();
      b.group(baseGroup, workGroup).channel(NioServerSocketChannel.class)
          .option(ChannelOption.SO_BACKLOG, 100)
          .handler(new LoggingHandler()).childHandler(

          new ChannelInitializer<SocketChannel>() {
            @Override
            protected void initChannel(SocketChannel ch) throws Exception {
//              使用protobuf
              ch.pipeline().addLast(new ProtobufVarint32FrameDecoder()).addLast(new ProtobufDecoder(
                  SubscribeReq.getDefaultInstance())).addLast(new ProtobufVarint32LengthFieldPrepender())
                  .addLast(new ProtobufEncoder())
                  .addLast(new SubReqServerHandler());
            }
          });

      ChannelFuture sync = b.bind(port).sync();
      sync.channel().closeFuture().sync();
    } catch (InterruptedException e) {
      workGroup.shutdownGracefully();
      baseGroup.shutdownGracefully();
    }
  }

  public static void main(String[] args) {
    new SubReqServer().bind(8080);
  }
}

@Slf4j
class SubReqServerHandler extends ChannelInboundHandlerAdapter {

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    SubscribeReq req = (SubscribeReq) msg;
    if ("alertcode".equalsIgnoreCase(req.getUserName())) {
      log.info("service accept client subscribe req:{}", req.toString());
      ctx.writeAndFlush(resp(req.getSubReqId()));
    }
    super.channelRead(ctx, msg);
  }


  private SubscribeResp resp(int subReqId) {
    Builder builder = SubscribeResp.newBuilder();
    builder.setSubReqId(subReqId);
    builder.setRespCode(1);
    builder.setDesc("success");
    return builder.build();
  }

  @Override
  public void channelActive(ChannelHandlerContext ctx) throws Exception {
    super.channelActive(ctx);
  }

  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    ctx.close();
  }
}

```

### Netty 的 Protobuf 客户端开发

```java
package top.alertcode.trainhigh.netty.protobuf;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.protobuf.ProtobufDecoder;
import io.netty.handler.codec.protobuf.ProtobufEncoder;
import io.netty.handler.codec.protobuf.ProtobufVarint32FrameDecoder;
import io.netty.handler.codec.protobuf.ProtobufVarint32LengthFieldPrepender;
import lombok.extern.slf4j.Slf4j;
import top.alertcode.trainhigh.netty.protobuf.SubscribeReqProto.SubscribeReq;
import top.alertcode.trainhigh.netty.protobuf.SubscribeReqProto.SubscribeReq.Builder;
import top.alertcode.trainhigh.netty.protobuf.SubscribeRespProto.SubscribeResp;


/**
 * @author alertcode
 * @date 2018-10-12
 * @copyright alertcode.top
 */
public class SubReqClient {

  void connect(String host, int port) {
    NioEventLoopGroup client = new NioEventLoopGroup();
    Bootstrap b = new Bootstrap();
    b.group(client).channel(NioSocketChannel.class).option(ChannelOption.TCP_NODELAY, true).handler(new ChannelInitializer<SocketChannel>() {

      @Override
      protected void initChannel(SocketChannel ch) throws Exception {
//        使用protobuf
        ch.pipeline().addLast(new ProtobufVarint32FrameDecoder()).addLast(new ProtobufDecoder(
            SubscribeResp.getDefaultInstance())).addLast(new ProtobufVarint32LengthFieldPrepender())
            .addLast(new ProtobufEncoder())
            .addLast(new SubReqClientHandler());
      }
    });
    ChannelFuture s = null;
    try {
      s = b.connect(host, port).sync();
      s.channel().closeFuture().sync();
    } catch (InterruptedException e) {
      e.printStackTrace();
      client.shutdownGracefully();
    }
  }

  public static void main(String[] args) {
    new SubReqClient().connect("localhost", 8080);
  }
}

@Slf4j
class SubReqClientHandler extends ChannelInboundHandlerAdapter {

  @Override
  public void channelActive(ChannelHandlerContext ctx) throws Exception {
    for (int i = 0; i < 10; i++) {
      ctx.write(subReq(i));
    }
    ctx.flush();
  }

  private SubscribeReq subReq(int i) {
    Builder builder = SubscribeReq.newBuilder();
    builder.setSubReqId(i);
    builder.setUserName("alertcode");
    builder.setProductName("netty book");
    builder.setAddress("liaoning dalian");
    return builder.build();
  }

  @Override
  public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
    ctx.close();
  }

  @Override
  public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
    ctx.flush();
  }

  @Override
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    log.info("Receive server response:[{}]", msg);
  }
}
```

### 运行结果

#### 服务端

```
...省略8次
21:17:43.348 [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message subReqId: 8
userName: "alertcode"
productName: "netty book"
address: "liaoning dalian"
 that reached at the tail of the pipeline. Please check your pipeline configuration.
21:17:43.350 [nioEventLoopGroup-3-1] DEBUG io.netty.channel.DefaultChannelPipeline - Discarded inbound message subReqId: 9
userName: "alertcode"
productName: "netty book"
address: "liaoning dalian"
 that reached at the tail of the pipeline. Please check your pipeline configuration.
```

#### 客户端

```
...省略8次
21:47:21.555 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.protobuf.SubReqClientHandler - Receive server response:[subReqId: 8
respCode: 1
desc: "success"
]
21:47:21.556 [nioEventLoopGroup-2-1] INFO top.alertcode.trainhigh.netty.protobuf.SubReqClientHandler - Receive server response:[subReqId: 9
respCode: 1
desc: "success"
]
```

### 总结

&emsp;&emsp;ProtobufDecoder 仅仅负责解码，它不支持读半包。因此，在 ProtobufDecoder 前面一定要有能够处理读半包的解码器，有以下三种方式可以选择。

- 使用 Netty 提供的 ProtobufVarint32LineBasedFrameDecoder,它可以处理半包信息
- 继承 Netty 提供的通用半包解码器 LengthFieldBaseFrameDecoder
- 继承 ByteToMessageDecoder 类，自己处理半包信息。
  &emsp;&emsp;如果只是用 ProtobufDecoder 解码器而忽略对半包消息的处理，程序是不能正常工作的。
