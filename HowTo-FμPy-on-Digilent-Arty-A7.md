# Arty A7 Setup

## Introduction

This how-to guide is for people who want to get started running MicroPython on a [Digilent Arty A7 FPGA board](https://store.digilentinc.com/arty-a7-artix-7-fpga-development-board-for-makers-and-hobbyists/) ([reference information](https://reference.digilentinc.com/reference/programmable-logic/arty-a7/start)), using [FμPy](https://fupy.github.io/).

By the end of this guide you will have a MicroPython [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) running on the board's FPGA using a Soft CPU.

You will need:

* A laptop running Linux, or a Linux VM (Ubuntu 16.04 LTS is well tested)
* A [Digilent Arty A7 board](https://store.digilentinc.com/arty-a7-artix-7-fpga-development-board-for-makers-and-hobbyists/)
* A micro USB cable to connect the Arty A7 to your laptop

This guide will show you how to;

* Configure the FPGA to run a "[[Soft CPU]]"
* Load a [[MicroPython]] interpreter onto the configured FPGA 
* Use the [[MicroPython]] REPL from your laptop

This guide was sourced from Ewen McNeill's [blog post](https://ewen.mcneill.gen.nz/blog/entry/2018-01-17-fupy-fpga-micropython-on-mimas-v2-and-arty-a7/).

All shell commands in this guide use `bash`

## Configure USB Connection

First we need to configure how we connect with the device by creating some rules for [`udev`](https://wiki.debian.org/udev):

```bash
git clone https://github.com/timvideos/HDMI2USB-mode-switch.git
cd HDMI2USB-mode-switch/udev
make install
make reload
```

Next we ensure that your user has permission to access the relevant device files:

```bash
sudo adduser $USER video
sudo adduser $USER dialout       # or 'uucp' group on some Linux distros
sudo reboot
```

(if you do not want to `reboot` it is sufficient to log your user out, and back in again; use `id` to check that your user is in the correct groups, particularly `dialout` or `uucp`).

## Download the Build Environment

Next we download and setup the build environment. This environment contains the tools that we need to build the [gateware](https://github.com/timvideos/litex-buildenv/wiki/Glossary) + [firmware](https://github.com/timvideos/litex-buildenv/wiki/Glossary) and load them onto the FPGA.

```bash
git clone https://github.com/timvideos/litex-buildenv.git
cd litex-buildenv/

# Build targets
CPU=lm32             # Soft CPU architecture
PLATFORM=arty        # Target platform, Arty A7
FIRMWARE=micropython
TARGET=base
export CPU PLATFORM TARGET FIRMWARE

# Download dependencies for the build environment
./scripts/download-env.sh
```

## Enter the build environment

Once the download is done (several minutes) you can enter your build environment.  You will need to do this every time you use the litex build tools (eg, in a new terminal window or a new login).  The environment variable settings are the same as above.

```bash
CPU=lm32
PLATFORM=arty
TARGET=base
FIRMWARE=micropython
export CPU PLATFORM TARGET FIRMWARE

source scripts/enter-env.sh
```

The Arty A7 board requires some proprietary tools to build the [FPGA gateware](https://github.com/timvideos/litex-buildenv/wiki/Glossary).

For this getting-started guide we will be using a prebuilt gateware image file, so you do not need to download the Xilinx tools at this point.

To customise your own gateware you will need the [Xilinx Vivado WebPACK FPGA synthesis tool](https://www.xilinx.com/products/design-tools/vivado/vivado-webpack.html) installed and reachable from `/opt/Xilinx`.  The "Webpack" version of Xilinx Vivado, which can be run with a no-cost license file, is sufficient for `litex-buildenv`.  We will not install Vivado for this guide. Instead, let's download a prebuilt gateware image:

```bash
sudo apt install subversion
# Download the prebuilt gateware file
./scripts/download-prebuilt.sh
make firmware-cmd       # Fix the file paths in the downloaded image
```

If you see segmentation faults and "`sudo dmesg`" shows that you have vsyscall issues, add `vsyscall=emulate` to your linux CMDLINE and reboot.  (This happens on some Linux kernels/distros because the downloaded tools were built for an older Linux kernel than the one you are running on, and thus use some older Linux kernel features which are not always included in modern Linux distros, but can be emulated.)

## Loading the gateware onto the Arty A7

Now we can load the prebuilt gateware onto the board.  Use your micro USB to connect your laptop to the board.  The 'send' and 'receive' lights near the board's USB ports should flash yellow when you type.

To load the gateware:

```bash
make gateware-load
```

## Building MicroPython (FμPy)

Next we will build the MicroPython firmware and load it onto the soft CPU:

```bash
./scripts/build-micropython.sh
make firmware-load
```

Once the output pauses in the USB serial console, hit the enter key, you should see a `BIOS>` prompt. To initialise the MicroPython REPL, run:

```
serialboot
```
This will then upload the MicroPython firmware to the soft CPU running on the Arty A7.  Once the upload finishes (several seconds) MicroPython should boot, and then you should reach a MicroPython REPL:

```
sL5DdSMmkekro
[FLTERM] Received firmware download request from the device.
[FLTERM] Uploading kernel (163668 bytes)...
[FLTERM] Upload complete (7.7KB/s).
[FLTERM] Booting the device.
[FLTERM] Done.
Executing booted program at 0x40000000
MicroPython v1.9.4-514-g7e26f0c-dirty on 2018-08-28; litex with lm32
>>> 
```