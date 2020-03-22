# LiteX Projects

Collections of projects with LiteX involved that could be useful for users to better understand LiteX, go further and/or find some inspiration.

## [Linux-on-LiteX-VexRiscv](https://github.com/litex-hub/linux-on-litex-vexriscv)
```
                                   __   _
                                  / /  (_)__  __ ____ __
                                 / /__/ / _ \/ // /\ \ /
                                /____/_/_//_/\_,_//_\_\
                                      / _ \/ _ \
                      __   _ __      _\___/_//_/ __             _
                     / /  (_) /____ | |/_/__| | / /____ __ ____(_)__ _____  __
                    / /__/ / __/ -_)>  </___/ |/ / -_) \ // __/ (_-</ __/ |/ /
                   /____/_/\__/\__/_/|_|    |___/\__/_\_\/_/ /_/___/\__/|___/

                   Copyright (c) 2019-2020, Linux on LiteX VexRiscv Developers
```
 - Have an FPGA board with 32MB of RAM and want to test Linux on it? 
 - Want to study/explore a RISC-V Linux capable SoC?
 - Want to create a full autonomous SoC with LiteX and its peripherals (SPI, I2C, SDCard, FrameBuffer, etc...) managed by Linux?

[Linux-on-LiteX-VexRiscv](https://github.com/litex-hub/linux-on-litex-vexriscv) project demonstrates how to create a Linux capable SoC with [VexRiscv](https://github.com/SpinalHDL/VexRiscv) CPU, a 32-bits Linux Capable RISC-V CPU written in Spinal HDL. A SoC around the VexRiscv CPU is created using LiteX as the SoC builder and LiteX's cores written in Migen Python DSL (LiteDRAM, LiteEth, LiteSDCard). All the components used to create the SoC are open-source and the flexibility of Spinal HDL/Migen allow targeting easily very various FPGA devices/boards: Lattice, Altera, Xilinx, Microsemi FPGAs with SDRAM/DDR/DDR2/DDR3/DDR4 RAMs, RMII/MII/RGMII/1000BASE-X Ethernet PHYs. On Lattice ECP5 FPGAs, the [open source toolchain](https://github.com/SymbiFlow/prjtrellis) allows creating full open-source SoC with open-source cores **and** toolchain!

This project demonstrates **how high level HDLs (Spinal HDL, Migen) enable new possibilities and complement each other**. Results shown there are the results of a productive collaboration between open-source communities.ant to test easily control LiteX cores.

## [NeTV2](https://www.crowdsupply.com/alphamax/netv2)
![enter image description here](https://bunniefoo.com/netv2/netv2-slogan-white.png)
![enter image description here](https://www.crowdsupply.com/img/222e/netv2-board-angle-bgd_jpg_project-body.jpg)
The NeTV2 is a HDMI capture/playback board based on an Xilinx Artix7 FPGA.

The [official SoC](https://github.com/AlphamaxMedia/netv2-fpga) is doing an overlay (up to 1080p60) from the Raspberry Pi 3B+ on an HDMI stream with or without HDCP. The SoC of the Artix7 FPGA is built with LiteX, a VexRiscv CPU is used for the control, [LiteDRAM](https://github.com/enjoy-digital/litedram) as the DDR3 controller to store the CPU firmware and the do HDMI buffering and LiteVideo](https://github.com/enjoy-digital/litevideo)  for the HDMI playback/capture.

The [libre SoC](https://github.com/enjoy-digital/netv2) makes use of the PCIe Gen2 X4 connector of the board with [LitePCIe](https://github.com/enjoy-digital/litepcie) to create a HDMI capture/playback device.

## [Fomu](https://www.crowdsupply.com/sutajio-kosagi/fomu)
![enter image description here](https://www.crowdsupply.com/img/9023/updated-fomu-penny_png_project-body.jpg)

The [Fomu](https://www.crowdsupply.com/sutajio-kosagi/fomu) is a tiny FPGA board that fits in your USB port. The SoC of the FPGA is built with LiteX,  a custom version of [VexRiscv](https://github.com/SpinalHDL/VexRiscv) CPU is used for the control and the [ValentyUSB](https://github.com/im-tomu/valentyusb) core allows the board to communicate with the Host computer and emulate various USB devices. Fomu demonstrates how iCE40 designs can be **tiny** and **powerful**: Fomu can run Micropython/Zephyr on its RISC-V CPU while allowing **software debug** over GDB and **gateware debug** over the USB bridge and is able to **reprogram itself** directly from USB with [Foboot](https://github.com/im-tomu/foboot). The [Fomu Workshop](https://workshop.fomu.im/en/latest/) workshop is also a great introduction to FPGA design.

## [A Trustworthy, Free (Libre), Linux Capable,  Self-Hosting 64bit RISC-V Computer](http://www.contrib.andrew.cmu.edu/~somlo/BTCP/)


## [USB3 PIPE](https://github.com/enjoy-digital/usb3_pipe)
![enter image description here](https://raw.githubusercontent.com/enjoy-digital/usb3_pipe/master/doc/breakout_board.jpg)

Current solutions for USB3 connectivity with an FPGA require the use of an external SerDes chip ([TI TUSB1310A - SuperSpeed 5 Gbps USB 3.0 Transceiver with PIPE and ULPI Interfaces](http://www.ti.com/product/TUSB1310A)) or external FIFO chip ([FTDI FT60X](https://www.ftdichip.com/Products/ICs/FT600.html) or Cypress [FX3](https://www.cypress.com/products/ez-usb-fx3-superspeed-usb-30-peripheral-controller)). The aim of this project is to experiment with [High Speed Transceivers (SERDES)](https://en.wikipedia.org/wiki/Multi-gigabit_transceiver) of popular FPGAs to create a [USB3.0 PIPE interface](https://www.intel.com/content/dam/www/public/us/en/documents/white-papers/phy-interface-pci-express-sata-usb30-architectures-3.1.pdf) and see if it's possible to just use the transceivers of the FPGA for the USB3 connectivity and have the USB3 PIPE directly implemented in the fabric (and then avoid any external chip!)

## [PCIe Analyzer](https://github.com/enjoy-digital/pcie_analyzer)
![enter image description here](https://github.com/enjoy-digital/pcie_analyzer/raw/master/doc/banner.jpg)

The aim of this project is to create a PCIe interposer + FPGA capture board for PCIe signals capture and analysis with cheap hardware.

## [Betrusted.io](https://betrusted.io/)
![enter image description here](https://betrusted.io/assets/images/betrusted-banner.png)

Betrusted is a protected place for your private matters. It’s built from the ground up to be checked by anyone, but sealed only by you. Betrusted is more than just a secure CPU – it is a system complete with screen and keyboard, because privacy begins and ends with the user.

Betrusted prototypes are built around two FPGAs SoC built with LiteX: The iCE40 based [Embedded controller](https://github.com/betrusted-io/betrusted-ec) SoC and the Spartan7 based [Secure Domain](https://github.com/betrusted-io/betrusted-soc) SoC.

## [Chubby75](https://github.com/q3k/chubby75)
![enter image description here](https://github.com/q3k/chubby75/raw/master/5a-75b/images/cl-5a-75b-v61-front-annotated.jpg)

Chubby75 project documents cheap FPGA based led controllers (15$ shipping included) that can be repurposed as FPGA development boards with 2x 1Gbps interfaces / Spartan6 or ECP5 FPGAs / SDRAM, be used as GPIOs over network devices or for their original purpose: control led panels for a [led cube](https://github.com/NiklasFauth/colorlight-led-cube) :) LiteX and [LiteEth](https://github.com/enjoy-digital/liteeth) have been tested on the reversed pinout and boards are supported in [LiteX-boards](https://github.com/litex-hub/litex-boards).