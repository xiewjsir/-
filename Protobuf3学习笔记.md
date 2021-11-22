##### 参考文档
- [Protobuf3学习笔记](https://www.jianshu.com/p/ea656dc9b037)
- [官方文档](https://developers.google.com/protocol-buffers/docs/proto3)

##### 一个简单的例子
```
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```
##### 版本号
对于一个pb文件而言，文件首个非空、非注释的行必须注明pb的版本，即syntax = "proto3";，否则默认版本是proto2。

##### Message
一个message类型看上去很像一个Java class，由多个字段组成。每一个字段都由类型、名称组成，位于等号右边的值不是字段默认值，而是数字标签，可以理解为字段身份的标识符，类似于数据库中的主键，不可重复，标识符用于在编译后的二进制消息格式中对字段进行识别，一旦你的pb消息投入使用，字段的标识就不应该再改变。数字标签的范围是[1, 536870911]，其中19000～19999是保留数字。

##### 类型
每个字段的类型（int32,string）都是scalar的类型，和其他语言类型的对比参见[官方文档](https://developers.google.com/protocol-buffers/docs/proto3)。

##### 修饰符
如果一个字段被repeated修饰，则表示它是一个列表类型的字段，如下所示：
```
message SearchRequest {
  repeated string args = 1 // 等价于java中的List<string> args
}
```
如果你希望可以预留一些数字标签或者字段可以使用reserved修饰符：
```
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
  string foo = 3 // 编译报错，因为‘foo’已经被标为保留字段
}
```

##### 默认值
- string类型的默认值是空字符串
- bytes类型的默认值是空字节
- bool类型的默认值是false
- 数字类型的默认值是0
- enum类型的默认值是第一个定义的枚举值
- message类型（对象，如上文的SearchRequest就是message类型）的默认值与 语言相关
- repeated修饰的字段默认值是空列表
> 如果一个字段的值等于默认值（如bool类型的字段设为false），那么它将不会被序列化，这样的设计是为了节省流量。

##### 枚举
每个枚举值有对应的数值，数值不一定是连续的。第一个枚举值的数值必须是0且至少有一个枚举值，否则编译报错。编译后编译器会为你生成对应语言的枚举类
```
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
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
一个数值可以对应多个枚举值，必须标明option allow_alias = true;
```
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
```
可以使用MessageType.EnumType的形式引用定义在其它message类型中的枚举。
>由于编码原因，出于效率考虑，官方不推荐使用负数作为枚举值的数值。

##### 使用其它的message类型
除了上述基本类型，一个字段的类型也可以是其它的message类型：
```
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```
从上面的例子可以看到，一个.proto文件中可以定义多个message。我们也可以引用定义在其它文件中的message：
```
import "myproject/other_protos.proto"; // 这样就可以引用在other_protos.proto文件中定义的message

```
>不能导入不使用的.proto文件。

import还有一种特殊的语法，先看下面的例子：
```
// new.proto
// 原来在old.proto文件中的定义移到这里

// old.proto
import public "new.proto"; // 把引用传递给上层使用方
import "other.proto"; // 引用old.proto本身使用的定义

// client.proto
import "old.proto";
// 此处可以引用old.proto和new.proto中的定义，但不能使用other.proto中的定义
```
从这个例子中可以看到import关键字导入的定义仅在当前文件有效，不能被上层使用方引用（client.proto无法使用other.proto中的定义），而import public关键字导入的定义可以被上层使用方引用（client.proto可以使用new.proto中的定义），import public的功能可以看作是import的超集，在import的功能上还具有传递引用的作用。

##### 嵌套类型
你可以在一个message类型中定义另一个message类型，并且可以一直嵌套下去，类似Java的内部类：
```
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```
可以使用Parent.Type的形式引用嵌套的message：
```
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```
##### Any
Any类型允许包装任意的message类型：
```
import "google/protobuf/any.proto";

message Response {
    google.protobuf.Any data = 1;
}
```
可以通过pack()和unpack()（方法名在不同的语言中可能不同）方法装箱/拆箱，以下是Java的例子：
```
People people = People.newBuilder().setName("proto").setAge(1).build();
// protoc编译后生成的message类
Response r = Response.newBuilder().setData(Any.pack(people)).build();
// 使用Response包装people

System.out.println(r.getData().getTypeUrl());
// type.googleapis.com/example.protobuf.people.People
System.out.println(r.getData().unpack(People.class).getName());
// proto
```
Any对包装的类型会生成一个URL，默认是 type.googleapis.com/packagename.messagename （在Java中可以通过这个特性进行反射操作）。

##### Oneof
如果你有一些字段同时最多只有一个能被设置，可以使用oneof关键字来实现，任何一个字段被设置，其它字段会自动被清空（被设为默认值）：
```
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```
>oneof块中的字段不支持repeated。

##### Maps
pb中也可以使用map类型（官方并不认为是一种类型，此处称之为类型仅便于理解），绝大多数scalar类型都可以作为key，除了浮点型和bytes，枚举型也不能作为key，value可以是除了map以外的任意类型：
```
// map<key_type, value_type> map_field = N;
map<string, Project> projects = 3;
```
>map类型字段不支持repeated，value的顺序是不定的。

map其实是一种语法糖，它等价于以下形式：
```
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```
##### 包
你可以用指定package以避免类型命名冲突：
```
package foo.bar;
message Open { ... }
```
然后可以用类型的全限定名来引用它
```
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```
指定包名后，会对生成的代码产生影响，以Java为例，生成的类会以你指定的package作为包名。

#### JSON映射
pb支持和JSON互相转换。如果一个字段不存在JSON数据中或者为null，那么pb中会被赋为该字段的默认值，反之，如果一个字段在pb中是默认值，那么不会写到JSON数据中以节省空间。

##### 选项
选项不对message的定义产生任何的效果，只会在一些特定的场景中起到作用，下面是一部分例子，完整的选项列表可以前往google/protobuf/descriptor.proto查看（Java语言可以在jar包中找到）：
- option java_package = "com.example.foo"; 编译器为以此作为生成的Java类的包名，如果没有该选项，则会以pb的package作为包名。
- option java_multiple_files = true; 该选项为true时，生成的Java类将是包级别的，否则会在一个包装类中。
- option optimize_for = CODE_SIZE; 该选项会对生成的类产生影响，作用是根据指定的选项对代码进行不同方面的优化。
- int32 old_field = 6 [deprecated=true]; 把字段标为过时的。









