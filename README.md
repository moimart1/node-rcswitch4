node-rcswitch2
==============

[![NPM version](https://badge.fury.io/js/rcswitch2.svg)](http://badge.fury.io/js/rcswitch2)

Node bindings for the [rcswitch](https://github.com/sui77/rc-switch).

This is an extended version of [node-rcswitch](https://github.com/marvinroger/node-rcswitch) project.

Comparing to original [node-rcswitch](https://github.com/marvinroger/node-rcswitch) it has:
* [#9](https://github.com/marvinroger/node-rcswitch/issues/9) - Replaced [outdated](https://github.com/r10r/rcswitch-pi/issues/25) [rcswitch-pi](https://github.com/r10r/rcswitch-pi) dependency with the latest version of [rc-switch](https://github.com/sui77/rc-switch) project
* [#11](https://github.com/marvinroger/node-rcswitch/issues/11) - Added code receiving bindings
* [#13](https://github.com/marvinroger/node-rcswitch/issues/13) - Exporting of class instead of already created instance

Although the project is now using generic RcSwitch library, it's still intended to be used on Raspberry Pi only.

It should be compatible with all versions of Node.js starting from 0.8.

## Requirements

* Like the C++ version of [rcswitch](https://github.com/sui77/rc-switch), [WiringPi must be installed](https://projects.drogon.net/raspberry-pi/wiringpi/download-and-install/) in order to compile.
* Both the data and the power Pins of the 315/433Mhz emitter must be connected to the RPi. Note the number of the WiringPi data Pin. (see http://wiringpi.com/pins/)
* The node command must be run with root access

## Usage

### Sending

```javascript
var RcSwitch = require('rcswitch'); // Might throw an error if wiring pi init failed, or exit process if no root (must work on that)
var rcswitch = new RcSwitch();

rcswitch.enableTransmit(0); // Use data Pin 0 (GPIO 17)
rcswitch.switchOn("10110", 1); // Switch on the first unit of 10110 (code 1x23x) group
rcswitch.switchOff("11000", 2); // Switch off the second unit of 11000 (code 12xxx) group
```

### Receiving

```javascript
var RcSwitch = require('rcswitch'); // Might throw an error if wiring pi init failed, or exit process if no root (must work on that)
var rcswitch = new RcSwitch();

rcswitch.enableReceive(2); // Use data Pin 2 (GPIO 27)
setInterval(function () {
    if (rcswitch.available()) { // Check if there is a pending code
        console.log(rcswitch.getReceivedValue()); // Read and print the code
        rcswitch.resetAvailable(); // Reset `available` state
    }
}, 500); // Set code checking interval
```

## API

### Sending

#### Configuration

##### rcswitch.enableTransmit(`pin`)

Enable transmission on the given pin (make it an OUTPUT). Must be called before any other functions.

* `pin` - (Number) data Pin to use following [the WiringPi schema](http://wiringpi.com/pins/)

Return true if `pin` is an integer, false otherwise.

##### rcswitch.disableTransmit()

Disable transmission (set the pin to -1 which disable any following function call).

Return true.

#### Type A

![Type A switch](https://raw.github.com/insolite/node-rcswitch2/master/img/type_a.jpg "Type A switch")

##### rcswitch.switchOn(`group`, `switch`)

Switch a remote switch on (Type A with 10 pole DIP switches).

* `group` - (String) code of the switch group (refers to DIP switches 1, 2, 3, 4 and 5 where "1" = on and "0" = off - e.g. if all DIP switches are on it's "11111")
* `switch` - (Number) switch number (can be 1 (if DIP switch A is on), 2 (if DIP switch B is on) and so on until 4)

Return true.

##### rcswitch.switchOff(`group`, `switch`)

Switch a remote switch off (Type A with 10 pole DIP switches).

* `group` - (String) code of the switch group (refers to DIP switches 1, 2, 3, 4 and 5 where "1" = on and "0" = off - e.g. if all DIP switches are on it's "11111")
* `switch` - (Number) switch number (can be 1 (if DIP switch A is on), 2 (if DIP switch B is on) and so on until 4)

Return true.

#### Type B

![Type B switch](https://raw.github.com/insolite/node-rcswitch2/master/img/type_b.jpg "Type B switch")

##### rcswitch.switchOn(`group`, `switch`)

Switch a remote switch on (Type B with two rotary/sliding switches).

* `group` - (Number) group (can be 1, 2, 3, 4)
* `switch` - (Number) switch (can be 1, 2, 3, 4)

Return true.

##### rcswitch.switchOff(`group`, `switch`)

Switch a remote switch off (Type B with two rotary/sliding switches).

* `group` - (Number) group (can be 1, 2, 3, 4)
* `switch` - (Number) switch (can be 1, 2, 3, 4)

Return true.

#### Type C

##### rcswitch.switchOn(`family`, `group`, `switch`)

Switch a remote switch on (Type C Intertechno).

* `family` - (String) familycode (can be a, b, c, d, e, f)
* `group` - (Number) group (can be 1, 2, 3, 4)
* `switch` - (Number) switch (can be 1, 2, 3, 4)

Return true.

##### rcswitch.switchOff(`family`, `group`, `switch`)

Switch a remote switch off (Type C Intertechno).

* `family` - (String) familycode (can be a, b, c, d, e, f)
* `group` - (Number) group (can be 1, 2, 3, 4)
* `switch` - (Number) switch (can be 1, 2, 3, 4)

Return true.

#### Other

##### rcswitch.send(`code`)

Send raw binary string code.

* `code` - (String) code

Return true.

##### rcswitch.sendTriState(`code`)

Send tri-state code.

* `code` - (String) tri-state code

Return true.

This function is useful for eg. micro-electric AS 73 which is also sold as REV Telecontrol in Germany (Version with house code with 6 DIP switches).

This socket has 10 DIP-Switches.

The house code uses the first 6 switches, the receiver code is set by the next 4 switches. For the house code, the switch position OFF is represented by F and switch position ON by 0.

Receiver codes:

Channel	Switches 7-10
* `0FFF` Channel A
* `F0FF` Channel B
* `FFF0` Channel C
* `FF0F` Channel D

* `FF` or `F0` Button on
* `0F` Button off

The input string for the function is `[homecode][channel][on/off value]` 
e.g. F000000FFFFF for homecode 100000, Channel A and button on.

### Receiving

#### rcswitch.enableReceive(`pin`)

Enable receiving on the given pin (make it an INPUT). Must be called before any other functions.

* `pin` - (Number) data Pin to use following [the WiringPi schema](http://wiringpi.com/pins/)

Return true if `pin` is an integer, false otherwise.

#### rcswitch.disableReceive()

Disable receiving (set the pin to -1 which disable any following function call).

Return true.

#### rcswitch.available()

Check if the code was received.

Return true if there is a pending code, false otherwise.

#### rcswitch.resetAvailable()

Reset `available` state to start recognizing of the next code.

Return true.

#### rcswitch.getReceivedValue()

Get the last received code.

Return integer representation of the code.

#### rcswitch.getReceivedBitlength()

Get bit length of the last received code.

Return integer length.

#### rcswitch.getReceivedDelay()

Get delay.

Return integer delay.

#### rcswitch.getReceivedProtocol()

Get protocol.

Return integer protocol.

#### rcswitch.getReceivedRawdata()

Get raw data.

Return array of raw data.
