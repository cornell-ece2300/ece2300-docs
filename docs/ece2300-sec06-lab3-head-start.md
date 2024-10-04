
Section 6: Lab 3 Head Start
==========================================================================

In this discussion section you will use what you have learned in the
previous discussion sections to get started on Lab 3. You will start by
implementing a D latch, D flip-flop, D flip-flop with reset and enable,
and a 8-bit register all at the gate-level. You will then implement a
parameterized multi-bit register and simple counter at the
register-transfer level. Feel free to copy any code you would like from
your work in this discussion section into your lab group repo.

In the past discussion sections, we have focused on how to use Verilog to
model combinational logic at both the gate-level and
register-transfer-level (RTL). In this discussion section, we will learn
how to use Verilog to model sequential logic. Gate-level modeling of
sequential logic largely uses similar syntax and semantics to modeling
combinational logic at the gate level, but RTL modeling of sequential
logic will require new syntax and semantics which we will need to
understand and use carefully. **Remember from last discussion section
that it is critical that students _always_ know what is the hardware they
are modeling whether they are working at the gate- or register-transfer
level.**

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
% git clone git@github.com:cornell-ece2300/ece2300-sec06-lab3-head-start sec06
% cd sec06
% tree
```

The repo includes the following files:

 - `Makefile.in`: Makefile for the build system
 - `configure`: Configure script for the build system
 - `configure.ac`: Used to generate the configure script
 - `scripts`: Scripts used by the build system
 - `Adder_8b_RTL.v`: 8-bit RTL adder (copy from lab 2)
 - `Mux2_1b_GL.v`: 8-bit RTL adder (copy from lab 2)
 - `DLatch_GL.v`: 1-bit D latch at gate-level
 - `DFF_GL.v`: 1-bit D flip-flop at gate-level
 - `DFFRE_GL.v`: 1-bit D flip-flop with reset and enable at gate-level
 - `Register_8b_GL.v`: 8-bit register with reset and enable at gate-level
 - `Register_RTL.v`: Parameterized register with reset and enable at RTL
 - `SimpleCounter_8b_RTL.v`: Simple counter that counts up in structural RTL
 - `test`: Directory with unit tests for each hardware module

Go ahead and create a build directory and run configure to generate a
Makefile.

```bash
% cd ${HOME}/ece2300/sec06
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

2. Mux and Adder from Lab 2
--------------------------------------------------------------------------

Copy your 1-bit 2-to-1 multiplexor and your 8-bit RTL adder from Lab 2
into the `hw` directory for this discussion section. Verify that these
hardware modules are working.

```bah
% cd ${HOME}/ece2300/sec06/build
% make Mux2_1b_GL-test
% make Adder_8b_RTL-test
% ./Mux2_1b_GL-test
% ./Adder_8b_RTL-test
```

3. Implementing and Testing D Latch, D Flip-Flops, and Register
--------------------------------------------------------------------------

We will start by exploring how to implement and test three different
single-bit sequential logic gates: D latch, D flip-flop, and a D flip-flop
with reset and enable.

### 3.1 D Latch

A D latch has a data input (`d`) and a clock input (`clk`) and one output
(`q`). A D latch is transparent when the clock is one (i.e., the input
passes directly to the output) and is opaque with the clock is zero
(i.e., the latch "remembers" the value right before the clock
transitioned to zero and holds this value while the latch is opaque).

We have provided you the interface a D latch in `DLatch_GL.v` and a test
bench in `DLatch_GL-test.v`. For the test bench, we have provided you a
template for an exhaustive test case; all you need to do is fill in the
correct output values for each provided check. Notice that our checks
look exactly like the simulation tables we have studied in lecture with
an explicit column for the clock signal. You can test the D latch as
follows.

```bash
% cd ${HOME}/ece2300/sec06/build
% make DLatch_GL-test
% ./DLatch_GL-test
```

!!! question "Activity 1: Implement and Test D Latch"

    Create a Verilog hardware design that implements a D latch using the
    explicit gate-level modeling (do not use Boolean logic equations).
    Finish the test bench and verify that your D latch is functionally
    correct.

### 3.2 D Flip-Flop

A D flip-flop has a data input (`d`) and a clock input (`clk`) and one
output (`q`). Unlike a D latch, a D flip-flop is never transparent. A D
flip-flop samples its input right before the positive edge of the clock
and then holds this value for the entire clock period until the next
rising edge. We can implement a D flip-flop with two latches: a leader
latch and a follower latch. The leader latch uses the complement of the
clock so that only one latch is ever transparent on either phase of the
clock.

We have provided you the interface a D flip-flop in `DFF_GL.v` and a test
bench in `DFF_GL-test.v`. For the test bench, we have provided you a
template for an exhaustive test case; all you need to do is fill in the
correct output values for each provided check. You can test the D
flip-flop as follows.

```bash
% cd ${HOME}/ece2300/sec06/build
% make DFF_GL-test
% ./DFF_GL-test
```

