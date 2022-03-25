# TPM modules pinout

This document describes a few different variants of the TPM module pinouts
which are available on some of the PC mainboards. This research was made to
design a few variants of the TPM modules with the most popular pinouts. In this
chapter below, the same pinouts of TPM modules were grouped. The analyzed
mainbaords are chosen based on what we have available for testing, but also
based on the best sellers positions from the
[Newegg.com](https://www.newegg.com/) for some of the most recent mainboards.

## LPC interface

LPC was (and still is) the most common interface for the TPM.

### AsRock Rack X470 D4U, ROMED8-2T Server MB, EPC621D8A, H170M PRO4

Below there are two different pinouts that could be treated as interchangeable.
PCICLK is a 33MHz standard clock (same with CK_33M_TPM). F_CLKRUN# is a signal
when asserted instructs chipset to supply a clock signal. On the pinout below
it is tied to GND which means always asserted (clock is always running).

![pinout](images/pinouts/AsRock_Rack_X470_D4U_Server.svg)
![pinout](images/pinouts/Asrock-H170M_PRO4.svg)

- Manuals:
  + [User Manual X470](https://download.asrock.com/Manual/X470D4U.pdf)
  + [User Manual ROMED8-2T](https://download.asrock.com/Manual/ROMED8-2T.pdf)
  + [User Manual EPC621D8A](https://download.asrock.com/Manual/EPC621D8A.pdf)
  + [User Manual H170M](https://download.asrock.com/Manual/H170M%20Pro4.pdf)

### Asus M5A99FX Pro r2.0, MAXIMUS VII HERO, Z87 PLUS, MAXIMUS VII RANGER

![pinout](images/pinouts/Asus-M5A99FX_Pro_r2.0.svg)

- Manuals:
  + [User Manual M5A99FX](https://images10.newegg.com/UploadFilesForNewegg/itemintelligence/ASUS/E7472_M5A99FX_PRO_R21401484751352.pdf)
  + [User Manual MAXIMUS VII HERO](http://dlcdnet.asus.com/pub/ASUS/mb/LGA1150/MAXIMUS-VII-HERO/E9192_Maximus_Vii_Hero.pdf)
  + [User Manual Z87 PLUS](https://images10.newegg.com/UploadFilesForNewegg/itemintelligence/ASUS/E7831_Z87_PLUS1403687574304.pdf)
  + [User Manual MAXIMUS VII RANGER](https://dlcdnets.asus.com/pub/ASUS/mb/LGA1150/MAXIMUS-VII-RANGER/E9798_maximus_vii_ranger_ug_v2_WEB.pdf)

### Asus MAXIMUS IX FORMULA, CROSSHAIR VIII DARK HERO, ROG STRIX B450-F Gaming

![pinout](images/pinouts/Asus-MAXIMUS_IX_FORMULA.svg)

- Manuals:
  + [User Manual MAXIMUS IX FORMULA](http://dlcdnet.asus.com/pub/ASUS/mb/LGA1151/MAXIMUS_IX_FORMULA/E12314_MAXIMUS_IX_FORMULA_UM_V3_WEB.pdf)
  + [User Manual ROG CROSSHAIR](https://media.s-bol.com/Bnx1GRw6DXXo/original.pdf)
  + [User Manual B450-F](https://www.bhphotovideo.com/lit_files/403638.pdf)

### Asus TPM-M R2.0, Z97 PRO GAMER

![pinout](images/pinouts/Asus-TPM-M_R2.0.svg)

- Manuals:
  + [User Manual TPM-M](https://www.manualslib.com/manual/1447187/Asus-Tpm-M-R2-0.html)
  + [User Manual Z97 PRO GAMER](https://dlcdnets.asus.com/pub/ASUS/mb/LGA1150/Z97-PRO_GAMER/E10265_Z97-PRO_GAMER_Guide_v2_web_hi_res.pdf)

### Gigabyte GA A320M-S2H AMD A320, GA B250 HD3P, B450 Aorus Pro, GA Z77 D3H, GA 970A-UD3P

Below there are two different pinouts that could be treated as interchangeable.
SUSCLK is a RTC clock signal which is not used for TPM so it could be ignored.

![pinout](images/pinouts/Gigabyte-GA-Z77X-D3H.svg)
![pinout](images/pinouts/Gigabyte-GA-A320M-S2H-AMD-A320.svg)

- Manuals:
  + [User Manual Z77X D3H](https://download1.gigabyte.com/Files/Manual/mb_manual_ga-z77x-d3h_e.pdf)
  + [User Manual Z77 D3H](https://download1.gigabyte.com/Files/Manual/mb_manual_ga-z77x-d3h_e.pdf)
  + [User Manual 970A-UD3P](https://download1.gigabyte.com/Files/Manual/mb_manual_ga-970a-ud3p_e.pdf)
  + [User Manual A320M-S2H](https://gzhls.at/blob/ldb/0/a/1/c/f15c2f507a34fa4fb5f2d78875ef5477838c.pdf)
  + [User Manual B250 HD3P](https://download.gigabyte.com/FileList/Manual/mb_manual_ga-b250-hd3p_e.pdf)
  + [User Manual B450 AORUS PRO](https://download.gigabyte.com/FileList/Manual/mb_manual_b450-aorus-pro-wifi_1002_e_190528.pdf)

### MSI B75MA E33, MSI B85M E45, MSI B350 TOMAHAWK, MSI X99A GAMING 9 ACK

![pinout](images/pinouts/MSI-B75MA-E33.svg)

- Manuals:
  + [User Manual B75MA E33](https://www.msi.com/Motherboard/b75ma-e33/support#down-manual)
  + [User Manual B85M E45](https://pl.msi.com/Motherboard/B85M-E45/support#down-manual)
  + [User Manual TOMAHAWK](https://pl.msi.com/Motherboard/B350-TOMAHAWK/support#down-manual)
  + [User Manual X99A](https://www.msi.com/Motherboard/x99a-gaming-9-ack/support#down-manual)

### Supermicro X10DAL-i, Supermicro MBD M12SWA

Below there are two different pinouts that could be treated as interchangeable.

![pinout](images/pinouts/Supermicro_X10DAL-i.svg)
![pinout](images/pinouts/Supermicro_MBD_M12SWA.svg)

- Manuals:
  + [User Manual Supermicro MBD M12SWA](https://www.supermicro.com/manuals/motherboard/M12/MNL-2336.pdf)
  + [User Manual X10DAL-i](https://www.supermicro.com/manuals/motherboard/C600/MNL-1701.pdf)

## SPI interface

SPI interface becomes more and more common for TPM modules, especially for the
more recent mainboards. We can expect that this interface will be more commonly
used in the modern mainbards, as the LPC interface becomes slowly deprecated for
modern processors.

### Asus-B550 PLUS TUF GAMING, Asus-ROG STRIX X570-F GAMING

Below there are two different pinouts that could be treated as interchangeable.

![pinout](images/pinouts/Asus-ROG_STRIX_X570-F_GAMING.svg)
![pinout](images/pinouts/Asus-B550-PLUS.svg)

- Manuals:
  + [User Manual STRIX X570-F](https://gamingprofy.com/wp-content/uploads/2021/06/E15827_ROG_STRIX_X570-F_GAMING_UM_v2_WEB.pdf)
  + [User Manual B550 PLUS](https://dlcdnets.asus.com/pub/ASUS/mb/SocketAM4/TUF_GAMING_B550-PLUS/E16576_TUF_GAMING_B550-PLUS_UM_WEB.pdf)

### MSI Z590 PRO WiFi

![pinout](images/pinouts/MSI-Z590_PRO_WiFi.svg)

- Manuals:
  + [User Manual Z590](https://download.msi.com/archive/mnu_exe/mb/M7D09v1.2.pdf)

### Gigabyte Z590 AORUS MASTER

![pinout](images/pinouts/Gigabyte-Z590_AORUS_MASTER.svg)

- Manuals:
  + [User Manual](https://download.gigabyte.com/FileList/Manual/mb_manual_z590-aorus-master_1002_e.pdf)

## Summary

There is a relation between TPM modules pinouts at the same vendors. For
typical motherboards from one and similar series, pinouts are the same. On the
other hand, the same manufacturer also supports different pinouts in the case
of different models of motherboards. So it couldn't be said that always the
same vendor supports the same pinout.

During our research, we have identified:

- 6 unique LPC TPM pinouts:
  + AsRock (18-pin)
  + Asus (20-pin)
  + Asus (14-pin)
  + Gigabyte (20-pin)
  + MSI (14-pin)
  + Supermicro (20-pin)
- 3 unique SPI TPM pinouts:
  + Asus (14-pin)
  + MSI (12-pin)
  + Gigabyte (12-pin)

These pinouts will be used to define hardware requirements for the TPM boards
and, later on, to deisgn TPM adapter boards.
