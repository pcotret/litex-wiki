```
                       __   _ __      _  __
                      / /  (_) /____ | |/_/
                     / /__/ / __/ -_)>  <
                    /____/_/\__/\__/_/|_|
                         Migen inside

                Build your hardware, easily!
              Copyright 2012-2019 / EnjoyDigital
```
[![](https://travis-ci.com/enjoy-digital/litex.svg?branch=master)](https://travis-ci.com/enjoy-digital/litex)
![License](https://img.shields.io/badge/License-BSD%202--Clause-orange.svg)
# Welcome to LiteX wiki!
LiteX is a FPGA design/SoC builder that can be used to build cores, create
SoCs and full FPGA designs.

LiteX is based on Migen and provides specific building/debugging tools for
a higher level of abstraction and compatibily with the LiteX core ecosystem.

Think of Migen as a toolbox to create FPGA designs in Python and LiteX as a
SoC builder to create/develop/debug FPGA SoCs in Python.

# Typical LiteX design flow:
```
                        +---------------+
                        |FPGA toolchains|
                        +----^-----+----+
                             |     |
                          +--+-----v--+
         +-------+        |           |
         | Migen +-------->           |
         +-------+        |           |        Your design
                          |   LiteX   +---> ready to be used!
                          |           |
+----------------------+  |           |
|LiteX Cores Ecosystem +-->           |
+----------------------+  +-^-------^-+
 (Eth, SATA, DRAM, USB,     |       |
  PCIe, Video, etc...)      +       +
                           board   target
                           file    file
```
LiteX already supports various softcores CPUs: LM32, Mor1kx, PicoRV32, VexRiscv
and is compatible with the LiteX's Cores Ecosystem:

| Name                                                         | Build Status                                                            | Description                   | Supported Standards                                                            | Supported Hardware                                                                                                           |
| ------------------------------------------------------------ | ----------------------------------------------------------------------- | ----------------------------- | ------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| [LiteDRAM](http://github.com/enjoy-digital/litedram)         | ![](https://travis-ci.org/enjoy-digital/litedram.svg?branch=master)     | Dynamic RAM controller        | SDRAM, DDR, LPDDR, DDR2, DDR3, DDR4                                            | Generic&nbsp;Verilog,<br>Xilinx&nbsp;Spartan&nbsp;6&nbsp;+&nbsp;7&nbsp;Series&nbsp;+&nbsp;Ultrascale,<br>Lattice&nbsp;ECP5   |
| [LiteEth](http://github.com/enjoy-digital/liteeth)           | ![](https://travis-ci.com/enjoy-digital/liteth.svg?branch=master)       | Ethernet                      | 100, 1000 Mbit, MII, GMII & RGMII and many high speed tranceivers              | Generic&nbsp;Verilog,<br>Xilinx&nbsp;Spartan&nbsp;6&nbsp;+&nbsp;7&nbsp;Series&nbsp;+&nbsp;Ultrascale,<br>Lattice&nbsp;ECP5   |
| [LitePCIe](http://github.com/enjoy-digital/litepcie)         | ![](https://travis-ci.com/enjoy-digital/litepcie.svg?branch=master)     | PCIe                          | Gen1, Gen2, x1, x2 x4                                                          | Xilinx&nbsp;7&#8209;series,<br>Intel Cyclone V,&nbsp;and<br>soon Lattice ECP5                                                |
| [LiteSATA](http://github.com/enjoy-digital/litesata)         | ![](https://travis-ci.com/enjoy-digital/litesata.svg?branch=master)     | SATA                          | 1.5/3.0/6.0 GBps                                                               | Xilinx&nbsp;7&#8209;series                                                                                                   |
| [LiteUSB](http://github.com/enjoy-digital/liteusb)           | ![](https://travis-ci.com/enjoy-digital/liteusb.svg?branch=master)      | USB transfer                  |                                                                                |                                                                                                                              |
| [LiteSDCard](http://github.com/enjoy-digital/litesdcard)     | ![](https://travis-ci.com/enjoy-digital/litesdcard.svg?branch=master)   | SD card                       | SD / SDHC / SDXC / SDUC, Default Speed, High Speed, UHS-I                      | Xilinx&nbsp;Spartan&nbsp;6&nbsp;+&nbsp;7&nbsp;Series                                                                         |
| [LiteICLink](http://github.com/enjoy-digital/liteiclink)     | ![](https://travis-ci.com/enjoy-digital/liteiclink.svg?branch=master)   | Inter-Chip communication      | Custom protocol over Single Ended or LVDS Pair                                 | Xilinx&nbsp;7&#8209;series&nbsp;+&nbsp;Ultrascale                                                                            |
| [LiteJESD204B](http://github.com/enjoy-digital/litejesd204b) | ![](https://travis-ci.com/enjoy-digital/litejesd204b.svg?branch=master) | JESD204B                      |                                                                                | Xilinx&nbsp;7&#8209;series&nbsp;+&nbsp;Ultrascale                                                                            |
| [LiteVideo](http://github.com/enjoy-digital/litevideo)       | ![](https://travis-ci.com/enjoy-digital/litevideo.svg?branch=master)    | DVI, HDMI                     | DVI, HDMI                                                                      | Xilinx&nbsp;Spartan&nbsp;6&nbsp;+&nbsp;7&#8209;series                                                                        |
| [LiteScope](http://github.com/enjoy-digital/litescope)       | ![](https://travis-ci.com/enjoy-digital/litescope.svg?branch=master)    | Embedded FPGA logic analyzer  | PCIe, UART, Ethernet                                                           | Generic&nbsp;Verilog                                                                                                         |
