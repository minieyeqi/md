## Protobuf(Protocol Buffers) 入门文档
> * 是什么
> * 关键点技术
> * 怎么用
> * 横向对比

## 是什么
* Protocol buffers 是一种语言中立，平台无关，可扩展的序列化数据的格式，可用于通信协议，数据存储等。
  * 传输协议
  * IDL
    > 接口描述性语言
  * 存储格式
    > pd的压缩比
* 核心竞争力
  * 向前向后兼容性
    > 无序考虑版本，前后版本兼容
  * 多语言自动代码生成
  * 快 & 小

## 关键点技术
![](https://github.com/minieyeqi/md/raw/main/images/protobuf_3%20(1).jpg)
* varints 编码
    > * 每个字节使用其中7位保存数字，最高位表示后面是否还有内容；
    > * 低位在前，高位在后；
    > * 保留 `fixed32` 和 `fixed64`，用于传递大整数；
    > * `int32`, `int64`, `uint32`, `unit64`, `bool`，序列化结果相互兼容，可以修改；
* zigzag 编码
    > * 传统上，负数最高位为1，小负数会浪费编码长度;
    > * `(n << 1)^(n >> 31)`;
    > * -1 将会被编码成 1，1 将会被编码成 2，-2 会被编码成 3；
    > * `sint32` 和 `sint64` 使用 zigzag 编码
* message structure 编码
  ![](https://github.com/minieyeqi/md/raw/main/images/protobuf_4.jpg)
    > * `Tag-Value`编码；
    > * `Tag = (field_number << 3)|wire_type ->varints`;
    > * `wire_type`:0 表示`varints`，1表示固定64，,表示固定32位；
    > * `wire_type`:2 表示`Tag-Length-Value`编码，`Length`使用`varints`；
    > * `string`，`bytes`，`message`嵌套，都采用TLV编码；
* repeated 编码
    > * 第一种方式：重复出现的相同`tag`；
    > * 第二种方式(`packed=true`)，TLVVV...编码；
    > 仅有数字类型才可以使用第二种方法，pd3中默认第二种；非`repeated`情况出现重复`tag`，后面的覆盖前面的，因此`optional`和`repeated`相互兼容；
* protoc 编译器
  > * C++ 编写的 proto 文件编译器；
  > * 支持合资语言的插件，使用进程间通讯传递信息；
  > * 安卓和iOS上有对应的插件支持，自动调用protoc；

## 怎么用
> 参考 proto3 官网教程 https://developers.google.com/protocol-buffers/ 结合自己理解，分为以下：
> * Proto3 的语法指南
> * Python 接口端使用
> * C++ 接口端使用

### Proto3 的语法指南
> 参考 https://developers.google.com/protocol-buffers/docs/proto3
> * 1.定义消息类型
> * 2.标量值类型
> * 3.默认值
> * 4.枚举
> * 5.嵌套类型
#### 1. 定义消息类型
建立 .proto文件（.proto 是用于定义消息类型的文件）；
* 指定字段类型；
* 分配字段编号；
* 指定字段规则；
* 添加更多消息类型
* 添加注释
* 保留字段
 
```
# 举个例子开始，
# 搜索请求消息的格式 - 每个搜索请求都对应一个查询字符串、感兴趣的特定页面以及每页结果数量：

syntax = 'proto3'; // 指定proto3的语法，否则认定proto2

message SearchRequest
{
    string query = 1;
    int32 page_number = 2;
    int32 result = 3;
}

# SearchRequest消息指定了三个字段（类型 名称 = 值队） 
 ```

 ##### 1.1 指定字段类型
 在上面的示例中，所有字段均为标量类型，如下所示：
 | 字段类型 | 字段名称 |
 | :-: | :-: |
 | `string` | `query`|
 | `int32` | `page_name` |
 | `int32` | `result` |
 但是，您也可以为字段指定复合类型，包括枚举和其他消息类型。

 #### 1.2 分配字段编号
 消息定义中的每个字段都有一个唯一的编号。这些数字用于标识消息二进制格式的字段。在上面的示例中，如下所示：
 | 字段类型 | 字段名称 | 字段编号 |
 | :-: | :-: | :-: |
 | `string` | `query`| `1` |
 | `int32` | `page_name` | `2` |
 | `int32` | `result` | `3` |
 
 请注意，范围为1到15的字段号需要一个字节来编码，包括字段号和字段的类型。

 #### 1.3 指定字段规则
 字段规则：
 相比较于proto2 移除了`required`规则，并把`optional`规则改为`singual`。
 * `singual`:格式正确的消息可以包含零个或多个，这是proto3语法的默认字段规则。
 * `repeated`:格式正确的消息可以包含零个或多个，重复值的顺序将保留，在proto3中，默认情况下打包标量数字类型的重复字段。

 | 字段规则 | 字段类型 | 字段名称 | 字段编号 |
 | :-: | :-: | :-: | :-: |
 | `sigual` | `string` | `query`| `1` |
 | `sigual` | `int32` | `page_name` | `2` |
 | `sigual` | `int32` | `result` | `3` |
 
#### 1.4 添加更多消息类型
可以在一个 .proto文件中定义多个消息类型。
 ```
# 在上面例子基础上再定义一个
message SearchRequest
{
    required string query = 1;
    optional int32 page_number = 2;
    optional int32 result = 3;
}
message SearchResponse
{
    ...
}
# 合并消息会导致膨胀。虽然可以在单个.proto文件中定义多种消息类型（例如消息，枚举和服务），
# 但是当在单个文件中定义大量具有不同依赖性的消息时，也可能导致依赖性膨胀。
# 建议每个.proto文件包含尽可能少的消息类型。
 ```
#### 1.5 添加注释
注释格式和C++一样
```
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  required string query = 1;
  optional int32 page_number = 2;  // Which page number do we want?
  optional int32 result_per_page = 3;  // Number of results to return per page.
}
 ```

#### 1.6 保留字段
如果您通过完全删除字段或将其注释掉来更新消息类型，则将来的用户在自己对该类型进行更新时可以重用该字段号。或者以后加载旧版本的旧版本，可能会导致严重的问题.proto，包括数据损坏，隐私错误等。
确保不会发生这种情况的一种方法是，将已删除字段的字段编号（和/或名称，也可能导致JSON序列化的问题）指定为reserved。如果将来有任何用户尝试使用这些字段标识符，协议缓冲区编译器会提示。
```
message Foo
{
    reserved 2, 15, 9 to 11;
    reserved "foo", "bar";
}
 ```

### 2. 标量值类型
字段的标量值类型如下：
![](https://github.com/minieyeqi/md/raw/main/images/protobuf_1.PNG)
![](https://github.com/minieyeqi/md/raw/main/images/protobuf_2.PNG)

### 3. 默认值
解析消息时，如果编码的消息不包含特定`sigual`的元素，则已解析对象中的相应字段将设置为该字段的默认值。这些默认值是特定于类型的：
* 对于字符串`string`，默认值为空字符串。
* 对于字节`bytes`，默认值为空字节。
* 对于布尔值`bool`，默认值为false。
* 对于数字类型，默认值为零。
* 对于枚举，默认值为第一个定义的枚举值，必须为0。
* 对于消息字段，未设置该字段。它的确切值取决于语言。

### 4.枚举
在定义消息类型时，可能希望其字段之一仅具有一个预定义的值列表。
承接上面的例子，添加一个`Corpus`类型字段到每个`SearchRequest`中，可通过枚举的方式；
 ```
syntax = 'proto3'

message SearchRequest
{
    string query = 1;
    int32 page_number = 2;
    int32 result_per_page = 3;
    enum Corpus
    {
        UNIVERSAL = 0；
        WEB = 1;
        IMAGES = 2;
        LOCAL = 3;
        NEWS = 4;
        PRODUCTS = 5;
        VIDEO = 6;
    }
    Corpus corpus = 4;
}
 ```

### 5.嵌套类型
可以在其他消息类型中定义和使用消息类型。
如上面例子中，`Result`消息是在`SearchResponse`消息内部定义的：
```
message SearchResponse
{
    message Result
    {
        string url = 1;
        string title = 2;
        repeated string snippets = 3;
    }
    repeated Result result = 1;
}
 ```
 如果要在父类消息以外的地方重用`Result`消息类型，方式是`_Parent_._Type_`:
 ```
 message SomeOtherMessage
 {
     SearchResponse.Result result = 1;
 }
 ```
 也可以根据需求深度嵌套消息：
 ```
 message Outer // level 0
 {
     message MiddleAA // level 1
     {
         message Inner // level 2
         {
             int64 ival = 1;
             bool booly = 2;
         }
     }
     message MiddleBB // level 1
     {
         message Inner // level 2 
         {
            int32 ival = 1;
            bool booly = 2;
         }
     }
 }
 ```

 ## Python 接口端使用
 > 参考 https://developers.google.com/protocol-buffers/docs/reference/python-generated
 > * 1.编译器调用
### 1. 编译器调用
步骤：
* 编译器下载，安装python的protobuf，和安装pb依赖grpc_tools.protoc工具就是protocol-compiler：
  ```
    # 安装python的protobuf（两种方式）

    # 1.pip3直接下载安装
    sudo pip3 install protobuf
     # 单独安装protocol-compiler
    sudo apt-get install protocol-compiler


    # 2.github下载编译安装 github:https://github.com/google/protobuf/releases 
     # config文件下的编译安装protocol-compiler
    sudo make & make install
    protoc --version
     # 进入python文件夹安装protobuf
    python3 setup.py build
    python3 setup.py test
    python setup.py install --cpp_implementation
  ```
* python编译，建立`.proto`文件 >> `.proto`消息类型完成 >> 建立`python_proto`文件夹 >> 编译`.proto`文件，路径指定`python_proto/` >> 得到`_pd2.py`:
    ```
    protoc --python_out=./proto_python  file.proto
    ``` 

## 横向对比
其他竞品：
* JSON - 自解释，易读；
* Thrift - 自带 rpc 方案，跨平台好；
* MessagePack - 可以没有 IDL，比 JSON 快和小；
* Apache Avro - 性能好， hadoop 生态中成熟；
* FlatBuffers - 无需反序列化；
  
![](https://github.com/minieyeqi/md/raw/main/images/protobuf_5.jpg)
![](https://github.com/minieyeqi/md/raw/main/images/protobuf_6.jpg)