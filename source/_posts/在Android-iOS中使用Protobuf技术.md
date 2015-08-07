title: 在Android/iOS中使用Protobuf技术
date: 2015-03-25 08:45:42
tags: [Android,iOS]
---

### 前言
在开发中，我们免不了要与服务器进行网络数据交互，而这些数据采用了各种各样的方式，现如今比较流行的是JSON格式，而XML格式因为其解析速度慢、传输数据量较大等性能上的诟病被大家逐渐抛弃了。今天的主角是Protobuf（Protocol Buffers），也是完成此类任务的一个选择。

### 什么是Protobuf

protobuf是google提供的一个开源序列化框架，类似于XML，JSON这样的数据表示语言，其最大的特点是基于二进制，因此比传统的XML表示高效短小得多。虽然是二进制数据格式，但并没有因此变得复杂，开发人员通过按照一定的语法定义结构化的消息格式，然后送给命令行工具，工具将自动生成相关的类，可以支持php、java、c++、python等语言环境（由于开源力量的支持，已经有很多其他的支持语言，比如objc、C#等）。通过将这些类包含在项目中，可以很轻松的调用相关方法来完成业务消息的序列化与反序列化工作。

protobuf在google中是一个比较核心的基础库，作为分布式运算涉及到大量的不同业务消息的传递，如何高效简洁的表示、操作这些业务消息在google这样的大规模应用中是至关重要的。而protobuf这样的库正好是在效率、数据大小、易用性之间取得了很好的平衡。更多详见：[protocol-buffers](https://developers.google.com/protocol-buffers/docs/overview)

### 如何使用Protobuf

首先需要在一个 .proto 文件中定义你需要做串行化的数据结构信息。每个ProtocolBuffer信息是一小段逻辑记录，包含一系列的键值对。这里有个非常简单的 .proto 文件定义了个人信息:

```
message Person {
required string name = 1;
required int32 id = 2;
optional string email = 3;
enum PhoneType {
MOBILE = 0;
HOME = 1;
WORK = 2;
}
message PhoneNumber {
required string number = 1;
optional PhoneType type = 2 [default = HOME];
}
repeated PhoneNumber phone = 4;
}
```

如你所见，消息格式很简单，每个消息类型拥有一个或多个特定的数字字段，每个字段拥有一个名字和一个值类型。值类型可以是数字(整数或浮点)、布尔型、字符串、原始字节或者其他ProtocolBuffer类型，还允许数据结构的分级。你可以指定可选字段，必选字段和重复字段。你可以在[这里](http://code.google.com/apis/protocolbuffers/docs/proto.html)找到更多关于如何编写 .proto 文件的信息。

一旦你定义了自己的报文格式(message)，你就可以运行ProtocolBuffer编译器，将你的 .proto 文件编译成特定语言的类。这些类提供了简单的方法访问每个字段(像是 query() 和 set_query() )，像是访问类的方法一样将结构串行化或反串行化。例如你可以选择C++语言，运行编译如上的协议文件生成类叫做 Person 。随后你就可以在应用中使用这个类来串行化的读取报文信息。你可以这么写代码:

```
Person person;
person.set_name("John Doe");
person.set_id(1234);
person.set_email("jdoe@example.com");
fstream.output("myfile",ios::out | ios::binary);
person.SerializeToOstream(&output);
```

然后，你可以读取报文中的数据:

```
fstream input("myfile",ios::in | ios:binary);
Person person;
person.ParseFromIstream(&input);
cout << "Name: " << person.name() << endl;
cout << "E-mail: " << person.email() << endl;
```

你可以在不影响向后兼容的情况下随意给数据结构增加字段，旧有的数据会忽略新的字段。所以如果使用ProtocolBuffer作为通信协议，你可以无须担心破坏现有代码的情况下扩展协议。


### Protobuf有什么缺陷

- 二进制格式导致可读性差：为了提高性能，protobuf采用了二进制格式进行编码。这直接导致了可读性差的问题（严格地说，是没有可读性）。虽然protobuf提供了TextFormat这个工具类，但终究无法彻底解决此问题。

- 缺乏自描述：一般来说，XML是自描述的，而protobuf格式则不是。给你一段二进制格式的协议内容，如果不配合相应的proto文件，那简直就像天书一般。

- 其他的关于Protobuf:

1、 从某种意义上讲，可以把proto文件看成是描述通讯协议的规格说明书（或者叫接口规范）。我们能从中看到AIDL的影子，它们的思想是相通的。

2、关于“向后兼容”和“向前兼容”：

为了叙述方便，我们把增加了“状态”属性的协议成为“新版本”；之前的叫“老版本”。

所谓的“向后兼容”（backward compatible），就是说，当模块B升级了之后，它能够正确识别模块A发出的老版本的协议。由于老版本没有“状态”这个属性，在扩充协议时，可以考虑把“状态”属性设置成非必填的，或者给“状态”属性设置一个缺省值。

所谓的“向前兼容”（forward compatible），就是说，当模块A升级了之后，模块B能够正常识别模块A发出的新版本的协议。这时候，新增加的“状态”属性会被忽略。

“向后兼容”和“向前兼容”有什么用处呢？举个例子：当你维护一个很庞大的分布式系统时，由于你无法同时升级所有模块，为了保证在升级过程中，整个系统能够尽可能不受影响，就需要尽量保证通讯协议的“向后兼容”或“向前兼容”。

3、跟Protobuf类似的还有thrift，thrift提供了全套RPC解决方案，包括序列化机制、传输层、并发处理框架等，支持的语言更多，功能更丰富，但是学习成本较高。