## Target TPM hardware interfaces

As we are designing a universal TPM device, we are targeting multiple
communication interfaces to talk to TPM. Our requirements are to use the most
popular ones (LPC and SPI).

The first interface, that we are looking to implement is SPI. That's mostly
because we think it is the easiest way to start.

We are planning to use lpnTPM with an embedded platform (such as Raspberry Pi)
for test purposes.
Some steps were made to archive it, but we are still in the development stage.
Our goal is to be able to use `tpm2-tools` and emulate the behavior of
already available TPM.

## Required SPI bus speed

As one of our requirements is to communicate with motherboards using SPI, we
should also implement such communication.

[TCG PC Client Platform TPM Profile Specification for TPM 2.0](https://trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2p0-v1p04_r0p37_pub-1.pdf)
document greatly explains the actual requirements of SPI interface. On page
`127` in the `7.4.1 Clocking` section, we can find the following must-have
conditions for SPI bus speed.

```
1. The TPM SHALL support an SPI clock frequency range of 10 - 24MHz.
2. The TPM MAY support running at lower frequencies.
3. The TPM SHOULD support higher frequencies
```

Let's focus on STM32L476 and its integrated SPI peripheral.
As mentioned on page `1450` of the
[STM32 RM0351 Reference manual](https://www.st.com/resource/en/reference_manual/rm0351-stm32l47xxx-stm32l48xxx-stm32l49xxx-and-stm32l4axxx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
STM32L4 limits SPI speed up to `APB2/2`, thus our maximum speed of the SPI
interface is 40MHz.

```
42.2 SPI main features:
  - Slave mode frequency up to fPCLK/2
```

Taking into account the TPM requirements we should be good to
initiate communication.

## Low pin count interface

LPC (Low Pin Count) interface is not available in popular microcontrollers.

We have some ideas for software implementations of such.
First of all, we would like to use some other hardware interfaces, to be able to
emulate this interface and initialize correct communication.

We found some interesting software that somehow could help us to start thinking
of software-based LPC bus emulation:
[State machine IO driver for the Low-Pin Count bus slave.](https://github.com/eddiecorrigall/LPCSlave)

### STM32 available interfaces

We will look towards `FSMC`, `SPI`, and `4-bit SDcard interface`. There is a
chance, that by preparing special data packets we can simulate the behavior of
the LPC interface using a more popular one.

In our opinion bit-banging of such a fast interface (33MHz) will not be
reasonable and will require us to choose a faster processor, as STM32L4 runs at
a maximum 80MHz.

We have found an example of
[Low-Pin Count bus slave state machine IO driver](https://github.com/eddiecorrigall/LPCSlave)
so it can be a great starting point for our development of software
emulation of the LPC bus.

The actual throughput of LPC bus is `2.47MB` -
[Table 18: Peripheral Initiated Memory Read Cycle](https://www.intel.com/content/dam/www/program/design/us/en/documents/low-pin-count-interface-specification.pdf)
so in case of bit-banging of the LPC interface we have to choose a
microcontroller like
[STM32G4 Series](https://www.st.com/en/microcontrollers-microprocessors/stm32g4-series.html)
with the maximum CPU clock speed of 170 MHz.

### Using SPI/LPC converters

From what we found the one and only chip converter that is out there, to convert
SPI to LPC is
[Microchip ECE1200](https://www.microchip.com/en-us/products/embedded-controllers-and-super-io/espi-to-lpc-bridge)

It uses Enhanced `Serial Peripheral Interface (eSPI)`, which is a slightly
modified version of the SPI interface.

This outcome and FPGA one, listed below would reduce the need for main 
uC computing power.

### Custom made FPGA converter

Lattice software stack
[offers LPC implementation for FPGA's](https://www.latticesemi.com/products/designsoftwareandip/intellectualproperty/referencedesigns/referencedesigns02/lpcbuscontroller)
and if we want to implement it that way, it would be a clear and elegant
solution to create some kind of SPI>LPC converter.

Lattice XO2 series, avaliable in WLCSP25/WLCSP36 package (2,5mm x 2,5mm) would
be sufficient.


