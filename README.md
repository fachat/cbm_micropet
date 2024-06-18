# Micro-PET R3

Note: this is a part of a larger set of repositories, with [upet_family[() as the main repository.

This is a re-incarnation of the Commodore PET computer(s) from the later 1970s.

It is build on a PCB slightly larger than a Eurocard board and has only parts that can still be obtained new in 2024.
The current version is 3.0A

![Picture of a MicroPET](images/cover.jpg)

Here is the [Overview video](https://youtu.be/COlfqcaY7rI)

Some more videos on the build process can be found here [YT 8-bit times](https://youtube.com/playlist?list=PLi1dzy7kw1iybjcUccgjCV4fhNH4IPWSx)

## Features

The board is built with a number of features:

- Commodore 3032 / 4032 / 8032 / 8296 with options menu to select at boot
  - Boot-menu to select different PET versions to run (BASIC 1, 2, 4)
  - 40 and 80 col character display
  - 8296 memory map emulation
  - IEEE488 interface (card edge and 24pin flat ribbon cable)
  - Tape connector (card edge, 5V only)
  - PET graphics keyboard, or alternatively a C64 keyboard
- Improved system design:
  - 512k video RAM, plus 512k fast RAM, accessible using banks on the W65816 CPU
  - boot from an SPI Flash ROM
  - up to 17.5 MHz mode (via configuration register)
  - Write protection for the PET ROMs once copied to RAM
  - lower 32k RAM mappable from all of the 512k fast RAM
  - Audio output using a DAC with DMA
- Improved Video output:
  - VGA colour video output
  - up to 96x72 characters on screen
  - hires modes up to a 768x576 resolution
  - 16 out of 64 colour palette, Colour-PET or C128 VDC-compatible, Sprites
  - Hires graphics mode (using a configuration register)
  - modifyable character set
  - multiple video pages mappable to $8000 video mem address

## Overview

The system architecture is actually rather simple, as you can see in the following graphics.

![MicroPET System Architecture](images/upet-system-architecture.png)

The main functionality is "hidden" inside the FPGA. It does:

1. clock generation and management
2. memory mapping
3. video generation.
4. SPI interface and boot
5. DAC DMA

On the CPU side of the FPGA it is actually a rather almost normal 65816 computer, 
with the exception that the bank register (that catches and stores the address lines 
A16-23 from the CPU's data bus) is in the FPGA, and that there is no ROM. The ROM has been
replaced with some code in the FPGA that copies the initial program to the CPU accessible
RAM, taking it from the Flash Boot ROM via SPI. This actually simplifies the design,
as 

1. parallel ROMs are getting harder to come by and
2. they are typically not as fast as is needed, and
3. with the SPI boot they don't occupy valuable CPU address space.

The video generation is done using time-sharing access to the video RAM.
The VGA output is 768x576 at 60Hz. So there is a 28ns slot per pixel on the screen, 
with a pixel clock of 35MHz.

The system runs at 17.5MHz, so a byte of pixel output (i.e. eight pixels) has four
memory accesses to VRAM. Two of them are reserved for video access, one for fetching the
character data (e.g. at $08xxx in the PET), and the second one to fetch the "character ROM"
data, i.e. the pixel data for a character. This is also stored in VRAM, and is being loaded
there from the Flash Boot ROM by the initial boot loader.

The FPGA reads the character data, stores it to fetch the character pixel data, and streams
that out using its internal video shift register.

For more detailled descriptions of the features and how to use them, pls see the 
[FPGA repository](https://github.com/fachat/upet_fpga) that contains the code for the FPGA,
as described in the next section.

## Building

Here are four subdirectories:

- [Board](Board/) that contains the board schematics and layout
- [Case](Case/) 3-D printed supports and keyboards to mount board in a C64c case

In addition, two other repositories are needed - they are separate as they are shared with other variants of the upet-family:

- [FPGA](https://github.com/fachat/upet_fpga) contains the VHDL code to program the FPGA logic chip used, and describes the configuration options - including the SPI usage, the DAC usage, and the Video features.
- [ROM](https://github.com/fachat/upet_roms) ROM contents to boot


### Board

To have the board built, you can use the gerbers that are stored in the zip file in the Board/production subdirectory.

To populate the board, there is an interactive bom (bill of materials) from KiCad, as well as the KiCad BOM CSV export in the [bom](bom/) folder.
Note at this time, part numbers are not available yet, please compare with the [Ultra-CPU](https://github.com/fachat/csa_ultracpu) and [PETIO](http://www.6502.org/users/andre/csa/petio/index.html) boards.

The BOM contains an Ethernet breakout board that is put into the connectors "above" the USB port. As an alternative you could use the 
[Wifi breakout board](https://github.com/fachat/upet_wifi). Note that at this time, this is not tested / programmed yet.

### FPGA

The FPGA is a Xilinx Spartan 6 programmable logic chip. It runs on 3.3V and it is programmed in VHDL.

To build the FPGA content, clone the [FPGA](https://github.com/fachat/upet_fpga) repository, and look for the ShellUPet.bin.
This needs to be programmed into the SPI flash chip containing the configration for the FPGA.

### ROM

To build the boot ROM image, clone the [ROM](https://github.com/fachat/upet_roms) repository, and build the spiimg70m file. This needs to be 
programmed into the SPI flash ROM containing the 65816 boot code.

The ROM image can be built using gcc, xa65, and make. Use your favourite EPROM programmer to burn it into the SPI Flash chip.

The ROM contains images of all required ROM images for BASIC 1, 2, and 4, and corresponding editor ROMs, including
some that have been extended with wedges.

The updated editor ROMs are from [Steve's Editor ROM project](http://www.6502.org/users/sjgray/projects/editrom/index.html) and can handle C64 keyboards, has a DOS wedge included, and resets into the Micro-PET boot menu.
For more details see the description in the ROM repository.

### Case

The [Case](Case/) subdirectory holds some supporting 3D models, to use an existing C64-II case for the Micro-PET. 

Also, it contains the models for custom key caps for a PET N-type keyboard, as is default for the Micro-PET. 
As keyboard PCB I suggest using Steve Gray's [PET keyboard replacements](http://cbmsteve.ca/mxkeyboards/index.html) using modern, MX-type key switches.

## Gallery (Still R2)

![Picture of a MicroPET](images/upet-c64kbd.jpg)

Note: the published schematics and board has the extra wires fixed/included.

![MicroPET with self-printed key caps](images/case-with-caps.jpg)

![MicroPET running the 8296 burnin](images/8296diag.jpg)

![MicroPET before adjusting a C64 case](images/upet.png)
 
