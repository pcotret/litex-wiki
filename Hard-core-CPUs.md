LiteX currently has minimal support (compiling and running BIOS, with UART serial terminal, connected to CSR bus in gateware) on these following hard ARM CPUs: Xilinx Zynq-7000, QuickLogic EOS S3, Gowin EMCU.

# Xilinx Zynq-7000

Demonstrated on [Digilent Zedboard](https://github.com/litex-hub/litex-boards/blob/master/litex_boards/targets/digilent_zedboard.py). 
As any complete workflow on Zynq devices requires a number of tools, scripts and build steps besides software and gateware binaries built by LiteX to produce and program boot images - refer to [this repository](https://github.com/sergachev/litex-template/tree/main/digilent_zedboard) for a complete implementation and details.
Serial terminal is on the USB-UART.

Note: producing a working gateware currently requires [patching LiteX locally](https://github.com/enjoy-digital/litex/issues/1154) in some way to remove address decoding on the CSR bus.


# QuickLogic EOS S3

Demonstrated on [QuickLogic QuickFeather](https://github.com/litex-hub/litex-boards/blob/master/litex_boards/targets/quicklogic_quickfeather.py).
Use `python lib/litex-boards/litex_boards/targets/quicklogic_quickfeather.py --build` with path to [QORC SDK](https://github.com/QuickLogic-Corp/qorc-sdk) provided in QORC_SDK environment variable to implement - refer to [this repository](https://github.com/sergachev/litex-template) for an example.
Serial terminal is on pins J3.2/J3.3.

Note: producing a working gateware currently requires [patching LiteX locally](https://github.com/enjoy-digital/litex/issues/1154) in some way to remove address decoding on the CSR bus.


# Gowin EMCU

Demonstrated on [Sipeed Tang Nano 4K](https://github.com/litex-hub/litex-boards/blob/master/litex_boards/targets/sipeed_tang_nano_4k.py). 
To implement use `python litex-boards/litex_boards/targets/sipeed_tang_nano_4k.py --cpu-type=gowin_emcu --build`. Program with Gowin Programmer in MCU mode using project.fs and bios.bin. Look at the [platform file](https://github.com/litex-hub/litex-boards/blob/master/litex_boards/platforms/sipeed_tang_nano_4k.py#L32) for the locations of the serial terminal pins.
