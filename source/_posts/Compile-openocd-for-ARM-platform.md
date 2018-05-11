---
title: Compile openocd for ARM platform
abbrlink: '233799e4'
date: 2018-05-11 16:11:20
tags:
---

## Background
The latest openocd we can get from the apt repository of Raspberry Pi is only up to 0.9.0 as of writing. I'd like to use my Raspberry Pi to download FPGA bit file, but openocd 0.9.0 has some problem to support jlink debugger. See errors below if it's like your case.
```
Pi@Raspberrypi:~/openocd-workdir $ /usr/bin/openocd -f interface/jlink.cfg -f xilinx-xc6v.cfg -f openocd-xc6v.cfg
Open On-Chip Debugger 0.9.0 (2018-01-22-06:14)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "jtag". To override use 'transport select <transport>'.
xc6v_program_iprog
adapter speed: 5000 kHz
Info : J-Link ARM V8 compiled May 27 2009 17:31:22
Info : J-Link caps 0xb9ff7bbf
Info : J-Link hw version 80000
Info : J-Link hw type J-Link
Info : J-Link max mem block 9752
Info : J-Link configuration
Info : USB-Address: 0x0
Info : Kickstart power on JTAG-pin 19: 0xffffffff
Info : Vref = 2.523 TCK = 1 TDI = 0 TDO = 1 TMS = 0 SRST = 1 TRST = 1
Info : J-Link JTAG Interface ready
Info : clock speed 5000 kHz
Info : JTAG tap: xc6v.tap tap/device found: 0x84250093 (mfg: 0x049, part: 0x4250, ver: 0x8)
Warn : gdb services need one or more targets defined
openocd: jlink.c:1474: jlink_tap_append_step: Assertion `index_var < JLINK_TAP_BUFFER_SIZE' failed.
Aborted
```

Howerver, it works well on the PC with the openocd 0.10.0 from [GNU MCU Eclipse OpenOCD](https://gnu-mcu-eclipse.github.io/openocd/), so I decided to compile it for my Raspberry Pi.

Unfortunately, GNU MCU Eclipse OpenOCD doesn't support compiling for arm, after some searching on the web, I found an openocd build [repository](https://github.com/arduino/OpenOCD-build-script) for Arduino which bundled all necessary libraries for building. The openocd included is a bit of old, we can replace it with the latest one.

## Cross Compiling
1. Check out the latest openocd code
   ```
   $ git clone git://git.code.sf.net/p/openocd/code openocd
   ```
2. Check out the openocd build script
   ```
   $ git clone https://github.com/arduino/OpenOCD-build-script.git openocd-build
   ```
3. Copy it to the build folder, replace the original `OpenOCD` folder
   ```
   $ mv openocd-build/OpenOCD{,-old}
   $ cp openocd openocd-build/OpenOCD -r
   ```
4. Add the cross compile arguments in the `compile_unix_openocd.sh`
   ```
   # Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA  02110-1301, USA.
   ARCH=`gcc -v 2>&1 | awk '/Target/ { print $2 }'`

   # Add two new arguments, check uname -m for your build PC's architecture
   BUILD="x86_64-pc-linux-gnu"
   HOST="arm-linux-gnueabihf"
   ```
5. Append the cross compile flags to all configuration steps, like this
   ```
   ...
   ./configure --enable-static --disable-shared --disable-blkid --disable-kmod  --disable-manpages --host=$HOST --build=$BUILD
   ...
   ```
6. Build it, install any missed packages if the build process reports, like gperf
   ```
   $ ./compile_unix_openocd.sh
   ```

However the binary generated this way is failed to run on the raspberry pi with a segment fault, not sure why. Instead, I went for a native building.

## Native Compiling On Raspberry Pi
Nothing need to change except updating the OpenOCD code. The new generated openocd solved the problem in the first place.