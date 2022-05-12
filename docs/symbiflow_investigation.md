## Getting acquainted with the open-source "Symbiflow" package for synthesizing, implementing and generating the FPGA configuration file

Here is a link to the main Github repository for the **Symbiflow** software package:

[Github Symbiflow repository](https://github.com/SymbiFlow)

First, I installed a modified version of the **Yosys** package (modifications for Symbiflow) from this report:

[Github Yosys Symbiflow repository](https://github.com/SymbiFlow/yosys)

Installation carried out according to the instructions on the main page of this repository. Yosys is only for FPGA synthesis phase. A list of Verilog primitive connections is created in ** JSON ** format.

The next tool is for the implementation phase (map processes, placement and routing). For this phase you need a "user constaraints" file in ** Pcf ** format for the selected FPGA.

Here is the link to the ** nextpnr ** tool repository used to implement and create a "bitstream" (binary configuration file) for a given FPGA chip:

[Github nextpnr Symbiflow repository](https://github.com/SymbiFlow/nextpnr)

**ATTENTION!** - installations should be performed according to the instructions on the Github repository page, it requires for different FPGA families (Lattice, Xilinx - Intel is not supported at all), installation of additional Symbiflow packages. For example, for the **ICE40** family of the **IceStorm** project Lattice from this repository:

[IceStorm ICE40 project repository](http://bygone.clairexen.net/icestorm/)

I installed all the additional tools for the FPGA families:

+ Lattice ICE40 (low resource FPGAs)
+ Lattice ECP5 (medium-sized FPGAs - equivalent to Xilinx's Artix-7 family)
+ Gowin FPGAs (unfortunately there were errors during the installation and the package probably does not work)
+ Generic FPGAs - you can define your own types of FPGAs here - but it is very complex.

Since package installations are done by compiling the sources from a repository, it takes a few hours to install all packages.

I also installed the **vtr-verilog-to-routing** package which is used for the implementation phase and bitstream generation for Xilinx's Artix-7 family chips. Unfortunately, after watching the video tutorial presenting the status of the **Symbiflow** project, I know that the bitstream generation support for Xilin chips is very incomplete (eg it does not include clock pins and Block_RAM memory, which are used in practically every project. **X-Ray** package needed to generate bitstream also requires "Xilinx Vivado" to be installed in the archived version 2017.2 (the installer package that you need to download from the Xilinx website is about 20 GB).

Here is the link to the video tutorial on Symbiflow (I think you must watch it):
[Youtube video tutorial for Symbiflow](https://www.youtube.com/watch?v=-xyAauPa__s)

It shows that VHDL support is very limited in Symbiflow, and that bitstream generation for all primitives of a given FPGA family is full only for Lattice's ICE40 chips. It's also not bad for the Lattice ECP5 family. The bitstream generation for Xilinx's Artix-7 family of FPGAs is partial.

#### An attempt to implement and generate bitstream for the Lattice ICE40 family of FPGAs

Since full support for FPGA implementation and bitstream generation is available for Lattice's ICE40 chips, I decided to make the first trial for such a chip.

The implementation and bitstream generation for a simple example of LED blinking (Hello world for FPGA devices) was carried out according to the tutorial from the project website **nextpnr**:

[Github nextpnr Symbiflow repository](https://github.com/SymbiFlow/nextpnr)

I launched the Linux console and went to the directory /home /bofh/Symbiflow/ nextpnr/ice40/ examples where I installed the nextpnr package:
Then I ran these commands:

> cd / blinky
>
> yosys -p 'synth_ice40 -top blinky -json blinky.json' blinky.v # synthesize into blinky.json
>
> nextpnr-ice40 --hx1k --json blinky.json --pcf blinky.pcf --asc blinky.asc # run place and route
>
> icepack blinky.asc blinky.bin # generate binary bitstream file

Before executing the commands, there were only the following files in the blinky directory:
+ blinky.v - verilog code of the main module of the project
+ blinky_tb.v - test-bench for the main module
+ blinky.pcf - "user constraints" file describing the pin mapping of the ICE40 FPGA chip for the project

All phases of the project ended without any errors being reported, and the bitstream **blinky.bin** file was created.

After this trial, I decided to implement some more complex Verilog project, consisting of several source files. First, I wanted to implement some module from the op0en-source "Libre-SOC" package, but all modules had a very large number of ports (I / O pins), often exceeding the number of pins available in the FPGA ICE40. This is due to the fact that such modules are used inside more complex modules, e.g. soft-CPU, and are encapsulated with other modules.

So my choice fell on a simple project in Verilog (Tx implementation of the UART module), such a project after bitstream generation can be tested on a physical FPGA set with the Lattice ICE40 chip. Besides, this project was once implemented on the "Elbert v.2" set with the Xilinx Spartan-3A chip and I have the "ISE 14.7" project for it

The project consists of several source files:
+ txstr.v - the main module of the project
+ uart_tx.v - project sub-module
+ baudgen_tx.v - project sub-module
+ baudgen.vh - file with constants included in the project
+ txstr.pcf - file with pin mapping for FPGA ICE40 (created by me)

In the /examples directory, I created a new folder /uart_tx and copied the source files mentioned above into it and went to that directory. Then in the system console I followed the instructions:

> yosys -p 'synth_ice40 -top txstr -json txstr.json' txstr.v uart_tx.v baudgen_tx.v

 where **txstr** is the main module of the project and all verilog source files are listed on the command line

The synthesis phase was successful (there were no errors) and a file was created: txstr.json with a list of system connections

Then I ran the command in the console:

> nextpnr-ice40 --hx1k --json txstr.json --pcf txstr.pcf --asc txstr.asc
    where ice40 --hx1k are trype and FPGA symbol

As a result of this command, the **txstr.asc** file was created with the result of the FPGA implementation phase. This phase was successful.

The last command given was:

> icepack txstr.asc txstr.bin

Based on the results of the implementation phase, it created a binary FPGA configuration file named **txstr.bin**.

As you can see, for Lattice's ICE40 FPGAs, you can implement a simple Verilog-based FPGA design using the **Symbiflow** tools.

I also checked the operation of the Verilog language simulator called **Verilator** - for this purpose I used a bench test from the first example of being able to use an LED diode. The simulator worked correctly. I have used Verilator in the past and I know how to use it to generate a VCD file with simulation timing diagrams, which can then be analyzed with the "GTKWave" program.

