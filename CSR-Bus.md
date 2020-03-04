LiteX CSRs: A Developer's Overview
----------------------------------

1. LiteX CSR definition and terminology

LiteX CSRs are MMIO Configuration/Status Registers used to interact with
and control various hardware peripheral devices included on a LiteX SoC.
NOTE: LiteX CSRs are *NOT* to be confused with the concept of CSRs as
defined within the RISC-V ISA! To further complicate things, it is
entirely possible for the LiteX SoC to be configured with a RISC-V soft
CPU, in which case the term "CSR" will be used in a RISC-V specific way
when discussing the CPU's behavior and internals, and in a LiteX specific
way (see below) when discussing "hardware peripheral MMIO registers" that
belong to the SoC, and are accessed by the CPU at the direction of various
pieces of system software (e.g., the Bios, bootloader, or OS kernel).

2. LiteX CSR Addressing

2.1. CSR Address Space: Base Address and `csr_address_width`

LiteX CSRs are placed in an MMIO segment starting at a Base Address
(`mem_map["csr"]` in `litex/soc/integration/soc_core.py:SoCCore:__init__()`).
The default Base Address is 0x82000000, but can (and often is) overridden
by other components (usually CPUs) as they are added to the SoC setup.

The size of the CSR segment, and also the number of bits dedicated to
accessing LiteX CSRs is dictated by `csr_address_width` (search for
`self.register_mem("csr", ...)` in `litex/soc/integration/soc_core.py`). 
The `register_mem` function will also ensure that our CSR segment does
not overlap with any other reserved segment in the overall `mem_map` of
the SoC.

E.g., assuming an address space of 32 bits, if the LiteX CSR Base Address
occupies the 8 MSBits, that leaves the remaining 24 bits for addressing
actual MMIO registers within the CSR segment. However, assuming that the
smallest addressable individual memory location is 32-bit wide, that also
removes the 2 LSBits (ignored by the Wishbone interface, as per thedefault
`adr_width=30` in `litex/soc/interconnect/wishbone.py:Interface:__init__()`),
leaving the actual number of bits for addressing individual CSR elements
(`csr_address_width`) to be max. 22.

**FIXME**: should Wishbone ignore 3 LSBs when `data_width == 64`?

CSR segment alignment (i.e., that the Base Address is aligned to allow for at
least `csr_address_width` variable bits to address individual (sub)registers)
is enforced in `litex/soc/integration/common.py:mem_decoder()`, which is
referenced in `SoCCore:add_wb_slave()`, used to connect the entire CSR
subsystem to the Wishbone bus.

**FIXME**: when the main RAM is connected to the CPU directly (via AXI), bypassing 
the Wishbone bus, alignment is NOT enforced (this FIXME does not have anything
to do with CSRs per se, but wanted to make sure I remember)

2.2. CSR Bank ID

All CSRs that belong to the same hardware device are grouped together in
a "bank", and given a common ID or "map address". 

Bank IDs are allocated in `SoCCore:add_csr()` where one (weak) sanity
check ensures an ID can't exceed `2^csr_address_width-1` (which would
reflect the extreme/degenerate case in which each bank contains exactly
one addressable element). CSR Banks are allocated the next available ID
in a natural sequence starting at 0.

Bank IDs (as variable name `mapaddr`) are then multiplied by 0x800 (i.e.,
left-shifted by 11 bits) and added to the CSR Base Address to determine
the physical bank address in the CPU's physical address space. This means
that, effectively, we are allowed bits [23..11], or 13 bits worth of bank
IDs (remember, bits [31..24] are reserved for the overall CSR segment's
Base Address).

**FIXME**: The left-shift amount (and basically the number of bits of
`csr_address_width` allocated to the Bank ID) should be derived and/or
asserted programmatically, from named parameters!

