LiteX is based on Migen but is also heavily used to integrate/reuse traditional Verilog/VHDL cores (commercial or open-source). It can also be used to integrate cores from Spinal-HDL, nMigen or other high level HDL DSL through Verilog.

## Reusing a Verilog core

In this example we will demonstrate the integration of a verilog core. Two steps are involved. The step first is to include the given core through call to platform.add_source given the verilog . The second step is to the instantiate this core.

We are going to modify [lab001](https://github.com/litex-hub/fpga_101/blob/master/lab001/base.py) to use verilog directly and start by writing a small verilog core called *blink.v* .

```
module blink(
    input clk,
    input rst,
    output led
);
  reg [25:0] cnt;
  assign led = cnt[25] == 1;

  always @(posedge clk) begin
    if(rst) begin
        cnt <= 25'd0;
    end else begin
        cnt <= cnt + 1'b1;
    end
  end
endmodule
```

This module can be added by calling `platform.add_source("blink.v")`

and then must be instantiate. This is done using *Instance*

```
#!/usr/bin/env python3
from migen import *
from litex_boards.platforms import tinyfpga_bx

# Create our platform (fpga interface)
platform = platform = tinyfpga_bx.Platform()
platform.add_source("blink.v")

# Create our module (fpga description)
module = Module()
module.specials += Instance("blink",
    i_clk = ClockSignal(),
    i_rst = ResetSignal(),
    o_led = platform.request("user_led")
)

# Build
platform.build(module)

```

The first parameter of the instance is the verilog module name (the verilog file will actually be included in the generated code). The parameters have the same names as the verilog module parameters with the exception that they are prefixed with the addition of a prefix "i_" for input "o_" for output and "p_" for parameter.

A more complex example is [the inclusion of an I2C core obtained through OpenCores](https://github.com/betrusted-io/gateware/blob/master/gateware/i2c/core.py)

## Reusing a VHDL core
TODO

## Reusing a nMigen core

In this example we will demonstrate the integration of an nMigen core into our project by
converting it to verilog first.

### Converting a nMigen core to Verilog
As an example, we will use this simple nMigen core:

```python
#!/usr/bin/env python3
"""converts a rising edge to a single clock pulse"""
from nmigen     import Elaboratable, Signal, Module
from nmigen.cli import main

class EdgeToPulse(Elaboratable):
    """
        each rising edge of the signal edge_in will be
        converted to a single clock pulse on pulse_out
    """
    def __init__(self):
        self.edge_in          = Signal()
        self.pulse_out        = Signal()

    def elaborate(self, platform) -> Module:
        m = Module()

        edge_last = Signal()

        m.d.sync += edge_last.eq(self.edge_in)
        with m.If(self.edge_in & ~edge_last):
            m.d.comb += self.pulse_out.eq(1)
        with m.Else():
            m.d.comb += self.pulse_out.eq(0)

        return m

if __name__ == "__main__":
    m = EdgeToPulse()
    main(m, name="edge_to_pulse", ports=[m.edge_in, m.pulse_out])
```
The last `if`-statement is instrumental for the conversion to verilog: The name attribute will determine the module name in verilog,
and the ports list will determine the ports that will face the outside of the module (`clk` and `rst` will be automatically added by nMigen).
We convert to verilog with the following command line:
```bash
$ python3 edgetopulse.py generate -t v edgetopulse.v
```
which, after cleaning up some code generation comments (to make it more readable for this presentation), 
looks like this:
```verilog
module edge_to_pulse(pulse_out, clk, rst, edge_in);
  reg \initial  = 0;
    wire \$1 ;
    wire \$3 ;
    input clk;
    input edge_in;
    reg edge_last = 1'h0;
    reg \edge_last$next ;
    output pulse_out;
  reg pulse_out;
    input rst;
  assign \$1  = ~ edge_last;
  assign \$3  = edge_in & \$1 ;
  always @(posedge clk)
    edge_last <= \edge_last$next ;
  always @* begin
    if (\initial ) begin end
    \edge_last$next  = edge_in;
        casez (rst)
      1'h1:
          \edge_last$next  = 1'h0;
    endcase
  end
  always @* begin
    if (\initial ) begin end
            casez (\$3 )
      1'h1:
          pulse_out = 1'h1;
      default:
          pulse_out = 1'h0;
    endcase
  end
endmodule
```

### Integrating the generated Verilog core
Now we can the generated nMigen verilog core into our LiteX design:
```python
#!/usr/bin/env python3
from migen                          import Instance
from migen.fhdl.module              import Module
from migen.fhdl.structure           import Signal, ClockSignal, ResetSignal, Finish, If
from litex.build.io                 import CRG
from litex.build.sim.platform       import SimPlatform
from litex.build.sim.config         import SimConfig
from litex.soc.integration.builder  import Builder
from litex.soc.integration.soc_core import SoCMini
from litex.build.generic_platform   import Pins

class VerilogDemo(Module):
    def __init__(self, platform, pads):
        self.edge_signal_in     = Signal()
        self.pulse_signal_out   = Signal()
        self.pulse_signal_out_n = Signal()

        # instantiate the verilog module
        self.specials += Instance("edge_to_pulse",
            i_clk       = ClockSignal(),
            i_rst       = ResetSignal(),
            i_edge_in   = self.edge_signal_in,
            o_pulse_out = self.pulse_signal_out
        )

        # add the verilog source
        # may need to adjust the path if it is
        # in a subdirectory
        platform.add_source("edgetopulse.v")

        # finally do something with it:
        self.comb += self.pulse_signal_out_n.eq(~self.pulse_signal_out)

class DemoPlatform(SimPlatform):
    default_clk_name = "sys_clk"
    clk_freq = int(100e6)

    _io = [
        ("sys_clk", 0, Pins(1)),
        ("sys_rst", 0, Pins(1))
    ]

    def __init__(self):
        SimPlatform.__init__(self, "SIM", self._io)

class SimSoc(SoCMini):
    def __init__(self):
        platform = DemoPlatform()
        SoCMini.__init__(self, platform, clk_freq=platform.clk_freq)
        dut = VerilogDemo(self.platform, pads=[])
        self.submodules += dut
        counter = Signal(16)
        self.sync += [
            counter.eq(counter + 1),
            If(counter[15], Finish())
        ]
        self.comb += [ 
            platform.trace.eq(1),
            dut.edge_signal_in.eq(counter[3])
        ]

def main():
    soc = SimSoc()
    builder = Builder(soc)
    sim_config = SimConfig(default_clk="sys_clk", default_clk_freq=soc.platform.clk_freq)
    builder.build(sim_config=sim_config, trace=True)

if __name__ == "__main__":
    main()
```
Note that when instantiating the Verilog module, the module's arguments are prepended with the following prefixes as annotations for Migen:
* `i_` for an input
* `o_` for an output
* `io_` for a bidirectional port
* `p_` for a module parameter (like eg. bit width of ports etc.)

When the simulation is run, we see the expected output:
![](https://user-images.githubusercontent.com/148607/104075071-56a77180-5244-11eb-9226-590946ea4ff5.png)

