## Target TPM hardware interfaces

As we are designing a universal TPM device, we are targeting multiple communication
interfaces to talk to TPM. Our requirements are to use the most popular ones (LPC
and SPI).

The first interface, that we looking to implement is SPI. That's mostly because we
think it is the easiest way is to start.

We are planning to use lpntpm with an embedded platform as Raspberry Pi.
Some steps were made to archive it, but we are still in the development stage.
Our goal is to be able to use `tpm2-tools` and emulate the behavior of
already available TPM.

## Required SPI bus speed

As one of our requirements is to communicate with motherboards using SPI we
should also implement such communication.

[TCG PC Client Platform TPM Profile Specification for TPM 2.0](https://trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2p0-v1p04_r0p37_pub-1.pdf)
page 127 - 7.4.1 Clocking

```
1. The TPM SHALL support an SPI clock frequency range of 10 - 24MHz.
2. The TPM MAY support running at lower frequencies.
3. The TPM SHOULD support higher frequencies
```

STM32L4 limits SPI speed up to `APB2/2` thus our maximum speed of SPI interface
is theoretically is 40MHz so we are good to go.

[RM0351 Reference manual](https://www.st.com/resource/en/reference_manual/rm0351-stm32l47xxx-stm32l48xxx-stm32l49xxx-and-stm32l4axxx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)

```
42.2 SPI main features:
  - Slave mode frequency up to fPCLK/2
```

## Low pin count interface

LPC (Low Pin Count) Interface it is not available in popular microcontrollers.

We have some ideas for software implementations of such.
First of all we would like to use some other hardware interfaces, to be able to
archive communication.

### STM32 avaliable interfaces

We will look towards `FSMC`, `SPI` and `4-bit SDcard interface`. There is a
chance, that by preparing special data packets we can simulate the behavior of
LPC interface using more popular one.

In our opinion bit-banging of such a fast interface (33MHz) will not be reasonable
and will require us to choose a faster processor, as STM32L4 runs at maximum
80MHz.

### Using SPI/LPC converters

From what we found the one and only chip converter that is out there, to convert
SPI to LPC is
[Microchip ECE1200](https://www.microchip.com/en-us/products/embedded-controllers-and-super-io/espi-to-lpc-bridge)

It uses Enhanced `Serial Peripheral Interface (eSPI)`, which is slightly modified
version of SPI interface.

This outcome and FPGA one, listed below would reduce the need for main 
uC computing power.


### Custom made FPGA converter

[Lattice Design software - LPC Controller](https://www.latticesemi.com/products/designsoftwareandip/intellectualproperty/referencedesigns/referencedesigns02/lpcbuscontroller)

Lattice software stack offers LPC implementation for FPGA's and if we want to
implement it that way, it would be a clear and elegant solution to create some
kind of SPI>LPC converter.

Lattice XO2 series, avaliable in WLCSP25/WLCSP36 package (2,5mm x 2,5mm) would
be sufficient.


