Being able to easily read/write the addressable space of the SoC from a Host PC with simple scripts can be very powerful and greatly help/speed up debugging.

# Add a UART bridge to your SoC:
The easiest way to add a UART bridge to a SoC is to use an unused UART of your FPGA board. If your Platform file already has a an unused UART you can just use it, otherwise you can simply add UART pins to your Platform file by adapting the following template:
```python3
   ("uart_bridge", 0,
        Subsignal("tx", Pins("XY")),
        Subsignal("rx", Pins("YX")),
        IOStandard("LVCMOSXY")
    ),
```
and connect a USB/UART dongle between your FPGA board and your Host PC.

In your SoC, instantiate the UART bridge:
```python3
from litex.soc.cores import uart
uart_bridge = uart.UARTWishboneBridge(
    pads     = platform.request("uart_bridge"),
    clk_freq = self.clk_freq,
    baudrate = 115200)
self.submodules += uart_bridge
self.add_wb_master(uart_bridge.wishbone)
```
and generate the *.csv* file that will be used by the tools:
```python3
builder  = Builder(soc, ..., csr_csv="csr.csv")
```

Once your bitstream is built and loaded to the board, start *litex_server* in UART mode:
```
litex_server --uart --uart-port=/dev/ttyUSBX
```

# Add an Ethernet bridge to your SoC:
Another very convenient way to debug a SoC is to add an Ethernet Bridge to it this allows debugging your SoC over your local ethernet network and provide better speeds.

If your Platform file already has a an unused Ethernet PHY you can just use it, otherwise you can simply add the Ethernet pins to your Platform file by adapting the following template (here for a RMII PHY):
```python3
    ("eth_clocks", 0,
        Subsignal("ref_clk", Pins("X")),
        IOStandard("LVCMOS33"),
    ),

    ("eth", 0,
        Subsignal("rst_n", Pins("X")),
        Subsignal("rx_data", Pins("X X")),
        Subsignal("crs_dv", Pins("X")),
        Subsignal("tx_en", Pins("X")),
        Subsignal("tx_data", Pins("X X")),
        Subsignal("mdc", Pins("X")),
        Subsignal("mdio", Pins("X")),
        Subsignal("rx_er", Pins("X")),
        Subsignal("int_n", Pins("X")),
        IOStandard("LVCMOS33")
     ),
```
For the other types of Ethernet PHYs, you can find some inspiration in [LiteX-Boards](https://github.com/litex-hub/litex-boards/tree/master/litex_boards/platforms).

In your SoC, instantiate the Etherbone bridge:

```python3
from liteeth.phy.rmii import LiteEthPHYRMII
self.submodules.ethphy = LiteEthPHYRGMII(
    clock_pads = self.platform.request("eth_clocks"),
    pads       = self.platform.request("eth"))
self.add_csr("ethphy")
self.add_etherbone(
    phy         = self.ethphy,
    ip_address  = "192.168.1.50",
    mac_address = "0x10e2d5000001"
)
```

and generate the _.csv_ file that will be used by the tools:

```python3
builder  = Builder(soc, ..., csr_csv="csr.csv")
```

Once your bitstream is built and loaded to the board, start _litex_server_ in UDP mode:

```
litex_server --udp --udp_ip=192.168.1.50
```

# Add a PCIe bridge to your SoC:
On PCIe SoCs, the PCIe bridge is already present in the SoC since used to access the memory map of the SoC from the Host through the BAR0. The bridge is added by the following lines in the SoC definition:
```python3
self.submodules.pcie_bridge = LitePCIeWishboneBridge(self.pcie_endpoint,
    base_address = self.mem_map["csr"])
self.add_wb_master(self.pcie_bridge.wishbone)
```

To use this bridge from the Host, make sure the SoC will generate the _.csv_ file that will be used by the tools:
```python3
builder  = Builder(soc, ..., csr_csv="csr.csv")
```

Then with *lspci*, find the location of the PCIe board on the bus, here `02:00.0`:
```
00:00.0 Host bridge: Intel Corporation Xeon E3-1200 v6/7th Gen Core Processor Host Bridge/DRAM Registers (rev 05)
[...]
00:1f.6 Ethernet controller: Intel Corporation Ethernet Connection (2) I219-V
02:00.0 Memory controller: Xilinx Corporation Device 7024
```
And use this to start _litex_server_ in PCIe mode:
```
litex_server --pcie --pcie-bar=/sys/bus/pci/devices/0000\:02\:00.0/resource0
```

# Create scripts to communicate from the Host PC with the bridge:

You can now easily create scripts to read/write from/to the main bus of your SoC:
```python3
#!/usr/bin/env python3

from litex import RemoteClient

wb = RemoteClient()
wb.open()

# # #

# Access SDRAM
wb.write(0x40000000, 0x12345678)
value = wb.read(0x40000000)

# Access SDRAM (with wb.mems and base address)
wb.write(wb.mems.main_ram.base, 0x12345678)
value = wb.read(wb.mems.main_ram.base)

# Trigger a reset of the SoC
wb.regs.ctrl_reset.write(1)
 
# Dump all CSR registers of the SoC
for name, reg in wb.regs.__dict__.items():
    print("0x{:08x} : 0x{:08x} {}".format(reg.addr, reg.read(), name))

[...]

# # #

wb.close()
```