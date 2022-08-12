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

### Project Github repository

Github repository with source code (Verilog RTL code and MCU C language program)
is located at this URL:

["TPM "LPC protocol" implementation](https://github.com/lpn-plant/lpntpm-lpc)

There are four catalogs in this repository:

+ LPC_Peripheral_Verilog_Implementation
+ LPC_Peripheral_Verilog_Simulation
+ SOC_EOS_S3_Application_Test_comunication
+ SOC_EOS_S3_Application_With_LPC_peripheral

In first catalog is Verilog implementation of "LPC Peripheral" (I/O LPC Cycles). In second catalog are source needed for performing simulation in "Icarus Verilog" ("LPC Peripheral" impl. "LPC Host" impl. and verilog test-bench). In third catalog is application for SOC "EOS S3" (Quicklogic) testing internal comunication between FPGA and MCU parts using in this purpose "Wisbone Bus" and interrupts. In fourth catalog is application for SOC "EOS S3" (Quicklogic) with embedded "LPC Protocol Peripheral" (implemented in FPGA part). Aplication is reading I/O LPC cycles (using I/O ports from FPGA sockets) and displaying these cycles data (LPC Address, LPC Data, and cycle type) by UART in SOC MCU part.

### Verilog (FPGA based) implementation of "Low Pin Count" (LPC) protocol

Beacause in TPM  part of project we only need handle I/O or TPM cycle of LPC protocol
we develop minimalistic implementation of "LPC Peripheral". Before writing RTL 
code in Verilog we study several open-source implementation of LPC protocol.

The basic reference for implementing LPC protocol was Intel company "Intel Low Pin
Count(LPC) interface Specification" document available at such WWW address:

["Intel Low Pin Count(LPC) interface Specification"](https://www.intel.com/content/dam/www/program/design/us/en/documents/low-pin-count-interface-specification.pdf)

These three Verilog RTL modules had been implemented
+ LPC Peripheral (Slave) which is as final step embedded in SoC TPM application
+ LPC Host which is embedded in test-baench for LPC Peripheral
+ LPC Peripheral test-bench (connected together with LPC Host in one Verilog source
  file

This implemeentation of "LPC Peripheral" can handle such types of LPC cycles:
+ I/O LPC cycles (1 byte)
+ TPM LPC cycles (1 byte)

Other cycles of LPC protocol (for example Memory, Firmware, DMA) are not supported
by this implementation.

The Verilog "LPC Peripheral" source is located at this path  in repository:

["LPC Peripheral Verilog source"](https://github.com/lpn-plant/lpntpm-lpc/tree/main/LPC_Peripheral_Verilog_Implementation)

This implementation is based on simple FSM (Finit state machine) handling individual
phases of LPC protocol cycle. There is also code for handling I/O ports and internal
signals states for every phase of LPC cycle.

##### Here is table with all I/O ports of "LPC Peripheral" module
| Direction | Type | Bus    | Port name       |
|-----------|------|--------|-----------------|
|   input   | wire |        |     lpc_lclk    |
|   input   | wire |        |   lpc_lreset_n  |
|   input   | wire |        |   lpc_lframe_n  |
|   inout   | wire | [ 3:0] |    lpc_lad_in   |
|   input   | wire |        |    i_addr_hit   |
|   output  |  reg | [ 4:0] | o_current_state |
|   input   | wire | [ 7:0] |      i_din      |
|   output  |  reg | [ 7:0] |  o_lpc_data_in  |
|   output  | wire | [ 3:0] |  o_lpc_data_out |
|   output  | wire | [15:0] |    o_lpc_addr   |
|   output  | wire |        |     o_lpc_en    |
|   output  | wire |        |   o_io_rden_sm  |
|   output  | wire |        |   o_io_wren_sm  |
|   output  |  reg | [31:0] |      TDATA      |
|   output  |  reg |        |      READY      | 
##### And here is table with I/O ports descriptions of "LPC Peripheral" module
| Port Name          |   Direction    | Description                                                                      |
|--------------------|:--------------:|----------------------------------------------------------------------------------|
| LPC Interface                                                                                                          |
| lpc_lclk           |      Input     | LPC clock (33,3 MHz) from LPC Host                                                   |
| lpc_lreset_n       |      Input     | Active-low reset signal                                                          |
| lpc_lframe_n       |      Input     | Active-low frame signal                                                          |
| lpc_lad_in         | Bi-directional | Multiplexed Command, Address and Data Bus                                        |
| Back-end Interface                                                                                                     |
| i_addr_hit         |      Input     |                                                                                  |
| o_current_state    |     Output     | Current peripheral state                                                         |
| i_din              |      Input     | Data sent when host requests a read                                              |
| o_lpc_data_in      |     Output     | Data received by peripheral for writing                                          |
| o_lpc_data_out     |     Output     | Data sent to host when a read is requested                                       |
| o_lpc_addr         |     Output     | 16-bit LPC Peripheral Address                                                    |
| o_lpc_en           |     Output     | Active-high status signal indicating the peripheral is ready for next operation. |
| o_io_rden_sm       |     Output     | Active-high read status                                                          |
| o_io_wren_sm       |     Output     | Active-high write status                                                         |
| TDATA              |     Output     | 32-bit register with LPC cycle: Address, Data(8-bit) and type of opertion        |
| READY              |     Output     | Active-high status signal indicating that new cycle data is on TDATA             |

As one can see in I/O ports of "LPC Peripheral" module there are four common 
signal of LPC protocol (LPC Host is connected to Peperipheral by these lines). 
These signals are:
+ lpc_lclk
+ lpc_lreset_n
+ lpc_lframe_n
+ lpc_lad_in  (this 4-bit bi-directional multiplexed bus)

Other signals are used for control the module and displaying information.

In this path in github repository is located Verilog file "lpc_peri_tb.v" with 
test-bench for performing simulation of "LPC Peripheral" module:
["LPC Peripheral Verilog source"](https://github.com/lpn-plant/lpntpm-lpc/tree/main/LPC_Peripheral_Verilog_Implementation)

In this file (from line 59) is fragment of code setting the name of .vcd dump file
with waveforms :
```verilog
initial
begin
 	// Initialize
    $dumpfile("lpc_peri_tb.vcd");
    $dumpvars(0,lpc_peri_tb);

```

In line (106) there is for loop generating 128 I/O cycles (alternately write and read):
```verilog
for (i = 0; i <= 128; i = i + 1) begin 
        // Perform write
        #40  LFRAME_in  = 0;
        IO_Read_Flag   = 0;
. . .
```
In this test-bench file is also embedded implementation of module "LPC_Host" and 
instantation of modules "LPC_Host" and "LPC_peri". All of them are needed for 
carrying ot the simulation of "LPC Peripheral" module.