# Analysis of the TPM project launch on SoftCore-CPU

An alternative to hardware Hard-CPU (Or SoC containing Hard-CPU) is SoftCore-CPU
implemented from FPGA programmable circuit resources. This solution, like any other,
has its advantages, but also disadvantages - we will try to describe them in this
short document.

## Advantages of using a solution based on SoftCore-CPU

The main advantage of using SoftCore-CPU is its flexibility. We are able to "sew"
our CPU as needed, adding or omitting the appropriate blocks with HDL code during
synthesis. For example, if we don't need floating point calculations we can omit
`FPU` unit. For a further example if we need fast calculating checksums we can add
extension with hardware calculating of checksums (implemented in FPGA fabric).
By incurring programming costs, we can extend the set of peripheral systems of
such SoftCore-CPU. Another advantage is the possibility of implementing SoftCore-CPU
in a standard FPGA chip without Hard-CPU.

## Disadvantages of using a solution based on SoftCore-CPU

One of the main disadvantages is the lower performance of SoftCore-CPUs than Hard-CPU.
The SoftCore-CPU's main clock frequency is usually just a bit over 100 MHz.
Also, the occupancy of both basic FPGA blocks, such as LUTs and Flip-Flops, or
more advanced ones, such as BRAM memory blocks or DSP blocks, is large.
Requirements for more complex SoftCore-CPUs can exceed 30K LUTs and several
thousand FFs. Therefore, it is necessary to select a large FPGA chip to implement
such a solution. The FPGA chip selected to implement SoftCore-CPU must also be
relatively fast.

There are also other limitations related to other aspects of the SoftCore-CPU design.
Often in SoftCore-CPU external RAM memory drivers are not implemented (for example
for DDR memory). There are also often limitations in some programming tools for
such SoftCore-CPU, such as `Bootloader` or C/C ++ compiler

## SoftCore-CPU selection criteria for the TPM hardware project

The SoftCore-CPU selection criteria for the hardware design of the TPM module have
been specified in previous documents - the most important of which are:

+ the existing driver for the external (in relation to FPGA) RAM memory for the CPU
+ The lower limit of the CPU RAM is 256 KB
+ 32-bit high-performance CPU
+ efficient system bus (preferably with 'burst' mode support)
+ fast interrupt controller
+ a big advantage would be a DMA controller with the possibility of transfers from
 BRAM FPGA memory to CPU RAM
+ flexible configuration options
+ flexible `bootloader` with the ability to boot RTOS and/or OS Linux
+ well-functioning toolchain - C compiler and loader
+ JTAG debbugger with `openOCD` support
+ complete and detailed documentation
+ sample programs showing how to use various soft-cpu modules
+ it would be an advantage if SoftCore-CPU could be fully synthesized in open
 source FPGA synthesis tools

## SoftCore-CPU project considered as a candidate for the implementation of the TPM module

At this stage of work, we chose SoftCore-CPU named `NEORV32` compatible with 32-bit
ISA`RISC-V`. It happened because this project meets most of the requirements set
out in the paragraph above. One of the important factors was the existing C ++
compiler and the ability to debug CPU programs with the JTAG debugger. The
existence of the UART based `Bootloader` is also important. Also the documentation
for this project seems to be complete. The biggest disadvantage of this particular
SoftCore-CPU is the lack of an external DDR3 memory driver, which will have to be
added to this processor. Problematic may also be that this Soft-CPU was written
in `VHDL` language and most open source tools only support `Verilog`.

Here is `Github` repository for `NEORV32` SoftCore-CPU [NEORV32 SoftCore](https://github.com/stnolting/neorv32)