!!! question "Activity 2: Implement and Test D Flip-Flop"

    Create a Verilog hardware design that implements a D flip-flop by
    instantiating two D latches and connecting them appropriately. You
    will also need a NOT gate (do not use Boolean logic equations).
    Finish the test bench and verify that your D flip-flop is
    functionally correct.

### 3.3 D Flip-Flop with Reset and Enable

We can also add a reset (`rst`) and enable (`en`) to our D flip-flop. The
reset signal is used to reset the D flip-flop to a known value. In our
case we will reset the flip-flop to zero. The enable signal is used to
decided whether or not the flip-flop is "enabled": if the flip-flop is
enabled (i.e., `en` is one) then the flip-flop will sample a new value on
the rising edge of the clock; but if the flip-flop is disabled then the
flip-flop will hold the old value and not sample a new value on the
rising edge of the clock. We can implement a D flip-fop with reset and
enable by adding a multiplexor, AND gate, and NOT gate to our D
flip-flop.

We have provided you the interface a D flip-flop with reset and enable in
`DFFRE_GL.v` and a test bench in `DFFRE_GL-test.v`. We have already
provided you all of the test cases you need for the D flip-flop with
reset and enable. You can test this new version of the D flip-flop as
follows.

```bash
% cd ${HOME}/ece2300/sec06/build
% make DFFRE_GL-test
% ./DFFRE_GL-test
```

!!! question "Activity 3: Implement and Test D Flip-Flop with Reset and Enable"

    Create a Verilog hardware design that implements a D flip-flop with
    reset and enable by instantiating a D flip-flop, 1-bit 2-to-1
    multiplexor, AND gate, and NOT gate and connecting them
    appropriately. Do not use Boolean logic equations. Verify that your
    new D flip-flop passes all of the provided test cases.

### 3.4 Gate-Level Register

To implement a multi-bit register we just need to instantiate a number of
D flip-flops and connect them appropriately. We have provided you the
interface an eight-bit register in `Register_8b_GL.v` and a test bench in
`Register_8b_GL-test.v`. For the test bench, we have provided you a
template for your test cases; all you need to do is fill in the correct
output values for each provided check. Take a closer look at the provided
test bench.

```verilog
  task test_case_1_basic();
    t.test_case_begin( "test_case_1_basic" );

    //    rst en d             q
    check( 0, 1, 8'b0000_0000, 8'b0000_0000 );
    check( 0, 1, 8'b0000_0001, 8'b0000_0000 );
    check( 0, 1, 8'b0000_0000, 8'b0000_0001 );
    check( 0, 1, 8'b0000_0010, 8'b0000_0000 );
    check( 0, 1, 8'b0000_0000, 8'b0000_0010 );

  endtask
```

Notice how we no longer explicitly include the clock signal in our
checks. **The ECE 2300 testing framework takes care of toggling the clock
appropriately, so we can think of each check corresponding to the row in
our simulation tables when the clock is one.** We will now always have
exactly one check per cycle. In the above basic test, we can see that the
output `q` is updated the cycle after we set the input value `d`. This is
because we are testing sequential logic. It is important to understand
how this works so you can effectively write tests for sequential logic.

You can run the test simulator for the register as follows.

```bash
% cd ${HOME}/ece2300/sec06/build
% make Register_8b_GL-test
% ./Register_8b_GL-test
```

!!! question "Activity 4: Implement and Test GL Register"

    Create a Verilog hardware design that implements an eight-bit
    register with reset and enable by instantiating eight D flip-flops
    and connecting them appropriately. Finish the test bench and verify
    that your register is functionally correct.

3. Implementing and Testing a Parameterized RTL Register
--------------------------------------------------------------------------

Our goal is to now implement a parameterized multi-bit register using RTL
modeling, but we need to learn two new concepts before we can complete
this task.

### 4.1. RTL Modeling of Sequential Logic

In the last discussion section, we learned how to use an `always_comb`
block for RTL modeling of combinational logic. `always_ff` is a new kind
of always block specifically for RTL modeling of sequential logic. The
following is an example of a D flip-flop modeled using an `always_ff`
block.

```verilog
module DFF_RTL
(
  input  logic clk,
  input  logic d,
  output logic q
);

  always_ff @(posedge clk) begin
    q <= d;
  end

endmodule
```

An `always_comb` block executes whenever any of the inputs to that block
change. An `always_ff @(posedge clk)` block only executes on the rising
edge of the clock. Notice how we are using a non-block assignment (`<=`)
instead of a blocking assignment (`=`). A non-block assignment has very
different semantics to a blocking assignment. In a non-blocking
assignment, the right-hand side (i.e., `d`) is evaluated and saved
_before_ the rising clock edge and the left-hand side (i.e., `q`) is only
updated _after_ the rising clock edge. Critically, the right-hand side is
evaluated and saved in _every_ `always_ff` across the entire hardware
design before updating _any_ left-hand side. This effectively models all
of these assignments happening concurrently on the rising edge of the
clock which is exactly what happens in hardware. We should _only_ use
non-blocking assignments in `always_ff` blocks, and we should _only_ use
blocking assignments in `always_comb` blocks. We should _never_ mix these
up!

