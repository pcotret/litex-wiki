LiteX can create SoCs with or without CPU. Some simple SoCs don’t use any CPU (bridging SoCs for example), some SoCs use a CPU but external to the FPGA (PCIe SoCs for example where the CPU is directly the CPU of the Host machine) but in most of the cases the SoC embedded a "Soft CPU" to control the system and/or ease splitting tasks between software/hardware.

# [Hard CPUs page](https://github.com/enjoy-digital/litex/wiki/Hard-core-CPUs)

# Summary of Soft CPUs

**FIXME**: This list is currently out of date missing the HW Group CPU, MicroWatt and BlackParrot.

Currently the supported Soft CPUs are:

 * [`lm32`](https://github.com/enjoy-digital/litex/tree/master/litex/soc/cores/cpu/lm32) -- a [LatticeMico32](https://en.wikipedia.org/wiki/LatticeMico32) soft core.

*   [`or1k`](https://github.com/enjoy-digital/litex/tree/master/litex/soc/cores/cpu/mor1kx) -- an [OpenRISC 1000](https://openrisc.io/or1k.html) soft core (see also [Open RISC on Wikipedia](https://en.wikipedia.org/wiki/OpenRISC)).

*   [`picorv32`](https://github.com/enjoy-digital/litex/tree/master/litex/soc/cores/cpu/picorv32) -- a [Small RISC V core by Clifford Wolf](https://github.com/cliffordwolf/picorv32), implementing the `rv32imc` instruction set (or configured subsets)

*   [`vexriscv`](https://github.com/enjoy-digital/litex/tree/master/litex/soc/cores/cpu/vexriscv) -- an [FPGA Friendly RISC V core by SpinalHDL](https://github.com/SpinalHDL/VexRiscv), implementing the `rv32im` instruction set (hardware multiply optional)

*   [`minerva`](https://github.com/enjoy-digital/litex/tree/master/litex/soc/cores/cpu/minerva) -- a CPU core that currently implements the RISC-V RV32I instruction set with its microarchitecture described in plain Python code using the [nMigen toolbox](https://github.com/m-labs/nmigen).

*   [`rocket`](https://github.com/enjoy-digital/litex/tree/master/litex/soc/cores/cpu/rocket) -- [Rocket Chip](https://github.com/chipsalliance/rocket-chip), a configurable, fully featured, 64-bit `rv64imafdc` capable core.

*   [`blackparrot`](https://github.com/enjoy-digital/litex/tree/master/litex/soc/cores/cpu/blackparrot) -- [BlackParrot](https://github.com/black-parrot/black-parrot), a 64-bit Linux-Capable accelerator host multicore that implements `rv64imafdc` instruction set.

*   [`neorv32`](https://github.com/enjoy-digital/litex/tree/master/litex/soc/cores/cpu/neorv32) -- [NEORV32](https://github.com/stnolting/neorv32), a small, platform-agnostic and highly extendable 32-bit RTOS-capable RISC-V core supporting the `rv32imcbu` instruction set.

# Soft CPU Variants

Most of these CPUs have multiple configuration variants which customize the configuration to target a specific type of firmware, performance and resource usage. All these CPUs can be used with your own bare metal firmware.

**FIXME**: This list is currently out of date - missing the debug + multicore variants.

## `minimal`

Aliases: `min`

Minimal is the smallest possible working configuration for a given CPU type. These features frequently disables a large number of useful such as illegal instruction exceptions and similar. It should only be used if the absolute smallest configuration is needed.

### Supported CPUs

 * lm32
 * picorv32
 * vexriscv
 * neorv32

## `lite`

**Aliases**: `zephyr`, `nuttx`, `light`

Lite is the configuration which should work okay for bare metal firmware and RTOS like NuttX or Zephyr on small big FPGAs like the Lattice iCE40 parts. It can also be used for designs which are more resource constrained.

### Recommended FPGAs

 * Lattice iCE40 Series - iCE40HX, iCE40LP, iCE40UP5K
 * Any resource constrained design.

### Supported CPUs

 * lm32
 * vexriscv
 * neorv32

## `standard`

**Aliases**: `std`

Standard is the default configuration which should work well for bare metal firmware and RTOS like NuttX or Zephyr on modern big FPGAs.

### Supported CPUs

 * lm32
 * minerva
 * picorv32
 * or1k
 * vexriscv
 * rocket
 * blackparrot
 * neorv32

### Recommended FPGAs

 * Xilinx 7-Series - Artix7, Kintex7, Spartan7
 * Xilinx Spartan6
 * Lattice ECP5

## `full`

This target enables **all** features of each CPU.

### Supported CPUs

 * (TODO) - lm32
 * (TODO) - minerva
 * (TODO) - picorv32
 * (TODO) - or1k
 * vexriscv
 * rocket
 * neorv32


## `linux`

This target enables CPU features such as MMU that are required to get Linux booting.

### Supported CPUs

 * or1k
 * vexriscv
 * rocket
 * blackparrot

---

# Soft CPU Extensions

Extensions are added to the CPU variant with a `+`. For example a `minimal` variant with the `debug` extension would be `minimal+debug`.

## `debug`

The debug extension enables extra features useful for debugging. This normally includes things like JTAG port.

### Supported CPUs

 * vexriscv
 * (TODO) - neorv32

## TODO - `mmu`

The `mmu` extension enables a memory protection unit.

### Supported CPUs

 * lm32 (untested)
 * vexriscv
 * or1k

## TODO - `hmul`

The `hmul` extension enables hardware multiplication acceleration.

## TODO - `fpu`

The `fpu` extension enables a floating point acceleration unit.

### Supported CPUs

 * or1k


---

# Binutils + Compiler

 * lm32 support was added to upstream GCC around ~2009, no clang support.
 * or1k support was added to upstream GCC in version 9.0.0, clang support was added upstream in version XXX
 * riscv support (VexRISCV, PicoRV32, Minerva, and Rocket) was added to upstream GCC in version 7.1.0, clang support was added upstream in version 3.1

You can compile your own compiler, download a precompiled toolchain or use an environment like [TimVideos LiteX BuildEnv](https://github.com/timvideos/litex-buildenv/) which provides precompiled toolchain for all three architectures.

Note: RISC-V toolchains support or require various extensions. Generally `rv32i` is used on smaller FPGAs, and `rv32im` on larger FPGAs -- the `rv32im` adds hardware multiplication and division (see [RISC V ISA base and extensions on Wikipedia](https://en.wikipedia.org/wiki/RISC-V#ISA_base_and_extensions) for more detail).

---

# SoftCPU options

## lm32 - [LatticeMico32](https://github.com/m-labs/lm32)

[LatticeMico32](https://en.wikipedia.org/wiki/LatticeMico32) soft core, small and designed for an FPGA.

### CPU Variants

 * minimal
 * lite
 * standard

### Tooling support

 * Upstream GCC
 * Upstream Binutils

### OS Support

 * No upstream Linux, very old Linux port
 * Upstream NuttX
 * No Zephyr support

### Community

 * No current new activity

---

## [mor1k - OpenRISC](https://github.com/openrisc/mor1kx)

An [OpenRISC 1000](https://openrisc.io/or1k.html) soft core (see also [Open RISC on Wikipedia](https://en.wikipedia.org/wiki/OpenRISC)).

### CPU Variants

 * standard
 * linux

### Tooling support

 * Upstream GCC
 * Upstream Binutils
 * Upstream clang

### OS support

 * No Zephyr support
 * No NuttX support
 * Upstream Linux

### Community

 * Reasonable amount of activity.

---

## RISC-V - [VexRiscv](https://github.com/SpinalHDL/VexRiscv)

A [FPGA Friendly RISC V core by SpinalHDL](https://github.com/SpinalHDL/VexRiscv), implementing the `rv32im` instruction set (hardware multiply optional).

### CPU Variants

 * minimal
 * minimal_debug
 * lite
 * lite_debug
 * standard
 * standard_debug
 * linux

### Tooling support

 * Upstream GCC
 * Upstream Binutils
 * Upstream clang

### OS support

 * Upstream Zephyr
 * Unknown NuttX support
 * Upstream Linux (in progress)

### Community

 * Lots of current activity
 * Currently supported under both LiteX & MiSoC

## RISC-V - [picorv32](https://github.com/cliffordwolf/picorv32/)

A [small RISC V core by Clifford Wolf](https://github.com/cliffordwolf/picorv32), implementing the `rv32imc` instruction set (or configured subsets).

### CPU Variants

 * minimal
 * standard

### Tooling support

 * Upstream GCC
 * Upstream Binutils
 * Upstream clang

### OS support

 * Out of tree Zephyr
 * Unknown NuttX support
 * Too small for Linux

### Community

 * Some activity

## RISC-V - [`minerva`](https://github.com/enjoy-digital/litex/tree/master/litex/soc/cores/cpu/minerva)

The Minerva is a CPU core that currently implements the RISC-V RV32I instruction set with its microarchitecture described in plain Python code using the [nMigen toolbox](https://github.com/m-labs/nmigen).

### CPU Variants

 * standard

### Tooling support

 * Upstream GCC
 * Upstream Binutils
 * Upstream clang

### OS support

 * Unknown Zephyr support
 * Unknown NuttX support
 * Unknown Linux support

### Community

 * Some activity

## RISC-V - [`rocket`](https://github.com/enjoy-digital/litex/tree/master/litex/soc/cores/cpu/rocket)

The Rocket Chip is a full-featured, configurable CPU core that implements up to the full RISC-V RV64IMAFDC (a.k.a. RV64GC) instruction set, with its microarchitecture described in [Chisel](https://github.com/freechipsproject/chisel3).

### CPU Variants

 * standard (`rv64imac` without MMU support)
 * linux (`rv64imac` with enabled hardware MMU)
 * full (`rv64imafdc` with enabled hardware MMU and FPU)

### Tooling support

 * Upstream GCC
 * Upstream Binutils
 * Upstream clang

### OS support

 * Full upstream Linux support (for models with MMU enabled)

### Community

 * [Lots of activity](https://github.com/chipsalliance/rocket-chip)
 * Reference design for several taped-out ASICs (e.g., from [SiFive](https://www.sifive.com/risc-v-core-ip))

## RISC-V - [`neorv32`](https://github.com/enjoy-digital/litex/tree/master/litex/soc/cores/cpu/neorv32)

The [NEORV32 RISC-V Processor](https://github.com/stnolting/neorv32) is a tiny, customizable and highly extensible MCU-class 32-bit RISC-V soft-core CPU and microcontroller-like SoC written in platform-independent VHDL.

### CPU Variants

 * minimal (`rv32i_Zicsr_Zifencei`)
 * lite (`rv32imc_Zicsr_Zifencei`)
 * standard (`rv32imc_Zicsr_Zifencei_Zicntr` + i-cache, fast MUL (DSPs) and barrel-shifter)
 * full (`rv32imcu_Zicsr_Zifencei_Zicntr_Zihpm` + i-cache + physical memory protection, fast MUL (DSPs) and barrel-shifter)

### Tooling support

 * Upstream GCC
 * Upstream Binutils
 * Upstream openOCD and GDB

### OS support

 * Upstream [Zephyr](https://docs.zephyrproject.org/latest/boards/riscv/neorv32/doc/index.html) support
 * FreeRTOS [port](https://github.com/stnolting/neorv32/tree/main/sw/example/demo_freeRTOS)

### Community

 * [gitter channel](https://gitter.im/neorv32/community)
 * community-driven [example setups and projects](https://github.com/stnolting/neorv32-setups) for various FPGAs, boards and toolchains