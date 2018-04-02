---
title: Use OpenOCD to program FPGA for Xilinx Virtex XC7 Series
date: 2018-04-02 14:19:26
tags:
---
## Work Environment
1. Windows 7 64bit
2. [Xilinx Virtex-7 FPGA VC709 Connectivity Kit](https://www.xilinx.com/products/boards-and-kits/dk-v7-vc709-g.html)
   ![](https://i.loli.net/2018/04/02/5ac1cd20310a8.jpg)

## Installation Guide

1. Unzip the [OpenOCD binary zip file](https://github.com/gnu-mcu-eclipse/openocd/releases) to anywhere you're used to put a tool, like `d:\tools\`
2. Add the OpenOCD executable path to the `PATH` environment, reboot or logout to put it into effect

    _Note: if you're using `cmder`, we can make it without a rebooting_

    * Go to Setting->Startup->Environment
    * Set the `PATH` environment: `set PATH=D:\tools\openocd\bin;%PATH%`
	* Open a new console to load the new environments
3. Install USB JTAG cable driver
   * Download [zadig](http://zadig.akeo.ie/)
   * Go to Options->List All devices
   * Select the first cable port (interface 0 or Port A), vid: 0403, pid: 6010
   * Click "Replace Driver".

     ![](https://i.loli.net/2018/04/02/5ac1cbc3a84f6.png)

## Programming
    
   Target Xilinx Virtex XC7 series as of now

  1. Put the OpenOCD [configuration](#openocd-configuration) file in the work dir together with the bit file (file content attached at the end).
     ```
     $ cd d:\workdir
     $ dir
     openocd.cfg
     a-bit-file.bit
     ```
  2. Modify the path to the bit file in the configuration
     ```
     ...
     init
     xc7_program xc7.tap
     pld load 0 a-bit-file.bit   #### this one ####
     exit
     ...
     ```
  2. Run openocd in the cmd.exe or a cmder console
     ```
     $ openocd --command "tcl_port disabled" --command "telnet_port disabled"
     ```
     If the configuration file is not in the current work directory, we can specifiy it in the command line
     ```
     $ openocd -f <path_to_config> --command "tcl_port disabled" --command "telnet_port disabled"
     ```
  3. Waiting for the programming done if no error raised
     ```bash
     $ openocd
     GNU MCU Eclipse 64-bits Open On-Chip Debugger 0.10.0+dev-00404-g20463c28 (2018-01-23-12:30)
     Licensed under GNU GPL v2
     For bug reports, read
             http://openocd.org/doc/doxygen/bugs.html
     none separate
     Info : auto-selecting first available session transport "jtag". To override use 'transport select <transport>'.
     adapter speed: 25000 kHz
     Info : ftdi: if you experience problems at higher adapter clocks, try the command "ftdi_tdo_sample_edge falling"
     Info : clock speed 25000 kHz
     Info : JTAG tap: xc7.tap tap/device found: 0x33691093 (mfg: 0x049 (Xilinx), part: 0x3691, ver: 0x3)
     Warn : gdb services need one or more targets defined
     loaded file a-bit-file.bit to pld device 0 in 16s 603000us
     ```
---
## OpenOCD Configuration
```tcl
source [find interface/ftdi/digilent-hs1.cfg]
source [find cpld/xilinx-xc7.cfg]
#source [find cpld/jtagspi.cfg]
adapter_khz 25000

init
xc7_program xc7.tap
pld load 0 a-bit-file.bit
exit

## programming SPI Flash
# init
# jtagspi_init 0 xc7_bscan_spi.bit
# jtagspi_program a-bit-file.bit 0
# xc7_program xc7.tap
# pld load 0 a-bit-file.bit
# exit
```