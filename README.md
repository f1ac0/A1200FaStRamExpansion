# A1200 FaSt RAM Expansion
This project is a 8MB autoconfig Fast SRAM and 1.5MB Ranger RAM trapdoor expansion for Amiga 1200, with 512KB kickstart mapROM.

Facts about this project :
- It uses fast SRAM so it does not need any waitstate.
- The 8MB range is 32 bits. The 1.5MB and mapROM are 16 bits only.
- PCMCIA compatibility / 4MB autoconfig RAM : for permanent setup, close the 4M switch before applying power ; for temporary setup, use the software.
- 2-layers PCB only.

# Disclaimer
This is a hobbyist project, it comes with no warranty and no support. Also remember that the Amiga machines are about 30 years old and may fail because of such hardware expansions.

I publish my work under the CC-BY-NC-SA license. The content of this repository is the result of weeks of work, requiring investment in tools, parts, prototypes, and risky tests on his own equipment. For this reason the author does not want third parties to sell products and keep all the profit, usually without even offering support to their customers. If you see people selling this without the consent of the author, don't buy from them.

If you find it useful and want to reward me : I am always looking for Amiga/Amstrad CPC hardware to repair and hack, please contact me.

# BOM
- 1x XC9536XL CPLD
- 5x CY62167ELL-45ZXI 5V SRAM (do not use 3.3V CY62167EV !). You may also install only 2 or 4 if you only need 4 or 8 MB
- 1x 3.3v LDO, either SPX3819M5-L-3-3 or XC6206P332MR
- 2x 1uF (or more) 0805 capacitors
- 8x 100nF 0805 capacitors
- 1x 10k 0805 resistor
- 1x FCN-224J 150 pins card-edge connector

# Making it
Components are SMD, and have thin pin pitch. You need to know what you are doing.

1.0 PCB require 1.0 firmware.

0.1 PCB can be used with 1.0 firmware but require the modifications below :
- cut the trace between CPLD pin 1 and the closest via
- remove soldermask from the via and connect a wire to CPLD pin 3
- wire pin 133 of the connector (/OVR) to CPLD pin 1

OBSOLETE : 0.1 PCB can be used with 0.1 firmwares with OVR, Ranger RAM and mapROM given the modifications below :
- install R1 between the pads of the 4M jumper and leave its footprint unpopulated
- wire pin 133 of the connector (/OVR) to pin 1 of the 4M jumper (the square one)
- always leave the 4M jumper open

0.1 PCB can be used with the 0.1 firmware without OVR, Ranger RAM nor MapROM
- do not install U7 since it will never be used

Check for shorts at least between 5V, 3.3V, and GND traces before applying power !

The programming port does not need to be soldered since it needs to be programmed just once : you can just hold it in place during the few seconds required for programming.

CPLD code is generated and built into xsvf using Xilinx ISE 14.7 IDE then iMPACT.

There are several methods to program the XC9536XL. I personally use xsvfduino : https://github.com/wschutzer/xsvfduino

# Using it
For 1.0 firmware with 1.0 PCB or 0.1 modified PCB
- Turn off Amiga
- For 4MB operation/PCMCIA compatibility, close the 4M jumper
- For 8MB operation, open the 4M jumper
- Turn on the Amiga
- Ranger RAM and mapROM are always available

For OVR compatible 0.1 firmware with modified 0.1 PCB
- Turn off Amiga
- For 4MB operation/PCMCIA compatibility, disconnect the /OVR bodge wire from the 4M jumper pad
- For 8MB operation with ranger RAM and mapROM, plug the /OVR bodge wire to the 4M jumper pad
- Turn on the Amiga

For non OVR 0.1 firmware with 0.1 PCB
- 4/8MB jumper is latched and can be changed live, new setting take effect at next reboot.
- NO Ranger RAM and NO mapROM

# Amiga FLACOntrol Software
You may use the the FLACOntrol software from A500-IDE-RAM repository to control the board from your OS.

It provides a MapROM function that writes a 256K or 512K ROM ROM image to F80000-FFFFFF, and an AddMem to make the 1.5MB at C00000-D7FFFF available to the system.
It also communicates with this expansion using a control register byte at $00E9Cxxx :
- bit 7 indicates if MapROM is active, write 0 to disable it at next reboot ;
- bit 6 indicates that the MapROM is available
- bit 5 is writable to force PCMCIA compatibility mode : write 1 to force 4MB, and 0 to revert to the default controlled by the 4M jumper ; it takes effect at next reboot.

To add the 1.5MB Ranger RAM to the system :
```
FLACOntrol -m
```
To force 4Mb mode ; this will take effect at next reboot :
```
FLACOntrol -4
```
It will revert to default at next power cycle or with :
```
FLACOntrol -8
```

To map the internal ROM (might speed up the system) :
```
FLACOntrol -i
```
Or a different ROM from file, 256K and 512K are supported :
```
FLACOntrol -f Files:path/kickstart-image-not-swapped-nor-split.rom
```
It will revert to the hardware ROM at next power cycle or with :
```
FLACOntrol -x
```
You may want to add MapROM to your startup-sequence. The -r flag tells the software to reboot the system after the new ROM is loaded, then during next reboot the software will exit since MapROM is already active, and your system will finish to boot with the new ROM :
```
FLACOntrol -f Files:path/kickstart-image-not-swapped-nor-split.rom -r
```

To use the Ranger RAM you could also use an AddMem software such as http://aminet.net/package.php?package=util/cli/AddMem.lha :
```
AddMem START C00000 TO D7FFFF PRI -5 NAME rangermem
```

