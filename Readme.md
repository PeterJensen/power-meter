# Power Meter

**Author: Peter Jensen**

This is my first attempt at a 'useful' IoT project.  I've had an Arduino Uno for a while, but didn't really use it for anything, except going through a blinking LED tutorial.  It was time to bring it to use.  I find it rewarding to think up a project idea and then seek out the information you need when you need it.  Of course, often times you find that others have done something similar, but that's OK, you'll still learn from the process.

I wanted a device to measure power consumption in my house and have it report it up to a 'cloud' service when things changed; lights turning on/off, etc.  Also, I wanted a display on the box, so I could read what the current power consumption was.

## Prototyping

I started out by hooking up a current sensors to an analog input pin; like this:

![](https://openenergymonitor.org/forum-archive/sites/default/files/Arduino%20AC%20current%20input%20A.png)\
Source: [https://openenergymonitor.org](https://openenergymonitor.org)

For calibration, I was using a multimeter that was capable of measuring up to 10A and hooked it up to a wire that was powering a three bulb lamp.  Each of the bulbs could be turned on and off individually.  With 60W bulbs in the sockets, I could get samples for 60W, 120W, and 180W.  

**Note, the current sensor must only go around one wire, in order for the AC current going through that wire to induce a current in the sensor.**

If all you do in the `loop()` function in your Arduino sketch is to call `analogRead()` and store the result away in a memory buffer, you will be able to get about 100 samples during one cycle (60Hz - 16.7ms).  The Arduino is running at 16MHz.  That should be enough to compute the root-mean-square (rms) value of the input fairly accurately.

Why do you need the rms value? The input on the analog input pin is the voltage drop over the load/burden resistor.  That voltage drop is directly proportional with the current going through the power line being measured at the time of measurement.

When the current and voltage are varying over time, the power is computed as the average power over the period of the 60Hz sinus wave, using the RMS values of I and V:

<img src="https://latex.codecogs.com/svg.latex?P_{AVG} = I_{RMS}V_{RMS}" style="height:20px; margin-left:50px">

The V<sub>RMS</sub> component is kept constant by the power company at 120V, so the interesting one is the I<sub>RMS</sub> value.  The Arduino analog-to-digital converters are 10-bits, and the circuitry is designed so that the mid-point of the sinus input is 2.5V, which should result in a reading of ~511.  If the readings are equally spaced in time, the I<sub>RMS</sub> value can be computed as:

<img src="https://latex.codecogs.com/svg.latex?I_{RMS} = K\sqrt{\frac{1}{N}\sum_{i=1}^{N}{(v_i - 511)^2}}" style="height:80px;margin-left:50px">

where N is the number of samples it takes to cover the full period of the 60Hz sinus wave, K is a calibration constant that will be determined after measuring the actual I<sub>RMS</sub> values with a multimeter, and v<sub>i</sub> are the values returned by the calls to `analogRead()`

\<TODO: Insert picture of test measurement setup>

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

## Software

### User Interface
### Wifi Communication
### Server
### Client/Browser


## 3D Printed Box

## Lessons Learned

## Next Version

* Use an OLED 128x64 display instead
* Use an ESP32 module instead
* More inputs (lower amps)
* Better fault toleracy (after power outage)

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
| [![][img01] subtext1][img01] | [![][img02] subtext2][img02] |
| [![][img01] subtext1][img01] | [![][img02] subtext2][img02] |

[img01]: ./images/IMG_6477.JPG
[img02]: ./images/IMG_6477.JPG

<!---
adfaadfadfadfasdfasdfadsf
-->

[Some Link](http://www.danishdude.com)


```
here's some code
```

Here's a table:

|1|2|
| | |
|adadf|adfadf|


