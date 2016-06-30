# NODEMCU

[NodeMC](http://nodemcu.com/index_en.html) is based on the interpreted Lua scripting language and allows event-based programming of the ESP8266. Firmware binaries can already be [customized and built online](http://nodemcu-build.com/), so strictly speaking, you don't have to compile from source. However, since we're not scared of getting our hands dirty, we're going to learn how to strip away the cruft with more precision and also give ourselves more control and transparency over what's actually going in your hardware. So, with that said, the first and possibly least exciting part of dealing with a new processor is getting the toolchain and its dependencies set up so it can actually be programmed. This tutorial will focus on downloading the NodeMCU firmware source and building tools from github, selecting only the specific modules we want, compiling the module, and flashing it to the ESP8266 from a Linux environment. The commands apply specifically to Ubuntu 15.04 Vivid Vervet but could very easily be adapted to other versions and distributions -- just be aware that there could be subltle differences.

#### Getting Ready to Get Ready 

1. In case you use [NodeMCU-Devkit](https://en.wikipedia.org/wiki/NodeMCU#/media/File:NodeMCU_DEVKIT_1.0.jpg), support for the CH341 USB-serial bridge is built into the mainline Linux kernel but Windows and OSX users need to install drivers. Verify the device is functioning by plugging in the NodeMCU-Devkit and typing to following into the terminal:

```sh
dmesg | grep tty
``` 

You should see something like:

```sh
[50859.311796] usb 2-1.2: ch341-uart converter now attached to ttyUSB0
``` 

(In case you use a custom UARt interface be sure chipset i soported on your OS and maje the same testing steps.)
 

This is the serial port we will be communicating with when we send code to the ESP8266. 

2. Install dependencies for compilation and communication to ESP8266:

```sh
sudo apt-get install make unrar autoconf automake libtool gcc g++ gperf flex bison texinfo gawk ncurses-dev libexpat-dev python python-serial sed git libtool-bin screen help2man
```

The software development kit that we're going to use is the [esp-open-sdk](https://github.com/pfalcon/esp-open-sdk). It contains the tools to cross compile code for the Xtensa processor as well as the hardware abstraction library and esptool, a python script that can convert and upload code to the device.

3) Create a user-owned directory and clone the esp-open-sdk and nodemcu-firmware repository from GitLab:

```sh
mkdir ~/.opt && cd ~/.opt
git clone --recursive https://github.com/Datumatik/nodemcu-build-setup.git # --recursive since this repo contains more repos
```

4) Compile a standalone version of the open SDK with vendor SDK files built in. Then add the SDK's bin directory to your PATH environment variable:

```sh
cd nodemcu-build-setup/esp-open-sdk
make STANDALONE=y
```

