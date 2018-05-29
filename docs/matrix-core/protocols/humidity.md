<h2 style="padding-top:0">Humidity</h2>

### Device Compatibility
<img class="creator-compatibility-icon" src="/img/creator-icon.svg">

## Overview
The Humidity driver reports values for:

* Relative Humidity on the Board 
* Temperature in Celsius
* RAW Values of temperature

<h3 style="padding-top:0">Available ZeroMQ Ports</h3>
* `Base port`: 20017

* `Error port`: 20019

* `Data update port`: 20020


The Humidity can be configured with:
* Current temperature in Celsius
* Flag to set software calibration.

### Protocol buffers

```language-protobuf
message Humidity {
  float humidity = 1;
  float temperature = 2;
  float temperature_raw = 3;
  bool temperature_is_calibrated = 4 ;
}

message HumidityParams{
  float current_temperature = 1; 
  bool do_calibration = 2;
}
```

The message is defined in <a href="https://github.com/matrix-io/protocol-buffers/blob/master/matrix_io/malos/v1/driver.proto" target="_blank">driver.proto</a>.

### Keep-alives

This driver needs keep-alive messages [as specified in the CORE protocol](../index.md#keep-alive-port).
If you start sending keep-alive messages it will start returning data every second so you can omit the configuration for this device.


### Errors

This driver report errors when an invalid configuration is sent.

### Read

The driver will send a serialized message of type `Humidity`.

```language-protobuf
message Humidity {
  float humidity = 1;
  float temperature = 2;
  float temperature_raw = 3;
  bool temperature_is_calibrated = 4 ;
}

message HumidityParams{
  float current_temperature = 1;
  bool do_calibration = 2;
}
```

This is a sample output given by the example described below.

```language-bash
$ node test_humidity.js 
Sending pings every 5 secondsMessage received: Humidity error: 0, Invalid specific configuration for Humidity driver.
{ humidity: 30.716121673583984,
  temperature: 20.246780395507812,
  temperature_raw: 35.456275939941406,
  temperature_is_calibrated: true }
{ humidity: 30.716121673583984,
  temperature: 20.246780395507812,
  temperature_raw: 35.456275939941406,
  temperature_is_calibrated: true }
```

### JavaScript example

Enhanced description of the <a href="https://github.com/matrix-io/matrix-creator-malos/blob/master/src/js_test/test_humidity.js" target="_blank">sample source code</a>.

First, define the address of the MATRIX Creator. In this case we make it be `127.0.0.1`
because we are connecting from the local host but it needs to be different if we
connect from another computer. There is also the base port reserved by CORE for
the Humidity driver.

```language-javascript
var creator_ip = '127.0.0.1'
var creator_humidity_base_port = 20013 + 4 
```

Load the protocol buffers used in the example.

```language-javascript
var protoBuf = require("protobufjs")

// Parse proto file
var protoBuilder = protoBuf.loadProtoFile('../../protocol-buffers/malos/driver.proto')
// Parse matrix_malos package (namespace).
var matrixMalosBuilder = protoBuilder.build("matrix_malos")
```

Subscribe to the errors reported by the driver. 

```language-javascript
var zmq = require('zmq')

var errorSocket = zmq.socket('sub')
errorSocket.connect('tcp://' + creator_ip + ':' + (creator_humidity_base_port + 2))
errorSocket.subscribe('')
errorSocket.on('message', function(error_message) {
  process.stdout.write('Message received: Humidity error: ' + error_message.toString('utf8') + "\n")
});
```
All the drivers are configured using the message `DriverConfig` (see <a href="https://github.com/matrix-io/protocol-buffers/blob/master/matrix_io/malos/v1/driver.proto" target="_blank">driver.proto</a>).
This is what the message looks like if we omit the fields that are not used in this example.
```language-protobuf
message DriverConfig {
  float delay_between_updates = 1;
  float timeout_after_last_ping = 2;
  HumidityParams humidity = 9;
}
```
The following snippet is telling the driver to send an update each 2 seconds
and stop sending updates if it doesn't receive a keep-alive message for 6 seconds.

```language-javascript
var configSocket = zmq.socket('push')
configSocket.connect('tcp://' + creator_ip + ':' + creator_uv_base_port)

var driverConfigProto = new matrixMalosBuilder.DriverConfig

driverConfigProto.delay_between_updates = 2.0
driverConfigProto.timeout_after_last_ping = 6.0
var hum_params_msg = new matrixMalosBuilder.HumidityParams
hum_params_msg.current_temperature = 23
hum_params_msg.do_calibration = false
driverConfigProto.set_humidity(hum_params_msg)

configSocket.send(driverConfigProto.encode().toBuffer())
```

Where is where the updates are received by subscribing to the `data update port` of the driver.
The subscription is initiated by the line `updateSocket.subscribe('')`.

```language-javascript
var updateSocket = zmq.socket('sub')
updateSocket.connect('tcp://' + creator_ip + ':' + (creator_humidity_base_port + 3))
updateSocket.subscribe('')
updateSocket.on('message', function(buffer) {
  var data = new matrixMalosBuilder.Humidity.decode(buffer)
  console.log(data)
});
```
An empty keep-alive message is sent to the driver every 5 seconds to make sure it keeps
sending data updates.

```language-javascript
var pingSocket = zmq.socket('push')
pingSocket.connect('tcp://' + creator_ip + ':' + (creator_humidity_base_port + 1))
process.stdout.write("Sending pings every 5 seconds");
pingSocket.send(''); // Ping the first time.
setInterval(function(){
  pingSocket.send('');
}, 5000);
```
