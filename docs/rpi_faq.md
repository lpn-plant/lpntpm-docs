# FAQ & Good to know
###Which chip does the TPM use?
<<<<<<< HEAD
Let's Trust TPM module [https://www.pi3g.com](https://www.pi3g.com) use the 
**Infineon OPTIGA SLB 9670 TPM 2.0** Firmware 7.85 or later. This chip is 
compliant with the TCG TPM 2.0 Specification, revision 1.38. Starting with 
Firmware Version 7.85 the SLB 9670 is certified with Common Criteria EAL4+ and 
FIPS 140-2.

###Can SPI still be used?
Yes, **CS0** can still be used, the TPM module uses **CS1**. 
It is possible to address the TPM module using **CS0**, by satisfying both 
conditions:
=======
Lets Trust TPM module [https://www.pi3g.com](https://www.pi3g.com) use the **Infineon OPTIGA SLB 9670 TPM 2.0** Firmware 7.85 or later. This chip is compiliant to the TCG TPM 2.0 Specification, revision 1.38. Starting with Firmware Version 7.85 the SLB 9670 is certified with Common Criteria EAL4+ and FIPS 140-2.

###Can SPI still be used?
Yes, **CS0** can still be used, TPM module uses **CS1**. 
It is possible to address the TPM module using **CS0**, by satisfy both conditions:
>>>>>>> 58f00eb... Init raspberry test chapter
<br />

* moving the 0-Ohm resistor from position R3 to R2, <br />
* patching the device tree overlay to talk to the module on CS0.
<<<<<<< HEAD
The component placing can be found at: 
[https://www.letstrust.de/uploads/letstrust-v2.0.placement.pdf](https://
www.letstrust.de/uploads/letstrust-v2.0.placement.pdf)
###Can I download a circuit diagram?
The circuit diagram is available at: 
[https://www.letstrust.de/uploads/letstrust-v2.0schematic.pdf](https://
www.letstrust.de/uploads/letstrust-v2.0schematic.pdf)
### How do I get support?
Many topics could be found at 
[https://www.letsTrust.de](https://www.letsTrust.de)<br />
For more information in-depth questions, please get in touch with support: 
**<suport@pi3g.com>**
### Can you supply a custom version of the TPM module?
Starting at just 100 modules Trust TPM can modify the design for special 
requests. More information at: **<suport@pi3g.com>**
=======
The componrnt placing can be found at: [https://www.letstrust.de/uploads/letstrust-v2.0.placement.pdf](https://www.letstrust.de/uploads/letstrust-v2.0.placement.pdf)
###Can I download a circuit diagram?
The circuit diagram is available at: [https://www.letstrust.de/uploads/letstrust-v2.0schematic.pdf](https://www.letstrust.de/uploads/letstrust-v2.0schematic.pdf)
### How do I get support?
Many topics could be found at [https://www.letsTrust.de](https://www.letsTrust.de)<br />
For more information in-depth questions, please get in touch with support: **<suport@pi3g.com>**
### Can you supply custom version of the TPM module?
Starting at just 100 modules Trust TPM can modify the design for special requests. More information at: **<suport@pi3g.com>**

>>>>>>> 58f00eb... Init raspberry test chapter
