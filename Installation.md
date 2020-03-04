# LiteX installation guide

1. Install Python 3.5+ and FPGA vendor's development tools and/or [Verilator](http://www.veripool.org/).
2. Install Migen/LiteX and the LiteX's cores:

```sh
$ wget https://raw.githubusercontent.com/enjoy-digital/litex/master/litex_setup.py
$ chmod +x litex_setup.py
$ ./litex_setup.py init install --user (--user to install to user directory)
```
  Later, if you need to update all repositories:
```sh
$ ./litex_setup.py update
```

> **Note:** On MacOS, make sure you have [HomeBrew](https://brew.sh) installed. Then do, ``brew install wget``.

> **Note:** On Windows, it's possible you'll have to set `SHELL` environment variable to `SHELL=cmd.exe`.

3. Install a RISC-V toolchain (Only if you want to test/create a SoC with a CPU):

On Linux (Ubuntu):
```sh
$ wget https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.1.0-2019.01.0-x86_64-linux-ubuntu14.tar.gz
$ tar -xvf riscv64-unknown-elf-gcc-8.1.0-2019.01.0-x86_64-linux-ubuntu14.tar.gz
$ export PATH=$PATH:$PWD/riscv64-unknown-elf-gcc-8.1.0-2019.01.0-x86_64-linux-ubuntu14/bin/
```
On MacOS:
```sh
$ wget https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-apple-darwin.tar.gz
$ tar -xvf riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-apple-darwin.tar.gz
$ export PATH=$PATH:$PWD/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-apple-darwin/bin/
```
On Windows:

You can Get and install the RISC-V toolchain from https://gnutoolchains.com/risc-v/. 

4. Build the target of your board...:

Go to litex-boards/litex_boards/targets and execute the target you want to build.

5. ... and/or install [Verilator](http://www.veripool.org/) and test LiteX directly on your computer without any FPGA board:

On Linux (Ubuntu):
```sh
$ sudo apt install libevent-dev libjson-c-dev verilator
$ lxsim --cpu-type=vexriscv
```

On MacOS:
```sh
$ brew install json-c verilator libevent
$ brew cask install tuntap
$ lxsim --cpu-type=vexriscv
```
6. Run a terminal program on the board's serial port at 115200 8-N-1.

  You should get the BIOS prompt like the one below.

<p align="center"><img src="https://raw.githubusercontent.com/enjoy-digital/litex/master/doc/bios_screenshot.png"></p>