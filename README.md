# go-igtimi

Example server for the Igtimi riot protocol. Can be used as the base for a custom server implementation.
See ./cmd/riot-server/main.go for the entry point.


## Tools
- buf cli - See https://buf.build/docs/installation
- protoc-gen-go - See https://grpc.io/docs/languages/go/quickstart/

## Building

```bash
buf generate
CGO_ENABLED=0 go build ./cmd/riot-server
```

## Running
Example usage:
```bash
./riot-server -riotAddress :6000 -webAddress :8080
```

The server needs to be run on a publicly reachable IPv4 address, e.g. using a VPS or a port forward.

### Device Configuration
Yachtbot/Windbot devices need to run at least 2.2.540 to be able to connect to a custom server.

Add the following to the device configuration:
```
riot server_address <address> <port>
```
The address can be an IPv4 address or a domain name. The port is the port the server is running on.


### Security
Make sure not to expose the example web interface to the public internet without authentication, as it will allow anyone to send commands to the devices. The web interface listen address can be set using the `-webAddress` flag. Per default it listens on Port 8080.
You can use the reverse proxy of your choice to add authentication to the web interface.

The server currently does not authenticate devices. It is recommended to add token based authentication to your solution for production purposes to avoid third parties sending data to your server.


## Protocol
The following is a short summary of the most important parts of the protocol. For an in-depth description see [Riot Protocol Documentation](https://support.yacht-bot.com/YachtBot%20Products/Riot%20Protocol/).

The devices send messages according to the riot protocol, as defined in [/proto/com/igtimi](/proto/com/igtimi/). The protocol is based on the [protobuf](https://developers.google.com/protocol-buffers) serialization format.

Devices connect to the server via a TCP connection (usually on port 6000). The messages are sent in a length-prefixed format, where each message is preceded by the length of the message in bytes encoded as a [protobuf Varint](https://protobuf.dev/programming-guides/encoding/#varints).
All messages sent from/to the device must adhere to [*com.igtimi.Msg*](./proto/com/igtimi/IgtimiStream.proto).

### Acknowledgments
Devices send all data messages to the server without prompting. No acknowledgement is needed from the server.
The server must send a heartbeat message to the devices every 15 seconds. The heartbeat submessage is defined in the protocol as part of [*com.igtimi.ChannelManagement*](./proto/com/igtimi/IgtimiStream.proto).
If the device does not receive a heartbeat message within 30 seconds, it will disconnect and attempt to reconnect.

### Authentication
The server may authenticate devices based on the so called device group token.
The token is a string that is sent initially after connection establishment using the [*com.igtimi.Authentication*](/proto/com/igtimi/IgtimiStream.proto) message. 

### Commands
The server may send commands to the devices using the [*com.igtimi.DeviceCommand*](/proto/com/igtimi/IgtimiDevice.proto) message.
