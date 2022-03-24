## TPM driver instrumentation

For development purposes, a set of scripts was created that allows for the
instrumentation of a specific module, allowing for tracing the execution
of module functions.

This could be especially useful in the process of development of different
protocol backends in our `lpntpn` like LPC or I2C.

For now, scripts are located inside our fork of
[ms-tpm-20-ref](https://github.com/lpn-plant/ms-tpm-20-ref) repository.
Look for `Samples/Nucleo-TPM/scripts/kfunc_tracer` directory on the dev branch.

We use the integrated functionality that gcc provides '-finstrument-function'
for function instrumentation.
This step allows us to register exact function call addresses and log them to
dmesg.

Next, we match the function addresses against kernel processes address space
that's located in `/proc/modules` to identify loaded module, that fn call
belongs to. By subtracting function address with start memory location of loaded
module, we obtain the real offset of a function call, that could be easily
matched with an exact function name.

The last step is to extract functions memory offsets located in non-striped
kernel modules. For this purpose we are using a simple readelf call:
`readelf -s MODULE_NAME.ko | grep FUNC`.

This process requires kernel recompilation, execution of module startup and
parsing of resulting trace files.

### Prepare Raspberry Pi

We are using Raspberry Pi 3, but with minor modifications, this process could
be used on different targets.

**Flash SD card with raspbian lite os.**

At the moment of writing image `2021-05-07-raspios-buster-armhf-lite.img` was
used.

**Enable ssh.**

Execute on raspberry pi with connected keyboard and display

```shell
# upgrade
sudo apt update
sudo apt upgrade

# change default password
passwd

# enable ssh
sudo systemctl enable ssh
sudo systemctl start ssh
```

**Check ssh connection.**

Environmental variable `RASPI_IP` will be used several more times during this
guide.

Run on your host machine to test ssh connection.
```shell
export RASPI_IP=pi@192.168.8.170
ssh-copy-id -i ~/.ssh/id_rsa.pub $RASPI_IP
ssh $RASPI_IP echo "success"
# should echo success without prompting user to enter password
```

### Kernel compilation

**Prepare build environment.**

Ubuntu 20.04 LTS was used in this example.

Install dependencies on your host machine and get source code.
```shell
sudo apt install crossbuild-essential-armhf     # default toolchain
sudo apt install python-is-python3              # no python2 in ubuntu 20 fix
sudo apt install bison flex
sudo apt install qtbase5-dev pkg-config         # xconfig
sudo apt install libncurses5-dev                # menuconfig

cd git
git clone --depth=1 https://github.com/raspberrypi/linux
cd linux
# at the moment of writing we are using rpi-5.10.y branch specifically
# 581049d718caf95f5feb00607ac748d5841cf27c commit
git checkout 581049d718caf95f5feb00607ac748d5841cf27c
```

**Patch kernel**

For future debug output we add `CONFIG_DYNAMIC_DEBUG=y` to bcm2709_defconfig.
The rest of this patch is the code instrumentation functions themself.

[instrument.patch](data/instrument.patch)

**Build kernel**

On your host machine:

```shell
cd git
cd linux
export KERNEL=kernel7
# output build directories
export BUILD_DIR=build
export INSTALL_PATH=install_tmp
# temp directory for new content of the /boot and /root partion 
export BOOT_PART_TMP=$BUILD_DIR/$INSTALL_PATH/boot

# make modules_install will create build/install_tmp/root/lib directory during
# install. we dont need to specify $BUILD_DIR prefix
export ROOT_PART_TMP=$INSTALL_PATH/root

# prepare output directory
mkdir -p $BOOT_PART_TMP
mkdir -p $BOOT_PART_TMP/overlays

make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 O=$BUILD_DIR bcm2709_defconfig
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 O=$BUILD_DIR scripts prepare modules_prepare
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 O=$BUILD_DIR zImage modules dtbs
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 O=$BUILD_DIR modules_install INSTALL_MOD_PATH=$ROOT_PART_TMP 

# export K_VERSION variable
export K_VERSION=$(make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- O=$BUILD_DIR -s kernelrelease)

cp $BUILD_DIR/arch/arm/boot/zImage $BOOT_PART_TMP/$KERNEL.img

cp $BUILD_DIR/arch/arm/boot/dts/*.dtb $BOOT_PART_TMP
cp $BUILD_DIR/arch/arm/boot/dts/overlays/*.dtb $BOOT_PART_TMP/overlays
```

**Deploy**

On your host machine:

```shell
export PI_HOME=/home/pi

cd ~/git/linux
tar -zcvf install_pkg.tar.gz $BUILD_DIR/$INSTALL_PATH
scp install_pkg.tar.gz $RASPI_IP:/home/pi/

ssh $RASPI_IP tar -xvzf $PI_HOME/install_pkg.tar.gz
ssh $RASPI_IP rm $PI_HOME/install_pkg.tar.gz
ssh $RASPI_IP sudo cp -r $PI_HOME/build/install_tmp/root/lib/* /lib
ssh $RASPI_IP sudo cp -r $PI_HOME/build/install_tmp/boot/* /boot
ssh $RASPI_IP rm -rf $PI_HOME/build
ssh $RASPI_IP sudo reboot
```

Test if dynamic debug option is enabled

```shell
ssh $RASPI_IP sudo modprobe configs
ssh $RASPI_IP "zcat /proc/config.gz | grep DYNAMIC_DEBUG"
# you should get the following output: 
# CONFIG_DYNAMIC_DEBUG=y
# CONFIG_DYNAMIC_DEBUG_CORE=y
```

**Build instrumented module**

```shell
cd ~/git/linux

export BUILD_DIR=build
export K_VERSION=$(make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- O=$BUILD_DIR -s kernelrelease)
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 O=$BUILD_DIR -C . M=drivers/char/tpm EXTRA_CFLAGS="-g -DDEBUG=1 -finstrument-functions"

export MOD_SRC=build/drivers/char/tpm
export MOD_TMP_DST=/home/pi
export MOD_DST=/lib/modules/$K_VERSION/kernel/drivers/char/tpm

scp $MOD_SRC/tpm.ko $RASPI_IP:$MOD_TMP_DST/
scp $MOD_SRC/tpm_tis_core.ko $RASPI_IP:$MOD_TMP_DST
scp $MOD_SRC/tpm_tis_spi.ko $RASPI_IP:$MOD_TMP_DST

ssh $RASPI_IP sudo mv tpm.ko $MOD_DST;
ssh $RASPI_IP sudo mv tpm_tis_core.ko $MOD_DST;
ssh $RASPI_IP sudo mv tpm_tis_spi.ko $MOD_DST;
```

At this point, we need to enable device tree overlay for tpm.

In Raspberry Pi ssh terminal:

```shell
sudo nano /boot/config.txt

# add at the bottom
dtparam=spi=on
dtoverlay=tpm-slb9670

# permanently enable debug log level
sudo nano /boot/cmdline.txt
add 'loglevel=7' at the end of the line

# reboot raspi
sudo reboot
```

At this point, just after raspi reboot you should be able to see tpm function
traces in `dmesg`.

Example trace:

In Raspberry Pi ssh terminal:

```console
$ dmesg  | grep tpm_addr
[    9.358361] tpm_addr_enter: 0x7f0ee070 0x801022a4 
[    9.358546] tpm_addr_enter: 0x7f0ee000 0x7f0ee15c 
[    9.359167] tpm_addr_exit: 0x7f0ee000 0x7f0ee15c 
[    9.359180] tpm_addr_exit: 0x7f0ee070 0x801022a4 
[    9.420744] tpm_addr_enter: 0x7f09f000 0x801022a4 
```

### Collect traces and get function names from addresses

Last step to get execution traces is to use provided scripts located in
`ms-tpm-20-ref/Samples/Nucleo-TPM/scripts/kfunc_tracer`

Pull logs from your host machine:
```console
$ ./pull_logs.sh 
RASPI_IP: pi@192.168.8.170
creating ./logs directory
entering ./logs directory
cleaning ./logs directory
collecting dmesg tpm_addr logs
collecting tpm related /proc/modules entries
```

Get function names.
Below you can see the full call stack of tpm module failing due to not
connected hardware of tpm module.
To be executed on your host machine.
```console
$ python get_call_stack.py 
tpm_tis_spi.ko: tpm_tis_spi_driver_probe()
tpm_tis_spi.ko: tpm_tis_spi_probe()
tpm_tis_spi.ko: tpm_tis_spi_init()
tpm_tis_core.ko: tpm_tis_core_init()
tpm.ko: tpmm_chip_alloc()
tpm.ko: tpm_chip_alloc()
tpm.ko: tpm2_init_space()
tpm.ko: tpm2_init_space()
tpm.ko: tpm_chip_alloc()
tpm.ko: tpmm_chip_alloc()
tpm_tis_core.ko: tpm_tis_clkrun_enable()
tpm_tis_core.ko: tpm_tis_clkrun_enable()
tpm_tis_core.ko: wait_startup()
tpm_tis_spi.ko: tpm_tis_spi_read_bytes()
tpm_tis_spi.ko: tpm_tis_spi_transfer()
tpm_tis_spi.ko: tpm_tis_spi_flow_control()
tpm_tis_spi.ko: tpm_tis_spi_flow_control()
tpm_tis_spi.ko: tpm_tis_spi_transfer()
tpm_tis_spi.ko: tpm_tis_spi_read_bytes()
tpm_tis_core.ko: wait_startup()
tpm_tis_core.ko: tpm_tis_clkrun_enable()
tpm_tis_core.ko: tpm_tis_clkrun_enable()
tpm_tis_core.ko: tpm_tis_remove()
tpm_tis_core.ko: tpm_tis_clkrun_enable()
tpm_tis_core.ko: tpm_tis_clkrun_enable()
tpm_tis_spi.ko: tpm_tis_spi_read32()
tpm_tis_spi.ko: tpm_tis_spi_read_bytes()
tpm_tis_spi.ko: tpm_tis_spi_transfer()
tpm_tis_spi.ko: tpm_tis_spi_flow_control()
tpm_tis_spi.ko: tpm_tis_spi_flow_control()
tpm_tis_spi.ko: tpm_tis_spi_transfer()
tpm_tis_spi.ko: tpm_tis_spi_read_bytes()
tpm_tis_spi.ko: tpm_tis_spi_read32()
tpm_tis_spi.ko: tpm_tis_spi_write32()
tpm_tis_spi.ko: tpm_tis_spi_write_bytes()
tpm_tis_spi.ko: tpm_tis_spi_transfer()
tpm_tis_spi.ko: tpm_tis_spi_flow_control()
tpm_tis_spi.ko: tpm_tis_spi_flow_control()
tpm_tis_spi.ko: tpm_tis_spi_transfer()
tpm_tis_spi.ko: tpm_tis_spi_write_bytes()
tpm_tis_spi.ko: tpm_tis_spi_write32()
tpm_tis_core.ko: tpm_tis_clkrun_enable()
tpm_tis_core.ko: tpm_tis_clkrun_enable()
tpm_tis_core.ko: tpm_tis_remove()
tpm_tis_core.ko: tpm_tis_core_init()
tpm_tis_spi.ko: tpm_tis_spi_init()
tpm_tis_spi.ko: tpm_tis_spi_probe()
tpm_tis_spi.ko: tpm_tis_spi_driver_probe()
tpm.ko: tpm_dev_release()
tpm.ko: tpm_dev_release()

```

### Whole logs from the test procedure

Below we have collected call stack for Raspberry Pi TPM driver startup. </br>

- [Log file with TPM module plugged in](./logs_tpm_module/log_with_tpm_get_call_stack.txt)
- [Log file without TPM](./logs_tpm_module/log_without_tpm_get_call_stack.txt)
