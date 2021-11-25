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

- building and running VCOM project `Samples/Nucleo-TPM/VCOM`
(Visual Studio 2017).
- first successful communication with TPM core
- investigation the root cause of response check failing
[link](https://github.com/lpn-plant/ms-tpm-20-ref/blob/master/Samples/Nucleo-TPM/VCOM/VCOM-TPM/VCOM-TPM.cpp#L198)

15.10.2021

- small fix for NDEBUG enabled compilation
- protocol analysis

18.10.2021

- protocol analysis

19.10.2021

- protocol analysis
- decoding command and response data frames
- setting up lib tpm2-tss and tpm2-pytss for future communication

20.10.2021

<<<<<<< HEAD
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
=======
- protocol analysis
- investigation of lib tpm2-tss and tpm2-pytss behaviour and wrong packet
reception
>>>>>>> lpn/main

