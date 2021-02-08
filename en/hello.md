# The first service

## Proto

Micro does not oblige to use protobuf. But for better experience in communication between microservices (client - server) strongly recommended to use .proto files. It insures developers on a both sides they are using the same generated .proto structures and services.

Usually .proto files located in the directory of the project. If amount of services which are using .proto file are growing up, the best way to share .proto files is using any of storage (github, gitlab etc.).

File .proto usually contains from three part:
- support - contains imports and package names
- service - contains service names and methods with annotations
- messages - description for sructures, which are using by service

# Example

See code below with .proto file example for better undestanding:

```protobuf
syntax = "proto3"; // syntax of proto file

package github; // service name
option go_package = ".;pb"; // optional, help code generator names packages in a right way

import "google/api/annotations.proto"; // for using http next to grpc we have to define import
import "protoc-gen-openapiv2/options/annotations.proto"; // for using openapi generation (describe our services) we have to define import 

service Github { // service name - will be using by code generator on the client side and server side support functions (create service client, create service server), in Endpoints and openapi description
  rpc LookupUser(LookupUserReq) returns (LookupUserRsp) { // description for method with structures to receive and respond
    option (grpc.gateway.protoc_gen_openapiv2.options.openapiv2_operation) = { // openapi annotaiont
      operation_id: "LookupUser"; // operation name in openapi
      responses: { // type of responses
        key: "default"; // using by any of response type except standart one described in the method
        value: {
          description: "Error response"; // openapi description
          schema: { json_schema: { ref: ".github.Error"; } }; // link to message type, consists with package name and message name
        }
      }
    };
    option (google.api.http) = { get: "/users/{username}"; }; // describes endpoint which should be used connecting to rpc LookupUser via http with method GET and path /users/username. In order to use POST, PUT, PATCH requests also may contain body. Body is defining the same way as path variable, but instead should be using link to message structure. If body is not pre-defined should be used body:'*' declaration.
  };
};

message LookupUserReq { // request description
  string username = 1; // the username field. Name have to be identical to path variable declaration in option google.api.http GET /users/{username}
};

message LookupUserRsp { // response description
  string name = 1; // here define only one field from api.github.com - name of user
};

message Error { // error description
  string message = 1; // message from api.github.com if user not found
};
```

# Code generation

After proto file has been created our next step will be code generation. For those purposes we are using protoc-gen-micro. It generates code base on proto description for server and client sides. The key thing here - imports have to be available. To make this happen we have to install them. The easiest way to do this - using file tools.go like example below:

```go
// +build tools

package main

import (
        _ "github.com/grpc-ecosystem/grpc-gateway/v2/protoc-gen-openapiv2"
)
```

For using this file only in compilation time, we have to define // +build tools and pass the flag tools (we won't do that). The command go build / go get see the import and add the dependency.

Next step - script for code generation:
```shell
#!/bin/sh -e

INC=$(go list -f '{{ .Dir }}' -m github.com/grpc-ecosystem/grpc-gateway/v2)
ARGS="-I${INC} -I${INC}/third_party/googleapis"

protoc $ARGS -Iproto --openapiv2_out=disable_default_errors=true,allow_merge=true:./proto/ --go_out=paths=source_relative:./proto/ --micro_out=components="micro|http",debug=true,paths=source_relative:./proto/ proto/*.proto
```

Here we said we want to generate swagger/openapi specification, go code with structures, micro interfaces and http client, server. All generated code will be stored in proto directory.

# Service

## Client

The code below is describing creation client for Github service

```go
package main

import (
	"context"

	mhttp "github.com/unistack-org/micro-client-http/v3"
	jsoncodec "github.com/unistack-org/micro-codec-json/v3"
	pb "github.com/unistack-org/micro-tests/client/http/proto"
	"github.com/unistack-org/micro/v3/client"
	"github.com/unistack-org/micro/v3/logger"
)

func main() {
	hcli := mhttp.NewClient(client.ContentType("application/json"), client.Codec("application/json", jsoncodec.NewCodec()))
	cli := client.NewClientCallOptions(hcli, client.WithAddress("https://api.github.com"))
	gh := pb.NewGithubService("github", c)

	rsp, err := gh.LookupUser(context.TODO(), &pb.LookupUserReq{Username: "vtolstov"})
	if err != nil {
		logger.Errorf(context.TODO(), err)
	}

	if rsp.Name != "Vasiliy Tolstov" {
		logger.Errorf(context.TODO(), "invalid rsp received: %#+v\n", rsp)
	}
}
```

## Server

```go

```
