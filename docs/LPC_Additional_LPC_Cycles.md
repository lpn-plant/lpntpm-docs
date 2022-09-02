## Why do we need to support additional LPC protocol cycles in "LPC Host"

In the implementation of the LPC (LPC Host and LPC Peripheral) protocol we wrote
earlier, only two types of LPC cycles were supported:
+ I / O cycles (Read and Write)
+ TPM cycles

And we really do not need to support more types of LPC cycles in the designed 
TPM module, but there is a need to check how the "LPC peripheral" module we wrote
reacts to other types of cycles appearing on the 4-bit LAD bus. For this purpose,
it was necessary to extend the "LPC Host" implementation with additional LPC protocol
cycle types. 

The "LPC peripheral" module should record all the occurrences on the LAD of the
`I/O` and `TPM` cycles and send their data (LPC address and LPC data) to the 
MCU module of SoC (by 32-bit output bus in FPGA). However, other LPC protocol cycle
types on the LAD bus should be ignored and their data should not be logged.

It is enough to add one additional type of supported cycles to the `LPC Host`
module as the response of the "LPC Peripheral" to all additional cycle types 
should be similar. The choice fell on `Memory Read/Write` cycles because they are
quite similar to `I/O` cycles and easy to implement.

## Changes related to the addition of "Memory" cycles support in "LPC Host"

First, let's look at the similarities and differences between the `I/O` and `Memory`
cycles.For this purpose we will look at Intel's `LPC Protocol` reference document.
See screen shots from this document. First we have description of `I/O` cycles
![LPC IO Cycles](images/I_O_LPC_Cycles.png)

The most important here are the `START` and `CYCTYPE` fields:

+ START has value: 0000b
+ CYCTYPE has values: 0000b or 0010b
And we also see that whole `I/O`c cycle has a length of 13 clock cycles.

Second we have description of `Memory R/W` cycles:
![LPC Memory cycles](images/LPC_Memory_Cycles_c1.png)
![LPC Memory cycles](images/LPC_Memory_Cycles_c2.png)
The most important here are also the `START` and `CYCTYPE` fields:

+ START has value: 0000b
+ CYCTYPE has values: 0100b or 0110b
And we also see that whole `Memory R/W`c cycle has a length of 21(read) or 17(write)
clock cycles. The field Address on LAD bus is longer comparing to `I/O` cycles.



