# 【附录】Protocol Buffer

[TOC]

## 简介

Protocol Buffers 是 google 的一种数据交换的格式，它独立于语言，独立于平台。google 提供了多种语言的实现：java、c#、c++、go 和 python，每一种实现都包含了相应语言的编译器以及库文件。由于它是一种二进制的格式，比使用 xml 进行数据交换快许多。可以把它用于分布式应用之间的数据通信或者异构环境下的数据交换。作为一种效率和兼容性都很优秀的二进制数据传输格式，可以用于诸如网络传输、配置文件、数据存储等诸多领域。



## 定义消息类型

```protobuf
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

- 第一行`syntax`表明协议版本，若不指定编译器则假定使用[proto2](https://developers.google.com/protocol-buffers/docs/proto)，这必须是文件的第一个非空的非注释行。
- `SearchRequest` 定义了三个字段 (name/value pairs)，每个字段都有一个名称和类型。

### 指定字段类型

在上面的示例中，所有字段都是[标量类型](###标量值类型)：两个整数（page_number和result_per_page）和一个字符串（query）。但是，您还可以为字段指定其他类型，包括枚举和其他消息类型。

### 分配字段标识号

正如上述示例，每个字段都有一个**唯一数字标识**。这些数字在**二进制编码**中用于标识字段，一旦使用就不应该更改这些字段编号。注意，1到15范围内的字段编号需要一个字节来编码，包括字段编号和字段的类型(您可以在协议缓冲区编码中了解更多)。16到2047之间的字段数占用两个字节。因此，您应该为非常频繁出现的消息元素保留数字1到15。记住要为将来可能添加的经常出现的元素留一些空间。

最小的标识号可以从1开始，最大到2^29 - 1, or 536,870,911。不可以使用其中的[19000－19999]的标识号， Protobuf协议实现中对这些进行了预留。如果在.proto文件中使用了这些预留标识号，编译时就会报错。

### 指定字段规则

消息字段可以是以下之一：

- 单数：格式良好的消息可以包含该字段中的零个或一个（但不超过一个）。
- `repeated`：此字段可以在格式良好的消息中重复任意次数（包括零）。将保留重复值的顺序。

在proto3中，`repeated`数字类型的字段默认使用`packed`编码。

`packed`您可以在[协议缓冲区编码中](https://developers.google.com/protocol-buffers/docs/encoding.html#packed)找到有关编码的更多信息。

### 多个消息类型

可以在单个`.proto`文件中定义多种消息类型。

```protobuf
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
 ...
}
```

### 添加注释

要为`.proto`文件添加注释，请使用C / C ++ - 样式`//`和`/* ... */`语法。

```protobuf
/* SearchRequest represents a search query, with pagination options to
 * indicate which results to include in the response. */

message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
```

### 保留字段

当你在某次更新消息中屏蔽或者删除了一个字段的话，未来的使用着可能在他们的更新中重用这个字段标识号来标记他们自己的字段。然后当他们加载旧的消息的时候就会出现很多问题，包括数据冲突，隐藏的bug等等。指定这个字段的标签数字（或者名字，名字可能在序列化为JSON的时候可能冲突）标记为`reserved`来保证他们不会再次被使用。如果以后的人试用的话protobuf编译器会提示出错。

```protobuf
message Foo {
  reserved 2, 15, 9 to 11; // 保留字段标识号
  reserved "foo", "bar"; // 保留字段名
}
```

注意一个reserved字段不能既有标签数字又有名字。

### 标量值类型

标量消息字段可以具有以下类型之一 - 该表显示`.proto`文件中指定的类型，以及自动生成的类中的相应类型：

