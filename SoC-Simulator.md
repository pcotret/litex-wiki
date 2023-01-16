TODO

if you want a simple example with litex_sim, you can do:

`litex_sim --integrated-main-ram-size=0x10000 --cpu-type=vexriscv --no-compile-gateware`

then

`litex_bare_metal_demo --build-path=build/sim/`

and

`litex_sim --integrated-main-ram-size=0x10000 --cpu-type=vexriscv --ram-init=demo.bin`

this will create a SoC with PicoRV32 and execute the demo app on it 



