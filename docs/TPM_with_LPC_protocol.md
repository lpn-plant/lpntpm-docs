## TPM module with "Low Pin Count Protocol" build on SoC
### Introduction

The main goal of this document is showing the current state of development of TPM 
module with "LPC Protocol" based on SoC (System on Chip), and also documenting
work carried out so far.

## Motivation for doing this project

We stiil see the necessity of making TPM module with "Low Pin Count" protocol.
There is still many computers on the market (especcialy servers) which have 
mother-boards supporting LPC protocol.
We considered possible hardware platforms in the past in this document:

["Target TPM hardware interfaces"](https://lpntpm.lpnplant.io/hardware_interfaces/#low-pin-count-interface)

Outside of fast "SPI Bus" and even "I2C" protocol which are relatively easy to 
implement on MCU haedware just "LPC protocol" make it difficult to implement on 
MCU.

We made an attempt of implementation "LPC protocol" (LPC Peripheral) on MCU, but 
this attempt hasn't been successfully completed. "ESP32" board has been chosen as 
hardware platform. This hardware was Dual Core Tensilica LX6 with 240 MHz clock 
and 512 KB SRAM, which is quite powerful MCU. The maximum frequency of switching
GPIO pins we could reach on this hardware was close to 1 MHz. The main reason for
such bad result had been big latency in interrupts handling. We also considered
some versions of STM32 family MCUs, but resulting from documentation that it is 
not possible to get better results than this of "ESP32" hardware.

We came to the conclusion that the best way of implementing "LPC protocol" would
be use of programmable device like a FPGA or CPDL. But we also need CPU for 
implementation of TPM module logic and quite much amount of RAM. So the need of
using together MCU and FPGA ICs in one project causes large size of PCB for TPM 
module. Fortunately in last years appeared ICs called SoC (System on Chip) which
are combinig these two MCU and FPGA in one chip.

## Hardware platform for this project

The most serious issue in our previous attempts to implement TPM module was lack
off enough SRAM memory amount available for MCU, so one of basic requirement for 
new hardware platform is big amount of SRAM memory. The second assumption for new
hardware is existence of programmble logic (FPGA) and fast bus connecting FPGA 
and MCU. The third assumption is efficient MCU with needed peripherals. The last
requirement is possibility of use open-source software for development applications
for this hardware.

Our choice for new hardware platform for this project is 
"QuickLogic EOS™ S3 MCU + eFPGA SoCs". It combine ARM Cortex-M4 MCU with 512 KB 
of SRAM (max. clock frequency is 80 Mhz) and FPGA.FPGA and MCU are connected by 
fast "Wisbone Bus" and can handle interrupts from FPGA to MCU. The MCU has many
peripherals like "SPI controllers", "I2C controllers", UARTs, Timers, Wtchdog timer.

"Quicklogic EOS S3 MCU + eFPGA SoC" had been chosen for hardware part of this 
project because it fulfills all requirements related to amount of RAM memory and 
overall performance. Here is link to description of this SoC:

[Quicklogic EOS S3 SoC](https://www.mouser.pl/new/quicklogic/quicklogic-eos-s3-mcu-efpga-socs/)

We carried out all development on open-hardware board called "Sparkfun QuickLogic
Thing Plus - EOS S3". This board is based on Quicklogic "EOS S3" SoC - here is
link to this product:

["Sparkfun QuickLogic Thing Plus - EOS S3"](https://www.sparkfun.com/products/17273)

Here is hardware manual from Quicklogic company:

["QuickLogic EOS S3 Ultra Low Power multicore MCU datasheet"]
(https://cdn.sparkfun.com/assets/7/a/c/c/e/QL-EOS-S3-Ultra-Low-Power-multicore-MCU-Datasheet-v3_3d.pdf)

And here is "Hookup Guide" for "Sparkfun QuickLogic Thing Plus - EOS S3" board:

["QuickLogic Thing Plus (EOS S3) Hookup Guide"]
(https://learn.sparkfun.com/tutorials/quicklogic-thing-plus-eos-s3-hookup-guide#hardware-overview)

Below is picture of circuit used for development of this project:

![Circuit used for development](images/QuickLogicThingPlus.png)


It consist of such parts:
+ "Sparkfun QuickLogic Thing Plus - EOS S3" board
+ breadboard
+ power adapter
+ J-Link JTAG programmer/debugger
+ USB 2 UART converter


### Software needed for development of this project

All development - both FPGA Verilog RTL part and ARM Cortex-M4 MCU development
had been carried out using "Qiucklogic QORC SDK". "QORC SDK" consist of the 
following parts:
+ Symbiflow package for making FPGA synthessis (Yosys) and "place and route" tool
  for implementation and generating bitstream for FPGA
+ GCC-cross compiler for ARM Cortex
+ FreeRTOS
+ Zephyr-RTOS

"QORC SDK" is in all it's parts open-source.

"QORC SDK" is only one software tool needed for development of all parts of 
application for SoC "EOS S3". However there is also very useful software for making 
Verilog RTL code simulation. We used for simulating this project open-source 
Verilog simulator called "Icarus Verilog". We also used open-source viewer for 
.vcd files called "GTKWave".

Here is link to "Icarus Verilog":

["Icarus Verilog simulator"](http://iverilog.icarus.com/)

, and here is link to "GTKWave" application:

["GTKWave" application](http://gtkwave.sourceforge.net/)






