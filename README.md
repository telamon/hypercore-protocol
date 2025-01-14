# hypercore-protocol

Stream that implements the [hypercore](https://github.com/mafintosh/hypercore) protocol

```
npm install hypercore-protocol
```

[![build status](https://travis-ci.org/mafintosh/hypercore-protocol.svg?branch=master)](https://travis-ci.org/mafintosh/hypercore-protocol)

For detailed info on the messages sent on each channel see [simple-hypercore-protocol](https://github.com/mafintosh/simple-hypercore-protocol)

Note that the latest version of this is Hypercore Wire Protocol 7, which is not compatible with earlier versions.

## Usage

``` js
const Protocol = require('hypercore-protocol')

// create two streams with hypercore protocol
const streamA = new Protocol(true) // true indicates this is the initiator
const streamB = new Protocol(false) // false indicates this is not the initiator

// open two feeds specified by a 32 byte key
const key = Buffer.from('deadbeefdeadbeefdeadbeefdeadbeef')
const feed = streamA.open(key)
const remoteFeed = streamB.open(key, {
  // listen for data in remote feed
  ondata (message) {
    console.log(message.value.toString())
  }
})

// add data to feed
feed.data({ index: 1, value: '{ block: 42 }'})

streamA.pipe(streamB).pipe(streamA)
```

`output => { block: 42 }`

## API

#### `const stream = new Protocol(initiator, [options])`

Create a new protocol duplex stream.

Options include:

``` js
{
  encrypt: true, // set to false to disable encryption if you are already piping through a encrypted stream
  timeout: 20000, // stream timeout. set to 0 or false to disable.
  keyPair: { publicKey, secretKey }, // use this keypair for the stream authentication
  onauthenticate (remotePublicKey, done) { }, // hook to verify the remotes public key
  onhandshake () { }, // function called when the stream handshake has finished
  ondiscoverykey (discoveryKey) { } // function called when the remote stream opens a feed you have not
}
```

#### `stream.on('discovery-key', discoveryKey)`

Emitted when the remote opens a feed you have not opened.
Also calls `stream.handlers.ondiscoverykey(discoveryKey)`

#### `stream.on('timeout')`

Emitted when the stream times out.
Per default a timeout triggers a destruction of the stream, unless you disable timeout handling in the constructor.

#### `stream.setTimeout(ms, ontimeout)`

Set a stream timeout.

#### `stream.setKeepAlive(ms)`

Send a keep alive ping every ms, if no other message has been sent.
This is enabled per default every timeout / 2 ms unless you disable timeout handling in the constructor.

#### `stream.prefinalize`

A [nanoguard](https://github.com/mafintosh/nanoguard) instance that is used to guard the final closing of the stream.
Internally this guard is ready'ed before the stream checks if all channels have been closed and the stream is finalised.
Call wait/continue on this guard if need asynchrously add more channels and don't want to stream to finalise underneath you.

#### `stream.remotePublicKey`

The remotes public key.

#### `stream.publicKey`

Your public key.

#### `const bool = stream.remoteVerified(key)`

Returns true if the remote sent a valid capability for the key when they opened the channel.
Use this in `ondiscoverykey` to check that the remote has the key corresponding to the discovery key.

#### `const bool = Protocol.isProtocolStream(stream)`

Static method to check if an object is a hypercore protocol stream.

#### `const keyPair = Protocol.keyPair()`

Static method to generate an static authentication key pair.

#### `const channel = stream.open(key, handlers)`

Signal the other end that you want to share a hypercore feed.

The feed key will be hashed and sent as the "discovery key" which protects the feed key from being learned by a remote peer who does not already possess it. Also includes a cryptographic proof that the local possesses the feed key, which can be implicitly verified using the above `remoteVerified` api.

[See the protobuf schema for more info on this messsage](https://github.com/mafintosh/simple-hypercore-protocol/blob/master/schema.proto#L7)

The `handlers` is an object of functions for handling incoming messages and is described below.

#### `stream.close(discoveryKey)`

You can call this method to signal to the other side that you do not have the key
corresponding to the discoveryKey. Normally you would use this together with the `ondiscoverykey` hook.

#### `stream.destroy([error])`

Destroy the stream. Closes all feeds as well.

#### `stream.finalize()`

Gracefully end the stream. Closes all feeds as well.
This is automatically called after the prefinalise guard and all channels have been closed.

#### `feed.options(message)`

Send an `options` message. [See the protobuf schema for more info on this messsage](https://github.com/mafintosh/simple-hypercore-protocol/blob/master/schema.proto#L13)

#### `feed.handlers.onoptions(message)`

Called when a options message has been received.

#### `feed.status(message)`

Send an `status` message. [See the protobuf schema for more info on this messsage](https://github.com/mafintosh/simple-hypercore-protocol/blob/master/schema.proto#L20)

#### `feed.handlers.onstatus(message)`

Called when a status message has been received.

#### `feed.have(message)`

Send a `have` message. [See the protobuf schema for more info on this messsage](https://github.com/mafintosh/simple-hypercore-protocol/blob/master/schema.proto#L26)

#### `feed.handlers.onhave(message)`

Called when a `have` message has been received.

#### `feed.unhave(message)`

Send a `unhave` message. [See the protobuf schema for more info on this messsage](https://github.com/mafintosh/simple-hypercore-protocol/blob/master/schema.proto#L34)


#### `feed.handlers.onunhave(message)`

Called when a `unhave` message has been received.

#### `feed.want(want)`

Send a `want` message. [See the protobuf schema for more info on this messsage](https://github.com/mafintosh/simple-hypercore-protocol/blob/master/schema.proto#L40)

#### `feed.handlers.onwant(want)`

Called when a `want` message has been received.

#### `feed.unwant(unwant)`

Send a `unwant` message. [See the protobuf schema for more info on this messsage](https://github.com/mafintosh/simple-hypercore-protocol/blob/master/schema.proto#L46)

#### `feed.handlers.onunwant(unwant)`

Called when a `unwant` message has been received.

#### `feed.request(request)`

Send a `request` message. [See the protobuf schema for more info on this messsage](https://github.com/mafintosh/simple-hypercore-protocol/blob/master/schema.proto#L52)


#### `feed.handlers.onrequest(request)`

Called when a `request` message has been received.

#### `feed.cancel(cancel)`

Send a `cancel` message. [See the protobuf schema for more info on this messsage](https://github.com/mafintosh/simple-hypercore-protocol/blob/master/schema.proto#L60)

#### `feed.handlers.oncancel(cancel)`

Called when a `cancel` message has been received.

#### `feed.data(data)`

Send a `data` message. [See the protobuf schema for more info on this messsage](https://github.com/mafintosh/simple-hypercore-protocol/blob/master/schema.proto#L67)

#### `feed.handlers.ondata(data)`

Called when a `data` message has been received.

#### `feed.extension(id, buffer)`

Send an `extension` message. `id` should be the index an extension name in the `extensions` list sent in a previous `options` message for this channel.

#### `feed.handlers.onextension(id, buffer)`

Called when an `extension` message has been received. `id` is the index of an extension name received in an extension list in a previous `options` message for this channel.

#### `feed.close()`

Close this feed. You only need to call this if you are sharing a lot of feeds and want to garbage collect some old unused ones.

#### `feed.handlers.onclose()`

Called when this feed has been closed. All feeds are automatically closed when the stream ends or is destroyed.

#### `feed.destroy(err)`

An alias to `stream.destroy`.

## Wire protocol

The hypercore protocol consists of two phases.
A handshake phase and a message exchange phage.

For the handshake Noise is used with the XX pattern. Each Noise message is sent with varint framing.
After the handshake a message exchange phased is started.

This uses a basic varint length prefixed format to send messages over the wire.

All messages contains a header indicating the type and feed id, and a protobuf encoded payload.

```
message = header + payload
```

A header is a varint that looks like this

```
header = numeric-feed-id << 4 | numeric-type
```

The feed id is just an incrementing number for every feed shared and the type corresponds to which protobuf schema should be used to decode the payload.

The message is then wrapped in another varint containing the length of the message

```
wire = length(message) + message + length(message2) + message2 + ...
```

## License

MIT