A CSR Bank is selected (see `litex/soc/interconnect/csr_bus.py:CSRBank()`)
if its Bank ID (as parameter `address`) matches the CSR Bus address
MSBits, starting with bit 9 and up (See the Figure below for a graphic
example of how CSR Bus address bits are matched up against the address
bits of the CPU's MMIO Bus).
NOTE: Some CSRs are implemented as simple SRAM memories, without being
backed by actual hardware. In such cases, the "bank" is selected in a
similar way in `litex/soc/interconnect/csr_bus.py:SRAM()`

```
Alignment (CPU XLen): 32
CSR Base Address = 0xe000_0000
csr_address_width = 14
UART Bank ID = 0x04
uart.ev_enable intra-bank "subreg." offset: 5
uart.ev_enable MMIO addr: 0xe000_2014:

<-------------------------- CPU MMIO Address ------------------------->
<- CSR Base -->
----------- --- --- --- ----------- ----------- ----------- -----------
31 30 29 28 27- 23- 19- 15 14 13 12 11 10 09 08 07 06 05 04 03 02 01 00
----------- --- --- --- ----------- ----------- ----------- -----------
 1  1  1  0  00  00  00  0  0  1  0  0  0  0  0  0  0  0  1  0  1  0  0
----------- --- --- --- ----------- ----------- ----------- -----------
                <⋅⋅⋅⋅⋅⋅ <--- Bank ---> <------- CSRsubreg ------>
                ⋅⋅⋅ ⋅⋅⋅ == =========== -- ----------- -----------
                21- 17- 13 12 11 10 09 08 07 06 05 04 03 02 01 00
                ⋅⋅⋅ ⋅⋅⋅ ----- ----------- ----------- -----------
                         0  0  1  0  0  0  0  0  0  0  0  1  0  1
                ⋅⋅⋅ ⋅⋅⋅ ----- ----------- ----------- -----------
                <-------------- CSR Bus Address ---------------->
```

2.3. Intra-Bank CSR (sub)register selection and `csr_data_width`

Within a CSR Bank, registers are stored in "slices" or "subregisters" of
up to `csr_data_width` bits. Supported values for `csr_data_width` are
8, 16, 32, and 64.

**FIXME**: 64 doesn't work ATM!

**FIXME**: must WB `soc_bus` also be 64bit for `csr-data-width=64` to work?

**FIXME**: setting `soc_bus = wishbone.Interface(data_width=64)` in
       `SoCCore:__init__()` isn't working right now, so maybe just disallow
       `csr_data_width > 32` for the time being, and resolve to fix this in
       the future?

A register with more than `csr_data_wdth` bits is spread across multiple
adjacent "subregisters", with subregisters holding more significant bits
stored at lower addresses, in a way similar to the Big Endian model.

Within a subregister, bits are stored in CPU-native endianness. This is
a consequence of how data wires are interpreted _inside_ the CPU, and
_not_ a feature of the LiteX SoC gateware itself!

Regardless of whether they fit in a single subregister or require being
spread across multiple subregisters, all registers in a CSR Bank will be
laid out as a sequence of adjacent subregisters, and the number of bits
required to enumerate all subregisters will be counted as `decode_bits`
(see `litex/soc/interconnect/csr.py:GenericBank()`).

**FIXME**: we should probably assert that `decode_bits` plus the number
of bits dedicated to representing the Bank ID should be less than or equal
to `csr_address_width` minus any LSBs ignored due to `alignment > 32`!

Once a bank is matched, see the latter part of the `__init__()` method
in `litex/soc/interconnect/csr_bus.py:CSRBank()` for how an individual
subregister is selected based on the lesser bits of the provided CSR Bus
address.

3. CSR Software Programmer's API

**FIXME**: write this section once the above FIXMEs have been resolved. Mainly,
it is important to know how the intra-subregister endianness works, before
we can either document this behavior authoritatively, or, further use this
information to write a universal OS-kernel bus driver for the LiteX CSR Bus!