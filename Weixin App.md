# Use MQTT on Weixin App

This document is to show how to use MQTT with [MQTT.js](https://github.com/mqttjs/MQTT.js)  library on Weixin App.

## Installation
Any of your folders

```sh
 npm install mqtt
```

then copy `mqtt.min.js` from `node_modules/mqtt/dist/mqtt.min.js` to you `utils` folder,
Now you can use it.

## Example

```js
var mqtt = require('/utils/mqtt.js')
var client = mqtt.connect('wx://q.emqtt.com:8083/mqtt')
// SSL
// var client = mqtt.connect('wxs://q.emqtt.com:8084/mqtt')
client.on('connect',function(){
	client.subscribe('topic')
	client.publish('topic','message')
})

client.on('message',function(topic, message){
	console.log(message.toString())
})
```

## API
  * <a href="#connect"><code>mqtt.<b>connect()</b></code></a>
  * <a href="#client"><code>mqtt.<b>Client()</b></code></a>
  * <a href="#publish"><code>mqtt.Client#<b>publish()</b></code></a>
  * <a href="#subscribe"><code>mqtt.Client#<b>subscribe()</b></code></a>
  * <a href="#unsubscribe"><code>mqtt.Client#<b>unsubscribe()</b></code></a>
  * <a href="#end"><code>mqtt.Client#<b>end()</b></code></a>
  * <a href="#removeOutgoingMessage"><code>mqtt.Client#<b>removeOutgoingMessage()</b></code></a>
  * <a href="#reconnect"><code>mqtt.Client#<b>reconnect()</b></code></a>
  * <a href="#handleMessage"><code>mqtt.Client#<b>handleMessage()</b></code></a>
  * <a href="#connected"><code>mqtt.Client#<b>connected</b></code></a>
  * <a href="#reconnecting"><code>mqtt.Client#<b>reconnecting</b></code></a>
  * <a href="#getLastMessageId"><code>mqtt.Client#<b>getLastMessageId()</b></code></a>
  * <a href="#store"><code>mqtt.<b>Store()</b></code></a>
  * <a href="#put"><code>mqtt.Store#<b>put()</b></code></a>
  * <a href="#del"><code>mqtt.Store#<b>del()</b></code></a>
  * <a href="#createStream"><code>mqtt.Store#<b>createStream()</b></code></a>
  * <a href="#close"><code>mqtt.Store#<b>close()</b></code></a>
-------

<a name="connect"></a>
### mqtt.connect([url], options)

Connects to the broker specified by the given url and options and
returns a [Client](#client).

The URL can be on the following protocols: 'mqtt', 'mqtts', 'tcp',
'tls', 'ws', 'wss'. The URL can also be an object as returned by
[`URL.parse()`](http://nodejs.org/api/url.html#url_url_parse_urlstr_parsequerystring_slashesdenotehost),
in that case the two objects are merged, i.e. you can pass a single
object with both the URL and the connect options.

You can also specify a `servers` options with content: `[{ host:
'localhost', port: 1883 }, ... ]`, in that case that array is iterated
at every connect.

For all MQTT-related options, see the [Client](#client)
constructor.

-------------------------------------------------------
<a name="client"></a>
### mqtt.Client(streamBuilder, options)

The `Client` class wraps a client connection to an
MQTT broker over an arbitrary transport method (TCP, TLS,
WebSocket, ecc).

`Client` automatically handles the following:

* Regular server pings
* QoS flow
* Automatic reconnections
* Start publishing before being connected

The arguments are:

* `streamBuilder` is a function that returns a subclass of the `Stream` class that supports
the `connect` event. Typically a `net.Socket`.
* `options` is the client connection options (see: the [connect packet](https://github.com/mcollina/mqtt-packet#connect)). Defaults:
  * `wsOptions`: is the WebSocket connection options. Default is `{}`.
     It's specific for WebSockets. For possible options have a look at: https://github.com/websockets/ws/blob/master/doc/ws.md.
  * `keepalive`: `60` seconds, set to `0` to disable
  * `reschedulePings`: reschedule ping messages after sending packets (default `true`)
  * `clientId`: `'mqttjs_' + Math.random().toString(16).substr(2, 8)`
  * `protocolId`: `'MQTT'`
  * `protocolVersion`: `4`
  * `clean`: `true`, set to false to receive QoS 1 and 2 messages while
    offline
  * `reconnectPeriod`: `1000` milliseconds, interval between two
    reconnections
  * `connectTimeout`: `30 * 1000` milliseconds, time to wait before a
    CONNACK is received
  * `username`: the username required by your broker, if any
  * `password`: the password required by your broker, if any
  * `incomingStore`: a [Store](#store) for the incoming packets
  * `outgoingStore`: a [Store](#store) for the outgoing packets
  * `queueQoSZero`: if connection is broken, queue outgoing QoS zero messages (default `true`)
  * `will`: a message that will sent by the broker automatically when
     the client disconnect badly. The format is:
    * `topic`: the topic to publish
    * `payload`: the message to publish
    * `qos`: the QoS
    * `retain`: the retain flag
  * `transformWsUrl` : optional `(url, options, client) => url` function
        For ws/wss protocols only. Can be used to implement signing
        urls which upon reconnect can have become expired.
  * `resubscribe` : if connection is broken and reconnects,
     subscribed topics are automatically subscribed again (default `true`)

In case mqtts (mqtt over tls) is required, the `options` object is
passed through to
[`tls.connect()`](http://nodejs.org/api/tls.html#tls_tls_connect_options_callback).
If you are using a **self-signed certificate**, pass the `rejectUnauthorized: false` option.
Beware that you are exposing yourself to man in the middle attacks, so it is a configuration
that is not recommended for production environments.

If you are connecting to a broker that supports only MQTT 3.1 (not
3.1.1 compliant), you should pass these additional options:

```js
{
  protocolId: 'MQIsdp',
  protocolVersion: 3
}
```

This is confirmed on RabbitMQ 3.2.4, and on Mosquitto < 1.3. Mosquitto
version 1.3 and 1.4 works fine without those.

#### Event `'connect'`

`function (connack) {}`

Emitted on successful (re)connection (i.e. connack rc=0).
* `connack` received connack packet. When `clean` connection option is `false` and server has a previous session
for `clientId` connection option, then `connack.sessionPresent` flag is `true`. When that is the case,
you may rely on stored session and prefer not to send subscribe commands for the client.

#### Event `'reconnect'`

`function () {}`

Emitted when a reconnect starts.

#### Event `'close'`

`function () {}`

Emitted after a disconnection.

#### Event `'offline'`

`function () {}`

Emitted when the client goes offline.

#### Event `'error'`

`function (error) {}`

Emitted when the client cannot connect (i.e. connack rc != 0) or when a
parsing error occurs.

#### Event `'message'`

`function (topic, message, packet) {}`

Emitted when the client receives a publish packet
* `topic` topic of the received packet
* `message` payload of the received packet
* `packet` received packet, as defined in
  [mqtt-packet](https://github.com/mcollina/mqtt-packet#publish)

#### Event `'packetsend'`

`function (packet) {}`

Emitted when the client sends any packet. This includes .published() packets
as well as packets used by MQTT for managing subscriptions and connections
* `packet` received packet, as defined in
  [mqtt-packet](https://github.com/mcollina/mqtt-packet)

#### Event `'packetreceive'`

`function (packet) {}`

Emitted when the client receives any packet. This includes packets from
subscribed topics as well as packets used by MQTT for managing subscriptions
and connections
* `packet` received packet, as defined in
  [mqtt-packet](https://github.com/mcollina/mqtt-packet)

-------------------------------------------------------
<a name="publish"></a>
### mqtt.Client#publish(topic, message, [options], [callback])

Publish a message to a topic

* `topic` is the topic to publish to, `String`
* `message` is the message to publish, `Buffer` or `String`
* `options` is the options to publish with, including:
  * `qos` QoS level, `Number`, default `0`
  * `retain` retain flag, `Boolean`, default `false`
  * `dup` mark as duplicate flag, `Boolean`, default `false`
* `callback` - `function (err)`, fired when the QoS handling completes,
  or at the next tick if QoS 0. An error occurs if client is disconnecting.

-------------------------------------------------------
<a name="subscribe"></a>
### mqtt.Client#subscribe(topic/topic array/topic object, [options], [callback])

Subscribe to a topic or topics

* `topic` is a `String` topic to subscribe to or an `Array` of
  topics to subscribe to. It can also be an object, it has as object
  keys the topic name and as value the QoS, like `{'test1': 0, 'test2': 1}`.
  MQTT `topic` wildcard characters are supported (`+` - for single level and `#` - for multi level)
* `options` is the options to subscribe with, including:
  * `qos` qos subscription level, default 0
* `callback` - `function (err, granted)`
  callback fired on suback where:
  * `err` a subscription error or an error that occurs when client is disconnecting
  * `granted` is an array of `{topic, qos}` where:
    * `topic` is a subscribed to topic
    * `qos` is the granted qos level on it

-------------------------------------------------------
<a name="unsubscribe"></a>
### mqtt.Client#unsubscribe(topic/topic array, [callback])

Unsubscribe from a topic or topics

* `topic` is a `String` topic or an array of topics to unsubscribe from
* `callback` - `function (err)`, fired on unsuback. An error occurs if client is disconnecting.

-------------------------------------------------------
<a name="end"></a>
### mqtt.Client#end([force], [cb])

Close the client, accepts the following options:

* `force`: passing it to true will close the client right away, without
  waiting for the in-flight messages to be acked. This parameter is
  optional.
* `cb`: will be called when the client is closed. This parameter is
  optional.

-------------------------------------------------------
<a name="removeOutgoingMessage"></a>
### mqtt.Client#removeOutgoingMessage(mid)

Remove a message from the outgoingStore.
The outgoing callback will be called withe Error('Message removed') if the message is removed.

After this function is called, the messageId is released and becomes reusable.

* `mid`: The messageId of the message in the outgoingStore.

-------------------------------------------------------
<a name="reconnect"></a>
### mqtt.Client#reconnect()

Connect again using the same options as connect()

-------------------------------------------------------
<a name="handleMessage"></a>
### mqtt.Client#handleMessage(packet, callback)

Handle messages with backpressure support, one at a time.
Override at will, but __always call `callback`__, or the client
will hang.

-------------------------------------------------------
<a name="connected"></a>
### mqtt.Client#connected

Boolean : set to `true` if the client is connected. `false` otherwise.

-------------------------------------------------------
<a name="getLastMessageId"></a>
### mqtt.Client#getLastMessageId()

Number : get last message id. This is for sent messages only.

-------------------------------------------------------
<a name="reconnecting"></a>
### mqtt.Client#reconnecting

Boolean : set to `true` if the client is trying to reconnect to the server. `false` otherwise.

-------------------------------------------------------
<a name="store"></a>
### mqtt.Store(options)

In-memory implementation of the message store.

* `options` is the store options:
  * `clean`: `true`, clean inflight messages when close is called (default `true`)

Other implementations of `mqtt.Store`:

* [mqtt-level-store](http://npm.im/mqtt-level-store) which uses
  [Level-browserify](http://npm.im/level-browserify) to store the inflight
  data, making it usable both in Node and the Browser.
* [mqtt-nedbb-store](https://github.com/behrad/mqtt-nedb-store) which
  uses [nedb](https://www.npmjs.com/package/nedb) to store the inflight
  data.
* [mqtt-localforage-store](http://npm.im/mqtt-localforage-store) which uses
  [localForage](http://npm.im/localforage) to store the inflight
  data, making it usable in the Browser without browserify.

-------------------------------------------------------
<a name="put"></a>
### mqtt.Store#put(packet, callback)

Adds a packet to the store, a packet is
anything that has a `messageId` property.
The callback is called when the packet has been stored.

-------------------------------------------------------
<a name="createStream"></a>
### mqtt.Store#createStream()

Creates a stream with all the packets in the store.

-------------------------------------------------------
<a name="del"></a>
### mqtt.Store#del(packet, cb)

Removes a packet from the store, a packet is
anything that has a `messageId` property.
The callback is called when the packet has been removed.

-------------------------------------------------------
<a name="close"></a>
### mqtt.Store#close(cb)

Closes the Store.

## Other 
You can visit [MQTT.js](https://github.com/mqttjs/MQTT.js) for more information.