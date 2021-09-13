gRPC: the g means Google; so gRPC is Google remote procedure call

gRPC是谷歌基于Protobuf开发的跨语言的开源RPC框架，它是基于HTTP/2.0协议设计。

![image-20201126223925635](http://akatsuke.com/image-20201126223925635.png)

```sh
protoc --go_out=. --go_opt=paths=source_relative  --go-grpc_out=. --go-grpc_opt=paths=source_relative hello.proto
```