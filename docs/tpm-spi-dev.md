# Intro

The goal of this document is to describe the current state of the lpnTPM
communication over SPI interface and what are the further plans in this area.

## Replacing USB CDC with SPI communication in ms-tpm-20-ref

[Initially](https://lpntpm.lpnplant.io/running/) we have enabled the sample code
from the `ms-tpm-20-ref`, which provided the USB CDC communication channel. Our
first goal towards the SPI communication was to replace the USB CDC channel
with SPI one.

We have added basic support for communication with TPM over SPI and dropped CDC
support (code available
[here](https://github.com/lpn-plant/ms-tpm-20-ref/tree/cmd_parsing_spi)).
Currently, communication protocol does not follow the protocol as defined in the
[TCG PC Client Specification](https://trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2p0-v1p05p_r14_pub.pdf).
This is planned for futher milestones of the project to be fully compatible with
existing TPM SPI drivers. For implementation to be compliant with TPM protocol
we must use the same header format (current format has only slight differences),
and implement full TPM register interface.

### Running

To build and run this application you need latest
[STM32Cube IDE](https://www.st.com/en/ecosystems/stm32cube.html) (at least
version 1.10.0). Instructions for building and running has been described
[here](https://lpntpm.lpnplant.io/building). TPM2 interface is exposed through
SPI2. Following pins are used:

| Pin  | Function |
| ---- | -------- |
| PC2  | MISO     |
| PC3  | MOSI     |
| PB10 | SCK      |
| PB12 | CS       |

STM32Cube IDE by default inserts breakpoint at `main()`, to resume program press
F8. After resuming program you should see similar log in `SWV ITM Data Console`
window

```text
=========================
= Nucleo-L476RG TPM 2.0 =
=========================
2000.01.01-00:00:00.003GMT: Generated tpmUnique
    64d35b5495b2960f085a53af30aa2b68
    1dcde64d11b81f1ba80207cd0546dbc8
    937003faae4ee104769877877e8a9f8b
    30a3ce2621641a56c9d3ce1ff95b9eee
2000.01.01-00:00:00.324GMT: Initialized 16kb NVFile.
2000.01.01-00:00:00.324GMT: NVFile loaded (16kb, 1970.01.01-00:00:00GMT created, 0 writes, NEVER last)
2000.01.01-00:00:00.327GMT: TPM_Manufacture(1) requested.
2000.01.01-00:00:00.654GMT: NVFile written (16kb, 1970.01.01-00:00:00GMT created, 0 writes, NEVER last)
2000.01.01-00:00:00.654GMT: _plat__SetNvAvail().
2000.01.01-00:00:00.654GMT: _plat__Signal_PowerOn().
2000.01.01-00:00:00.654GMT: _plat__Signal_Reset().
```

![](/images/tpm_spi2_con.jpg)

As SPI master we use Raspberry PI 3B, TPM is connected to RPI SPI1. To be able
to send test commands to TPM `spidev` driver must be enabled and bound to SPI1.
To enable `spidev`, modify `config.txt` in RPI boot partition by adding
`dtoverlay=spi1-1cs`. This line has to be added before any section starts,
preferably on top of file. RPI must be rebooted for the changes to take effect.

Device should be visible as `/dev/spidev1.0`.

### Testing

Basic testing can be done using
[spidev](https://github.com/STMicroelectronics/linux/blob/v5.10-stm32mp/tools/spi/spidev_test.c)
test tool.

Commands are sent after 4-byte header. Header first byte determines size of
transfer and second byte determines which register to read/write. There are 4
registers:

- 0x00 - write TPM command into buffer without executing it
- 0x01 - execute TPM command stored in buffer
- 0x02 - read TPM response
- 0x03 - send test response: TPM always responds with
  `0x42, 0x43, 0xab, 0xcd, 0xde, 0xef`
- 0x04 - read TPM response size

To do a simple test, first, send special request

```shell
$ spitest -v -D /dev/spidev1.0 -s 100000 -p '\x06\x03\x00\x00'
spi mode: 0x0
bits per word: 8
max speed: 100000 Hz (100 kHz)
TX | 06 03 00 00 __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __  |....|
RX | 00 00 00 00 __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __  |....|
```

TPM should start printing `SPI TX error: 3` which means it cannot send response
due to timeout, this is because host is not waiting for transfer. To receive
data, next transfer must be initiated. SPI works in duplex mode simultaneously
so some data must be transferred to TPM while receiving response. Here we send
NULL bytes.

```shell
$ spitest -v -D /dev/spidev1.0 -s 100000 -p '\x00\x00\x00\x00\x00\x00'
spi mode: 0x0
bits per word: 8
max speed: 100000 Hz (100 kHz)
TX | 00 00 00 00 00 00 __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __  |......|
RX | 42 43 AB CD DE EF __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __  |BC....|
```

> Note: number of NULLs must match the number declared in header (here 6 bytes).

To send real TPM command (here TPM_Startup) it must be first written into buffer
(normally TPM would use FIFO but here we use a simple buffer).

```shell
$ spitest -v -D /dev/spidev1.0 -s 100000 -p '\x0c\x00\x00\x00\x80\x01\x00\x00\x00\x0c\x00\x00\x01\x44\x00\x00'
spi mode: 0x0
bits per word: 8
max speed: 100000 Hz (100 kHz)
TX | 0C 00 00 00 80 01 00 00 00 0C 00 00 01 44 00 00 __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __  |.............D..|
RX | DE DE DE DE DE DE DE DE DE DE DE DE DE DE DE DE __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __  |................|
```

> Note: on RX we receive some junk. This is because test app does not enqueue
> TX transfer while receiving data and controller re-transfers remnants of
> previous transfer. This can be (partially) solved by always providing a NULL
> filled buffer, and using HAL_SPI_TransmitReceive instead of HAL_SPI_Receive.
> Zephyr (see the section below) already does that which partially solves the
> problem. However, if TPM does not enqueue transfer right in time, this causes
> noise on SPI.

And now, execute command:

```shell
$ spitest -v -D /dev/spidev1.0 -s 100000 -p '\x01\x01\x00\x00\x01'
```

You should see something like this on SWV console

```shell
TPM command: 80 01 00 00 00 0c 00 00 01 44 00 00
2000.01.01-00:20:18.201GMT: Executing command TPM_CC_Startup()
2000.01.01-00:20:18.525GMT: NVFile written (16kb, 1970.01.01-00:00:00GMT created, 0 writes, NEVER last)
2000.01.01-00:20:18.525GMT: Completion time 0'0" with ReturnCode {TPM_RC_SUCCESS}
Response size: 10
```

To read response, type:

```shell
$ spitest -v -D /dev/spidev1.0 -s 100000 -p '\x04\x04\x00\x00\x00\x00\x00\x00'
$ spitest -v -D /dev/spidev1.0 -s 100000 -p '\x00\x00\x00\x00'
spi mode: 0x0
bits per word: 8
max speed: 100000 Hz (100 kHz)
TX | 00 00 00 00 __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __  |....|
RX | 0A 00 00 00 __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __  |....|
```

Response size is 10, so we need to send 4 byte header and then prepare 10 byte
transfer

```shell
$ spitest -v -D /dev/spidev1.0 -s 100000 -p '\x04\x02\x00\x00\x00\x00\x00\x00'
$ spitest -v -D /dev/spidev1.0 -s 100000 -p '\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00'
spi mode: 0x0
bits per word: 8
max speed: 100000 Hz (100 kHz)
TX | 00 00 00 00 00 00 00 00 00 00 __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __  |..........|
RX | 80 01 00 00 00 0A 00 00 00 00 __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __ __  |..........|
```

Examples above show successful communication attempt, however communication is
unstable and it tends to fail frequently. Failure can be simply reproduced by
trying commands multiple times.

Also, please note that SPI is running at 100 kHz, where TPM specification
requires TPM to operate at range 10 MHz - 24 MHz. Currently, upon receiving
command TPM is waiting for host to receive response and will not accept further
commands until response is received. Before sending command and receiving
response there is a delay caused by executing `spitest` separately. `spitest`
calls may be chained together, but this may result in TPM not responding right
in time.

STM32 HAL does not provide API to sample SPI chip select, but TPM specification
requires some actions to be taken based on the CS value.

> From [TCG PC Client Platform TPM Profile Specification](https://trustedcomputinggroup.org/wp-content/uploads/PC-Client-Specific-Platform-TPM-Profile-for-TPM-2p0-v1p05p_r14_pub.pdf)
> section 6.5.1.1:
>
> 1. For Read Cycles, the TPM SHALL abort a cycle by driving 1 on MISO and
>    continue to hold MISO at 1 until its CS# signal is deasserted.

Also, TPM needs a way to recover to interrupted transfers. TPM data transfer
happens in a few byte chunks (header and data payload of size defined by
header). When TPM is deselected, transfer is aborted and TPM should go back into
idle state. This may require TPM firmware to sample CS state in order to update
its internal state machine.

Neither Zephyr provides such an API, except for GPIO chip select, which may not
be usable from within SPI slave.

### Summary of the HAL-based approach

The STM32 HAL approach was continued as it was the easiest path forward for
idea verification. Unfortunataly, we faced several obstacles and decided not to
continue this path, as this approach is not the target one for multiple
reasons:

- the STM32L476 is
  [short on memory](https://lpntpm.lpnplant.io/issues/#memory-usage) when
  running the `ms-tpm-20-ref` stack
- the STM32L476 will not be able to provide LPC interface, which is common
  interface for TPM modules
- due to the global supply chain issues, the STM32L476 (as most of the others
  STM32, and many more MCUs) is infeasible to source

Some more considerations on the hardware replaceement could be found
[here](https://lpntpm.lpnplant.io/further_project_development/).

One of the paths to move forward is to port the `ms-tpm-20-ref` stack to some
RTOS (such as [Zephyr](https://zephyrproject.org/)) to gain some more
flexibility in tems of hardware switching in the current global situation.

## Prototyping TPM SPI communication with Zephyr

Further SPI work has been done on Zephyr, the code is available
[here](https://github.com/lpn-plant/zephyr-spi-app). This code tests
communication with a dummy TPM implementation as `ms-tpm-ref-20` has not been
fully ported to Zephyr yet.

Zephyr has it's own SPI driver, which still uses low level HAL API, however its
behaviour is different from high level HAL API we were using previously. First,
receiving of multiple bytes at once works fine, which simplifies code - instead
of filling RX buffer one byte at a time we can do this in 2 transfers

- 4 byte TPM header which contains: mode (read or write),
  target register address and transfer size
- data read or write transfer with matching size as defined in TPM header

> Note: Current SPI implementation uses different header format than defined by
> TPM spec. Mode is not present and, on read,  TPM always sends full register,
> ignoring header size field.

| Byte | Usage            |
| ---- | ---------------- |
| 0    | Size of transfer |
| 1    | Register address |
| 2    | Unused (zero)    |
| 3    | Unused (zero)    |

Currently, there are 3 registers (mininum required to send command and receive
response):

- TPM_REG_CMD - register used for sending command, there is no  TPM FIFO, entire
  command is executed immediatelly after transfer is complete
- TPM_REG_STATUS - status register, currently contains only response size
- TPM_REG_RESP - response, there is no FIFO so response must be read during a
  single transfer

## Running Zephyr sample

Zephyr sample communicates SPI1 (there were problems in getting SPI2 to work
under Zephyr). Following pins are used:

| Pin | Function |
| --- | -------- |
| PA5 | SCK      |
| PA6 | MISO     |
| PA7 | MOSI     |

> Note: Zephyr uses GPIO chip select on SPI1 which works only for master, not
> slave. So, device is always selected and listening on the bus and there is no
> need to attach CS. In real world scenario CS is required, in that case SPI
> controller dedicated CS should be used instead of GPIO.

![](/images/tpm_spi1_con.jpg)

To run Zephyr sample, use the following commands:

```shell
$ docker run -ti \
    --privileged \
    -v /dev:/dev \
    -u $(id -u) \
    -v $PWD:/workdir \
    -v $(dirname $SSH_AUTH_SOCK):$(dirname $SSH_AUTH_SOCK) \
    -e SSH_AUTH_SOCK=$SSH_AUTH_SOCK  \
    ghcr.io/zephyrproject-rtos/zephyr-build:v0.23.3
(docker)$ west init -m git@github.com:lpn-plant/zephyr-spi-app.git --mr spi zephyr-spi-workspace
(docker)$ cd zephyr-spi-workspace
(docker)$ west update
(docker)$ export ZEPHYR_BASE=$PWD/zephyr
(docker)$ west build -p auto -b nucleo_l476rg -s zephyr-spi-app/app/
(docker)$ west flash
```

> Note: Zephyr outputs to emulated UART over USB which should be visible as
> /dev/ttyACM*

### Testing SPI communication

We have provided `tpmctl` available in
[zephyr-spi-app](https://github.com/lpn-plant/zephyr-spi-app) repo which can be
used to send TPM command and receive response. `tpmctl` can be built simply by
invoking GCC as it does not depend on any external libraries. If cross compiling
you can use `arm-linux-gnueabihf-gcc` or build directly on the target device.

```shell
$ gcc tpmctl.c -o tpmctl
```

Command is sent by piping raw, binary command to `tpmctl`, output can be either
in binary form or hexdump:

```shell
$ printf '\x80\x01\x00\x00\x00\x0c\x00\x00\x01\x44\x00\x00' | tpmctl -H
80 01 00 00 00 0a 00 00 00 00
```

> Note: here we send TPM startup which is the only TPM command supported by SPI
> test application.

Communication works fine with smaller commands (up to 16 bytes), albeit SPI
driver keeps complaining that transfer failed even if host got all data.

```shell
[00:00:08.242,000] <err> spi_ll_stm32: spi_stm32_get_err: err=64
[00:00:08.242,000] <err> spi: SPI TX failed: -5 (size 10)

[00:00:10.248,000] <err> spi_ll_stm32: spi_stm32_get_err: err=64
[00:00:10.248,000] <err> spi: SPI TX failed: -5 (size 10)

[00:00:11.386,000] <err> spi_ll_stm32: spi_stm32_get_err: err=64
[00:00:11.387,000] <err> spi: SPI TX failed: -5 (size 10)
```

As per TPM spec, TPM must support transfers up to 64 bytes (excluding 4 byte
header). With transfers larger than 8 bytes communication becomes unstable.
Transfer from TPM to Host is generally more unstable - Host to TPM transfer
(command) can be **usually** transferred uncorrupted, however responses, usually
get corrupted right after 8 byte mark.

```shell
$ printf '\x80\x01\x00\x00\x00\xfa\x00\x00\x01\x44\x00\x00\x03\x29\xab\x33\x64\xd2\x41\xec\x2d\x8c\x31\x7f\x3c\x64\x68\x5a\x7d\x8e\xce\xbe\xf5\xfa\xa0\xc7\x06\xe8\x07\x5e\xb3\x12\xfc\xbc\x4e\x61\x72\x05\x3f\x34\xf2\x28\x44\xb5\x40\x2e\x21\x35\xcc\x57\xfa\xa0\xc7\x06' | tpmctl -H
ec 32 48 4e 9d 22 9c db 9d 9d 9d 9d 9d 4e ce ce
ce a7 67 67 67 53 b3 b3 b3 a9 d9 d9 d9 d4 ec ec
ec ea 76 76 76 75 3b 3b 3b 3a 9d 9d 9d 9d 4e ce
ce ce a7 67 67 67 53 b3 b3 b3 a9 d9 d9 d9 d4 ec
```

Here command is transferred properly and TPM responds with a hardcoded 64 byte
response, but the response becomes corrupted (note repeating patterns). The
correct response is

```shell
ec 32 48 4e 9d 22 9c db 29 9e b4 4a 23 e2 ec e1
36 4e 1b c2 09 44 eb 65 21 aa db 00 da 60 50 a6
53 de da 22 5c a5 df 2f 75 b3 9c e0 10 4a 1d a9
94 e2 0c b8 c1 82 b0 34 d6 ea 10 e1 72 98 88 24
```

For SPI to work reliably both host and device must keep transfers in sync, to
achieve this on host we execute multiple transfers during a single IOCTL and let
the driver handle this. On TPM side, we have to process request quickly and
queue response before host initiates next transfer. This adds complexity as the
code has to be very well optimized. Currently, there is a workaround that adds
delays between transfers to let TPM get ready after executing command, otherwise
we won't be able to get response. This workaround should be replaced with better
optimized SPI handling on TPM site, as well as implementation of wait states
(TPM specific extension to SPI).

SPI currently runs at 100 KHz where TPM specification requires support for
10 - 24 MHz range. Currently, communication is completely broken at 1 MHz.

## Further work

SPI stability problems must be solved and device must be able to reliably
operate at 24 MHz (TPM specification recommends higher frequencies, however
these are not mandatory currently).

TPM specific SPI extensions must be implemented, such as wait states and bus
aborts, which depend on non-standard behaviour not supported directly by SPI
drivers/controllers. Implementing this is possible, but complex.

Currently, we have implemented only 3 registers, but a real TPM has much more.
To be able operate with existing software/hardware we have to implement TPM
register interface, along with TPM FIFO interface, interrupt interface and all
related registers.

Requirements have been also described
[here](https://github.com/lpn-plant/lpntpm-docs/blob/main/docs/tpm_lpc_spi_interface.md)
