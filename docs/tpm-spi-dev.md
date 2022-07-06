# TPM communication over SPI

We have added basic support for communication with TPM over SPI and dropped CDC
support (code available
[here](https://github.com/lpn-plant/ms-tpm-20-ref/tree/cmd_parsing_spi)).
Currently, communication protocol does not follow the protocol as defined in TCG
PC Client Specification. Due to problems with programming STM32 SPI controller
by using directly HAL, resulting instability in communication and the fact that
we are moving to Zephyr based implementation, we have dropped HAL based SPI
attempts.

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

Currently, there are 3 registers (mininum required to send command and receive
response):

- TPM_REG_CMD - register used for sending command, there is no FIFO, entire
  command is executed immediatelly after transfer is complete
- TPM_REG_STATUS - status register, currently contains only response size
- TPM_REG_RESP - response, there is no FIFO so response must be read during a
  single transfer

## Testing SPI communication

We have provided `tpmctl` available in
[zephyr-spi-app](https://github.com/lpn-plant/zephyr-spi-app) repo which can be
used to send TPM command and receive response. Command is sent by piping raw,
binary command to `tpmctl`, output can be either in binary form or hexdump:

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
operate at 24 MHz (TPM specification recommends larger frequencies, however
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
