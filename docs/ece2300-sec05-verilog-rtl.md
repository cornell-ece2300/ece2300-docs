
Section 5: Verilog Combinational RTL Design
==========================================================================

In the past discussion sections, we have learned how to model hardware at
the gate-level (GL). In this discussion section, you will learn a new
approach to modeling hardware using Verilog at the _register-transfer
level_ (RTL). By the end of this discussion section, you will have
explore both a GL and RTL implementation of an absolute difference unit.
RTL modules are usually implemented at a much higher level of abstraction
compared to GL models. This improves designer productivity and gives the
FPGA tools more flexibility in optimizing the final hardware, but it also
means the designer might not fully understand the final hardware and has
less control over the details of the final hardware. This is a key
trade-off we will need to balance carefully when using RTL modeling. It
is also possible to write RTL models that cannot map to any kind of
reasonable hardware. **It is critical that students _always_ know what
is the hardware they are modeling whether they are working at the gate-
or register-transfer level.**

1. Logging Into `ecelinux` with VS Code
--------------------------------------------------------------------------

Follow the same process as previous discussion sections. Find a free
workstation and log into the workstation using your NetID and standard
NetID password. Then complete the following steps (described in more
detail in the last discussion section):

 - Start VS Code
 - Install the Remote-SSH, Verilog, and Surfer extensions
 - Use _View > Command Palette_ to execute _Remote-SSH: Connect Current
    Window to Host..._
 - Enter `netid@ecelinux.ece.cornell.edu`
 - Install the Verilog and Surfer extensions on the server
 - Use _View > Explorer_ to open your home directory on `ecelinux`
 - Use _View > Terminal_ to open a terminal on `ecelinux`

There is no need to fork the repo for today's discussion section. Simple
clone the repo as follows.

```bash
% source setup-ece2300.sh
% mkdir -p ${HOME}/ece2300
% cd ${HOME}/ece2300
% git clone git@github.com:cornell-ece2300/ece2300-sec05-verilog-rtl sec05
% cd sec05
% tree
```

The repo includes the following files:

 - `Makefile.in`: Makefile for the build system
 - `configure`: Configure script for the build system
 - `configure.ac`: Used to generate the configure script
 - `scripts`: Scripts used by the build system
 - `hw/FullSubtractor_GL.v`: One-bit full subtractor at gate level
 - `hw/SubtractorRippleCarry_4b_GL.v`: Four-bit ripple-carry subtractor at gate level
 - `hw/GTComparator_1b_GL.v`: One-bit greater-than comparator at gate level
 - `hw/GTComparator_4b_GL.v`: Four-bit greater-than comparator at gate level
 - `hw/Mux2_1b_GL.v`: One-bit two-to-one multiplexor at gate level
 - `hw/Mux2_4b_GL.v`: Four-bit two-to-one multiplexor at gate level
 - `hw/AbsDiff_4b_GL.v`: Absolute difference unit at gate level
 - `hw/Subtractor_4b_RTL.v`: Four-bit subtractor at register-transfer level
 - `hw/GTComparator_4b_RTL.v`: Four-bit greater-than comparator at register-transfer level
 - `hw/Mux2_4b_RTL.v`: Four-bit two-to-one multiplexor at register-transfer level
 - `hw/AbsDiff_4b_RTL.v`: Absolute difference unit at register-transfer level
 - `test`: Directory with unit tests for each hardware module
 - `sim/mux-rtl-sim.v`: interactive simulator for experimenting with mux RTL modeling

Go ahead and create a build directory and run configure to generate a
Makefile.

```bash
% cd ${HOME}/ece2300/sec05
% mkdir -p build
% cd build
% ../configure
```

To make it easier to cut-and-paste commands from this handout onto the
command line, you can tell Bash to ignore the `%` character using the
following command:

```bash
% alias %=""
```

Now you can cut-and-paste a sequence of commands from this tutorial
document and Bash will not get confused by the `%` character which begins
each line.

