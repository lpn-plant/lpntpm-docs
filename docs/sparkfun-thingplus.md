# Developing on SparkFun Thing+ board

SparkFun Thing+ is a board based on, and compatible with QuickLogic FeatherLite
board. It has EOS S3 which is ARM Cortex-M4 MCU with embedded FPGA. See
[this link](https://learn.sparkfun.com/tutorials/quicklogic-thing-plus-eos-s3-hookup-guide)
for official documentation.

## First boot

Device comes pre-flashed with diagnostic firmware, when device boots it flashes
blue led for a few seconds, then pressing reset button will trigger diagnostic
firmware and device exposes serial port over USB (CDC ACM).

## Flashing firmware to device

Flashing can be performed in two way, either through bootloader or through SWD.
Bootloader flashing mode can be triggered by pressing the reset button followed
by pressing the user button. Green LED should start breathing. If LED blinks
blue and green then an error has occurred (this happened on the firmware that
came pre-installed on the device, flashing through SWD was required).

Device came with broken firmware version, which prevents firmware flashing,
please see [Recovering from bad flash](#recovering-from-bad-flash) section -
recovery process involves reflashing bootloader and USB FPGA with a working
properly working version.

> Note: original bootloader version had a weird quirk - having UART connected,
> was causing bootloader to actually enter download mode, however attempt to
> flash m4app resulted in corrupting bootloader and USB FPGA.

### Flashing through OpenOCD

For flashing through OpenOCD following equipment is required

- Debugger supporting OpenOCD, e.g.
  [Olimex ARM-USB-OCD-H](https://www.olimex.com/Products/ARM/JTAG/ARM-USB-OCD-H/)
  (requires also [ARM-JTAG-SWD](https://www.olimex.com/Products/ARM/JTAG/ARM-JTAG-SWD/)
  adapter)
- [SWD cable](https://www.adafruit.com/product/1675)

When flashing firmware over SWD you need to put jumpers on J2 and J3. You need
OpenOCD version 0.11, if your distro provides older version you have to build
OpenOCD yourself.

```
apt build-dep openocd
git clone https://github.com/openocd-org/openocd --branch v0.11.0 --depth=1
cd openocd
./bootstrap
mkdir build && cd build
../configure --prefix=/opt/openocd
make -j$(nproc)
make install
```

Flash device is an external SPI flash and OpenOCD can't handle it - in order to
write flash bootloader is required. Prebuilt bootloader is available
[here](https://github.com/QuickLogic-Corp/quick-feather-dev-board),
source is also available in
[qorc-sdk](https://github.com/QuickLogic-Corp/qorc-sdk) repo.

Clone repository with prebuilt bootloader

```shell
git clone https://github.com/QuickLogic-Corp/quick-feather-dev-board
```

Connect board to debugger using SWD cable, make sure jumpers are in place, plug
USB cable and then type the following command

```shell
/opt/openocd/bin/openocd -f interface/ftdi/olimex-arm-usb-ocd-h.cfg \
    -f interface/ftdi/olimex-arm-jtag-swd.cfg \
    -f board/quicklogic_quickfeather.cfg \
    -c init \
    -c reset \
    -c "load_image quick-feather-dev-board/binaries/qf_loadflash.bin 0x0 bin" \
    -c reset \
    -c exit
```

Green LED should immediatelly light up and host should detect new CDC ACM device

```
[881016.294497] usb 3-2: new full-speed USB device number 25 using xhci_hcd
[881016.443375] usb 3-2: New USB device found, idVendor=1d50, idProduct=6130, bcdDevice= 0.00
[881016.443384] usb 3-2: New USB device strings: Mfr=0, Product=0, SerialNumber=0
[881016.445132] cdc_acm 3-2:1.0: ttyACM0: USB ACM device
```

Now device can be flashed through USB.

### Preparing flashing tools

Firmware can be flashed through [QuickLogic's TinyFPGA programmer](https://github.com/QuickLogic-Corp/TinyFPGA-Programmer-Application/)
fork. Please follow the instructions below to prepare programmer.

```shell
git clone https://github.com/QuickLogic-Corp/TinyFPGA-Programmer-Application --recurse-submodules
cd TinyFPGA-Programmer-Application
git checkout 97149cab9126e70cc1aeeca6ba023520ae18ce60
pip3 install tinyfpgab
sed -i 's@^#!python3\.7@#!/usr/bin/env python3@' tinyfpga-programmer-gui.py
chmod +x tinyfpga-programmer-gui.py
```

Programmer cannot be easily installed because has no setuptools script, unless
you want to copy to script (including submodules) into your bin, please consider
putting an alias into your `.bashrc`

```shell
alias qfprog="/full/path/to/tinyfpga-programmer-gui.py"
```

Install `71-QuickFeather.rules`, otherwise qfprog may conflict with ModemManager
(which may detect device as GSM modem).

```shell
sudo cp 71-QuickFeather.rules /etc/udev/rules.d/
sudo udevadm control --reload
```

Unplug and plug device back in for the changes to take effect.

## Running Zephyr hello world

To avoid messing with your environment you may use official Zephyr container.

```shell
$ mkdir zephyr
$ cd zehpyr
$ docker run -it \
    --privileged \
    -v /dev:/dev \
    -u $(id -u) \
    -v $PWD:/workdir \
    ghcr.io/zephyrproject-rtos/zephyr-build:v0.23.3
(docker)$ west init .
(docker)$ west update
(docker)$ west build -b quick_feather zephyr/samples/hello_world
```

To flash Zephyr you need to enter download mode, either by pressing the user
button or by downloading bootloader through OpenOCD which has been described in
sections above. Then to flash Zephyr use the following command

```shell
qfprog --mode m4 --m4app build/zephyr/zephyr.bin
```

> To boot from flash you need to take off jumpers and also to boot into Zephyr
> you need press reset button.

Zephyr by default outputs to UART0 instead of UART-over-USB (USB even isn't
enabled when Zephyr boots). To be able to see anything you have to connect UART
to `44/TX` and `45/RX` pins on J6 connector (you may also see official
documentation linked at the top of this document).

[Qorc SDK](https://github.com/QuickLogic-Corp/qorc-sdk) has an example app
(`qf_uart2usbser`) which may be used to get USB UART working on Zephyr.

## Other components, bootloader, FPGAs

QuickLogic EOS comes with few components available as part of
[Qorc SDK](https://github.com/QuickLogic-Corp/qorc-sdk)

- Bootloader (`qf_bootloader`) - Cortex-M4 application responsible for booting
  platform and loading FPGAs.
- Cortex-M4 - target application to run on Cortex-M4, Qorc SDK contains a few
  applications, including `qf_helloworldsw` which is the diag application that
  comes preinstalled.
- bootfpga - this is USB serial FPGA. See the section below for how to recover.
- FPGA for user application - FPGA which can be used for any purpose.

![](/images/qorc-flash-memory-map-addresses.svg)

Qorc SDK comes with `envsetup.sh` script that automatically downloads all
required components (compilers, etc.) and configures environment, tested on
Ubuntu 20.04 with Qorc SDK rev `6968338b8778cb129ae441f6944444c9a1298390`.

`qfprog` can flash a few types of image

```shell
usage: tinyfpga-programmer-gui.py [-h] [--mode [fpga-m4]] [--m4app app.bin] [--appfpga appfpga.bin] [--bootloader boot.bin] [--bootfpga fpga.bin] [--reset] [--port /dev/ttySx] [--crc] [--checkrev]
                                  [--update] [--mfgpkg qf_mfgpkg/]

optional arguments:
  -h, --help            show this help message and exit
  --mode [fpga-m4]      operation mode - m4/fpga/fpga-m4
  --m4app app.bin       m4 application program
  --appfpga appfpga.bin
                        application FPGA binary
  --bootloader boot.bin, --bl boot.bin
                        m4 bootloader program WARNING: do you really need to do this? It is not common, and getting it wrong can make you device non-functional
  --bootfpga fpga.bin   FPGA image to be used during programming WARNING: do you really need to do this? It is not common, and getting it wrong can make you device non-functional
  --reset               reset attached device
  --port /dev/ttySx     use this port
  --crc                 print CRCs
  --checkrev            check if CRC matches (flash is up-to-date)
  --update              program flash only if CRC mismatch (not up-to-date)
  --mfgpkg qf_mfgpkg/   directory containing all necessary binaries
```

For device to boot, bootloader, m4app and bootfpga partitions are required.
Bootfpga is actually USB serial FPGA and it is required for programming device
and for booting application that use serial over USB.

## Recovering from bad flash

`qf_loadflash` doesn't require a valid USB serial FPGA in SPI flash as it
contains its own embedded USB FPGA, so it can be used for recovering from
corrupted flash.

To recover from bad flash you have to flash `bootloader` and `bootfpga`.
`qf_bootfpga` provided in `quick-feather-dev-board` repo is broken and device
will hang when attempting to load, but `qf_bootfpga` can be built as part
`qf_bootloader`. To built it go to `qorc-sdk` directory, run `envsetup.sh`, then
go to `qf_apps/qf_bootloader` and run `make`. Then flash resulting binaries

```shell
qfprog --mode m4 --bootloader GCC_Project/output/qf_bootloader.bin
qfprog --mode m4 --bootfpga GCC_Project/output/qf_bootfpga.bin
```

## FPGA synthesis

QuickLogic uses its own SymbiFlow fork for synthesis. `qorc-sdk` contains
Verilog sources for usb2serial among others (located in `s3-gateware`
directory). Sources for many components can built without problems by simply
running `make`.

## FPGA demo applications

`qf_apps` contains a few applications using a custom FPGA, including
`qf_helloworldhw` and `qf_advancedfpga`. These applications can be flashed
through bootloader as any other application.

Despite using FPGA, the two applications must be flashed in m4 mode.
`qf_advancedfpga` uses UART0 instead of USB. Applications must be told to power
on the LED, please see
[Qorc SDK readme](https://github.com/QuickLogic-Corp/qorc-sdk#lesson-3-advanced-fpga-m4--fpga-qf_advancedfpga)

QuickLogic has 3 modes of operation: `m4`, `fpga` and `fpga-m4`. This controls
bootloader behaviour, so that bootloader loads either Cortex-M4 application,
FPGA or both. In case of applications that load FPGA by themselves, mode should
be `m4`.

> Note: You can use `qfprog --mode` to get current mode.

If operation mode is set to `fpga` or `fpga-m4` bootloader loads FPGA from the
`appfpga` region, see relevant code
[here](https://github.com/QuickLogic-Corp/qorc-sdk/blob/6968338b8778cb129ae441f6944444c9a1298390/qf_apps/qf_bootloader/src/appfpga_loader.c#L113)

## Resources

- [Thing+ guide](https://learn.sparkfun.com/tutorials/quicklogic-thing-plus-eos-s3-hookup-guide) -
  Getting started for Sparkfun Thing+, contains HW overview, including images
  and detailed pin descriptions, schematics and other useful resources.