### 4.2. Parameterized Hardware Modules

So far all of our designs have been for a fixed bitwidth. For example, we
have implemented various adders specifically designed for eight-bit
inputs and outputs. If we need a new adder for 12-bit inputs and outputs
we would need to implement and test this module from scratch. We can
develop _parameterized_ hardware modules by using Verilog parameters.
Here is an example of how we can implement a parameterized RTL adder that
can operate on inputs and outputs of any bitwidth.

```verilog
module Adder_RTL
#(
  parameter p_nbits = 1
)(
  (* keep=1 *) input  logic [p_nbits-1:0] in0,
  (* keep=1 *) input  logic [p_nbits-1:0] in1,
  (* keep=1 *) input  logic               cin,
  (* keep=1 *) output logic               cout,
  (* keep=1 *) output logic [p_nbits-1:0] sum
);

  assign {cout,sum} = in0 + in1 + { {p_nbits-1}{1'b0}, cin };

endmodule
```

Parameters are specified as a list using `#()` after the module name. In
the above example we have one parameter named `p_nbits` with a default
value of 1. This parameter is then used to specify the bitwidths of the
input ports `in0` and `in1` as well as the bitwidth of the output port
`sum`. Notice how we are also using this parameter along with the Verilog
repeat operator to zero-extend `cin`.

You can specify parameters using `#()` when you instantiate a module.
Here is how we would instantiate a parameterized RTL adder as an
eight-bit adder and connect its ports to wires.

```verilog
  logic [7:0] adder_in0;
  logic [7:0] adder_in1;
  logic       adder_cin;
  logic       adder_cout;
  logic [7:0] adder_sum;

  Adder_RTL#(8)
  (
    .in0  (adder_in0),
    .in1  (adder_in1),
    .cin  (adder_cin),
    .cout (adder_cout),
    .sum  (adder_sum)
  );
```

### 4.3. Parameterized RTL Register

We have provided you the interface for a parameterized eight-bit register
in `Register_RTL.v`.

```
module Register_RTL
#(
  parameter p_width = 1
)(
  input  logic               clk,
  input  logic               rst,
  input  logic               en,
  input  logic [p_width-1:0] d,
  output logic [p_width-1:0] q
);
```

We have provided you a test bench in `Register_RTL-test.v`. Notice how
the test bench instantiates this parameterized register as an eight-bit
register.

```
  Register_RTL#(8) register
  (
    .clk (clk),
    .rst (reset || dut_rst),
    .en  (dut_en),
    .d   (dut_d),
    .q   (dut_q)
  );
```

You should be able to just copy your tests from your gate-level register
and reuse them for your RTL register. You can run the test simulator for
the register as follows.

```bash
% cd ${HOME}/ece2300/sec06/build
% make Register_RTL-test
% ./Register_RTL-test
```

!!! question "Activity 5: Implement and Test RTL Register"

    Use what you have learned in Section 4.1 and 4.2 to create a Verilog
    hardware design that implements a parameterized register with reset
    and enable. You should have a single `always_ff` block with an
    if/else conditional operation to handle the reset and enable signals.
    Verify that your register is functionally correct.

5. Simple Counter
--------------------------------------------------------------------------

We now want to implement the following simple counter.

![](img/sec06-counter.png)

The counter has a single reset input signal which should reset the
counter register to zero. The counter then simply counts up by one every
cycle forever. If the counter reaches the maximum value (255) it just
rolls over back to zero.

We could definitely implement this simple counter using our 8-bit
gate-level adders from lab 2, but let's stick with just using the RTL
adder. We will experiment with using both the gate-level and RTL register
developed in this discussion section.

We have provided you a test bench with three test cases. Notice how we
can use a loop to test roll over:

```verilog
  task test_case_2_directed_rollover();
    t.test_case_begin( "test_case_2_directed_rollover" );

    for ( int i = 0; i < 275; i = i+1 )
      check( 0, 8'(i) );

  endtask
```

You can run the test simulator for the simple counter as follows.

```bash
% cd ${HOME}/ece2300/sec06/build
% make SimpleCounter_RTL-test
% ./SimpleCounter_RTL-test
```

!!! question "Activity 6: Implement and Test Simple Counter"

    Implement a simple counter by instantiating an eight-bit register and
    the RTL adder. Try using both the gate-level and the RTL register.
    Verify that both versions are functionally correct.

6. Clean Build
--------------------------------------------------------------------------

As a final step, do a clean build to verify everything is working
correctly.

```bash
% cd ${HOME}/ece2300/sec06
% trash build
% mkdir -p build
% cd build
% ../configure
% make check
```

