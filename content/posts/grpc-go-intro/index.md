+++
title = "gRPC-Go 使用指南"
description = '本文简述了 gRPC 的基本概念及 gRPC-Go 环境的搭建方法，并提供了 gRPC-Go 的示例程序，包括服务端与客户端的编写及测试步骤。'
date = 2021-10-04
[taxonomies]
tags= ["gRPC", "Golang"]
+++

### 1 RPC 框架原理

![gRPC](grpc.png)

<font color=red>目的：让服务调用更加的简单和透明。</font>
方法：由 RPC 框架负责屏蔽底层的传输方式（TCP 或者 UDP）、序列化方式（XML/Json/ 二进制）和通信细节。服务调用者可以像调用本地接口一样调用远程的服务提供者，而不需要关心底层通信细节和调用过程。通过近一段时间的使用体验可以发现相比于传统的 HTTP 服务的开发和使用确实要方便不少，只需要<font color=red>ip+port</font>就可以调用另外一个服务。

### 2 gRPC 框架

gRPC 是一个高性能、开源和通用的 RPC 框架，面向服务端和移动端，基于 HTTP/2 设计。

#### 2.1 gRPC 特点

1. 语言中立，支持多种语言；
2. 基于 IDL 文件定义服务，通过 proto3 工具生成指定语言的数据结构、服务端接口以及客户端 Stub；
3. 通信协议基于标准的 HTTP/2 设计，支持双向流、消息头压缩、单 TCP 的多路复用、服务端推送等特性，这些特性使得 gRPC 在移动端设备上更加省电和节省网络流量；
4. 序列化支持 PB（Protocol Buffer）和 JSON，PB 是一种语言无关的高性能序列化框架，基于 HTTP/2 + PB, 保障了 RPC 调用的高性能。

#### 2.2 gRPC 服务创建示例

<font color=red>以 golang 语言为例进行演示</font>

1. 安装 golang

```bash
wget https://golang.org/dl/go1.17.linux-amd64.tar.gz
sha256sum go1.17.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.17.linux-amd64.tar.gz
# 修改环境变量, 即在~/.bashrc文件末尾添加如下语句即可：
export PATH=$PATH:/usr/local/go/bin
export GO111MODULE=on
export GOPROXY=https://mirrors.tencent.com/go/,goproxy.cn,direct
export GOSUMDB=off
```

2. 安装依赖

- 安装 protoc,保证版本大于 3.0
  源码安装：
  执行此操作前，先确保安装了相应的工具链，如：make、autoconf、aclocal。

```bash
git clone https://github.com/protocolbuffers/protobuf
# v3.6.0+以上版本支持map解析，syntax=2、3消息序列化后是二进制兼容的，用root执行以下命令
cd protobuf
git checkout v3.6.1.3
./autogen.sh
./configure
make -j8
make install
```

- 安装 gprc protocol 插件

```bash
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.1
```

- 配置环境变量

```bash
追加配置  `export PATH=$PATH:$GOPATH/bin`  至 `~/.bashrc`
```

3. 编写 pb 文件 helloworld.proto

```bash
syntax = "proto3";

option go_package = "github.com/botao/helloworld";

package helloworld;

// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}

  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

4. 编写自动生成框架代码脚本

```bash
#!/bin/bash
cd $(dirname $0) || exit 1
mkdir -p v1

protoc --go_out ./v1 --go_opt paths=source_relative \
       --go-grpc_out ./v1 --go-grpc_opt paths=source_relative helloworld.proto

# 当要使用grpc-gateway特性时可采取下列模版生成代码
# 替换成对应的目录，对应内容：https://github.com/googleapis/googleapis，https://github.com/protocolbuffers/protobuf
# PB_INCLUDE="-I/root/go/src/github.com/googleapis/ -I/root/go/src/github.com/protocolbuffers/protobuf/src"

# protoc -I . $PB_INCLUDE \
#     --go_out ./v1 --go_opt paths=source_relative \
#     --go-grpc_out ./v1 --go-grpc_opt paths=source_relative helloworld.proto

# 当要使用grpc-gateway特性时使用
# protoc -I . $PB_INCLUDE \
#     --grpc-gateway_out ./v1 \
#     --grpc-gateway_opt logtostderr=true \
#     --grpc-gateway_opt paths=source_relative \
#     --grpc-gateway_opt generate_unbound_methods=true helloworld.proto
```

5. 编写服务端代码

```go
package main

import (
	"context"
	"log"
	"net"

	pb "github.com/botao/helloworld/proto/v1"
	"google.golang.org/grpc"
)

const (
	port = ":50051"
)

// server is used to implement helloworld.GreeterServer.
type server struct {
	pb.UnimplementedGreeterServer
}

// SayHello implements helloworld.GreeterServer
func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	log.Printf("Received: %v", in.GetName())
	return &pb.HelloReply{Message: "Hello " + in.GetName()}, nil
}

func (s *server) SayHelloAgain(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
	log.Printf("Received: %v", in.GetName())
	return &pb.HelloReply{Message: "Hello again" + in.GetName()}, nil
}

func main() {
	lis, err := net.Listen("tcp", port)
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	s := grpc.NewServer()
	pb.RegisterGreeterServer(s, &server{})
	log.Printf("server listening at %v", lis.Addr())
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

6. 编写客户端代码

```go
package main

import (
    "context"
    "log"
    "os"
    "time"

    "google.golang.org/grpc"
    pb "github.com/botao/helloworld/proto/v1"
)

const (
    address     = "localhost:50051"
    defaultName = "world"
)

func main() {
    // Set up a connection to the server.
    conn, err := grpc.Dial(address, grpc.WithInsecure(), grpc.WithBlock())
    if err != nil {
        log.Fatalf("did not connect: %v", err)
    }
    defer conn.Close()
    c := pb.NewGreeterClient(conn)

    // Contact the server and print out its response.
    name := defaultName
    if len(os.Args) > 1 {
        name = os.Args[1]
    }
    ctx, cancel := context.WithTimeout(context.Background(), time.Second)
    defer cancel()
    r, err := c.SayHello(ctx, &amp;pb.HelloRequest{Name: name})
    if err != nil {
        log.Fatalf("could not greet: %v", err)
    }
    log.Printf("Greeting: %s", r.GetMessage())

    r, err = c.SayHelloAgain(ctx, &amp;pb.HelloRequest{Name: name})
    if err != nil {
        log.Fatalf("could not greet: %v", err)
    }
    log.Printf("Greeting: %s", r.GetMessage())
}
```

7. 执行测试
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9fd702d41b034d7a0af6a1346a8ff514.png)
8. 代码结构
   ![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/06707b392a19bac4166bdfcb06609a4f.png)
