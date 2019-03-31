# grpc-php-to-golang-demo

#参考文档
    https://www.liangzl.com/get-article-detail-96591.html
#安装protobuf
    wget https://github.com/protocolbuffers/protobuf/releases/download/v3.6.1/protoc-3.6.1-osx-x86_64.zip
    查看版本 ：
    tar -zxvf protoc-3.6.1-osx-x86_64.zip
    cd protoc-3.6.1-osx-x86_6/bin  
    ./protoc --version
#Golang grpc
    go get google.golang.org/grpc
    git clone https://github.com/grpc/grpc-go.git $GOPATH/src/google.golang.org/grpc
    git clone https://github.com/golang/net.git $GOPATH/src/golang.org/x/net
    git clone https://github.com/golang/text.git $GOPATH/src/golang.org/x/text
    git clone https://github.com/golang/sys.git $GOPATH/src/golang.org/x/sys
    go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
    git clone https://github.com/google/go-genproto.git $GOPATH/src/google.golang.org/genproto
    cd $GOPATH/src/
    go install google.golang.org/grpc
#应用
    cd $GOPATH/src
    mkdir -p grpc-php-to-golang-demo/protobuf
    cd grpc-php-to-golang-demo/protobuf
    vim helloworld.proto
    代码：
        syntax = "proto3";
        
        option java_multiple_files = true;
        option java_package = "io.grpc.examples.helloworld";
        option java_outer_classname = "HelloWorldProto";
        option objc_class_prefix = "HLW";
        
        package helloworld;
        
        // The greeting service definition.
        service Greeter {
          // Sends a greeting
          rpc SayHello (HelloRequest) returns (HelloReply) {}
        }
        
        // The request message containing the user's name.
        message HelloRequest {
          string name = 1;
        }
        
        // The response message containing the greetings
        message HelloReply {
          string message = 1;
        }
    
    mkdir -p go-server/helloworld
    protoc --go_out=plugins=grpc:./go-server/helloworld ./helloworld.proto
    cd go-server/helloworld/
    cd $GOPATH/src/grpc-php-to-golang-demo
    mkdir -p golang/holleworld
    cd golang/holleworld
    vim server.go 
    代码
        package main
        
        import (
            "log"
            "net"
        
            pb "grpc-php-to-golang-demo/protobuf/go-server/helloworld"
        
            "google.golang.org/grpc"
            "golang.org/x/net/context"
            "google.golang.org/grpc/reflection"
        )
        
        const (
            port = ":50051"
        )
        
        // server is used to implement helloworld.GreeterServer.
        type server struct{}
        
        // SayHello implements helloworld.GreeterServer
        func (s *server) SayHello(ctx context.Context, in *pb.HelloRequest) (*pb.HelloReply, error) {
            return &pb.HelloReply{Message: "Hello " + in.Name}, nil
        }
        
        func main() {
            lis, err := net.Listen("tcp", port)
            if err != nil {
                log.Fatalf("failed to listen: %v", err)
            }
            s := grpc.NewServer()
            pb.RegisterGreeterServer(s, &server{})
            // Register reflection service on gRPC server.
            reflection.Register(s)
            if err := s.Serve(lis); err != nil {
                log.Fatalf("failed to serve: %v", err)
            }
        }
        
    vim client.go
    代码
        package main
        
        import (
            "log"
            "os"
            "time"
        
            pb "grpc-php-to-golang-demo/protobuf/go-server/helloworld"
        
            "google.golang.org/grpc"
            "golang.org/x/net/context"
        )
        
        const (
            address     = "localhost:50051"
            defaultName = "world"
        )
        
        func main() {
            // Set up a connection to the server.
            conn, err := grpc.Dial(address, grpc.WithInsecure())
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
        
            go func() {
                r, err := c.SayHello(context.Background(), &pb.HelloRequest{Name: name})
                if err != nil {
                    log.Fatalf("could not greet: %v", err)
                }
                log.Printf("Greeting: %s", r.Message)
            }()
        
            time.Sleep(10 * time.Second)
        
        }
#运行测试
    go build server.go
    go build client.go
    ./server
    ./client
   