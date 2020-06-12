For SoCs with CPU(s), LiteX offers various approaches/methods to load and jump to application's code from the BIOS.

# ROM Boot:
Running the application code from a embedded ROM is the fastest way to execute code and could be interesting when the code is small enough or on large devices where many block-rams are available at almost no cost, or simply when the execution speed is critical. 

An boot ROM with contents from `bootrom.bin` file, located at `0x20000000` can be added to the SoC with:
```python3
self.add_rom("bootrom", 0x20000000, contents=get_mem_data("bootrom.bin", endianness="big"))
```
Defining `ROM_BOOT_ADDRESS` in the SoC will make it jump to the defined value at startup:
```python3
self.add_constant("ROM_BOOT_ADDRESS", 0x20000000)
```
# Serial Boot:
Loading application code from serial is very convenient for development since provides an easy way
to (re)load application code, boot to it and do quick iterations. LiteX supports very various serial interfaces that can be configured for the board with:

`--uart_name=serial`: _Regular_ UART at 115200Bauds, ~10-15KB/s upload.

`--uart_name=serial --uart_baudrate=1e6`: _Boosted_ UART at 1MBauds, ~80-90KB/s upload.

`--uart_name=serial --uart_baudrate=3e6`: _Overboosted_ UART at 3MBauds :), ~250KB/s upload.

`--uart_name=usb_fifo`: UART through FT245 FIFO, ~300KB/s upload.

`--uart_name=usb_acm`: UART through USB ACM Valenty USB core, ~160KB/s upload.

`--uart_name=jtag_uart` UART through JTAG core, **TBD** upload.

`--uart_name=etc... or create yours!`

Serial boot is generally slow compared to Ethernet or SDCard boot but is the easiest to setup and usable on all boards so is still very interesting for easy use or when no alternative are available.

LiteX's [lxterm](https://github.com/enjoy-digital/litex/blob/master/litex/tools/litex_term.py) tool is used to ulpload the application code and is directly installed with LiteX, available with the `lxterm` command.

Serial boot has priority over the other boot methods and the BIOS will always try to boot from it. If `lxterm` is running on the Host, it upload the binary(ies) at startup: 
```bash
--============== Boot ==================--
Booting from serial...
Press Q or ESC to abort boot completely.
sL5DdSMmkekro
[LXTERM] Received firmware download request from the device.
[LXTERM] Uploading buildroot/Image to 0x40000000 (4545524 bytes)...
[LXTERM] Upload complete (162.0KB/s).
[LXTERM] Uploading buildroot/rootfs.cpio to 0x40800000 (8029184 bytes)...
[LXTERM] Upload complete (163.3KB/s).
[LXTERM] Uploading buildroot/rv32.dtb to 0x41000000 (1995 bytes)...
[LXTERM] Upload complete (191.2KB/s).
[LXTERM] Uploading emulator/emulator.bin to 0x41100000 (9600 bytes)...
[LXTERM] Upload complete (161.1KB/s).
[LXTERM] Booting the device.
[LXTERM] Done.
Executing booted program at 0x41100000
--============= Liftoff! ===============--
...
```
Otherwise it will try the next available boot method, here for example the falling back to SDCard:
```bash
--============== Boot ==================--
Booting from serial...
Press Q or ESC to abort boot completely.
sL5DdSMmkekro
Timeout
Booting from SDCard in SPI-Mode...
...
```

 1. Uploading a single binary:

To load `boot.bin` to default location (`RAM at 0x4000000`), start `lxterm` with:
```bash
lxterm /dev/ttyUSBX --kernel=boot.bin
```
 2. Uploading multiple binaries with a *JSON* description file:

