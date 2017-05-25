# BeagleBone FreeBSD Tinderbox Log

# Goal

Build a physical [Tinderbox](https://ci.freebsd.org/tinderbox/) that uses LED lights to display the current FreeBSD build status.

# Prerequisites

- A working installation of FreeBSD
- BeagleBone Green with a 4GB micro-SD card and a serial cable
- An addressible LED RGB strip [Sparkfun APA102](https://www.sparkfun.com/products/14015)

# Steps

## Install FreeBSD on the micro-SD card

### 1. Build or download a FreeBSD image

To get started, you can download an image from the [FreeBSD Snapshot](https://download.freebsd.org/ftp/snapshots/ISO-IMAGES/) site with filename labeled as BeagleBone, in this case we download:

```
FreeBSD-12.0-CURRENT-arm-armv6-BEAGLEBONE-20170519-r318502.img.xz
```

You can always choose to build FreeBSD from source code if you want to experience the latest changes for the support of BeagleBone and are comfortable with the process. [Crochet](https://github.com/freebsd/crochet) is the tool to use, and you can find a detailed guide on GitHub.

### 2. Install the image

[dd(1)](https://www.freebsd.org/cgi/man.cgi?dd(1) utility is used for raw data copying such as, in this case, initializing a disk from a raw image.

We specify if (input file), of (output file) and bs (block size of copying). These arguments should be changed to match the actual file and device name.

```bash
$ dd if=FreeBSD-BeagleBone.img of=/dev/da7 bs=8m
```

*Specifying a block size is not necessary, but the default setting may result in very slow operation.*

After the operation finished, you can insert the micro-SD card into BeagleBone.

## Boot the BeagleBone Green

### 1. Connect the serial cable

A serial cable might not be necessary as you can wait until it boots and try to ssh to it (the system configuration might prevent you from sshing as root though). But since BeagleBone Green doesn't have a HDMI output, you can see what is going on through the whole booting process with a serial cable, making it much easier to diagnose if something go wrong.

The serial console of BeagleBone Green is exposed on a 6-pin header. Connect the USB to TTL cable to BeagleBone and computer, then open a terminal window and execute the following:

```bash
$ sudo cu -s 115200 -l /dev/ttyU0 # Or appropriate tty device
```

We use the [cu(1)](https://www.freebsd.org/cgi/man.cgi?cu(1) utility on FreeBSD and specify the line speed of 115200 baud. You won't see any output yet.

### 2. Boot up and log in

The BeagleBone Black can boot from either the onboard eMMC or a micro-SD card, and by default it boots from eMMC. To boot from micro-SD, first hold down the boot switch, the apply power. Don't release the button until you see it starts booting FreeBSD (or count to 5).

After booting, log in as root (default password is root as well).

*Tip: Making a Beaglebone Black always boot from the Micro-SD  
The AM335x chip on board actually boots from the first partition that has the active flag set. After using the "holding the boot button" method described above to boot FreeBSD and log in as root, we are able to turn off the bootable flag of the onboard eMMC to make it always boot from the Micro-SD:*
```bash
$ gpart unset -a active -i 1 mmcsd1
```
*To restore this change and make the eMMC available again do:*
```bash
$ gpart set -a active -i 1 mmcsd1
```
*Alternatively, you can copy the FreeBSD image to eMMC so no pressing the button is needed.*

## Test the GPIO

[gpioctl(8)](https://www.freebsd.org/cgi/man.cgi?query=gpioctl&sektion=8)

## SPI bit banging

Awesome! GPIO is working well with BeagleBone, it's time to start using the addressible LED strip.

The LED RGB strip we got is packed with 60 APA102s and can be controlled with a standard SPI interface, however, at this moment FreeBSD has no userland support for SPI devices. We use [Bit banging](https://en.wikipedia.org/wiki/Bit_banging) to simulate the [SPI Protocol](https://en.wikipedia.org/wiki/Serial_PeripherA complete description of `fbsd_gpio` can be found [here](https://pypi.python.org/pypi/fbsd_gpio/0.4.0).
al_Interface_Bus) as a walkaround.

We use Python and the `fbsd_gpio` python bindings for the code. You may want to install python and pip first, and then `cffi` and `fbsd_gpio` libraries via PyPI.

### 1. Write SPI bit banging functions

Import the library and create a controller:
```python
from fbsd_gpio import GpioController
gpioc = GpioController(0) # Using gpio controller unit 0 (/dev/gpioc0)
```

Set which pins we are using:
```python
SCLK = 2 # CI (Blue)
MOSI = 3 # DI (Green)
```
*Note:  
SCLK: Serial Clock (output from master)  
MOSI: Master Output Slave Input (data output from master), or DI from LED*

Provide SPI init and write functions:
```python
def spi_init():
    gpioc.pin_output(SCLK)
    gpioc.pin_output(MOSI)
    gpioc.pin_set(SCLK, 0)
    gpioc.pin_set(MOSI, 0)


def spi_write_byte(b):
    for i in xrange(8):
        gpioc.pin_set(SCLK, 0)
        gpioc.pin_set(MOSI, int(format(b, "08b")[i]))
        gpioc.pin_set(SCLK, 1)


def spi_write(buf):
    for i in buf:
        spi_write_byte(i)
```

*A complete description of `fbsd_gpio` can be found [here](https://pypi.python.org/pypi/fbsd_gpio/0.4.0).*

### 2. Work with APA102 LEDs

Now we've set up the SPI functions and ready to send SPI data, but what to send in order to light up any LEDs we want? Follow the [APA102 Manual](https://cdn-shop.adafruit.com/datasheets/APA102.pdf), we are able to find out the data format:

![](https://cpldcpu.files.wordpress.com/2014/08/programming.png)

Each update consists of a start frame of 32 zeroes, 32 bits for every LED and an end frame of 32 ones. So our send function will most likely to work as follows:

```python
# Start Frame
spi_write([0b00000000, 0b00000000, 0b00000000, 0b00000000])

#LED Frames
spi_write([0b11111111, 0b00000001, 0b00000000, 0b00000000]) # First LED, brightness full, blue
...

# End Frame
spi_write([0b11111111, 0b11111111, 0b11111111, 0b11111111])
```

Read this article if you want to [Understand the APA102 “Superled”](https://cpldcpu.com/2014/11/30/understanding-the-apa102-superled/) better.


## Display the status

### 1. Get data from Jenkins

Many objects of Jenkins provide remote access APIs. We use the provided Python one to get status of all jobs:

```python
import ast
import urllib
JENKINS_URL = "https://ci.freebsd.org/api/python?tree=jobs[name,color]"
data = ast.literal_eval(urllib.urlopen(JENKINS_URL).read())["jobs"]
```

*The Jenkins API manual can be found [here](https://ci.freebsd.org/api/).*

### 2. Light up the LEDs



*See [my repository](https://github.com/yzgyyang/freebsd-ci-ledstrip) for the full code.*

## Add some final touches

We cut the strip to parts and stick them inside a picture frame. Looking good!

## Further reading

Here is a complete guide of building, installing and updating FreeBSD on a BeagleBone:
[Getting Started with FreeBSD on BeagleBone Black](https://www.freebsdfoundation.org/wp-content/uploads/2015/12/vol1_no1_beaglebone_dkr.pdf)

## Thanks

Ed, Siva
