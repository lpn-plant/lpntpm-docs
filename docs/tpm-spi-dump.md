# SPI communication dump

## Required hardware

SPI TPMs typically operate at range 10 MHz - 24 MHz as required by TPM
specification (larger frequencies are possible). To monitor SPI we are using
DSLogic Plus which is capable of operating at up to 400 MHz (but with 3 channels
only). As a TPM module we are using ASUS 90MC07D0-M0XBN0 (with Nuvoton NPCT750).
Since LPN TPM must run at least at 24 MHz to be compliant with specification we
are capturing at 24 MHz, however 90MC07D0-M0XBN0 has been confirmed to work at
frequencies as low as 1 MHz.

As SPI host we are using Raspberry PI 3B v1.2

## Required software

[DSView](https://github.com/DreamSourceLab/DSView) is an open-source software
for DSLogic devices. No Linux binaries are provided and unless your distribution
already provides a binary package, DSView has to be built manually. Building
process is simple and is not covered here.

## Preparing Raspberry PI

Recent 32-bit RPI OS had a problem with non-working SPI chip select, we
recommend using 64-bit version which worked properly. OS can be installed using
[Raspberry Pi Imager](https://www.raspberrypi.com/software/). Please remember to
configure user passwords before installing OS to avoid login problems.

![RPI Imager password setup](/images/rpi_imager_cfg.png)

For testing purposes we are going to use Linux driver, which requires changes to
device tree. Please save the following contents to temporary file.

```c
/dts-v1/;
/plugin/;
/ {
	compatible = "bcrm,bcm2835";

	fragment@0 {
		target = <&spi0>;
		tpm: __overlay__ {
			#address-cells = <0x01>;
			#size-cells = <0x00>;
			tpm@0 {
				compatible = "tcg,tpm_tis-spi";
				reg = <0x00>;
				/* Maximum frequency in Hz */
				spi-max-frequency = <24000000>;
				status = "okay";
			};
		};
	};

	fragment@1 {
		target = <&spidev0>;
		__overlay__ {
			/* Disable spidev0 to avoid conflict with TPM */
			status = "disabled";
		};
	};
};
```

Then within RPI OS boot directory run the following command

```shell
dtc -I dts -O dtb /tmp/tpm.dts -o overlays/spi0-tpm.dtbo
```

Then to actually enable overlay add following lines to `config.txt` file

```ini
dtoverlay=spi0-2cs
dtoverlay=spi0-tpm
```

> Note: these lines must be added before any section starts, we recommend adding
> them directly at the top of the file.

When booting `tpm_tis_spi` should be automatically loaded. Added devicetree
nodes should be visible at
`/sys/firmware/devicetree/base/soc/spi@7e204000/tpm@0`.

## Attaching TPM

ASUS TPM has a small female header, standard jump wires will not fit unless
theirs plastic heads are taken off. To avoid short circuits, cables can be
wrapped with electrical tape.

This is the pinout of the TPM used

![ASUS TPM pinout](/images/asus_tpm_pinout.png)

Following connections must be done

| TPM       | Raspberry Pi |
| --------- | ------------ |
| VCC       | 3.3V_PWR     |
| GND       | GND          |
| MISO_TPM  | SPI0_MISO    |
| MOSI_TPM  | SPI0_MOSI    |
| SCLK#_TPM | SPI0_SCLK    |
| CS#       | SPI0_CS0     |
| RST#_TPM  | GND          |

![RPI2 header pinout](/images/rp2_pinout.png)

When the TPM is connected to RPI, try running these commands as root

```shell
rmmod tpm_tis_spi
modprobe tpm_tis_spi
```

After successful TPM probe, `/dev/tpm0` and `/dev/tpmrm0` nodes should appear.
If probe failed, then probably TPM is not connected properly (or CS bug
mentioned above, try again with CS# connected to GND).

Complete setup looks like this:

![TPM to RPI connection](/images/tpm_rpi_dslogic_connection.jpg)

## Monitoring SPI bus

In DSView open Device Options window, set operation mode to buffered (required
for DSLogic Plus to operate at 100 MHz with 4 channels). Set voltage threshold
to 2.0V and select max 4 channels.

![DSView configuration](/images/dsview_cfg.png)

DSView has built-in decoders for various protocols including TPM over SPI. To
enable decoding click on the `Decode` button from toolbar and search for the
`SPI TPM` protocol. Select proper CLK, MISO, MOSI, CS#, and leave other settings
at theirs defaults.

![DSView protocol decoder configuration](/images/dsview_proto_cfg.png)

When configured properly DSView should show TPM registers we read or write, and
raw data that goes to/from these registers.

![Communication dump during TPM init](/images/dsview_trace_tpm_init.png)

## Communication dumps

`data` directory in this repository contains a few `*.dsl` files which can be
opened in DSView for analysis.

- `driver_init.dsl` - dump of communication right after loading `tpm_tis_spi`
  driver.
- `tpm2_startup-c.dsl` - result of running `tpm2_startup -c` command.
- `tpm2_pcrread.dsl` - result of running `tpm2_pcrread` command. At this point
  PCRs were at theirs default values.
