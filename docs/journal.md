## Journal

04-08.10.2021

- Fixing compilation problems.
- Converting project to STM32CubeIDE.
- Setup ITM trace logging.

11-15.10.2021

- Getting USB CDC work.
- Experiments with tpm2-tools.
- Writing documentation.
- Investigation of memory usage and choosing best suited hw platform.

14.10.2021

- Building and running VCOM project `Samples/Nucleo-TPM/VCOM`
(Visual Studio 2017).
- First successful communication with TPM core.
- Investigation the root cause of response check failing.
[link](https://github.com/lpn-plant/ms-tpm-20-ref/blob/master/Samples/Nucleo-TPM/VCOM/VCOM-TPM/VCOM-TPM.cpp#L198)

15.10.2021

- Small fix for NDEBUG enabled compilation.
- Protocol analysis.

18.10.2021

- Protocol analysis.

19.10.2021

- Protocol analysis
- Decoding command and response data frames.
- Setting up lib tpm2-tss and tpm2-pytss for future communication.

20/21.10.2021

- Protocol analysis.
- Investigation of lib tpm2-tss and tpm2-pytss behavior and wrong packet
reception.

26.10.2021

- USB communication reworked, added circular buffer and functions responsible
for command validation.
- Development of python scripts responsible for serial communication and
investigation the root cause of tpm2_tool @ /dev/ACM* failures.

27.10.2021

- Cleanup and refactor of python scripts.
- Writing documentation.

28.10.2021

- Kernel development setup.
- Browsing tpm related kernel code.

02.11.2021

- Writing docs.
- Function profiler prototype refactor.
- Experimenting with tpm module debug build.

03.11.2021

- Investigation of dynamic debug behavior.
- Writing how-to for kernel module recompilation.

04.11.2021
- Writing how-to for kernel instrumentation.
  https://github.com/lpn-plant/lpntpm-docs/pull/7

05-09.11.2021
- Raspberry Pi / STM32 SPI communication.
- TPM SPI specification research.

12.11.2021
- TPM SPI specification research.

22.11.2021
- Hardware interfaces requirements - documentation.
  https://github.com/lpn-plant/lpntpm-docs/pull/6/files

10.11.2021

- Raspberry Pi demonstration with TPM module
