---
title: Netty（四）：解编码技术
date: 2018-10-12 00:51:24
tags: ["Netty"]

categories: ["Netty"]
---

&emsp;&emsp;基于 java 提供的对象输入/输出流 ObjectInputStream 和 ObjectOutputStream,可以直接把 Java 对象作为可存储的字节数组写入文件，也可以传输到网络上。对程序员来说，基于 JDK 默认的序列化机制可以避免操作底层的字节数组，从而提升开发效率。
java 序列化的目的主要有两个：

- 网络传输
- 对象持久化
  &emsp;&emsp;当进行远程跨进程服务调用时，需要把被传输的 Java 对象编码为字节数组或者 ByteBuffer 对象。而当远程服务读取到 ByteBuffer 对象或者字节数组时，需要将其解码为发送时的 Java 对象。这被称为 Java 对象解编码技术。
  <!--more-->

### Java 序列化的缺点

- 无法跨语言
- 序列化后的码流太大
- 序列化性能太低

### 业界主流的解编码框架

专门针对 Java 语言的：Kryo，FST 等等,跨语言的：Protostuff，ProtoBuf，Thrift，Avro，MsgPack 等等

- Google 的 Protobuf
  &emsp;&emsp;将数据结构以.proto 文件进行描述，通过代码生成工具可以生成对应数据结构的 POJO 对象和 Protobuf 相关的方法和属性。特点：
- 结构化数据存储格式（XML,JSON 等）
- 高效的解编码性能
- 语言无关、平台无关、拓展性能好
- 官方支持 Java、C++和 Python 三种语言
- Protostuff 在 Protobuf 基础上不用编写.proto 文档来实现序列化
- Facebook 的 Thrift
  &emsp;&emsp;在多种不同的语言之间通信，Thrift 可以作为高性能的通信中间件使用，它支持数据（对象）序列化和多种类型的 RPC 服务。Thrift 适用于静态的数据交换，需要先确定好它的数据结构，当数据结构发生变化时，必须重新编辑 IDL 文件，生成代码和编译，这一点跟其他 IDL 工具相比可以视为是 Thrift 的弱项。Thrift 适用于搭建大姓数据交换及存储的通用工具，对于大型系统中的内部数据传输，相对于 JSON 和 XML 在性能和传输大小上都有明显的优势。
- EsotericSoftware 开源组织的 Kyro
- Apache Avro
  &emsp;&emsp;Hadoop 平台上的多个项目正在使用（或者支持）Avro 作为数据序列化的服务。