| .protoType | Notes                                                        | Java/Kotlin Type[1] | Python Type[3] | Go Type | Ruby Type                      | C++ Type | C# Type    | PHP Type          | Dart Type |
| ---------- | :----------------------------------------------------------- | ------------------- | -------------- | ------- | ------------------------------ | -------- | ---------- | ----------------- | --------- |
| double     |                                                              | double              | float          | float64 | Float                          | double   | double     | float             | double    |
| float      |                                                              | float               | float          | float32 | Float                          | float    | float      | float             | double    |
| int32      | 使用变长编码。对负数的编码效率很低——如果你的字段可能有负值，使用sint32代替。 | int                 | int            | int32   | Fixnum or Bignum (as required) | int32    | int        | integer           | int       |
| int64      | 使用变长编码。对负数的编码效率很低——如果你的字段可能有负数，使用sin64代替。 | long                | int/long[4]    | int64   | Bignum                         | int64    | long       | integer/string[6] | Int64     |
| uint32     | 使用变长编码。                                               | int[2]              | int/long[4]    | uint32  | Fixnum or Bignum (as required) | uint32   | uint       | integer           | int       |
| uint64     | 使用变长编码。                                               | long[2]             | int/long[4]    | uint64  | Bignum                         | uint64   | ulong      | integer/string[6] | Int64     |
| sint32     | 使用变长编码。签署的int值。与常规的int32相比，它们对负数的编码效率更高。 | int                 | int            | int32   | Fixnum or Bignum (as required) | int32    | int        | integer           | int       |
| sint64     | 使用变长编码。签署的int值。与常规的int64相比，它们对负数的编码效率更高。 | long                | int/long[4]    | int64   | Bignum                         | int64    | long       | integer/string[6] | Int64     |
| fixed32    | 总是四个字节。如果值通常大于2^28，则比uint32更有效。         | int[2]              | int/long[4]    | uint32  | Fixnum or Bignum (as required) | uint32   | uint       | integer           | int       |
| fixed64    | 总是8个字节。如果值经常大于2^56，则比uint64更有效。          | long[2]             | int/long[4]    | uint64  | Bignum                         | uint64   | ulong      | integer/string[6] | Int64     |
| sfixed32   | 总是4字节。                                                  | int                 | int            | int32   | Fixnum or Bignum (as required) | int32    | int        | integer           | int       |
| sfixed64   | 总是8个字节。                                                | long                | int/long[4]    | int64   | Bignum                         | int64    | long       | integer/string[6] | Int64     |
| bool       |                                                              | boolean             | bool           | bool    | TrueClass/FalseClass           | bool     | bool       | boolean           | bool      |
| string     | 字符串必须包含UTF-8编码或7位ASCII文本，且长度不能超过2^32。  | String              | str/unicode[5] | string  | String (UTF-8)                 | string   | string     | string            | String    |
| bytes      | 可以包含不超过2^32的任意字节序列。                           | ByteString          | str            | []byte  | String (ASCII-8BIT)            | string   | ByteString | string            | List      |

### 默认值

解析消息时，如果编码消息不包含特定的单数元素，则解析对象中的相应字段将设置为该字段的默认值。这些默认值是特定于类型的：

- 对于字符串，默认值为空字符串。
- 对于字节，默认值为空字节。
- 对于bools，默认值为false。
- 对于数字类型，默认值为零。
- 对于[枚举](https://developers.google.com/protocol-buffers/docs/proto3#enum)，默认值是第**一个定义的枚举值**，该**值**必须为0。
- 对于消息字段，未设置该字段。它的确切值取决于语言。有关详细信息， 请参阅[生成的代码指](https://developers.google.com/protocol-buffers/docs/reference/overview)

## 拓展：Protocol Buffer3 - Java

### 简单示例

```protobuf
syntax = "proto2";

package tutorial;

option java_multiple_files = true;
option java_package = "com.example.tutorial.protos";
option java_outer_classname = "AddressBookProtos";

message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    optional string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

.proto文件以声明语法版本开始，接下来声明了`package`这有助于防止不同项目之间的命名冲突。即使您已经提供了一个`java_package`，您仍然应该定义一个`package`，以避免在Protocol Buffers名称空间中以及在**非java语言**中发生名称冲突。



在声明包之后，可以看见三个特定于java的选项:`java_multiple_files`、`java_package`和`java_outer_classname`。

- `java_package`指定生成的类应该在哪个Java包中存在。如果不显式指定，它只匹配`package`声明出的包名，但这些名称通常不适合Java包名(因为它们通常不以域名开头)。

- `java_outer_classname`选项定义了将代表此文件的包装器类的类名。如果您没有显式地给出`java_outer_classname`，那么它将通过将文件名转换为大驼峰来生成。例如：“my_proto.proto” -》 MyProto。
- `java_multiple_files` = true选项允许为每个生成的类生成单独的.java文件，而不是生成单个.java文件。(使用包装器类作为外部类，并将所有其他类嵌套在包装器类中)。