2. Absolute Difference Unit Interface
--------------------------------------------------------------------------

We will be implementing an absolute difference unit using both GL and RTL
modeling. Both implementations will have similar interfaces. The GL
interface is as follows:

```verilog
module AbsDiff_4b_GL
(
  (* keep=1 *) input  wire [3:0] in0,
  (* keep=1 *) input  wire [3:0] in1,
  (* keep=1 *) output wire [3:0] diff
);
```

The RTL interface as the same ports and bitwidths, except instead of
declaring these ports as `wire` we declare them with `logic`. In RTL
modeling, we will always use `logic` for both ports and internal signals.

```verilog
module AbsDiff_4b_RTL
(
  (* keep=1 *) input  logic [3:0] in0,
  (* keep=1 *) input  logic [3:0] in1,
  (* keep=1 *) output logic [3:0] diff
);
```

The absolute difference unit takes as input two four-bit unsigned binary
numbers and outputs the corresponding absolute difference on the `diff`
output port.

2. Gate-Level Implementation of Absolute Difference Unit
--------------------------------------------------------------------------

We have provided you a gate-level implementation of an absolute
difference unit in these files:

 - `hw/FullSubtractor_GL.v`
 - `hw/SubtractorRippleCarry_4b_GL.v`
 - `hw/GTComparator_1b_GL.v`
 - `hw/GTComparator_4b_GL.v`
 - `hw/Mux2_1b_GL.v`
 - `hw/Mux2_4b_GL.v`
 - `hw/AbsDiff_4b_GL.v`

!!! question "Activity 1: Draw Block Diagram of Gate-Level Absolute Difference Unit"

    Inspect the above files and draw a detailed block diagram of how all
    of the blocks are instantiated and connected. You can use either a
    bottom-up approach (i.e., start from the logic gates within the
    `FullSubtractor_GL`, `GTComparator_1b_GL`, and `Mux2_1b_GL` modules
    and work up to the `AbsDiff_4b_GL` module) or a top-down approach
    (i.e., start from the `AbsDiff_4b_GL` module and work down to the
    logic gates in the `FullSubtractor_GL`, `GTComparator_1b_GL`, and
    `Mux2_1b_GL` modules). You must be able to point to the actual AND,
    OR, NOT, etc gates in your diagram.

3. RTL Implementation of Building Blocks
--------------------------------------------------------------------------

In this section, we will experiment with RTL implementations of the key
building blocks that make-up the absolute difference detector: a four-bit
greater-than comparator, a four-bit subtractor, and a four-bit two-to-one
multiplexor.

### 3.1. RTL Implementation of Four-Bit Subtractor

In Verilog GL modeling, we must explicitly instantiate gates. When
writing Verilog Boolean equations, we can use the `assign` statement
along with a limited number of operators (i.e., `~`, `&`, `|`, `^`).
The most basic form of RTL modeling simply enables designers to use more
complex operators in an `assign` statement. For example, we can implement
a greater-than comparator by simply using the `-` operator.

Open the `Subtractor_4b_RTL.v` file and implement the subtractor using
RTL modeling like this:

```verilog
  assign {bout,diff} = in0 - in1 - {3'b0,bin};
```

Notice how this implementation correctly hands both the borrow in and the
borrow out. Run all of the provide tests to verify your implementation.

```bash
% cd ${HOME}/ece2300/sec05/build
% make Subtractor_4b_RTL-test
% Subtractor_4b_RTL-test +test-case=-1
```

Notice how simple the RTL implementation is compared to the GL
implementation. An RTL implementation enables the designer to express the
subtractor at a very high level, and then trust that the FPGA tools will
be able to choose an appropriate subtractor hardware implementation. Of
course the downside is the designer has less control over what specific
subtractor hardware implementation is actually used on the FPGA.

In addition to more complex operators in `assign` statements, RTL
modeling also enables designers to use an `always_comb` block to express
operations. For example, the following `always_comb` block is equivalent
to the above `assign` statement.

