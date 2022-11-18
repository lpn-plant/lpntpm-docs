# Using Gowin GW1NSR-4C for the construction of a hardware TPM module

Some time ago "Gowin Semiconductor" released a series of SoC chips (combination
of hardware ARM Cortex-M3 MCU with FPGA chip) GW1NSR-4C.

In a small QN48P housing there is a 32-bit ARM Cortex-M3 MCU clocked up to 80 MHz
and an FPGA chip containing, among others, about 5K LUTs and 3.5K Flip-flops.
These two systems are connected by a fast AHB bus.

Datasheet for these SoCs is available
[here](https://www.gowinsemi.com/upload/database_doc/47/document/5c36e77302539.pdf?_file=database_doc%2F47%2Fdocument%2F5c36e77302539.pdf)

## The purpose of the study

The purpose of the test is to determine whether the chip from this series can be
used to implement a hardware TPM module.

## SoC requirements for TPM

As previously stated (and described), a TPM candidate must meet the following
requirements:

+ sufficiently efficient CPU (hardware or softcore)
+ sufficiently large RAM memory (minimum 256KB)
+ must have programmable circuit (FPGA) allowing the implementation of the LPC
  protocol driver
+ must have a high-speed bus connecting programmable logic to the CPU
+ it must have an efficient interrupt handler
+ it should have a DMA module that allows for transmissions between RAM from CPU
  with BRAM memory from FPGA
+ they have the necessary peripheral systems, such as the hardware SPI and I2C
  controller

## Requirements conformity

The GW1NSR-4C series meet most of the above-described requirements for the
hardware TPM module - they have an efficient CPU and needed peripherals. The
programmable circuitry (FPGA) has sufficient resources and is connected to the
CPU by a high-speed bus. The only problem that can be noticed is the very low
amount of RAM for the CPU - 16KB. The RAM memory for the CPU is made up of
internal BRAM blocks in the programmable circuit.

Fortunately, the SoC systems from the described series have built-in PSRAM or
HyperRAM memory modules with a capacity of 64 Mbit (8 MB). Both types of memory
are pseudo-static memories - internally they are DRAM memories, but refreshing
is done internally. Unfortunately, it seems that these memories do not have a
hardware driver in the CPU, to use them you need to implement a driver (in Verilog)
in the FPGA part. There is also an IP core provided by Gowin itself
([Gowin HyperRAM Memory Interface](https://www.gowinsemi.com/en/support/ip_detail/57/))

## HyperRAM test

We choose the [Tang Nano 4K](https://wiki.sipeed.com/hardware/en/tang/Tang-Nano-4K/Nano-4K.html)
board for hardware testing. This board is based on the
`GW1N-LV4CQN48PC7/16` chip. We created a project in
[Gowin EDA](https://www.gowinsemi.com/en/support/home/) with an implementation
(Verilog) of the HyperRAM memory driver based on Gowin IP Core.

We also wrote a test routine that writes data to the HyperRAM memory and checks
the data read from the memory and in case of errors it lights a LED on the
board. This test is included in the source file `hpram_test.v` end port named
`error` is connected to the LED. Source code is availble
[here](https://github.com/3mdeb/fpga-tests/tree/master/HyperRAM_Test_Gowin).

This test on Tang Nano 4K was completed without errors.

## Using HyperRAM as the main memory

Since there is no hardware HyperRAM memory driver and a software driver in FPGA
part is required, it is not possible to use this memory as RAM for the hard CPU.

This fact disqualifies Gowin's SoC from the GW1NSR-4C series as a hardware TPM
module.

## Open source tools for FPGA synthesis

There is project [YosysHQ/apicula](https://github.com/YosysHQ/apicula) with open
source tools for synthesis of Gowin FPGAs. Installation and use of these tools
is described on the project page.

The project documentation shows that, among others:

+ BRAM blocks in FPGA fabric are not properly handled
+ PLL clocks are not supported

## The decision

Due to the inability to connect a sufficiently large RAM memory to the CPU and
the open source tools not supporting the necessary FPGA blocks - Gowin's SoC
systems are not suitable for a hardware module for the implementation of TPM.
