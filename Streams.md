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

Note that in this example we had only a single input signal (`payload.data`). This is because the Endpoint definition for the UART
was defined as a record with a single member (`data`):
```python
self.sink = stream.Endpoint([("data", 8)])
```
This means that streams can carry bundles of related signals (migen `Record` type).
Click [here](https://github.com/m-labs/migen/blob/master/examples/basic/record.py) for a more in-depth example of using `Record`.

## A Classic Streams Example
While the UART example above may have been easy to get started with, it violates the Streams protocol:
Actually when both `valid` *and* `ready` are asserted, then _only_ may the data 'legally' handed on.
We illustrate this with a very prominent example, which you are likely to encounter: Clock Domain Crossings.

Since streams transport dataflows from one peripheral to another (for example camera to USB), they almost surely will cross clock domains,
because those peripherals operate with different clock frequencies and thus are said to be in different _clock domains_.
High speed USB, for example, operates at 60MHz, while the default system clock in LiteX tends to be at 50MHz, or more, depending on the board. Also, cameras and audio equipment operate at different clock frequencies altogether.
Since all those devices represent streams of continually flowing data, LiteX provides a standard way of crossing from one clock domain into another:
```python
from migen                         import *
from litex.soc.interconnect.stream import ClockDomainCrossing
from litex.build.generic_platform  import *

dut = ClockDomainCrossing(layout=[("data", 8)], cd_from="usb", cd_to="sys", depth=4)
dut.clock_domains += ClockDomain("usb")

def write(value: int, first=False, last=False):
    if first:
        yield dut.sink.first.eq(1)
    if last:
        yield dut.sink.last.eq(1)
    yield dut.sink.payload.data.eq(value)
    yield dut.sink.valid.eq(1)
    yield
    if first:
        yield dut.sink.first.eq(0)
    if last:
        yield dut.sink.last.eq(0)
    yield dut.sink.payload.data.eq(0)
    yield dut.sink.valid.eq(0)
    #yield

def testbench_usb():
    yield from write(0xaa, first=True)
    yield from write(0xbb)
    yield from write(0xcc)
    yield from write(0xdd)
    yield
    yield
    yield
    yield
    yield from write(0x55)
    yield from write(0xff, last=True)
    for i in range(0, 4):
        yield

def testbench_sys():
    yield
    yield dut.source.ready.eq(0)
    for i in range(0, 10):
        yield
    yield dut.source.ready.eq(1)

run_simulation(dut, dict(usb = testbench_usb(), sys = testbench_sys()),
                    vcd_name="cdc-slow-to-fast.vcd",
                    clocks={"sys": 10, "usb": 16})
```
In this code we a data stream flows from the slower USB domain (period 16ns) into the faster sys clock domain (period 10ns).
As we can clearly see, the data will only pass through the stream, if both `valid` and `ready` are asserted. Data which is valid, but not ready at the receiving end, will be dropped:
![image](https://user-images.githubusercontent.com/148607/112723471-37fe7000-8f41-11eb-8a64-711a1743c542.png)
We also can see nicely here, how `first` and `last` mark the first and the last payload byte of a packet of transferred data.

I would strongly encourage you to copy and paste the simulation and experiment with it: Try to move valid and ready around, and see how the signals react. This will give you a good intuition of how it behaves.




