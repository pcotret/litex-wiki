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

| Name                                                         | Build Status                                                            | Description                   |
| ------------------------------------------------------------ | ----------------------------------------------------------------------- | ----------------------------- |
| [LiteDRAM](http://github.com/enjoy-digital/litedram)         | [![](https://travis-ci.org/enjoy-digital/litedram.svg?branch=master)](https://travis-ci.org/enjoy-digital/litedram)     | DRAM        |
| [LiteEth](http://github.com/enjoy-digital/liteeth)           | [![](https://travis-ci.com/enjoy-digital/liteth.svg?branch=master)](https://travis-ci.com/enjoy-digital/liteth)       | Ethernet                      |
| [LitePCIe](http://github.com/enjoy-digital/litepcie)         | [![](https://travis-ci.com/enjoy-digital/litepcie.svg?branch=master)](https://travis-ci.com/enjoy-digital/litpcie)     | PCIe                          |
| [LiteSATA](http://github.com/enjoy-digital/litesata)         | [![](https://travis-ci.com/enjoy-digital/litesata.svg?branch=master)](https://travis-ci.com/enjoy-digital/litesata)     | SATA                          |
| [LiteSDCard](http://github.com/enjoy-digital/litesdcard)     | [![](https://travis-ci.com/enjoy-digital/litesdcard.svg?branch=master)](https://travis-ci.com/enjoy-digital/litesdcard)   | SD card                       |
| [LiteICLink](http://github.com/enjoy-digital/liteiclink)     | [![](https://travis-ci.com/enjoy-digital/liteiclink.svg?branch=master)](https://travis-ci.com/enjoy-digital/liteiclink)   | Inter-Chip communication      |
| [LiteJESD204B](http://github.com/enjoy-digital/litejesd204b) | [![](https://travis-ci.com/enjoy-digital/litejesd204b.svg?branch=master)](https://travis-ci.com/enjoy-digital/litejesd204b) | JESD204B                      |
| [LiteVideo](http://github.com/enjoy-digital/litevideo)       | [![](https://travis-ci.com/enjoy-digital/litevideo.svg?branch=master)](https://travis-ci.com/enjoy-digital/litevideo)    | VGA, DVI, HDMI                     |
| [LiteScope](http://github.com/enjoy-digital/litescope)       | [![](https://travis-ci.com/enjoy-digital/litescope.svg?branch=master)](https://travis-ci.com/enjoy-digital/litescope)    | Logic analyzer  |
