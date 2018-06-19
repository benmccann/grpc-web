## Overview

gRPC-Web provides a Javascript client library that lets browser clients
access a gRPC server. You can find out much more about gRPC in its own
[website](https://grpc.io).

The current release is a Beta release, and we expect to announce General-Available by Oct. 2018. 

The JS client library has been
used for some time by Google and Alphabet projects with the
[Closure compiler](https://github.com/google/closure-compiler)
and its TypeScript generator (which has not yet been open-sourced).

gRPC-Web clients connect to gRPC servers via a special gateway proxy: our
provided version uses Nginx. 

We have also added built-in gRPC-Web support to
[Envoy](https://github.com/lyft/envoy), which will become the default gateway for gRPC-Web by GA. 

In future, we expect gRPC-Web to be supported in
language-specific Web frameworks, such as Python, Java, and Node. See the
[roadmap](https://github.com/grpc/grpc-web/blob/master/ROADMAP.md) doc.

Protocol Buffers compared to JSON:

* Use less data to communicate same information
* Generate native classes for both client and server (so structure and types are built-in)
* Offer the ability to rename fields and change compatible types
* Are slower to parse

## Quick start

Try gRPC-Web and run a quick Echo example from the browser!

From the repo root directory:

```sh
$ docker build -t grpc-web --build-arg with_examples=true \
  -f net/grpc/gateway/docker/ubuntu_16_04/Dockerfile .
$ docker run -t -p 8080:8080 grpc-web
```

Open a browser tab, and inspect

```
http://localhost:8080/net/grpc/gateway/examples/echo/echotest.html
```

## How it works

Let's take a look at how gRPC-Web works with a simple example. You can find out
how to build, run and explore the example yourself in
[Build and Run the Echo Example](net/grpc/gateway/examples/echo).

### 1. Define your service

The first step when creating any gRPC service is to define it. Like all gRPC
services, gRPC-Web uses [protocol buffers](https://developers.google.com/protocol-buffers/)
to define its RPC service methods and their message request and response types.

```
service EchoService {
  rpc Echo(EchoRequest) returns (EchoResponse);

  rpc ServerStreamingEcho(ServerStreamingEchoRequest)
      returns (stream ServerStreamingEchoResponse);
}
```


### 2. Build the server

Next you need to have a gRPC server that implements the service interface and a
gateway that allows the client to connect to the server. Our example builds a
simple C++ gRPC backend server and the Nginx gateway. You can find out more in
the [Echo Example](net/grpc/gateway/examples/echo).



### 3. Write your JS client

Once the server and gateway are up and running, you can start making gRPC calls
from the browser!

Create your client

```js
var echoService = new proto.grpc.gateway.testing.EchoServiceClient(
  'http://localhost:8080');
```

Make a unary RPC call

```js
var unaryRequest = new proto.grpc.gateway.testing.EchoRequest();
unaryRequest.setMessage(msg);
echoService.echo(unaryRequest, {},
  function(err, response) {
    console.log(response.getMessage());
  });
```

Server-side streaming is supported!

```js
var stream = echoService.serverStreamingEcho(streamRequest, {});
stream.on('data', function(response) {
  console.log(response.getMessage());
});
```