```verilog
  always_comb begin
    {bout,diff} = in0 - in1 - {3'b0,bin};
  end
```

Any operations that are valid for use with an `assign` statement are also
valid within an `always)comb` block. In this course, we _require_
students to use `always_comb` when modeling combinational logic. Never
use the `always` keyword on its own to model combinational logic.

The real power of an `always_comb` block is that it enables expressing a
_sequence_ of operations that are executed _sequentially_ (unlike
`assign` statements which execute in parallel). The following example
illustrates modeling a subtractor with a sequence of four operations in
an `always_comb` block.

```verilog
  logic [4:0] temp;
  always_comb begin
    temp = in0 - in1;
    temp = temp - {3'b0,bin};
    bout = temp[4];
    diff = temp[3:0];
  end
```

We first subtract `in1` from `in0` and store the result in a temporary
signal. We then subtract the borrow input and store the result back in
the temporary signal. Then we can extract the most-significant bit as the
borrow output and the remaining bits as the difference.

Open the `Subtractor_4b_RTL.v` file and change your implementation of the
subtractor to use `always_comb` block. Run all of the provide tests to
verify your implementation.

```bash
% cd ${HOME}/ece2300/sec05/build
% make Subtractor_4b_RTL-test
% Subtractor_4b_RTL-test +test-case=-1
```

### 3.2. RTL Implementation of Four-Bit Greater-Than Comparator

An RTL model of the greater-than comparator can use the RTL `>` operator.
Open the `GTComporator_4b_RTL.v` file and change your implementation of
the subtractor to use the `>` operator in either an `assign` statement or
`always_comb` block. Run all of the provided tests to verify your
implementation.

```bash
% cd ${HOME}/ece2300/sec05/build
% make GTSubtractor_4b_RTL-test
% GTSubtractor_4b_RTL-test +test-case=-1
```

### 3.3. RTL Implementation of Four-Bit Two-to-One Multiplexor

An RTL model of a multiplexors can use the ternary operator. Open the
`Mux2_4b_RTL.v` file and implement the subtractor using RTL modeling like
this:

```verilog
  assign out = ( sel ) ? in1 : in0;
```

If the expression in the parentheses evaluates to one, then we assign
`in1` to the output. If the expression in the parentheses evaluates to
zero, then we assign `in0` to the output.

Run all of the provide tests to verify your implementation.

```bash
% cd ${HOME}/ece2300/sec05/build
% make Mux2_4b_RTL-test
% Mux2_4b_RTL-test +test-case=-1
```

You can use ternary operators in an `always_comb` block, but an
`always_comb` block also enables using more traditional `if`/`else`
conditional operations.

Open the `Mux2_4b_RTL.v` file and implement the subtractor using RTL
modeling like this:

```verilog
  always_comb begin
    if ( sel == 0 )
      out = in0;
    else
      out = in1;
  end
```

Run all of the provide tests to verify your implementation.

```bash
% cd ${HOME}/ece2300/sec05/build
% make Mux2_4b_RTL-test
% Mux2_4b_RTL-test +test-case=-1
```

### 3.3. Subtle Issues with Using If/Else in RTL Modeling

There are two critically important yet subtle issues that arise when
using `if`/`else` conditional operations in `always_comb` blocks. We have
provided you a simple interactive simulator that will enable us to
explore these two issues. You can build and run the interactive simulator
like this:

```bash
% cd ${HOME}/ece2300/sec05/build
% make mux-rtl-sim
% mux-rtl-sim +in0=0101 +in1=1111 +sel=0
% mux-rtl-sim +in0=0101 +in1=1111 +sel=1
```

Let's assume we accidentally forget to include the `else` clause in our
`always_comb` block. Go ahead and update your implementation to look like
this:

```verilog
  always_comb begin
    if ( sel == 0 )
      out = in0;
  end
```

Then rebuild the interactive simulator.

