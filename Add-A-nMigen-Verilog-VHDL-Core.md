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
The last `if`-statement is instrumental for the conversion to verilog: The name attribute will determine the module name in verilog,
and the ports list will determine the ports that will face the outside of the module (`clk` and `rst` will be automatically added by nMigen).
We convert to verilog with the following command line:
```bash
$ python3 edgetopulse.py generate -t v edgetopulse.v
```
which, after cleaning up some code generation comments, looks like this:
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