![alt tag](http://www.allaboutcircuits.com/uploads/articles/long_time_terminal.png)


The **make** command will take a while so go grab a cup of coffee or something while you wait. It's worth noting that while the standalone SDK is slightly easier to write code for locally, there are a few caveats. See the note from pfalcon [here](https://github.com/pfalcon/esp-open-sdk#building) on portability and licenses. Then add the newly built tools' binary folder to the PATH environment variable. That way your system knows where to look when you try to compile Xtensa code.

```sh
echo 'PATH=$PATH:/opt/nodemcu-build-setup/esp-open-sdk/xtensa-lx106-elf/bin' >> ~/.profile
PATH=$PATH:~/.opt/nodemcu-build-setup/esp-open-sdk/xtensa-lx106-elf/bin
 ```

Also:

```sh
cd ~/.opt/nodemcu-build-setup/esp-open-sdk
PATH=$PATH:$PWD/xtensa-lx106-elf/bin
```

 Now that we have the tools to make everything, now we can actually get the NodeMCU firmware and configure it to our specific needs.
 
 5. Change the Nodemcu firmware folder:

```sh
cd ~/.opt/nodemcu-build-setup/nodemcu-firmware
  ```

Nuts and bolts time! A common problem many users of NodeMCU are discussing is running out of room on the chip for program. Since the ESP8266 only has 64 KiB of instruction RAM and 96 KiB of data RAM, space is more or less at a premium. Why would you install I2C and u8g graphics libraries if you have no need for them for this project? By default, all the modules are built, so we're going to comment out the ones we don't want at this time: 

6. Select modules to compile and install

Using your favorite editor, open **../nodemcu-firmware/app/include/user_modules.h** 

Depending on the project you are working on, you should enable only the [modules](http://nodemcu.readthedocs.io/en/master/) you are going to use:

```c
#ifndef __USER_MODULES_H__
#define __USER_MODULES_H__

#define LUA_USE_BUILTIN_STRING          // for string.xxx()
#define LUA_USE_BUILTIN_TABLE           // for table.xxx()
#define LUA_USE_BUILTIN_COROUTINE       // for coroutine.xxx()
#define LUA_USE_BUILTIN_MATH            // for math.xxx(), partially work
// #define LUA_USE_BUILTIN_IO                   // for io.xxx(), partially work

// #define LUA_USE_BUILTIN_OS                   // for os.xxx(), not work
// #define LUA_USE_BUILTIN_DEBUG
#define LUA_USE_BUILTIN_DEBUG_MINIMAL // for debug.getregistry() and debug.traceback()

#ifndef LUA_CROSS_COMPILER

// The default configuration is designed to run on all ESP modules including the 512 KB modules like ESP-01 and only
// includes general purpose interface modules which require at most two GPIO pins.
// See https://github.com/nodemcu/nodemcu-firmware/pull/1127 for discussions.
// New modules should be disabled by default and added in alphabetical order.
#define LUA_USE_MODULES_ADC
//#define LUA_USE_MODULES_AM2320
//#define LUA_USE_MODULES_APA102
#define LUA_USE_MODULES_BIT
//#define LUA_USE_MODULES_BMP085
//#define LUA_USE_MODULES_BME280
#define LUA_USE_MODULES_CJSON
//#define LUA_USE_MODULES_COAP
#define LUA_USE_MODULES_CRYPTO
#define LUA_USE_MODULES_DHT
#define LUA_USE_MODULES_ENCODER
#define LUA_USE_MODULES_ENDUSER_SETUP // USE_DNS in dhcpserver.h needs to be enabled for this module to work.
#define LUA_USE_MODULES_FILE
#define LUA_USE_MODULES_GPIO
#define LUA_USE_MODULES_HTTP
//#define LUA_USE_MODULES_HX711
//#define LUA_USE_MODULES_I2C
#define LUA_USE_MODULES_MDNS
//#define LUA_USE_MODULES_MQTT
#define LUA_USE_MODULES_NET
#define LUA_USE_MODULES_NODE
//#define LUA_USE_MODULES_OW
//#define LUA_USE_MODULES_PERF
//#define LUA_USE_MODULES_PWM
//#define LUA_USE_MODULES_RC
//#define LUA_USE_MODULES_ROTARY
//#define LUA_USE_MODULES_RTCFIFO
//#define LUA_USE_MODULES_RTCMEM
//#define LUA_USE_MODULES_RTCTIME
//#define LUA_USE_MODULES_SIGMA_DELTA
#define LUA_USE_MODULES_SNTP
//#define LUA_USE_MODULES_SPI
//#define LUA_USE_MODULES_STRUCT
#define LUA_USE_MODULES_TMR
//#define LUA_USE_MODULES_TSL2561
//#define LUA_USE_MODULES_U8G
#define LUA_USE_MODULES_UART
//#define LUA_USE_MODULES_UCG
#define LUA_USE_MODULES_WIFI
//#define LUA_USE_MODULES_WS2801
//#define LUA_USE_MODULES_WS2812


#endif  /* LUA_CROSS_COMPILER */
#endif  /* __USER_MODULES_H__ */
```

With the Devkit Version 0.9, we also have to modify the **../nodemcu-firmware/app/include/user_config.h** to enable some v0.9-specific settings. Find line 4 and uncomment it so it looks like this: 

``` c
#define DEVKIT_VERSION_0_9 1     // define this only if you use NodeMCU devkit v0.9
```

#### Let's Build It Already! #### 

7. Compile NodeMCU:

``` sh
cd ~/.opt/nodemcu-build-setup/nodemcu-firmware # make your way back to the firmware dir if you weren't there already
make # this might take a while
```
 
#### Flashing ESP8266 ####

8. Add user to dialout group and flash the firmware to the device:

Before you can upload the compiled image or communicate with the device, you have to have permission to access to the serial port. **/dev/ttyUSB0** is part of the dialout group, so to add yourself, type:

``` sh
sudo adduser $USER dialout
 ```

Now you should be able to upload the NodeMCU image to your device:

``` sh
make flash # this assumes the tty device is /dev/ttyUSB0
 ```

Or also:

``` sh
cd ~/.opt/nodemcu-build-setup/nodemcu-firmware/
./tools/esptool.py --port /dev/ttyUSB0 write_flash 0x00000 ./bin/0x00000.bin 0x10000 ./bin/0x10000.bin
 ```
