# Flip Dot Sign Compendium

This is a collection of information related to the Mega Max 3000 series of Flip Dot signs produced by Luminator. Specifically you will find that the content includes information on the 98x16 Mega Max 3000 front sign, and the Mega Max 3000 90x7 side sign. There is a more common front sign with a resolution of 112x16, however I do not have specific information on this sign.

### Arduino C++ Projects using RS-485 Protocol
  1. [98x16 Front Sign with Graphics flipdotSoftwaregfx](https://github.com/hshutan/flipdotSoftwareDrivergfx) - An Arduino based project that communicates with a 98x16 Luminator Mega Max 3000 sign via RS-485 Modbus ASCII and makes use of the Adafruit_GFX library for easy text/graphics output.
  2. [98x16 Front Sign bare control flipdotSoftware](https://github.com/hshutan/flipdotSoftwareDriver) - Same of the above but without the Adafruit_GFX. Bare bones control.
  3. [90x7 Side Sign basic control flipdotSoftwareDriver_90x7_sidesign](https://github.com/hshutan/flipdotSoftwareDriver_90x7_sidesign) - An adaptation of my 98x16 front sign code to be used with a 90x7 side sign.

---

### Modbus ASCII over RS-485 and Connection Information

The information in this section is regarding the protocol-based projects and other general sign information.

#### Logic Connection Diagram
The diagram below is used with [flipdotSoftwaregfx](https://github.com/hshutan/flipdotSoftwareDrivergfx) or [flipdotSoftware](https://github.com/hshutan/flipdotSoftwareDriver), this is a known working configuration. The left connector is either of the two circular connectors on the sign (these connectors are the same pin out). *Please note the orientation of the circular connector.* The middle device is a no-name brand TTL to RS-485 converter, and the device on the right is a [Teensy 3.2](http://pjrc.com/) microcontroller. A Teensy 3.2 is not required, however a microcontroller with similar RAM and CPU should be used.

![Connection Diagram](https://github.com/hshutan/FlipDotCompendium/blob/master/Images/ConnectionDiagram.png)

#### Power Connection Diagram
The diagram below can be used to wire up power to any Mega Max 3000 sign. *Please note the orientation of the circular connector.* The sign draws all current through the 24vDC supply. The two 12vDC pins are signal connections, no current draw. The following information will help you select a power supply for a Luminator Mega Max 3000 98x16 front sign. Other signs may require more or less power.

* Current draw at idle: 210mA at 24 volts
* Current draw during flipping: 2.5 Amps at 24 volts
* Current draw of LEDs alone: 2 Amps at 24 volts
* Max current draw possible: 4.5 Amps at 24 volts
* Suggested power supply capability: at least 7 Amps at 24 volts
* Suggested in-line fuse on common negative: 5 Amps

![Power Diagram](https://github.com/hshutan/FlipDotCompendium/blob/master/Images/PowerReqs.png)

Note that the connection diagrams shown here are head-on photos of the 7-pin male connectors. These connectors are found in pairs on Max 3000 systems, both connectors share the same pin outs.

#### Sign ID Information

Typically there are multiple signs communicating on a shared [RS-485](https://en.wikipedia.org/wiki/RS-485) bus. Thus a method of setting a sign's unique ID exists, and would be applicable to configure in a real-world use of multiple signs. The protocol-based projects listed here rely on the sign being configured to a known sign ID, and do not support multiple signs on a single RS-485 bus.

##### A Note on Sign IDs

For the code in the 98x16 front sign protocol-based projects to work, set the sign's ID to = **6**. For the code in the 90x7 side sign protocol-based project to work, set the sign's ID to = **3**. As seen in the example image below, the sign ID is set to **6**. In the image: up is off, and **DOWN** is **ON**. *Do not adjust the sign ID with power applied to the sign.*

![Sign ID Set to 6](https://github.com/hshutan/FlipDotCompendium/blob/master/Images/SignID6.png)

##### How to set Sign IDs in General

The ID is encoded in binary. Look at the table below to see example settings for various sign IDs. *Do not adjust the sign ID with power applied to the sign.*

| Switch 1 | Switch 2 | Switch 3 | Switch 4 | Switch 5 | Switch 6 | Switch 7 | Switch 8 |  Resulting Sign ID  |
|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|:-------------------:|
|    **ON**   |     off    |     off    |     off    |     off    |     off    |     off    |     off    | Activates Test Mode |
|    **ON**   |     **ON**    |     off    |     off    |     off    |     off    |     off    |     off    | Activates Timer Mode |
|     off    |     off    |     off    |     off    |     off    |     off    |     off    |    **ON**   |          1          |
|     off    |     off    |     off    |     off    |     off    |     off    |    **ON**   |     off    |          2          |
|     off    |     off    |     off    |     off    |     off    |     off    |    **ON**   |    **ON**   |          3          |
|     off    |     off    |     off    |     off    |     off    |    **ON**   |     off    |     off    |          4          |
|     off    |     off    |     off    |     off    |     off    |    **ON**   |     off    |    **ON**   |          5          |
|     off    |     off    |     off    |     off    |     off    |    **ON**   |    **ON**   |     off    |          6          |
|     off    |     off    |     off    |     off    |     off    |    **ON**   |    **ON**   |    **ON**   |          7          |
|     off    |     off    |     off    |     off    |    **ON**   |     off    |     off    |     off    |          8          |

##### Sign Test Mode

Somewhat interesting is the test mode option seen in the table above. This is a useful way to see if a sign is operating properly independent of other variables, like communications protocols, or custom software.

1. Remove all power from the sign
2. Set the dipswitches to test mode as indicated above
3. Apply power to the sign (24vDC and 12vDC to appropriate connections)

##### Timer Mode

A mode that counts up in seconds starting at 00:00:00. 

1. Remove all power from the sign
2. Set the dipswitches to timer mode as indicated above
3. Apply power to the sign (24vDC and 12vDC to appropriate connections)

#### Protocol Information

##### Overview

The protocol appears to be very similar to standard [Modbus ASCII](https://en.wikipedia.org/wiki/Modbus). The communication bus is RS-485 at 19200 baud rate. Modbus and RS-485 are both very common in automotive and harsh electrical environments.

All command lines begin with a ":" character. This character is not included in the checksum calculation.

All command lines end with a newline (specifically must be CRLF). This is not included in the checksum calculation.

Command lines can be of varying length, as long as they are ended with the two character checksum which is performed on all data inbetween the ":" and CRLF.

The sign will send replies to many commands. Due to the simplex nature of RS-485, only one device can talk at a time, you cannot send data to the sign while it is replying. Also in certain Modbus standards, a pause/delay of 10 milliseconds must be added after newline or EOL (CRLF). This delay creates a gap where the sign can send its reply if needed, and also satisfies the time gap required by the Modbus protocol. The sign will not understand commands without enough EOL delay.

##### Checksum (LRC)

Each command line sent to the sign must contain a checksum. The checksum is an "LRC" or "8bit 2's Compliment". The below resources are very helpful and show how to calculate the exact two character LRC that the sign expects. All credit goes to the original authors.

* A webpage with an interactive LRC generator: [AnalyseDataHex and See CheckSum8 2s Complement section](http://www.scadacore.com/field-applications/programming-calculators/online-checksum-calculator/)
* Arudino C++ code: [LRC calculation for Modbus ASCII Protocol In Arduino](http://anilarduino.blogspot.com/2015/05/lrc-calculation-for-modbus-ascii.html)
* C# code to generate LRC: [SoapHexBinary class in System.Runtime.Remoting.Metadata.W3cXsd2001](http://stackoverflow.com/questions/12942904/calculate-twos-complement-checksum-of-hexadecimal-string/12943029)

##### Checksum Examples

|                   Data                   	| CRC 	|               Complete Command              	|
|----------------------------------------	|:---:	|-------------------------------------------	|
| 01000603A2                               	|  54 	| :01000603A254                               	|
| 00000101                                 	|  FE 	| :00000101FE                                 	|
| 10000000010A0000FFFFFFFFFFFFFFFFFFFFFFFF 	|  F1 	| :10000000010A0000FFFFFFFFFFFFFFFFFFFFFFFFF1 	|


##### Bit/Byte Order for the 98x16 Front Sign

When sending images to the 98x16 sign, the bit order, or byte order, must be constructed in the below specific fashion.

![Bitorder Diagram](https://github.com/hshutan/FlipDotCompendium/blob/master/Images/BitOrder.png)

This example image shows the order of bits for the first two columns of a sign. When [sending an image to the sign](#sending-an-image---all-dots-on) you must correctly order the bits, and then represent them as HEX in ASCII.

As an over simplified example, imagine the first two columns pictured above are alternating on/off dots ``01010101010101010101010101010101`` this binary would be converted to and sent as ``55555555`` in ASCII. Remember not to send raw ``0x55 0x55 0x55 0x55`` HEX out the serial port, it must be represented as ASCII. Meaning the data that actually gets sent out over serial is twice as much: ``0x35 0x35 0x35 0x35 0x35 0x35 0x35 0x35`` This is because Modbus ASCII sacrifices performance for human readability.



##### Sign Init 98x16 Front Sign

These commands **need** to be sent to the sign at least once after it is initially enabled and powered up (having both 24vDC and 12vDC correctly applied).

```
:01000502FFF9
:01000602FFF8
:01000603A155
:100000000447000F101C1C1C1C1000000000000006
:00000101FE
:0100060200F7
```

##### Sign Init 90x7 Side Sign

These commands **need** to be sent to the sign at least once after it is initially enabled and powered up (having both 24vDC and 12vDC correctly applied).

```
:01000502FFF9
:01000602FFF8
:01000303A158
:1000000004200006071E1E1E00080000000000005D
:00000101FE
:0100030200FA
```

##### Sign Close 98x16 Front Sign

These commands can be sent to the sign before you cut all power. They will instruct the sign to do a hard OFF write to every single dot. Note that instead of this command, removing 12vDC sign enable power while maintaining 24vDC power will also cause the sign to do this same shutdown procedure.

```
:01000603A94D
:01000603AA4C
:01007F02FF7F
:0100060255A2
:01000603A650
```

##### Sending an Image - 98x16 Front Sign - All Dots On

These commands will send an example image, 98x16 pixels with all dots ON, into the sign's memory and finally tell the sign to display the image. The sections of zeros are pixels that the 98x16 sign does not possess, are invisible, and can be left turned off.

```
:01000603A254
:10000000010A0000FFFFFFFFFFFFFFFFFFFFFFFFF1
:10001000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF0
:1000200000000000000000000000000000000000D0
:10003000000000000000000000000000FFFFFFFFC4
:10004000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFC0
:10005000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFB0
:10006000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFA0
:10007000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF90
:10008000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF80
:10009000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF70
:1000A000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF60
:1000B000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF50
:1000C000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF40
:1000D000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF30
:1000E000FFFFFFFF00000000000000000000000014
:00000F01F0
:0100060200F7
:0100060600F3
:0100060200F7
:01000603A94D
```

##### Color Coded Example - 98x16 Front Sign - All Dots On
This image is the same as the above example with the addition of color coding to highlight sections of the commands.

![Example Dots All On.png](https://github.com/hshutan/FlipDotCompendium/blob/master/Images/ExampleDotsAllOn.png)


##### Sending an Image - 90x7 Side Sign - All Dots On

These commands will send an example image, 90x7 pixels with all dots ON, into the sign's memory and finally tell the sign to display the image.

```
:01000303A257
:1000000001100000FFFFFFFFFFFFFFFFFFFFFFFFEB
:10001000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFF0
:10002000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFE0
:10003000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFD0
:10004000FFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFFC0
:10005000FFFFFFFFFFFFFFFFFFFFFFFFFFFF0000AE
:00000C01F3
:0100030200FA
:0100030600F6
:0100030200FA
:01000303A950
```


---

### Legal Note
Please note that the various flip dot products and technologies discussed are both copyright and also property of their respective owners. The communication protocol is not mine. The information in these home hobby projects may not be used for profit or commercial uses of any kind. Any for profit or commercial use should be performed through manufacturer approved retail channels.
