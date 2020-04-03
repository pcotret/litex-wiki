When developing a new core/interface or bringing up a new system composed of various cores, things are often not behaving as expected... This could be due to various reasons like a bug in a core, a miss interpretation of a specification, difference in interpretation between developers, etc...

To understand why the design is not behaving correctly, the internal signals of the FPGA need to be observed, in a similar way signals are observed during simulation to help/assist design.

The difference between hardware and simulation is that on hardware the signals have to be captured in real time, without perturbing the system, and with the limited resources available in the FPGA.

[LiteScope](https://github.com/enjoy-digital/litescope) is a small footprint and configurable embedded FPGA logic analyzer that has been developed for that purpose and we'll explain in the following sections how to integrate it in the SoC and how to use it.

# Add a bridge to your SoC:

With LiteScope, we want to capture internal signals of our design, store their values in an embedded memory of the FPGA and upload these datas to a Host PC to visualize them and debug/understand the issue.

To use LiteScope in LiteX, the first step is then to add a bridge connection between the Host PC and our FPGA board to allow the Host PC to access the main bus of our SoC and interact with it.

LiteX provides natives bridges that can operate over a [UART](https://github.com/enjoy-digital/litex/wiki/Use-Host-Bridge-to-control-debug-a-SoC#add-a-uart-bridge-to-your-soc), [Ethernet](https://github.com/enjoy-digital/litex/wiki/Use-Host-Bridge-to-control-debug-a-SoC#add-an-ethernet-bridge-to-your-soc) or [PCIe](https://github.com/enjoy-digital/litex/wiki/Use-Host-Bridge-to-control-debug-a-SoC#add-a-pcie-bridge-to-your-soc) and the integration of a such bridge in the SoC is explained [here](https://github.com/enjoy-digital/litex/wiki/Use-Host-Bridge-to-control-debug-a-SoC).

Alternative ways to create a bridges have also been developed, such as USB/SPI (used on Fomu), SPI. #FIXME: add more infos.

# Add a LiteScope Analyzer to your SoC:

TODO

# Use the Analyzer:

TODO
