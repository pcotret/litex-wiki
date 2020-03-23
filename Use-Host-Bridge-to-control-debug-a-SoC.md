# Add a UART bridge to your SoC:

Being able to easily read/write the addressable space of the SoC from a Host PC with simple scripts can be very powerful and greatly help/speed up debugging.

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

# Add a Ethernet bridge to your SoC:
FIXME: TODO

# Add a PCIe bridge to your SoC:
FIXME: TODO

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