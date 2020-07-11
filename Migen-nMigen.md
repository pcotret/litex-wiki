# Why is LiteX still Migen based and not already ported to nMigen?

To give a bit of context/history, Migen DSL has been around since 2012 and we have been involved in its development since then while also working on LiteX and creating the ecosystem of cores.

Since 2012, LiteX has been used to create very various FPGA based systems for [open-source and commercial projects](https://github.com/enjoy-digital/litex/wiki/Projects) and most of the codebase is currently tightly linked to Migen since written with its DSL. Most of these projects are still maintained and can't for now justify a complete rewrite.

nMigen has been developed since 2018, so several years after the initial LiteX development, is clearly improving the concepts introduced by Migen and pushing things further and we are clearly willing to use and/or switch to it; but such changes require hard work, time (and/or money :))...

# Why isn't LiteX using nMigen compat mode?

nMigen has a Migen compatibility layer that works great for small/medium size projects, but:
 - This is  not a long term solution.
 - This still requires changes to the codebase.
 - This does not mix well with LiteX relying on some Migen internals for simulation, build, logic analyzer, etc...

At some point, using nMigen compat mode was envisaged but for the foregoing reasons this does not seem practical and a switch to pure nMigen DSL would probably be a better long term solution.

# Is it already possible to use nMigen modules/cores with LiteX?

LiteX is a very permissive SoC builder and the fact that the main modules/cores  are still written with Migen DSL does not prevent integrating nMigen modules in a LiteX SoC by elaborating the nMigen module to verilog and integrate it in the design. A good example for this is the  [Minerva CPU](https://github.com/lambdaconcept/minerva) integration in LiteX, the CPU is elaborated at build-time from nMigen sources with appropriate parameters, exported to verilog and integrated in the LiteX design as any other CPU.

Even if nMigen allows developers to avoid VHDL/Verilog with FOSS toolchains, VHDL/Verilog will probably remain inter-echange formats for quite some time and nMigen cores developers will probably create standalone verilog core generator to facilitate reuse from other flows, similarly to what we are already doing with [LiteDRAM](https://github.com/enjoy-digital/litedram), [LitePCIe](https://github.com/enjoy-digital/litepcie), [LiteEth](https://github.com/enjoy-digital/liteeth), etc...

So reusing nMigen modules with LiteX can already be done in two different ways:
- Reusing a pre-generated verilog provided by developers.
- Elaborating the verilog from the nMigen sources during the LiteX build (as done with Minerva CPU).

The latter is more flexible/powerful since allows elaborating the nMigen module with appropriate parameters.

# Will I be limited by this?

Not really, nMigen has some advantages over Migen and LiteX has features and a rich ecosystem of cores not already available in others projects, so while waiting for the planets to be aligned, the best is probably to take the best of both worlds:

 - Develop in the language you find the most appropriate for you (VHDL/Verilog/nMigen*/Migen, etc....)
 - Reuse other open-source cores when you don't have the time or don't want to develop it yourself. (nMigen core exported to verilog and integrated in a LiteX design, LiteX core exported to verilog and integrated in a nMigen design, nMigen/LiteX core exported to verilog and integrated in a traditional VHDL/Verilog flow, etc....)
 - Use the right tool for the right job.
 - ... and most important: Have fun and understand things are still not perfect and that your help would be welcome to go in that direction!

*If you are new to FPGA and want to start learning a Python based language, starting learning nMigen is probably the best and will not really limit you if you want to also use LiteX.

# What are the long term plans regarding LiteX/nMigen?

Things are still not fully decided, if:
 - current LiteX codebase should be converted to nMigen.
 - current LiteX codebase should stay in Migen and new modules/cores developed with nMigen.
 - if a different project should be created with a nMigen codebase.

In the case of a full nMigen switch, only the DSL/simulators would probably be reused and LiteX would still handle the others parts (platforms/targets/build backends, cores, etc...).
 
...But for now, there **are still lots of things to do**, in the short term Migen is not really limiting the project and a full nMigen switch not really yet justified, so the project is still going to evolves slowly but surely and **if developers want to help us improving things** on the different fronts (including the nMigen one) we'll be **happy to collaborate**!