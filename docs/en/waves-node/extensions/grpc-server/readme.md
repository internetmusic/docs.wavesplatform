# gRPC Server

**gRPC Server** is a [node extension](/en/waves-node/extensions)  that allows running [gRPC](https://en.wikipedia.org/wiki/GRPC) services on a [node](/en/blockchain/node).

gRPC services provide information about:

* [accounts](/en/blockchain/account)
* [blockchain](/en/blockchain/blockchain)
* [blocks](/en/blockchain/block)
* [tokens](/en/blockchain/token)
* [transactions](/en/blockchain/transaction)

## Client generation

The clients [generated](https://grpc.io/docs/tutorials/) from [.proto files](https://github.com/wavesplatform/protobuf-schemas) are used to connect to gRPC services.

Examples of usage of gRPC clients generated from .proto files:

* [Connecting to transactions service in Java](https://github.com/wavesplatform/WavesJ/blob/master/examples/src/main/java/GRPCTest.java)
* [Retrieving blocks in C#](https://github.com/wavesplatform/WavesCS/blob/master/WavesCSTests/ProtobufTest.cs)
