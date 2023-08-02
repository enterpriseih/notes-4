# 示例

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
    PHONE_TYPE_UNSPECIFIED = 0;
    PHONE_TYPE_MOBILE = 1;
    PHONE_TYPE_HOME = 2;
    PHONE_TYPE_WORK = 3;
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



# 字段格式

限定修饰符  数据类型  字段名称  =  字段编码值  [字段默认值]

## 1、限定修饰符

包含 `required` \ `optional` \ `repeated`

-   **required**：表示是一个必须字段
-   **optional**：表示是一个可选字段
-   **repeated**：表示该字段可以包含 0～N 个元素



## 2、数据类型

-   bool	布尔类型	1字节	bool

-   double	64位浮点数	N	double

-   float	32为浮点数	N	float

-   int32	32位整数、	N	int

-   uin32	无符号32位整数	N	unsigned int

-   int64	64位整数	N	__int64

-   uint64	64为无符号整	N	unsigned __int64

-   sint32	32位整数，处理负数效率更高	N	int32

-   sing64	64位整数 处理负数效率更高	N	__int64

-   fixed32	32位无符号整数	4	unsigned int32

-   fixed64	64位无符号整数	8	unsigned __int64

-   sfixed32	32位整数、能以更高的效率处理负数	4	unsigned int32

-   sfixed64	64为整数	8	unsigned __int64

-   string	只能处理 ASCII字符	N	std::string

-   bytes	用于处理多字节的语言字符、如中文	N	std::string

-   enum	可以包含一个用户自定义的枚举类型uint32	N(uint32)	enum

-   message	可以包含一个用户自定义的消息类型	N	object of class

>   N 表示打包的字节并不是固定。而是根据数据的大小或者长度。 例如int32，如果数值比较小，在0~127时，使用一个字节打包。 关于枚举的打包方式和uint32相同。 关于message，类似于C语言中的结构包含另外一个结构作为数据成员一样。 关于 fixed32 和int32的区别。fixed32的打包效率比int32的效率高，但是使用的空间一般比int32多。因此一个属于时间效率高，一个属于空间效率高。根据项目的实际情况，一般选择fixed32，如果遇到对传输数据量要求比较苛刻的环境，可以选择int32.



## 3、字段名称

命名方式与java几乎相同



## 4、字段编码值

通信双方通过编码值互相识别对方的字段。相同的编码值，其限定修饰符和数据类型必须相同。 

可以使用 [1, 19000) 和 (19999, 2^29 - 1] 区间内的任意编号用来标识字段，中间 [19000, 19999] 是为 ProtoBuf 的实现所预留的；编号 1 到 15 需要占用 1 个字节，16 到 2047 需要占用 2 个字节，因此一般会将常用的字段对应到编号 1 到 15上以节约空间；

项目投入运营以后涉及到版本升级时的新增消息字段全部使用optional或者repeated，尽量不使用required。如果使用了required，需要全网统一升级，如果使用optional或者repeated可以平滑升级。



## 5、默认值

当在传递数据时，对于required数据类型，如果用户没有设置值，则使用默认值传递到对端。当接受数据是，对于optional字段，如果没有接收到optional字段，则设置为默认值。

