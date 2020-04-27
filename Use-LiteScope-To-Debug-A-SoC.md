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

Now that the SoC is instrumented and built, we can start using the analyzer. The first step is to run the LiteX server on the Host to allow communicating with the SoC and execute scripts. The previous example is integrated in [LiteX Sim](https://github.com/enjoy-digital/litex/blob/master/litex/tools/litex_sim.py) and we are going to use it here.

The simulation can be run with `litex_sim --with-ethernet --with-etherbone` and the LiteX server started with `litex_server --udp --udp-ip=192.168.1.51`.

To analyze the instruction bus of the CPU, we create the following `litescope_analyzer.py` script:

```python3
#!/usr/bin/env python3

import sys
import argparse

from litex import RemoteClient
from litescope import LiteScopeAnalyzerDriver

parser = argparse.ArgumentParser()
parser.add_argument("--ibus_stb",  action="store_true", help="Trigger on IBus Stb rising edge.")
parser.add_argument("--ibus_adr",  default=0x00000000,  help="Trigger on IBus Adr value.")
parser.add_argument("--offset",    default=128,         help="Capture Offset.")
parser.add_argument("--length",    default=512,         help="Capture Length.")
args = parser.parse_args()

wb = RemoteClient()
wb.open()

# # #

analyzer = LiteScopeAnalyzerDriver(wb.regs, "analyzer", debug=True)
analyzer.configure_group(0)
if args.ibus_stb:
	analyzer.add_rising_edge_trigger("simsoc_cpu_ibus_stb")
elif args.ibus_adr:
    analyzer.configure_trigger(cond={"simsoc_cpu_ibus_adr": int(args.ibus_adr, 0)})
else:
    analyzer.configure_trigger(cond={})
analyzer.run(offset=int(args.offset), length=int(args.length))

analyzer.wait_done()
analyzer.upload()
analyzer.save("dump.vcd")

# # #

wb.close()

```

This script will allow us to trigger on a `ibus_stb` rising edge or on a specific `ibus_adr` value.

The LiteX simulation is running and we are able to interact with the BIOS, let's configure the analyzer to trigger on `ibus_stb`rising edge and press enter, this will cause the CPU to fetch some data on the instruction bus and will trigger the capture:

<p align="center"><img src="https://user-images.githubusercontent.com/1450143/80379942-144d2880-889f-11ea-9200-434ab7828c74.png"></p>

```
$./litescope_analyzer.py --ibus_stb
[running]...
[uploading]...
|====================>| 100%
[writing to dump.vcd]...
```
We can then open the wave with GTKWave and analyze the access:
<p align="center"><img src="https://user-images.githubusercontent.com/1450143/80379951-1911dc80-889f-11ea-829c-8c820e4e1d01.png"></p>

Now let's trigger on `ibus_adr=0x00000000` (which is the reset address of the CPU) and run the reboot command of the BIOS:
<p align="center"><img src="https://user-images.githubusercontent.com/1450143/80379955-1b743680-889f-11ea-9151-d05700cea637.png"></p>

```
$./litescope_analyzer.py --ibus_adr=0x00000000
[running]...
[uploading]...
|====================>| 100%
[writing to dump.vcd]...
```
We can then open the wave with GTKWave and analyze the access:
<p align="center"><img src="https://user-images.githubusercontent.com/1450143/80379959-1d3dfa00-889f-11ea-8f04-23b4ca910fc1.png"></p>