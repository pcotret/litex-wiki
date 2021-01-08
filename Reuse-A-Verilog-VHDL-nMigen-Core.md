Because there is a large variety of open cores available on the internet and
also you might want to choose another HDL than Migen for several reasons,
it is very likely you will want to integrate cores written in other languages into your project.

In this example we will demonstrate the integration of an nMigen core into our project by
converting it to verilog first.

## Converting nMigen to Verilog to prepare for integration
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
The last `if`-statement is instrumental for the conversion to verilog: The name argument will determine the module name in verilog,
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

## Integrating a Verilog Core
Now we can now to integrate the verilog into our module:
```python
#!/usr/bin/env python3
from migen import Instance
from migen.fhdl.module import Module
from migen.fhdl.structure import Signal, ClockSignal, ResetSignal
from litex.build.sim.platform import SimPlatform
from litex.build.sim.config import SimConfig
from litex.soc.integration.builder import Builder
from litex.soc.integration.soc_core import SoCMini
from litex.build.generic_platform import Pins

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

    _io = [
        ("sys_clk", 0, Pins(1)),
        ("sys_rst", 0, Pins(1))
    ]

    def __init__(self):
        SimPlatform.__init__(self, "SIM", self._io)

class SimSoc(SoCMini):
    def __init__(self):
        platform = DemoPlatform()
        sys_clk_freq = int(100e6)
        SoCMini.__init__(self, platform, clk_freq=sys_clk_freq)
        dut = VerilogDemo(self.platform, pads=[])
        self.submodules += dut

def main():
    soc = SimSoc()
    builder = Builder(soc)
    sim_config = SimConfig(default_clk="sys_clk")
    builder.build(sim_config=sim_config)

if __name__ == "__main__":
    main()
```
Note that when instantiating the verilog module, the module's arguments are prepended with
the following prefixes as annotations for migen:
* `i_` for an input
* `o_` for an output
* `io_` for a bidirectional port
* `p_` for a module parameter (like eg. bit width of ports etc.)

## Integrating a VHDL Core
TODO: (probably analogous to verilog?)



