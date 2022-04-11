[proto3 语法](https://developers.google.com/protocol-buffers/docs/proto3)

# message

message.proto：

```protobuf
syntax = "proto3"; // 声明当前使用的语法为 proto3，不写则 protoc 默认使用 proto2 语法，proto 文件的第一行必须是非空、非注释行！

// 
message SearchRequest{
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

