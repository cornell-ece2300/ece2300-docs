
Lab 4 (Part A): TinyRV1 Processor - Implementation and Verification
==========================================================================

Lab 4 will give you experience designing, implementing, testing, and
prototyping two simple processor microarchitectures which both implement
the TinyRV1 instruction set. The lab reinforces several lecture topics
including instruction set architectures, single-cycle processors, and
multi-cycle processors. The lab will continue to provide opportunities to
leverage the three key abstraction principles: modularity, hierarchy, and
regularity.

The lab will include several parts, but currently only Part A is
released.

 - Part A: Processor Datapath Components

Part A is submitted by simply pushing the appropriate code to GitHub.
Part A is due on Thursday, November 7 at 11:59pm.

This handout assumes that you have read and understand the course
tutorials and that you have attended the discussion sections. To get
started, use VS Code to log into an `ecelinux` server, source the setup
script, and clone your individual remote repository from GitHub:

```bash
 % source setup-ece2300.sh
 % mkdir -p ${HOME}/ece2300
 % cd ${HOME}/ece2300
 % git clone git@github.com:cornell-ece2300/groupXX
 % cd ${HOME}/ece2300/groupXX
 % tree
```

where `XX` should be replaced with your group number. You can both pull
and push to your remote repository. If you have already cloned your
remote repository, then use `git pull` to ensure you have any recent
updates before working on your lab assignment.

```bash
 % cd ${HOME}/ece2300/groupXX
 % git pull
 % tree
```

Go ahead and create a build directory in the `lab3-music` directory for
this lab, and run configure to generate a Makefile.

```
% cd ${HOME}/ece2300/groupXX/lab4-proc
% mkdir -p build
% cd build
% ../configure
```

Your repo contains the following files which are part of the automated
build system:

 - `Makefile.in`: Makefile for the build system
 - `configure`: Configure script for the build system
 - `configure.ac`: Used to generate the configure script
 - `scripts`: Scripts used by the build system

The following table shows all of the hardware modules for Part A.

![](img/lab4-hardware-module-table.png)

Before starting, you should copy over all of the listed hardware modules
and the associated test benches for the seven-segment display from Lab 1
and the adders and multiplexors from Lab 2. Make sure all of these
hardware modules pass all of your test cases.

Remember that GL implementations must be implemented using either explicit
gate-level modeling or Boolean equations. For these designs, students are
only allowed to use these Verilog constructs:

 - `wire`, `assign`
 - `not`, `and`, `nand`, `or`, `nor`, `xor`, `xnor`
 - `~`, `&`, `|`, `^`
 - `1'b0`, `1'b1`, `1'd0`, `1'd1`, and other literals
 - `{}` (concatenation operator)
 - `{N{}}` (repeat operator)
 - module instantiation

Hardware modules marked in the table as GL* must _only_ use explicit
gate-level modeling (i.e., you cannot use `~`, `&`, `|`, `^`).

RTL implementations can use all of the GL constructs in addition to the
following Verilog constructs.

 - `logic`
 - `+`, `-`
 - `>>`, `<<`, `>>>`
 - `==`, `!=`, `<`, `>`, `<=`, `>=`
 - `&&`, `||`, `!`
 - `&`, `~&`, `|`, `~|`, `^`, `^~` (reduction operators)
 - `?:` (ternary operator)
 - `always_comb`, `always_ff @(posedge clk)`
 - `if`, `else if`, `endif`
 - `case`, `default`, `endcase`

Note that some hardware modules have more specific restrictions; see the
source comments for more details. Using unallowed Verilog constructs will
result in significant penalties for code functionality and code quality.
If you have any questions on what Verilog constructs can and cannot be
used, please ask an instructor. There are no restrictions on Verilog
constructs in test benches.

**It is critical for students to work together to complete the lab
assignment.** It is unlikely one student can complete the entire lab on
their own. A very productive approach is to have one student work on the
design of a few hardware modules while the other student works on the
test benches for those same hardware modules. Then work together to test
and debug these modules. Then switch roles and move on the next few
modules.

1. Interface and Implementation Specification
--------------------------------------------------------------------------

This section describe the required _interface_ (i.e., the ports for the
module and the module's functional behavior) before describing the
required _implementation_ (i.e., what goes inside the module) for each
hardware module.

### 1.1. Parameterized Multiplexors and Registers

Implement 2-to-1, 4-to-1, and 8-to-1 multiplexors using RTL modeling.
Each multiplexor should be parameterized by the bitwidth of the
corresponding input and output ports.

Implement a multi-bit register which supports reset and enable using RTL
modeling. The register should be parameterized by the bitwidth of the
corresponding input and output ports.

### 1.2. Arithmetic Units

Implement four arithmetic units.

 - Implement a 32-bit adder by instantiating four 8-bit carry select
   adders from Lab 2. The 32-bit adder will be the only module
   implemented at the gate-level in your final processor implementation.

 - Implement a 32-bit equality comparator using RTL modeling.

 - Compose the adder, the equality comparator, and a 2-to-1 multiplexor
   to create a simple ALU. The ALU takes as `op` input port which
   specifies whether the ALU should do an add (`op` is zero) or an
   equality comparison (`op` is one).

 - Implement a 32-bit by 32-bit multiplier using RTL modeling.

### 1.3. Immediate Generation Unit

Implement an immediate generation unit suitable for use in generating
immediates from TinyRV1 instructions. The immediate generation unit
uses the following encoding for the `imm_type` input:

 - `imm_type == 0`: I-type (ADDI)
 - `imm_type == 1`: S-type (SW)
 - `imm_type == 2`: J-type (JAL)
 - `imm_type == 3`: B-type (BNE)

See the TinyRV1 ISA manual for more details.

### 1.4. Register Files

Implement two different register files. Both register files have 32
32-bit registers. For both register files, reading register 0 should
_always_ return the value zero. For both register files, writing and
reading the same register results in reading the old value. The key
difference is one register file provides one read port and one write
port, whiel the other register file provides two read ports and one write
port.

