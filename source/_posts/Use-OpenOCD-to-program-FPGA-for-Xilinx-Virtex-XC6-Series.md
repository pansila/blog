---
title: Use OpenOCD to program FPGA for Xilinx Virtex-6 XC6 Series
abbrlink: 67d7a1ab
date: 2018-05-04 14:18:53
tags:
---

_Note: Currently it's only tested for Virtex-6 XC6VLX240T, suppose to be able to extend for more virtext-6 FPGAs_

## Work Environment
1. Windows 7 64bit
2. [Xilinx Virtex-6 FPGA ML605 Evaluation Kit](https://www.xilinx.com/products/boards-and-kits/ek-v6-ml605-g.html)
   ![](https://i.loli.net/2018/05/04/5aebfeb3647e0.jpg)

## Installation Guide

1. Unzip the [OpenOCD binary zip file](https://github.com/gnu-mcu-eclipse/openocd/releases) to anywhere you're used to put a tool, like `d:\tools\`
2. Add the OpenOCD executable path to the `PATH` environment, reboot or logout to put it into effect

    _Note: if you're using `cmder`, we can make it without a rebooting_

    * Go to Setting->Startup->Environment
    * Set the `PATH` environment: `set PATH=D:\tools\openocd\bin;%PATH%`
	* Open a new console to load the new environments
3. Install USB JTAG cable driver
   The adapter DLC9G that comes with Xilinx Virtex-6 Evaluation Kit is not supported by OpenOCD due to its proprietary protocol, here we'll use jlink instead.
   * Download [zadig](http://zadig.akeo.ie/)
   * Go to Options->List All devices
   * Select the jlink device, vid: 1366, pid: 0101
   * Click "Replace Driver".

     ![](https://i.loli.net/2018/05/04/5aebff31927f0.jpg)
   * Jumper wiring jlink pins to jtags pins on board

     ![](https://i.loli.net/2018/05/04/5aec028dd3c0e.jpg)
     ![](https://i.loli.net/2018/05/04/5aec02c03db0d.jpg)

## Programming
    
  1. Put the OpenOCD [configuration](#openocd-configuration) file in the work dir together with the bit file (file content attached at the end).
     ```
     $ cd d:\workdir
     $ dir
     openocd-xc6v.cfg
     xilinx-xc6v.cfg
     bitstream.bit
     ```
  2. Modify the path to the bit file in the configuration
     ```
     ...
     init
     xc6v_program xc6v.tap
     pld load 0 bitstream.bit   #### this one ####
     exit
     ...
     ```
  2. Run openocd in the cmd.exe or a cmder console
     ```
     $ openocd -f interface/jlink.cfg -f xilinx-xc6v.cfg -f openocd-xc6v.cfg
     ```
  3. Waiting for the programming done if no error raised
     ```bash
     $ openocd -f interface/jlink.cfg -f xilinx-xc6v.cfg -f openocd-xc6v.cfg
     GNU MCU Eclipse 64-bits Open On-Chip Debugger 0.10.0+dev-00404-g20463c28 (2018-01-23-12:30)
     Licensed under GNU GPL v2
     For bug reports, read
             http://openocd.org/doc/doxygen/bugs.html
     Info : auto-selecting first available session transport "jtag". To override use 'transport select <transport>'.
     xc6_program_iprog
     adapter speed: 5000 kHz
     Info : J-Link ARM V8 compiled May 27 2009 17:31:22
     Info : Hardware version: 8.00
     Info : VTarget = 2.478 V
     Info : clock speed 5000 kHz
     Info : JTAG tap: xc6.tap tap/device found: 0x84250093 (mfg: 0x049 (Xilinx), part: 0x4250, ver: 0x8)
     Warn : gdb services need one or more targets defined
     loaded file bitstream.bit to pld device 0 in 46s 322000us
     ```
---
## OpenOCD Configuration (openocd-xc6v.cfg)
```tcl
adapter_khz 5000

init
xc6v_program xc6v.tap
pld load 0 bitstream.bit
exit
```
## OpenOCD Target Configuration (xilinx-xc6v.cfg)
```tcl
if { [info exists CHIPNAME] } {
	set _CHIPNAME $CHIPNAME
} else {
	set _CHIPNAME xc6v
}

set XC6V_CFG_IN 0x05
set XC6V_JSHUTDOWN 0x0d
set XC6V_JPROGRAM 0x0b
set XC6V_JSTART 0x0c
set XC6V_BYPASS 0x3f

# Device ID Code IR Length Part Name
# 84250093 10 XC6VLX240T
# 44244093 10 XC6VLX75T
# c2a96093 10 XC6VLX50T
# 442a8093 10 XC6VHX380T

jtag newtap $_CHIPNAME tap -irlen 10 -ignore-version \
	-expected-id 0x84250093
pld device virtex2 $_CHIPNAME.tap

proc xc6v_program {tap} {
	global XC6V_JSHUTDOWN XC6V_JPROGRAM XC6V_JSTART XC6V_BYPASS
	irscan $tap $XC6V_JSHUTDOWN
	irscan $tap $XC6V_JPROGRAM
	irscan $tap $XC6V_JSTART
	irscan $tap $XC6V_BYPASS
}
```