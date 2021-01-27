LiteX's streams are group of signals (Migen's Record) used to create Sink/Source Endpoints on Modules and easily transfer data between them. LiteX's streams are a simplified compromise between AXI-4/Avalon-ST:
* `valid`: similar to AXI-4's `tvalid`.
* `ready`: similar to AXI-4's `tready`.
* `first` (used for packets only when needed): similar to Avalon-ST's `sop`, because it's sometimes useful to just easily know the start of a packet instead of deducing it from last as done in AXI-4.
* `last` (used for packets) : similar to AXI-4's `last`.
* `payload`: a Record with its own layout, similar to AXI-4's `tdata` that can evolve at each valid/ready transaction.
*  `param` (optional): a Record with its own layout, similar to AXI-4's `tuser` that can evolve at each start of packet.

LiteX's streams can be directly mapped to AXI-4 and adapted easily to AvalonST with: https://github.com/enjoy-digital/litex/blob/master/litex/soc/interconnect/avalon.py.

A basic real world example is provided by `litex.soc.cores.uart`.
It has a `StreamEndpoint` called `sink`, which
enables the user of the module to stream data into the module as follows:
```python
#!/usr/bin/env python3

from migen import *
from litex.soc.cores.uart import RS232PHYTX
from litex.build.generic_platform import *


baudrate = 115200
clk_freq = 50e6
tuning = int((baudrate/clk_freq)*2**32)
pads = Record([
         ("tx", 1),
         ("rx", 1)
       ])
        
dut = RS232PHYTX(pads, tuning)

def write(value: int):
    yield dut.sink.payload.data.eq(value)
    yield dut.sink.valid.eq(1)
    yield
    yield dut.sink.payload.data.eq(0)
    yield
    yield dut.sink.valid.eq(0)
    while (yield dut.sink.ready != 1):    
        yield

def testbench():
    yield from write(0)
    yield from write(0xaa)
    yield from write(0x55)
    yield from write(0xff)

run_simulation(dut, testbench(), vcd_name="rs232-tx.vcd")
```
![image](https://user-images.githubusercontent.com/148607/105935498-a11e5000-6084-11eb-94f4-6fe27b320bb5.png)

When the sink becomes ready (ie the UART is ready to transmit another byte), the `ready` signal is asserted, and data
can be written by setting up the payload data as above and asserting the `valid` signal.
When the `valid` signal is asserted, the data is read by the module. In the example above, you can see that it is clocked into the transmit register of the UART.

The whole trace of the above example then looks like this:
![image](https://user-images.githubusercontent.com/148607/105935950-71237c80-6085-11eb-901b-a56f5dd62c64.png)

