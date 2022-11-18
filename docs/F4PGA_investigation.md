# Getting acquainted with F4PGA fully open source toolchain for the development of FPGAs

F4PGA (previously known as SymbiFlow) is a fully open source toolchain for the
development of FPGAs of multiple vendors. Currently, it targets the Xilinx
7-Series, Lattice iCE40, Lattice ECP5 FPGAs, QuickLogic EOS S3 and is gradually
being expanded to provide a comprehensive end-to-end FPGA synthesis flow

## How it works

Please refer to the [Official Documentation](https://f4pga.readthedocs.io/en/latest/how.html).

## Supported architectures

Please refer to the [Official Documentation](https://f4pga.readthedocs.io/en/latest/status.html)
for a list of supported architectures.

## Getting F4PGA

The entire toolchain installation was done according to the
[Official Documentation](https://f4pga-examples.readthedocs.io/en/latest/getting.html).

> Note: `F4PGA_INSTALL_DIR` must point to the installation directory for F4PGA
> tools to work, we recommend exporting this variable from `.bashrc`

## Building examples (Quicklogic EOS S3)

Before building the example, make sure you have the environment variable
`FPGA_FAM` set to `eos-s3`

```bash
export FPGA_FAM="eos-s3"
```

Next step is preparing the environment:

```bash
conda env create -f $FPGA_FAM/environment.yml
```

After this step we can activate the environment:

```bash
conda activate $FPGA_FAM
```

Now, we can invoke the build:

```bash
make -C eos-s3/btn_counter
```

after make is invoked, a summary of the phases of the synthesis and routing process
is displayed:

```bash
Project status:
    [S] bitstream:  bitstream -> /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/build/top.bit
    [O] build_dir:  /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/build
    [S] eblif:  synth -> /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/build/top.eblif
    [S] fasm:  fasm -> /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/build/top.fasm
    [S] fasm_extra:  synth -> /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/build/top_fasm_extra.fasm
    [S] io_place:  ioplace -> /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/build/top.ioplace
    [S] net:  pack -> /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/build/top.net
    [O] pcf:  /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/chandalar.pcf
    [S] place:  place -> /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/build/top.place
    [S] place_constraints:  place_constraints -> /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/build/top_constraints.place
    [S] route:  route -> /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/build/top.route
    [S] sdc:  prepare_sdc -> /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/build/top.sdc
    [O] sdc-in:  /home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/dummy.sdc
    [O] sources:  ['/home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/btn_counter.v']

```

After each phase is completed, a summary is displayed (for example):

```bash
Executing module `yosys`:
    [1/3] : Synthesizing sources: ['/home/bofh/F4PGA/f4pga-examples/eos-s3/btn_counter/btn_counter.v']...
Module `yosys` has finished its work!

```

After completing all phases, the bitstream file for the FPGA chip (bit or bin)
will be created and we can load it to the SoC EOS S3 using J-Link or openOCD.

![Test Circuit Diagram](images/SOC_EOS_S3.png)

## Building examples (Xilinx Artix-7)

The steps are similiar to these from the EOS S3 example. First, make you sure
you have `FPGA_FAM` configured accordingly:

```bash
export FPGA_FAM="xc7"
```

Next step is preparing and activating the environment:

```bash
conda env create -f $FPGA_FAM/environment.yml
```

After this step we can activate the environment:

```bash
conda activate $FPGA_FAM
```

Now, we can invoke the build:

```bash
TARGET="arty_35" make -C xc7/counter_test
```

As one can see the target in this case is
[Arty 35](https://www.xilinx.com/products/boards-and-kits/arty.html) FPGA board.

As the FPGA bitstream file (bit or bin) is created, we can load it into the
FPGA set with the command:

```bash
TARGET="arty_35" make download -C counter_test
```

Alternatively, we can use the
[openFPGALoader](https://github.com/trabucayre/openFPGALoader) application to
load the FPGA bitstream file:

```bash
openFPGALoader -b arty_a7_35t top.bit
```

## Other FPGA tools

While the F4PGA toolchain tools are sufficient to develop an FPGA-based digital
chip design, there are other programs that make this process easier: HDL
simulators and waveform viewing programs.

We are going to mention two programaac here: the Verilog (and SystemVerilog)
language simulator called Verilator and waveforms viewer called GTKWave.

### Verilator - Verilog and SystemVerilog simulator

Verilator is a tool that compiles Verilog and SystemVerilog sources to highly
optimized (and optionally multithreaded) cycle-accurate C++ or SystemC code.
The converted modules can be instantiated and used in a C++ or a SystemC testbench,
for verification and/or modelling purposes.

#### Getting Verilator

The installation of Verilator can be done using the OS package manager or using
Git from the source (install from source is preferred). The entire installation
process can be performed following the [Verilator Installation](https://verilator.org/guide/latest/install.html)
tutorial.

#### Getting started with Verilator

Please refer to [Verilator getting started](https://itsembedded.com/dhd/verilator_1/)
which is a very good tutorial. For mmore information please refer to
[Verilator User's Guide](https://verilator.org/guide/latest/).

### GTKWave waveforms viewer

GKWave is application very useful when one want to see waveforms of circuit
generated before by simulator. The process of instalation of this util is
described in process of installation Verilator given above.

## What next

If someone would like to have a broader view of the entire F4PGA toolchain,
we recommend reading a very good tutorial: [F4PGA open source flow](<https://antmicro.com/blog/2022/09/f4pga-new-build-system-and-cli-tool/>)
