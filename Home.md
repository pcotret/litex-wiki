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

- LiteDRAM: [![](https://travis-ci.org/enjoy-digital/litedram.svg?branch=master)](https://travis-ci.com/enjoy-digital/litedram) http://github.com/enjoy-digital/litedram
- LiteEth: ![](https://travis-ci.com/enjoy-digital/liteth.svg?branch=master) http://github.com/enjoy-digital/liteeth
- LitePCIe: ![](https://travis-ci.com/enjoy-digital/litepcie.svg?branch=master) http://github.com/enjoy-digital/litepcie
- LiteSATA: ![](https://travis-ci.com/enjoy-digital/litesata.svg?branch=master) http://github.com/enjoy-digital/litesata
- LiteUSB: ![](https://travis-ci.com/enjoy-digital/liteusb.svg?branch=master) http://github.com/enjoy-digital/liteusb
- LiteSDCard: ![](https://travis-ci.com/enjoy-digital/litesdcard.svg?branch=master) http://github.com/enjoy-digital/litesdcard
- LiteICLink: [![](https://travis-ci.com/enjoy-digital/liteiclink.svg?branch=master)](https://travis-ci.com/enjoy-digital/liteiclink) http://github.com/enjoy-digital/liteiclink
- LiteJESD204B: [![](https://travis-ci.com/enjoy-digital/litejesd204b.svg?branch=master)](https://travis-ci.com/enjoy-digital/litejesd204b) http://github.com/enjoy-digital/litejesd204b
- LiteVideo: ![](https://travis-ci.com/enjoy-digital/litevideo.svg?branch=master) http://github.com/enjoy-digital/litevideo
- LiteScope: ![](https://travis-ci.com/enjoy-digital/litescope.svg?branch=master) http://github.com/enjoy-digital/litescope