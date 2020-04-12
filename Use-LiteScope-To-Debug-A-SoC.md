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

Let's say we have a CPU in our design and want to be able to visualize the Wishbone accesses of the instruction bus, the first thing we do is creating the list of Migen signals we want to be observe:

```python3
analyzer_signals = [
    self.cpu.ibus.stb,
    self.cpu.ibus.cyc,
    self.cpu.ibus.adr,
    self.cpu.ibus.we,
    self.cpu.ibus.ack,
    self.cpu.ibus.sel,
    self.cpu.ibus.dat_w,
    self.cpu.ibus.dat_r,
]
```

Then we create the Litescope Analyzer that will be able to capture these signals:

```python3
self.submodules.analyzer = LiteScopeAnalyzer(analyzer_signals,
    depth        = 512,
    clock_domain = "sys",
    csr_csv      = "analyzer.csv")
self.add_csr("analyzer")
```

The Analyzer is configured with a `depth` of *512* samples, doing the capture in the *sys* `clock domain` of the SoC and exporting the its configuration (`csr_csv`) to *analyzer.csv* file that will be used by the software during the trigger/capture.

> Note: Since samples are stored in embedded block rams of the FPGA, the depth will be limited by the number of available block rams in your design/FPGA. The total number of bits used for the capture is len(analyzer_signals)*depth.

> Note: LiteScope also accepts Migen's Records, so in our example we could just have used:  `analyzer_signals = [self.cpu.ibus]` but it would have been less understandable. Imagine we also want to capture the data bus, we could just do:  `analyzer_signals = [self.cpu.ibus, self.cpu.dbus]`. 

We can now build our SoC and start using the Analyzer!

# Use the Analyzer:

TODO
