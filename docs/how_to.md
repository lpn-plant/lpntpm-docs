## How to

lpnTPM is developed around [Official TPM 2.0 Reference Implementation
(by Microsoft)](https://github.com/microsoft/ms-tpm-20-ref).

The project consists of a cross-platform simulator and samples both for Cortex M
and Cortex A.

The original port was developed on STM32L4A6RG and STM32L476RG microcontrollers
using Atollic Studio.
For future compatibility, we converted project files to
[STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html), which
is a successor of Atollic software. The project doesn't provide any other build
systems but we are definitely looking forward to use one.

During the early development stage, we are using Nucleo L476RG board with USB
cable connected to STM32 IO.

Due to the limited amount of SRAM memory of such uC soon we will probably switch
to a different one. We are currently looking at L4/F4 series in LQFP64 package.
Probably for convenience, we will use some ready-made dev board as Nucleo L452RE
with 160 KB of SRAM.

At the current stage of development, we are using Nucleo USB CDC port to
communicate to TPM core.
We are targetting different interfaces - SPI, I2C, LPC is a must.

STlink integrated serial port, as well as ITM traces, are used for debugging
purposes.
