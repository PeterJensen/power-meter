# Power Meter

**Author: Peter Jensen**

## Table of Contents

* [Introduction](#introduction)
* [Prototyping](#prototyping)
* [Computing Measured Current](#computing-measured-current)
* [Components](#components)
* [Hardware](#hardware)
* [Software](#software)

## Introduction

This is my first attempt at a 'useful' IoT project.  I've had an Arduino Uno for a while, but didn't really use it for anything, except going through a blinking LED tutorial.  It was time to bring it to use.  I find it rewarding to think up a project idea and then seek out the information you need when you need it.  Of course, often times you find that others have done something similar, but that's OK, you'll still learn from the process.

I wanted a device to measure power consumption in my house and have it report it up to a 'cloud' service when things changed; lights turning on/off, etc.  Also, I wanted a display on the box, so I could read what the current power consumption was.

## Prototyping

I started out by hooking up a current sensors to an analog input pin; like this:

![][openenergymonitor-diagram]\
Source: [openenergymonitor.org][openenergymonitor]

For calibration, I was using a multimeter that was capable of measuring up to 10A and hooked it up to a wire that was powering a three bulb lamp.  Each of the bulbs could be turned on and off individually.  With 60W bulbs in the sockets, I could get samples for 60W, 120W, and 180W.  

**Note, the current sensor must only go around one wire, in order for the AC current going through that wire to induce a current in the sensor.**

## Computing Measured Current

The measured current is the current that goes through the power line that the current sensor wraps around.

If all you do in the `loop()` function in your Arduino sketch is to call `analogRead()` and store the result away in a memory buffer, you will be able to get about 100 samples during one cycle (60Hz - 16.7ms).  The Arduino is running at 16MHz.  That should be enough to compute the root-mean-square (rms) value of the input fairly accurately.

Why do you need the rms value? The input on the analog input pin is the voltage drop over the load/burden resistor.  That voltage drop is directly proportional with the current going through the power line being measured at the time of measurement.

When the current and voltage are varying over time, the power is computed as the average power over the period of the 60Hz sinus wave, using the RMS values of I and V:

<!--
<img src="https://latex.codecogs.com/svg.latex?P_{AVG}=I_{RMS}V_{RMS}" style="height:20px; margin-left:50px">
-->
>![][power]

The V<sub>RMS</sub> component is kept constant by the power company at 120V, so the interesting one is the I<sub>RMS</sub> value.  The Arduino analog-to-digital converters are 10-bits, and the circuitry is designed so that the mid-point of the sinus input is 2.5V, which should result in a reading of ~511.  If the readings are equally spaced in time, the I<sub>RMS</sub> value can be computed as:

<!--
<img src="https://latex.codecogs.com/svg.latex?I_{RMS}=K\sqrt{\frac{1}{N}\sum_{i=1}^{N}{(v_i-511)^2}}" style="height:80px;margin-left:50px">
-->
>![][i-rms]

where N is the number of samples it take to cover the full period of the 60Hz sinus wave, K is a calibration constant that will be determined after measuring the actual I<sub>RMS</sub> values with a multimeter, and v<sub>i</sub> are the values returned by the calls to `analogRead()`

More info on the math can be found here: [wikipedia][wikipedia]

<TODO: Insert picture of test measurement setup>

## Components

I bought everything I needed on Amazon Prime.  If you're not a Prime subscriber you might end up paying a bit more.  For some items, it made sense to buy more than one unit, since it was only a few dollars more and it's always nice to have a spare in case you fry one.  Also, if you want to make a second (or third) unit you already have what you need.

|Item|Total Cost|Unit Cost|
|----|---------:|--------:|
|[Arduino Pro Mini (3-pack)][1]                    | $11|  $4|
|[3.3V/5V Power Supply Module (5-pack)][2]         |  $9|  $2|
|[ESP8266 Esp-01 (4-pack)][3]                      | $14|  $4|
|[2x SCT-013-000 Non-invasive AC Current Sensor][4]| $26| $13|
|[16x2 LCD Display Module][5]                      |  $6|  $6|
|[110VAC->9V Adapter][6]                           |  $6|  $6|
|[10pcs 4x6cm Double Side Prototype PCB][7]        |  $7|  $1|
|[10pcs 3.5mm female PCB mounted jacks][8]         |  $8|  $2|
|Misc: Capacitor, resistors, button, wires         | ~$2|  $2|
|**Total**                                         |**$89**|**$40**|

[1]: https://www.amazon.com/gp/product/B00SFALVA0
[2]: https://www.amazon.com/gp/product/B01H6QNLEC
[3]: https://www.amazon.com/gp/product/B01EA3UJJ4
[4]: https://www.amazon.com/gp/product/B01C5JL5IY
[5]: https://www.amazon.com/gp/product/B019D9TYMI
[6]: https://www.amazon.com/gp/product/B06X43R1Z5
[7]: https://www.amazon.com/gp/product/B00CGV6TZG
[8]: https://www.amazon.com/gp/product/B008SNZUYC

## Hardware

Here's how I wired everything together:

![][power-meter]

I used the [Digikey Scheme-it software][digikey] to create the hardware wiring diagram above.  It runs in a browser!

There's a few things to point out:

* The [power supply unit][2] can take an unregulated input voltage (6.5 - 12V), or USB input power.  The 5V supply is used for the Arduino and the 3.3V is used for the ESP8266.
* I opted not to use a an I2C 16x2 LCD display, because there were enough I/O pins available on the Arduino to drive it directly, and not having the I2C circuitry saves a little power.
* The CT1 and CT2 'inductor' like symbols represent the [SCT-013][4] current sensors
* The 2, 3, 4, and 5 input pins on the Arduino are attached in reverse order.  This just makes the diagram look a bit nicer.

## Software

### User Interface

The information shown on the 2 line LCD cycles through this information, when the button is pressed:

* Total Power
* Line Power
* Total Current
* Line Current
* Total Energy Usage
* Local IP address
* Advanced Options - \<\<Long Press\>\>

If a long press is detected when 'Advanced Options' is shown these additional screens are included in the rotation:

* WiFi Transmit - On/Off
* Samples Transmit - On/Off
* Reset Data

In order to change the On/Off status or reset the data a long press on the button must be performed.


|     |     |
|:---:|:---:|
| [![][img20] Total Power][img20] | [![][img21] Line Power][img21] |
| [![][img22] Line Current][img22] | [![][img23] Total Energy Use since last reset][img23] |
| [![][img24] Local IP Address][img24] | [![][img25] Long Press for Advanced Options][img25] |
| [![][img26] WiFi transmission on/off][img26] | [![][img27] Transmit each sample value][img27] |
| [![][img29] Transmission in progress][img29] | |

### Wifi Communication
### Server
### Client/Browser

The browser display/interface currently looks like this:

![][browser-ui]

The black and red lines represent the power drawn from each of the two phases coming into the house.  The left and right arrow buttons on top will go to the previous day or the next day.  If you're at todays day, new data will be fetched.  No need to hit 'refresh'!

Here's some of the things that can be derived from this:

* The 'black' standby consumption is roughly 125W, and the 'red' one is 25W.  This is the parasitic draw by all the devices that are in standby mode (TV, microwave, computers, webcams, etc)

* The up/down square pattern on the 'black' line is the compressor in the fridge turning on and off.

* I woke up around 6am and started turning lights and the TV on.  The TV is drawing from the 'black' line.

* The spike in the 'red' is my CO2 fan on my water heater.  The water heater runs for about 10 minutes

* The smaller spike after the water heater spike is the garage door opening.

## 3D Printed Box

I used openSCAD to design a box.  The .scad file and rendered .stl files are in this repo.  The box with the cutouts for the display, button, and various connectors looks like this:

![][3d-box]

## Lessons Learned

## Next Version

* Use an OLED 128x64 display instead
* Use an ESP32 module instead
* More inputs (lower amps)
* Better fault toleracy (after power outage)
* Turn off the LED backlight after inactivity

## Add on devices

## Problems 

* **Flaky power supply**
  
  Relying on the power supply from the USB caused some flakiness in the analog pin readings.  Also, in order to supply power to the ESP-01, I needed a 3.3V supply.  The Arduino Pro Mini doesn't have a 3.3V voltage regulator, like the Uno does.  The very inexpensive [power supply module][2] fit the bill, and seems to deliver a stable enough power to ensure stable readings.

* **Baud rate of ESP8266 ESP-01**

  I had to use a couple of digital I/O pins on the Arduino for the serial communication between the Arduino and the ESP-01 module.  The regular TX/RX pins are needed for flashing the software onto the Arduino.  When using the SoftwareSerial module with digital I/O pins it wasn't possible to get a reliable communication link to the ESP-01 at the default 115200 baud rate.  Since the amount of data exchange between the Arduino and the ESP-01 is fairly limited I set the baud rate to a safe 9600 baud.

* **Running out of memory on the Arduino**

  The Arduino has only 2K worth of RAM memory.  The RAM is used for all global data, stack data, and even constant string data.  The Atmel 328 gcc compiler does a good job eliminating all dead code and data, and I'm pretty amazed that you can actually run a 1000+ lines C++ program in that small amount of RAM.  It does take some fiddling to make it fit though.

  * *Avoid using the `new` operator to allocate objects:*
  * *Move all strings to flash memory:*
  * *Avoid using `String` objects:*
  * *Insert probes to check the stack usage:*
  * *Use smaller integers whenever possible:*

## Resources



## Images

|     |     |
|:---:|:---:|
| [![][img01] Top view][img01] | [![][img02] Power supply connectors][img02] |
| [![][img03] CT connectors][img03] | [![][img04] Antenna part of the ESP8266][img04] |
| [![][img05] Bottom view][img05] | [![][img06] Bottom view without Arduino][img06] |
| [![][img07] Bottom view without Arduino and ESP8266][img07] | [![][img08]][img08] |
| [![][img09] Button and LCD attached to the top][img09] | [![][img10]][img10] |
| [![][img11] PCB underside][img11] | [![][img16] Box with lid][img16] |

<!--
-- Images
-->
[img01]: ./images/IMG_6468.JPG
[img02]: ./images/IMG_6469.JPG
[img03]: ./images/IMG_6470.JPG
[img04]: ./images/IMG_6471.JPG
[img05]: ./images/IMG_6472.JPG
[img06]: ./images/IMG_6473.JPG
[img07]: ./images/IMG_6474.JPG
[img08]: ./images/IMG_6475.JPG
[img09]: ./images/IMG_6476.JPG
[img10]: ./images/IMG_6477.JPG
[img11]: ./images/IMG_6478.JPG
[img12]: ./images/IMG_6479.JPG
[img13]: ./images/IMG_6480.JPG
[img14]: ./images/IMG_6481.JPG
[img15]: ./images/IMG_6482.JPG
[img16]: ./images/IMG_6483.JPG
[img20]: ./images/IMG_20170805_083951.jpg
[img21]: ./images/IMG_20170805_084005.jpg
[img22]: ./images/IMG_20170805_084014.jpg
[img23]: ./images/IMG_20170805_084026.jpg
[img24]: ./images/IMG_20170805_084036.jpg
[img25]: ./images/IMG_20170805_084043.jpg
[img26]: ./images/IMG_20170805_084055.jpg
[img27]: ./images/IMG_20170805_084103.jpg
[img28]: ./images/IMG_20170805_084111.jpg
[img29]: ./images/IMG_20170805_084158.jpg
[img30]: ./images/IMG_20170723_183119.jpg
[i-rms]: ./images/i-rms.svg
[power]: ./images/power.svg
[power-meter]: ./images/power-meter.svg
[openenergymonitor-diagram]: https://openenergymonitor.org/forum-archive/sites/default/files/Arduino%20AC%20current%20input%20A.png
[3d-box]: ./images/box.png
[browser-ui]: ./images/browser-ui.png

<!--
-- Links
-->
[wikipedia]: https://en.wikipedia.org/wiki/Root_mean_square#Average_electrical_power
[openenergymonitor]: https://openenergymonitor.org
[digikey]: https://www.digikey.com/schemeit

<!---
Here's a comment
-->


```
here's some code
```

Here's a table:

|1|2|
| | |
|adadf|adfadf|


