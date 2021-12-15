## The TPM concept
According to the many varied pinouts necessary was design the concept of TPM 
module which can fit the motherboard pinout. There was the consideration of 
two kinds of the interface (SPI and LPC). The whole setup would be made of two 
PCBs. One is the main PCB with the MCU and SPI or LPC configured socket and the 
second is the adapter that fits the motherboards pinouts.
![Flash](images/renders/koncepcja_TPM_adapter.svg)

## The main PCB
On the PCB there will be soldering only one chosen type of socket (SPI or LPC). 
This PCB will include the MCU, decoupling capacitors, and all necessary passive 
components.
![Flash](images/renders/2.png)
<center>![Flash](images/renders/3.png)</center>

## The adapter
This PCB is the second part to fit the SPI or LPC pinout to the motherboard 
pinouts. There will be created many variants (according to the pinout research) 
of adapters. We consider creating a straight and angled socket to be ready for
more existing mechanical design.
<center>![Flash](images/renders/4.png)</center>
<center>![Flash](images/renders/5.png)</center>
<center>![Flash](images/renders/6.png)</center>
<center>![Flash](images/renders/8.png)</center>

## Assembly
According to the adapter PCB, we consider a straight or angled connector. The 
size of PCBs is indicative.
<center>![Flash](images/renders/9.png)</center>
<center>![Flash](images/renders/7.png){: style="height:425px"}</center>

## Example of connection with motherboar
<center>![Flash](images/renders/MB1.png)</center>
<center>![Flash](images/renders/MB2.png)</center>
