There are already various possibilities to run Linux on a LiteX SoC or with LiteX's components:

## [Linux-on-LiteX-VexRiscv](https://github.com/litex-hub/linux-on-litex-vexriscv)
<div>
<img src="https://linuxgizmos.com/files/gsd_orangecrab_frontback.jpg" width="300">
<img src="https://pbs.twimg.com/media/EM6jskWXUAAflwE.jpg" width="300">
</div>
 
 - Have an FPGA board with 32MB of RAM and want to test Linux on it? 
 - Want to study/explore a RISC-V Linux capable SoC?
 - Want to create a full autonomous SoC with LiteX and its peripherals (SPI, I2C, SDCard, FrameBuffer, etc...) managed by Linux?

[Linux-on-LiteX-VexRiscv](https://github.com/litex-hub/linux-on-litex-vexriscv) project demonstrates how to create a Linux capable SoC with [VexRiscv](https://github.com/SpinalHDL/VexRiscv) CPU, a 32-bits Linux Capable RISC-V CPU written in Spinal HDL. A SoC around the VexRiscv CPU is created using LiteX as the SoC builder and LiteX's cores written in Migen Python DSL (LiteDRAM, LiteEth, LiteSDCard). All the components used to create the SoC are open-source and the flexibility of Spinal HDL/Migen allow targeting easily very various FPGA devices/boards: Lattice, Altera, Xilinx, Microsemi FPGAs with SDRAM/DDR/DDR2/DDR3/DDR4 RAMs, RMII/MII/RGMII/1000BASE-X Ethernet PHYs. On Lattice ECP5 FPGAs, the [open source toolchain](https://github.com/SymbiFlow/prjtrellis) allows creating full open-source SoC with open-source cores **and** toolchain!

This project demonstrates **how high level HDLs (Spinal HDL, Migen) enable new possibilities and complement each other**. Results shown there are the results of a productive collaboration between open-source communities.ant to test easily control LiteX cores.

With recent improvements on VexRiscv SMP and SDCard support, it can even boot Linux from reset in just a few seconds, as demonstrated on the [Arty A7](https://twitter.com/enjoy_digital/status/1285996750696742912).

![enter image description here](https://user-images.githubusercontent.com/1450143/89118966-96567a00-d4aa-11ea-9827-999a8ffd1444.jpg)

## [A Trustworthy, Libre, Self-Hosting 64bit RISC-V Linux Computer](http://www.contrib.andrew.cmu.edu/~somlo/BTCP/)
<img src="http://www.contrib.andrew.cmu.edu/~somlo/BTCP/RocketLitexHi.png">

Adding the [Rocket Chip](https://github.com/chipsalliance/rocket-chip) (64-bit RISC-V CPU) to the LiteX ecosystem, this project is capable of booting a *nearly* unmodified 64-bit upstream Linux kernel (with only the LiteEth network driver currently patched in).

Targeted toward the [ECP5-5G-Versa](https://www.latticesemi.com/en/Products/DevelopmentBoardsAndKits/ECP55GVersaDevKit) and the [TrellisBoard](https://github.com/daveshah1/TrellisBoard) (but also known to work on the [Nexys 4 DDR](https://store.digilentinc.com/nexys-a7-fpga-trainer-board-recommended-for-ece-curriculum/)), the ultimate goal is to boot a Linux distro (e.g., [Fedora](https://fedoraproject.org/wiki/Architectures/RISC-V)), and use native riscv64 builds of the Libre FPGA toolchain ([yosys](https://github.com/YosysHQ/yosys)/[trellis](https://github.com/SymbiFlow/prjtrellis)/[nextpnr](https://github.com/YosysHQ/nextpnr)) to build the underlying FPGA-based computer's **_own_** bitstream!

For the latest details, check out [http://www.contrib.andrew.cmu.edu/~somlo/BTCP](http://www.contrib.andrew.cmu.edu/~somlo/BTCP).

## OpenRisc / Mor1kx

TODO

## [Linux-On-LiteX-BlackParrot](https://github.com/scanakci/linux-on-litex-blackparrot)
![BlackParrot](https://github.com/black-parrot/black-parrot/raw/master/docs/bp_logo.png)

The [BlackParrot](https://github.com/black-parrot/black-parrot) CPU support has been added recently to LiteX and Linux demonstrated on it in Linux-On-LiteX-BlackParrot project:

![BlackParrot](https://user-images.githubusercontent.com/1450143/89119006-fcdb9800-d4aa-11ea-98f0-ca27c6a9b11f.jpeg)

[https://www.youtube.com/watch?v=npeDkfEMsoI&feature=youtu.be](https://www.youtube.com/watch?v=npeDkfEMsoI&feature=youtu.be)

## [Boot Linux on Microwatt](https://shenki.github.io/boot-linux-on-microwatt/)

![Microwatt](https://shenki.github.io/images/microwatt-title.png)

The Microwatt CPU is supported by LiteX but not yet capable of booting Linux directly when integrated in LiteX. The [Boot Linux on Microwatt](https://shenki.github.io/boot-linux-on-microwatt/) blogpost demonstrates how to boot Linux on the Microwatt CPU with an interesting approach where LiteX is not used as the SoC builder,  but just providing the DRAM controller and initialization code as a standalone core generated with [LiteDRAM generator](https://github.com/enjoy-digital/litedram/blob/master/litedram/gen.py) and integrated in a VHDL SoC. This **approach is really interesting** since also demonstrates that **just some components** of the LiteX ecosystem **can be reused separately in a more traditional flow** and that user is not enforced to use LiteX for the integration.