When multiple binaries need to be loaded at different locations, a *JSON* description file with file/address tuples can be used, as for example the `images.json` file used for [Linux-on-LiteX-VexRiscv](https://github.com/litex-hub/linux-on-litex-vexriscv):
```json
{
	"buildroot/Image":        "0x40000000",
	"buildroot/rootfs.cpio":  "0x40800000",
	"buildroot/rv32.dtb":     "0x41000000",
	"emulator/emulator.bin":  "0x41100000"
}
```

The **last file/address tuple** is used for the CPU **jump** (so here it will jump `0x41100000`).

To use a _JSON_ file, start `lxterm` with:
```bash
lxterm /dev/ttyUSBX --images=images.json
```
> **Note:** For large binaries, it's possible boost upload speed (and live dangerously :)) by disabling the CRC check with --no-crc.

# Ethernet Boot:
On SoCs with Ethernet capability, loading application code with it can be very convenient for development since flexible  and fast (on a Linux SoC, loading binaries only takes a few seconds).

Ethernet boot requires a *TFTP* server on the Host. LiteX uses `192.168.1.50` as default Host IP address, this default IP address can be changed by defining the `REMOTEIPX` constants in the SoC:
```python3
self.add_constant("REMOTE1", 192)
self.add_constant("REMOTE2", 192)
self.add_constant("REMOTE3", 192)
self.add_constant("REMOTE4", 192)
```

 1. Uploading multiple binaries with a *JSON* description file:

The BIOS will first try to load a `boot.json` file over TFTP that will provide the list of files that need to be loaded and their location in the SoC, as for example the `boot.json` file used in [Linux-on-LiteX-VexRiscv](https://github.com/litex-hub/linux-on-litex-vexriscv)
```json
{
	"Image":        "0x40000000",
	"rootfs.cpio":  "0x40800000",
	"rv32.dtb":     "0x41000000",
	"emulator.bin": "0x41100000"
}
```
2. Uploading a single binary:

When `boot.json` is available on the TFTP (or when boot.json is invalid), the BIOS will fallback to `boot.bin`. So when only one binary is used, it could be simply named `boot.bin` and put on the TFTP, the BIOS will loaded it and will jump to default location (`RAM at 0x4000000`).

Example of Linux ethernet boot on [Arty A7](https://store.digilentinc.com/arty-a7-artix-7-fpga-development-board-for-makers-and-hobbyists/) board:
![enter image description here](https://cdn10.bigcommerce.com/s-7gavg/product_images/attribute_rule_images/6425_zoom_1527801259.png)

```bash
--============== Boot ==================--
Booting from serial...
Press Q or ESC to abort boot completely.
sL5DdSMmkekro
Timeout
Booting from network...
Local IP : 192.168.1.50
Remote IP: 192.168.1.100
Booting from boot.json...
Copying Image to 0x40000000... (4545524 bytes)
Copying rootfs.cpio to 0x40800000... (8029184 bytes)
Copying rv32.dtb to 0x41000000... (2167 bytes)
Copying emulator.bin to 0x41100000... (9600 bytes)
Executing booted program at 0x41100000

--============= Liftoff! ===============--
```

# SPIFlash Boot:
TODO

# SDCard Boot:
Loading application code from SDCard is very interesting for standalone embedded systems and allow very large binaries. 

The BIOS supports loading files from FAT16/FAT32 partitions and boot scheme is similar to Ethernet boot: 
File format is identical, BIOS will first  look for a valid `boot.json` file, will eventually fallback to `boot.bin` if `boot.json` is not available or not valid, and will fails if neither of these are found/valid.

> **Note:** SDCard Boot currently only supports SPI-Mode but support for native SDCard 4-bit interface will be added in the future (so with the increased load speed).

Example of Linux SDCard boot on the [OrangeCrab](https://groupgets.com/campaigns/710-orangecrab):
![enter image description here](https://raw.githubusercontent.com/gregdavill/OrangeCrab/master/documentation/hugo-files/static/r0.2/orangeCrab-1.jpg)
[![asciicast](https://asciinema.org/a/cFQ7JRH96mgJNcuI0Ntey663H.svg)](https://asciinema.org/a/cFQ7JRH96mgJNcuI0Ntey663H)