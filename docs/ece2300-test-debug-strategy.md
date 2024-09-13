
ECE 2300 Testing and Debugging Strategy
==========================================================================

This document discuses the testing and debugging strategy we will be
using for the lab assignments in ECE 2300.

1. Testing Strategy
--------------------------------------------------------------------------

Testing is the process of checking whether a program behaves correctly.
Testing a large design can be hard because bugs may appear anywhere in
the design, and multiple bugs may interact. Good practice is to test
small parts of the design individually, before testing the entire design,
which can more readily support finding and fixing bugs.

### 1.1. Unit vs. Integration Testing

We will use both unit testing and integration testing. Unit testing is
the process of individually testing a small part or unit of a design,
typically a single module. A unit test is typically conducted by creating
a testbench, a.k.a. test harness, which is a separate program whose sole
purpose is to check that a module returns correct output values for a
variety of input values. Each unique set of input values is known as a
test vector. Manually examining printed output is cumbersome and error
prone. A better test harness would only print a message for incorrect
output. Integration testing involves testing the composition of various
modules and should only be attempted after we have unit tested those
modules.

### 1.2. Black-Box vs. White-Box Testing

We will be using a mix of black-box and white-box testing. Black box
testing is where your test cases only test the _interface_ of your
design. Black-box testing does not _directly_ test any of the internals
within your design. Obviously, black-box testing will _indirectly_ test
the internals though. White-box testing is where your test cases directly
test the internals by perhaps poking into the design using hierarchical
signal references. White-box testing is pretty fragile so we won't really
be using it in this course. We also might do what I call "gray-box"
testing. This is where you choose specific test vectors that are
carefully designed to trigger complex behavior in a specific
implementation. Since gray-box tests can be applied to any
implementation, they are like black-box tests. Since they attempt to
trigger complex implementation-specific behavior, they are like white-box
tests.

### 1.3. Directed vs. Random Testing

We will primarly be using directed testing and random testing. Directed
testing is where the designer explicitly specifies the inputs and the
correct outputs. Directed tests are carefully crafted to enable good
coverage of many different hardware behaviors. Random testing is where
the designer randomly generates inputs and then verifies that the
function produces the right output. This of course begs the question,
"How do we know what the right output is, if we are randomly generating
the input?" There are two approaches. First, the designer can assert that
property is valid on the output. For example, if the module is meant to
sort a sequence of values, the random test can assert that the final
values are is indeed sorted. Second, the designer can use a golden
reference implementation. For example, the programmer might use a
functional-level model of the module.

### 1.4. Value vs. Delay Testing

We will also use value and delay testing. Value testing focuses on
applying different input values and checking the corresponding output
values. Delay testing focuses more on changing the delays between _when_
input values are provided and possibly also changing the delays between
_when_ output values are accepted. Delay testing is particularly
important when working with memory interfaces to ensure the
design-under-test can correctly handle wait states. A variant of delay
testing involves verifying that the delay of a module is as expected. So
for example we might assert that the module takes no longer than a
specific number of cycles to executes a specific transaction.

### 1.5. Mixing Styles of Testing

Note that we usually mix and match these different styles of testing. So
we can use unit, black-box, directed, value testing or integration,
black-box, random, delay testing. Note that ad-hoc testing should _not_
an important part of your testing strategy. It is neither automatic nor
systematic.

2. Debugging Strategy
--------------------------------------------------------------------------

Here is our recommended systematic debugging process.

### Step 1: Configure the build system

Start by creating a build directory and using `configure` to configure
the build system.

```bash
% mkdir build
% cd build
% ../configure
```

### Step 2: Run all test benches

Fun all of the tests to get a high-level view of what test cases are
passing and what test cases are failing. This gives you the big pictures.
Are many tests failing? Are just a few tests failing?

```bash
% make check
```

### Step 3: Zoom-in on one test bench in isolation

Pick one failing test bench to focus on. Choose the failing test bench
that tests the _simplest_ hardware module. So if your design has many
modules, start by focusing on the simplest child module. Let's assume we
want to focus on the test bench named `Foo-test.v`. Build and run the
failing test bench in isolation like this:

```bash
% make Foo-test
% ./Foo-test
```

You can build and run the failing test bench in a single step using
command chaining like this:

```bash
% make Foo-test && ./Foo-test
```

### Step 4: Zoom-in on one test case in isolation

Each test bench will contain many test cases. Pick one failing test case
to focus on. Choose the simplest failing test case. Let's assume the test
bench includes a basic test case, several directed test cases, and a
random test case. If all of the test cases are failing, then start by
focusing on the basic test case. If the basic test case is passing, but
all of the remaining test cases are failing, then start by focusing on
the simplest directed test case. Only focus on a random test case if it
is the only failing test case. Lest's assume we want to focus on test
case number 2. Build and run just failing test test case in isolation
like this:

```bash
% make Foo-test && ./Foo-test +test-case=2
```

When you run a test case in isolation, the testing framework will display
a trace showing the values of various signals over time. You will also
see a more detailed error message showing the value from the design under
test that does not match the expected value.

### Step 5: Determine the observable error

