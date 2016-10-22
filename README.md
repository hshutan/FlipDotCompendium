# Flip Dot Sign Compendium

This is a collection of information related to the Mega Max 3000 series of Flip Dot signs produced by Luminator. Specifically you will find the content is tailored to a 98x16 MegaMax 3000 front sign that I own. Note that various modifications would be needed to drive other dimension signs, such as the more common 112x16 versions.

### Index

* Protocol-based Projects
  1. flipdotSoftwaregfx - An Arduino based project that communicates with a 98x16 Luminator Mega Max 3000 sign via RS-485 MODBUS ASCII and makes use of the Adafruit_GFX library for easy text/graphics output.
  2. flipdotSoftware - Same of the above but without the Adafruit_GFX. Bare bones control.
* Hardware-based Project
  1. 45x7 Flipdot Sign - Unlike the above projects, this project involved reverse engineering the electromechanical hardware and designing a custom driver board for one of the 45x7 panels inside a 90x7 Mega Max 3000 side sign.
* MODBUS ASCII over RS-485 and Connection Information for the Protocol-based projects above
* Legal Note


##

### MODBUS ASCII over RS-485 and Connection Information

The information in this section is regarding the protocol-based projects and other general sign information.

#### Logic Connection Diagram
The diagram below is used with flipdotSoftwaregfx or flipdotSoftware, this is a known working configuration. The left connector is either of the circular connectors (they are the same pin out) on the sign. *Please note the orientation of the circular connector.* The middle device is a no-name brand TTL to RS-485, and the device on the right is a Teensy 3.2 microcontroller.

![Connection Diagram](https://s16.postimg.org/63xtuotvp/Connection_Diagram_2.png)

#### Power Connection Diagram
The diagram below can be used to wire up power to the sign. Please note the orientation of the circular connector. The sign draws all current through the 24vDC supply. The 12vDC connections are only signal connections, no current draw. The following information will help you select a power supply for a Luminator Mega Max 3000 98x16 front sign:
* Current draw at idle: 210mA at 24 volts
* Current draw during flipping: 2.5 Amps at 24 volts
* Current draw of LEDs alone: 2 Amps at 24 volts
* Max current draw possible: 4.5 Amps at 24 volts
* Suggested power supply capability: at least 7 Amps at 24 volts
* Suggested in-line fuse on common negative: 5 Amps

![Power Diagram](https://s10.postimg.org/5ius06k6h/Power_Reqs.png)

#### Sign ID Information

Typically there are multiple signs all communicating on a shared line ([RS-485](https://en.wikipedia.org/wiki/RS-485)). Thus a method of setting a sign's ID exists and would be applicable to configure in a real-world use of multiple signs. The protocol-based projects listed above rely on the sign being configured to sign ID 6.

##### How to set Sign ID to 6

For the code in my protocol-based projects to work, set the sign's ID to = 6. To do this, turn dipswitches 6 and 7 both ON as seen in the image below. In the image: up is off, and **DOWN** is **ON**. *Do not adjust the sign ID with power applied to the sign.*

![signid6.png](https://s10.postimg.org/uqrxxkwvd/signid6.png)

##### How to set Sign IDs in General

The Mega Max 3000 series of signs use dipswitches to set the sign ID. The ID is encoded in binary. Look at the table below to see example settings for various sign IDs. Cross reference the row for sign ID 6 with the image above to see how the on/off states correlate. *Do not adjust the sign ID with power applied to the sign.*

| Switch 1 | Switch 2 | Switch 3 | Switch 4 | Switch 5 | Switch 6 | Switch 7 | Switch 8 |  Resulting Sign ID  |
|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|:--------:|:-------------------:|
|    **ON**   |     off    |     off    |     off    |     off    |     off    |     off    |     off    | Activates Test Mode |
|     off    |     off    |     off    |     off    |     off    |     off    |     off    |    **ON**   |          1          |
|     off    |     off    |     off    |     off    |     off    |     off    |    **ON**   |     off    |          2          |
|     off    |     off    |     off    |     off    |     off    |     off    |    **ON**   |    **ON**   |          3          |
|     off    |     off    |     off    |     off    |     off    |    **ON**   |     off    |     off    |          4          |
|     off    |     off    |     off    |     off    |     off    |    **ON**   |     off    |    **ON**   |          5          |
|     off    |     off    |     off    |     off    |     off    |    **ON**   |    **ON**   |     off    |          6          |
|     off    |     off    |     off    |     off    |     off    |    **ON**   |    **ON**   |    **ON**   |          7          |
|     off    |     off    |     off    |     off    |    **ON**   |     off    |     off    |     off    |          8          |

##### Sign Test Mode

Somewhat interesting is the test mode seen in the table above. This is an incredibly useful way to see if a sign is operating properly. As noted above, remove all power from the sign, set the dipswitches to test mode, then apply power to the sign (24vDC and 12vDC to appropriate connections).

#### Protocol Information

##### Overview

The protocol appears to be very similar to standard [MODBUS ASCII](https://en.wikipedia.org/wiki/Modbus). The communication bus is RS-485 at 19200 baud rate. MODBUS and RS-485 are both very common in automotive and harsh electrical environments.

All command lines begin with a ":" character. This character is not included in the checksum calculation.

All command lines end with a newline (specifically must be CRLF). This is not included in the checksum calculation.

Command lines can be of varying length, as long as they are ended with the two character checksum which is performed on all data inbetween the ":" and CRLF.

The sign will reply back to some commands. Due to the nature of RS-485 (it is simplex, only one device can talk at a time), and also due to the MODBUS standards, a pause/delay of 10 milliseconds must be added after each newline (CRLF). This creates a gap where the sign can send its reply if needed, and also the time gap is required by the MODBUS protocol.

##### Checksum (LRC)

Each command line sent to the sign must contain a checksum. The checksum is an "LRC" or "8bit 2's Compliment". The below resources are very helpful and show how to calculate the exact two character LRC that the sign expects. All credit goes to the original authors.

* A webpage with an interactive LRC generator: [AnalyseDataHex and See CheckSum8 2s Complement section](http://www.scadacore.com/field-applications/programming-calculators/online-checksum-calculator/)
* Arudino C++ code: [LRC calculation for MODBUS ASCII Protocol In Arduino](http://anilarduino.blogspot.com/2015/05/lrc-calculation-for-modbus-ascii.html)
* C# code to generate LRC: [SoapHexBinary class in System.Runtime.Remoting.Metadata.W3cXsd2001](http://stackoverflow.com/questions/12942904/calculate-twos-complement-checksum-of-hexadecimal-string/12943029)

##### Checksum Examples

|                   Data                   	| CRC 	|               Complete Command              	|
|----------------------------------------	|:---:	|-------------------------------------------	|
| 01000603A2                               	|  54 	| :01000603A254                               	|
| 00000101                                 	|  FE 	| :00000101FE                                 	|
| 10000000010A0000FFFFFFFFFFFFFFFFFFFFFFFF 	|  F1 	| :10000000010A0000FFFFFFFFFFFFFFFFFFFFFFFFF1 	|


##### Sign Init

##### Sign Close

##### Sending an Image - All Dots On


##### Color Coded Example - All Dots On
This is the same as the above example with color coding added to highlight sections of the commands.

![Example Dots All On.png](https://s3.postimg.org/klvazxiwz/Example_Dots_All_On.png)

##

### Legal Note
Please note that the various flip dot products and technologies discussed are both copyright and also property of their respective owners. The communication protocol is not mine. The information in these home hobby projects may not be used for profit or commercial uses of any kind. Any for profit or commercial use should be performed through manufacturer approved retail channels.
