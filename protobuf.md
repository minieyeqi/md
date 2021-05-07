## protobuf(Protocol Buffers) 入门文档
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
* varints编码
    >1. 每个字节使用其中7位保存数字，最高位表示后面是否还有内容；
    >2. 低位在前，高位在后；
    >3. 保留 fixed32 和 fixed64，用于传递大整数；
    >4. int32, int64, uint32, unit64, bool，序列化结果相互兼容，可以修改；
* zigzag编码
    >1. 传统上，负数最高位为1，小负数会浪费编码长度;
    >2. (n << 1)^(n >> 31);
    >3. -1 将会被编码成 1，1 将会被编码成 2，-2 会被编码成 3；
    >4. sint32 和 sint64 使用 zigzag 编码

## 怎么用
> 参考 protobuf 官网教程 https://developers.google.com/protocol-buffers/ 结合自己理解，分为以下：
> * Proto3 的语法指南
> * Python API
> * C++ API

### Proto3 的语法指南
> * 1.定义消息类型
> * 2.标量值类型
> * 3.可选和默认值
> * 4.枚举
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

message SearchRequest
{
    required string query = 1;
    optional int32 page_number = 2;
    optional int32 result = 3;
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
 
 请注意，范围为1到15的字段号需要一个字节来编码，包括字段号和字段的类型
 >（您可以在Protocol Buffer Encoding中找到有关此内容的更多信息）。

 #### 1.3 指定字段规则
 字段规则：
 * `required`:格式正确的消息必须至少有一个。
 * `optional`:格式正确的消息可以包含零个或多个。
 * `repeated`:格式正确的消息可以包含零个或多个。
 
 | 字段规则 | 字段类型 | 字段名称 | 字段编号 |
 | :-: | :-: | :-: | :-: |
 | `required` | `string` | `query`| `1` |
 | `optional` | `int32` | `page_name` | `2` |
 | `repeated` | `int32` | `result` | `3` |

```
repeated int32 samples =4 [packed=true] 
 ```

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


### 3. 可选和默认值

### 4.枚举