```bash
% cd ${HOME}/ece2300/sec05/build
% make mux-rtl-sim
```

`verilator` will complain that `in1` is not used, but more importantly
notice how `verilator` is warning about an _inferred latch_. The above
RTL mode actually does _not_ model _combinational_ logic. It models
_sequential_ logic. The signal `out` is no longer a function of just the
inputs `sel` and `in0`. If `sel` is first zero and is then changed to
one, the hardware must _remember_ what was the previous value of `out` so
that it stays the same even if `in0` changes. Inferred latches can cause
all kinds of subtle bugs which is why we are using an `always_comb` block
so `verilator` can detect inferred latches. **To avoid inferred latches
ensure that any signal written in an `always_comb` block is always
written no matter what path we take through `if`/`else` conditional
operators.**

Go ahead and update your implementation as it was before:

```verilog
  always_comb begin
    if ( sel == 0 )
      out = in0;
    else
      out = in1;
  end
```

Now let's experiment with what happens when one of our inputs is
undefined (i.e., is modeled using an `X` in Verilog). Try these inputs.

```bash
% cd ${HOME}/ece2300/sec05/build
% make mux-rtl-sim
% mux-rtl-sim +in0=xxxx +in1=1111 +sel=0
% mux-rtl-sim +in0=xxxx +in1=1111 +sel=1
% mux-rtl-sim +in0=0101 +in1=xxxx +sel=0
% mux-rtl-sim +in0=0101 +in1=xxxx +sel=1
% mux-rtl-sim +in0=0101 +in1=1111 +sel=x
```

Are the results as expected? Prof. Batten will talk more about `X`
propagation and how to ensure `X` values are correctly propagated through
your RTL models.

4. RTL Implementation of Absolute Difference Unit
--------------------------------------------------------------------------

Now that we have RTL models of our building blocks, we can create an RTL
model for the entire absolute difference unit.

Let's start with a _structural_ RTL model. Open `AbsDiff_4b_GL.v` and
copy the structural implementation into `AbsDiff_4b_RTL.v`. Change the
building blocks to all be the RTL implementation. Then rerun all the
tests to verify that your structural RTL modeling is functionally
correct.

```bash
% cd ${HOME}/ece2300/sec05/build
% make AbsDiff_4b_RTL-test
% AbsDiff_4b_RTL-test +test-case=-1
```

Notice how structural RTL modeling like this provides a nice balance
between productive modeling and control over the hardware implementation.
We can let the FPGA tools optimize each building block individually, but
we still get explicit control over how these building blocks are
composed.

We can also use a _flat_ RTL model which does not instantiate any
building blocks at all, but instead simply uses a single `always_comb`
block to model the entire absolute difference unit. Open
`AbsDiff_4b_RTL.v` and change the RTL implementation to be as follows:

```verilog
  always_comb begin
    if ( in0 > in1 )
      diff = in0 - in1;
    else
      diff = in1 - in0;
  end
```

Then rerun all the tests to verify that your structural RTL modeling is
functionally correct.

```bash
% cd ${HOME}/ece2300/sec05/build
% make AbsDiff_4b_RTL-test
% AbsDiff_4b_RTL-test +test-case=-1
```

Notice how flat RTL modeling enables the designer to express their intent
at a very high level, but the designer now has very little control over
the final hardware implementation. Not only will the FPGA tools optimize
the building blocks, but hopefully the FPGA tools will realize an
implementation only needs a single subtractor. When using flat RTL
modeling we have lost much of the intuition about the hardware
implementation that was so explicit in our GL modeling and still somewhat
present in our structural RTL modeling. **Students will need to carefully
navigate the tension between productivity and control when developing
gate-level models, boolean equation models, structural RTL models, and
flat RTL models.**

5. Clean Build
--------------------------------------------------------------------------

As a final step, do a clean build to verify everything is working
correctly.

```bash
% cd ${HOME}/ece2300/sec05
% trash build
% mkdir -p build
% cd build
% ../configure
% make check
```