Look at the trace and the error message. Determine the exact observable
error. The observable error might be something like "signal X is supposed
to be 0 but is instead 1". The observable error is never "my code doesn't
work"; the observable error needs to be a specific statement about a
signal and how its value is different from what is expected.

### Step 6: Look at the test case

Look at the actual test case. Make absolutely sure you know what the test
case is testing and that the test case is valid. You have no hope of
debugging your design if you do not understand what correct execution you
expect to happen!

### Step 7: Zoom-in both in terms of space and time

Try to zoom-in in terms of space. Look at the trace and see if you can
move the observable error in space. For example, let's say you have a
parent module with three child modules A, B, and C. The input of the
parent module is connected to the input of A, the output of A is
connected to the input of B, the output of B is connected to the input of
C, and the output of C is connected to the output of the parent module.
Let's say after step 5, the observable error is that the output value of
the parent module does not match the expected output value. From the
trace we might be able to determine that the input to B is correct, but
the output of B is incorrect. This means we have just moved the
observable error from the output of the parent module to the output of
module B. We no longer need to focus on module A or module C, we can
zoom-in and focus on debugging module B.

Try to zoom-in in terms of time. Work _backwards_ from the observable
error on the trace. For designs with just combinational logic (i.e., labs
1 and 2) there might not be any need to work backwards through the trace
since the outputs always directly depend on the inputs. For designs with
sequential logic working backwards through the trace is much more
important. There may be an observable error on cycle 10, but if we work
backwards through the trace we might realize that something first goes
wrong on cycle 5 and eventually this casues the incorrect output on cycle
10. We no longer need to focus on cycles 6 through 10. We can zoom-in and
focus on cycle 5.

Essentially, for designs with sequential logic we want to try and
work backwards through the trace to find the first cycle where the
previous cycle looks correct and the next cycle looks incorrect. Our goal
is to move the observable error as far backwards in the trace as
possible.


Step 7: Based on the narrowed focus from step 6, make a hypothesis on
what might be wrong. Take a quick look at the corresponding code. Check
for errors in bitwidth, in signal naming, or in connectivity. If you
cannot spot anything obvious then go to the next step. If you spot
something obvious skip to step 10.

Step 8: Use the --dump-vcd option to generate a VCD file. Open the VCD
file in gtkwave. Add the clock, maybe add the inst_D, inst_X, inst_M,
inst_W fields to the waveform view. Use the narrowed focus from Step 6
and the hypothesis from Step 7 to zoom in on a specific cycle and a
specific part of the design where you can clearly see a specific signal
that is incorrect.

Step 9: Work _backwards_ from the signal which is incorrect. Work
backwards in the datapath -- keep working backwards component by
component. For each component look at the inputs (all inputs, look at
data inputs and control signals) and look at the outputs (all outputs,
look at data outputs and status signals). Check for one of three things:
(1) are the inputs incorrect and the outputs incorrect for this
component? if so you need to continue working backwards -- if the
incorrect input is a control signal then you need to start working
backwards into the control unit; (2) are the inputs correct and the
outputs incorrect for this component? if so then you have narrowed the
bug to be inside the component (maybe it is a bug in the ALU? maybe it is
a bug in some other module?); or (3) are the inputs correct and the
outputs correct for this component? Then you have gone backwards too far
and you need to go forward in the design again to find a signal which is
incorrect.

Step 10: Once you find a bug, make a hypothesis about what should happen
if you fix the bug. Your hypothesis should not just be "fixing the bug
will make the test pass." It should instead be something like "fixing
this bug should make this specific signal be 1 instead of 0" or "fixing
this bug should make this specific instruction in the line trace stall".
Fix the bug and see what happens by looking at the line trace and/or
waveform. Don't just see if it passes the test -- literally check the
line trace and/or waveform and see if the behavior confirms the line
trace. One of four things will happen: (1) the test will pass and the
linetrace/waveform behavior will match your hypothesis -- bug fixed! (2)
the test will fail and the linetrace/waveform will not match your
hypothesis -- you need to keep working -- your bug fix did not do what it
was supposed to, and it did not fix the error -- undo the bug fix and go
back to step 6. (3) the test will fail but the linetrace/waveform _will_
match your hypothesis -- this means your bug fix did what you expected
but there might be another bug still causing trouble -- you need to keep
working -- go back to step 6. (4) the test will pass and the
linetrace/waveform _will not_ match your hypothesis -- you need to keep
working -- your bug fix did not do what you thought it would even though
it cause the test to pass -- there might be something subtle going on --
go back to step 6 to figure out why the bug fix did not do what you
thought it would.

Note a couple things about this systematic 10 step process. First, it is
a systematic process ... it does not involve randomly trying things.
Second, the process uses all tools at your disposable: output from
pytest, traceback, line tracing, and VCD waveforms. You really need to
use all of these tools. If you use line tracing but never use VCD
waveforms or you use VCD waveforms and never use line tracing then you
are putting yourself at a disadvantage. Third, the process requires you
to think critically and make a hypothesis about what should change -- do
not just change something, pass the test, and move on -- change something
and see if the line trace and waveforms change in the way you expect.
Otherwise you can actually introduce more bugs even though you think are
fixing things.

