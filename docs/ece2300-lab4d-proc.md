
Lab 4 (Part D): TinyRV1 Processor - FPGA Analysis/Prototype
==========================================================================

Lab 4 will give you experience designing, implementing, testing, and
prototyping a single-cycle processor microarchitecture and a specialized
accelerator. The processor will implement the TinyRV1 instruction set.
The instruction set manual is located here:

 - <https://cornell-ece2300.github.io/ece2300-docs/ece2300-tinyrv1-isa>

The lab reinforces several lecture topics including instruction set
architectures, single-cycle processors, and finite-state machines. The
lab will continue to provide opportunities to leverage the three key
abstraction principles: modularity, hierarchy, and regularity.

You should have already worked in simulation to verify all processor
components and your single-processor in Lab 4 Parts A and B. In Lab 4
Part D, we will be using the FPGA to prototype the single-cycle processor
and demonstrate the processor running two TinyRV1 assembly programs. In
Lab 4 Part E, you will study an accumulation workload executing on both
the single-cycle TinyRV1 processor and a specialized accelerator.

This handout assumes that you have read and understand the course
tutorials, discussion sections, and successfully completed Labs 1-3. Here
are the steps to get started:

 - Step 1. Find your lab partner
 - Step 2. Find a free workstation
 - Step 3. Ask the TAs for a lab check-off sheet (each student needs
    their own check-off sheet)

Throughout this handout you will see two kinds tasks: lab check-off tasks
and lab report tasks.

For each _lab report task_ you must take some notes, save a screenshot,
and/or record some data for your lab report. The lab report is due in the
during the last week of classes.

For each _lab check-off task_ you must raise your hand and have a TA come
to check-off your work. The TA will ask you the questions included as
part of the lab check-off task and the assess your understanding using
the following rubric: mastery; accomplished; emerging; beginning. If the
TA and students together feel the students have not mastered the lab
check-off task, the students are encouraged to take a few minutes and try
again.

!!! success "Lab Check-Off Task 1: Setup FPGA Board"

    Request an FPGA board from the TAs. The TAs will record the board
    number on your check-off sheet. Use the power cord to plug the FPGA
    board into an outlet, and use the USB cable to plug the FPGA board
    into the workstation.

1. Verifying Single-Cycle TinyRV1 Processor
--------------------------------------------------------------------------

Before starting to work on an FPGA prototype, you must make sure you have
a working Verilog hardware design that has been _thoroughly_ tested in
simulation. One student should start VS Code on the workstation, log into
the `ecelinux` servers, source the setup script, and make sure their
group repository is up to date.

```bash
% source setup-ece2300.sh
% cd ${HOME}/ece2300/groupXX
% git pull
% tree
```

Where `XX` is your group number. Now run all of the tests from a clean
build to ensure your design is fully functional.

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc
% trash build
% mkdir build
% cd build
% ../configure
% make check
```

We now need to get the files for your design from `ecelinux` onto the
workstation. This requires multiple steps.

 - Step 1. Click _Microsoft Edge_ on the desktop to open a web-browser on
   the workstation to log into GitHub and then find your repository

 - Step 2. Start PowerShell by clicking the _Start_ menu then searching
   for _Windows PowerShell_

 - Step 3. Clone your repo onto the workstation by using this command in
   PowerShell (where `netid` is your Cornell NetID, **notice we are using
   https!**):

```
% git clone https://github.com/cornell-ece2300/groupXX
```

 - Step 4. In the _Connect to GitHub_ pop-up, click _Sign in with your
   browser_

 - Step 5. You may be asked for your GitHub username again and you may be
   asked to authorize the Git Credential Manager; click _authorize
   git-ecosystem_

 - Step 6. Verify that you have successfully cloned your repo by changing
   into your repo and using `tree` on the workstation:

```
% cd groupXX
% tree
```

!!! success "Lab Check-Off Task 2: Verify Tests"

    Show a TA that your hardware designs are passing all of your tests.
    The TA will ask the students to explain their testing strategy for
    two different instructions. Students should show the TA their tests
    and explain how these tests ensure correct functionality of the
    TinyRV1 processor.

2. Setup Quartus Project
--------------------------------------------------------------------------

Click _Quartus (Quartus Prime 19.1)_ on the desktop to start Quartus, and
click _Run the Quartus Prime software_. You might need to try starting
Quartus twice. Setup a new Quartus project using the _New Project
Wizard_:

 - Directory, Name, Top-Level Entity
    + Working directory: `C:\Users\netid\lab4`
    + Name of this project: `lab4`
    + Name of top-level design entity: `lab4`
    + Click _Next_
 - Directory does not exist. Do you want to create it?
    + Click yes
 - Project Type
    + Choose _Empty Project_
    + Click _Next_
 - Add Files
    + Click triple dots to right of _File name_
    + Click on _This PC_, then navigate to your cloned repo by choosing
       _Windows (C:) >  Users > netid > netid_ where _netid_ is your
       Cornell NetID
    + Shift-click on every Verilog hardware design file (do not include
       any test files)
    + Click _Open_
    + Click _Next_
 - Family, Device, and Board Settings
    + Click _Board_ tab
    + Family: _Cyclone V_
    + Select _DE0-CV Development Board_
    + Make sure _Create top-level design file_ is checked
    + Click _Next_
 - EDA Tool Settings
    + Click _Next_
 - Summary
    + Click _Finish_

Since we are now using RTL modeling, there is one new step, similar to
Labs 2 and 3. You must choose _Assignments > Settings_ from the menu.
Then select the category _Compiler Settings > Verilog HDL Input_ and
under _Verilog version_ click _SystemVerilog_. Then click _OK_.

3. Demonstrating Simple TinyRV1 Program
--------------------------------------------------------------------------

We will start by verifying your TinyRV1 processor running a very simple
program that we provide for you.

### 3.1. Simulate Simple TinyRV1 Program

We provide you two simulators which emulate what will happen when your
processor is configured on the FPGA. These simulators and the actual FPGA
will use the following connections:

 - `in0[4:0]` is connected to first five switches
 - `in1[4:0]` is connected to second five switches
 - `in2[0]` is connected to a push-button
 - `out0[4:0]` is connected to the two seven-segment displays
 - `out1[4:0]` is connected to the two seven-segment displays
 - `out2[4:0]` is connected to the two seven-segment displays

Take a look at the `sim/proc-sim-prog1.v` file on `ecelinux` using VS
Code.

```verilog
task proc_sim_prog1();

  asm( 'h000, "addi x1, x0, 2"   );
  asm( 'h004, "addi x2, x1, 2"   );
  asm( 'h008, "csrw out0, x2"    );
  asm( 'h00c, "jal  x0, 0x00c"   );

endtask
```

The programs that run on the simulator look very similar to the tests you
wrote in Lab 4 Part B. One key difference is that our processor
prototypes will be using a very small 256-byte memory which only has
space for 64 32-bit words. This means we need to keep our programs pretty
short for now! Before continuing, make sure you understand the expected
behavior of this assembly program.

You can run this program on the FL processor simulator on `ecelinux` like
this:

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-fl-sim
% ./proc-fl-sim +prog-num=1
```

Confirm that the behavior is as expected. Now run the program on the
single-cycle processor simulator like this:

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-scycle-sim
% ./proc-scycle-sim +prog-num=1
```

Ideally, an effective testing strategy will ensure that your single-cycle
processor is fully correct by the time we start using these interactive
simulators. However, if the behavior is not as expected then you will
have no choice but to try and debug what has gone wrong. You will need to
use waveforms to carefully examine each cycle. You can dump waveforms
using the `+dump-vcd=waves.vcd` command line option. You should probably
use what you learn to add more directed tests cases.

The simulators support up to three different programs located in
`sim/proc-sim-prog1.v`, `sim/proc-sim-prog2.v`, and
`sim/proc-sim-prog3.v`. Once you have verified a program works, then we
need to take an extra step to choose which program will actually run on
the processor once it has been configured to the FPGA. The program which
will run on the processor once it has been configured to the FPGA is
located in the `hw/ProcMem.v` module. Go ahead and take a look at this
file on `ecelinux` using VS Code. You will see a region of the module
that looks like this:

```verilog
   if ( rst ) begin
      mem[   0] <= 32'h00200093; // 00000000 addi x1, x0, 2
      mem[   1] <= 32'h00208113; // 00000004 addi x2, x1, 2
      mem[   2] <= 32'h7c211073; // 00000008 csrw out0, x2
      mem[   3] <= 32'h0000006f; // 0000000c jal  x0, 0x00c
    end
```

This is where we ensure the memory has the desired program when the FPGA
is reset. Writing this by hand would be tedious, so our simulators
provide the `+dump-bin` command line option which will dump out what you
need to copy into `hw/ProcMem.v`. For example,

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-fl-sim
% ./proc-fl-sim +prog-num=1 +dump-bin

      mem[   0] <= 32'h00200093; // 00000000 addi x1, x0, 2
      mem[   1] <= 32'h00208113; // 00000004 addi x2, x1, 2
      mem[   2] <= 32'h7c211073; // 00000008 csrw out0, x2
      mem[   3] <= 32'h0000006f; // 0000000c jal  x0, 0x00c

```

So once you have verified one of your assembly programs works, use
`+dump-bin` and then copy-and-paste the resulting lines into
`hw/ProcMem.v`. You can use program number 0 to verify that the program
currently stored in `hw/ProcMem.v` works as expected:

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-scycle-sim
% ./proc-scycle-sim +prog-num=0
```

!!! success "Lab Check-Off Task 3: Simulate Simple Program"

    Show a TA the provided simple assembly program running on (1) the FL
    processor simulator; (2) the single-cycle processor simulator; and
    (3) the single-cycle processor simulator with the `ProcMem`. Here are
    the steps you need to show the TA.

    ```bash
    % cd ${HOME}/ece2300/groupXX/lab4-proc/build
    % make proc-fl-sim
    % make proc-scycle-sim
    % ./proc-fl-sim     +prog-num=1
    % ./proc-scycle-sim +prog-num=1
    % ./proc-scycle-sim +prog-num=0
    ```

### 3.2. Synthesize and Analyze Single-Cycle TinyRV1 Processor

Now that we know our single-cycle processor can successfully execute the
simple assembly program in simulation, we want to see if we can verify
the same program running on the processor FPGA prototype. As in previous
labs, the _New Project Wizard_ creates a top-level Verilog module for us
which has ports for all of the switches, LEDs, seven-segment displays,
and pins on the FPGA development board. Here is the code you can use for
your top-level design.

```verilog
  logic clk;
  assign clk = CLOCK_50;

  logic        imemreq_val;
  logic [31:0] imemreq_addr;
  logic [31:0] imemresp_data;

  logic        dmemreq_val;
  logic        dmemreq_type;
  logic [31:0] dmemreq_addr;
  logic [31:0] dmemreq_wdata;
  logic [31:0] dmemresp_rdata;

  logic [31:0] proc_out0;
  logic [31:0] proc_out1;
  logic [31:0] proc_out2;

  logic        proc_trace_val_unused;
  logic [31:0] proc_trace_addr_unused;
  logic [31:0] proc_trace_data_unused;

  ProcScycle proc
  (
    .clk             (clk),
    .rst             (~RESET_N),

    .imemreq_val     (imemreq_val),
    .imemreq_addr    (imemreq_addr),
    .imemresp_data   (imemresp_data),

    .dmemreq_val     (dmemreq_val),
    .dmemreq_type    (dmemreq_type),
    .dmemreq_addr    (dmemreq_addr),
    .dmemreq_wdata   (dmemreq_wdata),
    .dmemresp_rdata  (dmemresp_rdata),

    .in0             ({27'b0,SW[9:5]}),
    .in1             ({27'b0,SW[4:0]}),
    .in2             ({31'b0,~KEY[0]}),

    .out0            (proc_out0),
    .out1            (proc_out1),
    .out2            (proc_out2),

    .trace_val       (proc_trace_val_unused),
    .trace_addr      (proc_trace_addr_unused),
    .trace_data      (proc_trace_data_unused)
  );

  ProcMem mem
  (
    .clk             (clk),
    .rst             (~RESET_N),

    .imemreq_val     (imemreq_val),
    .imemreq_addr    (imemreq_addr),
    .imemresp_data   (imemresp_data),

    .dmemreq_val     (dmemreq_val),
    .dmemreq_type    (dmemreq_type),
    .dmemreq_addr    (dmemreq_addr),
    .dmemreq_wdata   (dmemreq_wdata),
    .dmemresp_rdata  (dmemresp_rdata)
  );

  Display_GL proc_out0_display
  (
    .in       (proc_out0[4:0]),
    .seg_tens (HEX5),
    .seg_ones (HEX4)
  );

  Display_GL proc_out1_display
  (
    .in       (proc_out1[4:0]),
    .seg_tens (HEX3),
    .seg_ones (HEX2)
  );

  Display_GL proc_out2_display
  (
    .in       (proc_out2[4:0]),
    .seg_tens (HEX1),
    .seg_ones (HEX0)
  );
```

Spend a few minutes making sure you understand this top-level
composition. Once you are happy with your understanding, you just need to
copy this code into the _DE0_CV_golden_top.v_. As in previous labs, we
need to create a _timing constraint_ file. Here are the steps to create
an initial timing constraint file:

 - Choose _File > New_ from the menu
 - Click _Synopsys Design Constraints File_
 - Click _OK_
 - Enter the constraints shown below
 - Click _File > Save_ from the menu
 - Name the file _timing.sdc_
 - Save the file in the _lab4_ directory

We will use the following initial constraints:

```
set_max_delay -from [all_inputs] -to [all_outputs] 20
set_min_delay -from [all_inputs] -to [all_outputs] 0

create_clock -name clk -period 20 [get_ports {clk}]

set_output_delay -add_delay -clock clk -max 0 [all_outputs]
set_output_delay -add_delay -clock clk -min 0 [all_outputs]

set_input_delay  -add_delay -clock clk -max 0 [all_inputs]
set_input_delay  -add_delay -clock clk -min 0 [all_inputs]
```

These constraints tell the FPGA tools that:

 - Our critical path delay constraint is `20ns` from all inputs to all
   outputs as well
 - We have a clock signal named `clk`
    - There should be setup time violations with respect to
      `clk` when the period is `20ns`
    - There should be no hold time violations with respect to `clk`
 - The output ports have a setup time of 0 (max constraint) and a hold
   time of 0 (min constraint)
 - The input ports have clock-to-port propagation delay of 0 (max
   constraint) and a clock-to-port contamination delay of 0 (min
   constraint)

Choose _Processing > Start Compilation_ from the menu to synthesize your
design. You will need to wait 5-10 minutes for synthesis to complete. Be
patient!

It is very likely that your design will not meet timing, so go ahead and
analyze the timing (i.e., the critical path delay) of your design. We
will analyze timing for the _Slow 1100mV 85C Model_ which is the default
choice in the Timing Analyzer.

 - Choose _Tools > Timing Analyzer_ from the menu
 - Double-click _Update Timing Netlist_
 - Choose _Reports > Custom Reports > Report Timing_ from the menu
 - Report Timing
    + Clocks - From clock: _clk_
    + Clocks - To clock: _clk_
    + Targets - From: _[get_registers *]_
    + Targets - To: _[get_registers *]_
    + Report number of paths: _1_
    + Click _Report Timing_
 - Identify the propagation delay of the displayed path
 - Look at the actual critical path (i.e., _Data Arrival Path_) which
    shows the longest path from one of the inputs through your
    design to one of the outputs
 - Choose _File > Close_ from the menu to close the timing analyzer

Since our design did not meet timing we need to slow down the clock
frequency. We have provided you a clock divider in `hw/ClockDiv_RTL.v`
which will reduce the clock frequency by a factor of four. Go ahead and
instantiate the clock divider at the top-level like this:

```
  logic clk;
  // assign clk = CLOCK_50;
  ClockDiv_RTL clock_div
  (
    .clk_in  (CLOCK_50),
    .clk_out (clk)
  );
```

We also need to update the timing constraints to tell the tools that the
processor now only needs to meet an 80ns (12.5MHz) timing constraint.
update the timing constraint file as follows.

```
set_max_delay -from [all_inputs] -to [all_outputs] 20
set_min_delay -from [all_inputs] -to [all_outputs] 0

create_clock -period 20 [get_ports {CLOCK_50}]
create_clock -name clk -period 80 [get_nets {ClockDiv_RTL:clock_div|count[1]}]

set_output_delay -add_delay -clock clk -max 0 [all_outputs]
set_output_delay -add_delay -clock clk -min 0 [all_outputs]

set_input_delay  -add_delay -clock clk -max 0 [all_inputs]
set_input_delay  -add_delay -clock clk -min 0 [all_inputs]
```

Now re-synthesize your design and confirm it meets timing. Look at the
critical path using the Timing Analyzer.

!!! success "Lab Check-Off Task 4: Successfully Synthesize Single-Cycle Processor"

    Show a TA your top-level design with the clock divider and the final
    timing report. Use the final timing report to discuss with the TA
    where the critical path goes through the single-cycle processor.

### 3.3. Configure TinyRV1 Processor and Run Simple Program

Now we can configure the FPGA:

 - Choose _Tools > Programmer_ from the menu
 - Click _Hardware Setup_
 - Currently selected hardware: _USB-Blaster [USB-0]_
 - Click _Close_
 - Click _Start_

You probably need to press the reset button on the FPGA board to start
the execution of the assembly program. Confirm that the seven-segment
displays show the exact same output as the simulation.

!!! success "Lab Check-Off Task 5: Demonstrate TinyRV1 Processor Running Simple Program"

    First, show the TA the same simulation you did earlier on ecelinux
    like this:

    ```bash
    % cd ${HOME}/ece2300/groupXX/lab4-proc/build
    % make proc-scycle-sim
    % ./proc-scycle-sim +prog-num=0
    ```

    Then press the reset button the FPGA board to show the TA that your
    FPGA prototype produces the expected output.

4. Demonstrate Complex TinyRV1 Programs
--------------------------------------------------------------------------

In the previous part, we provided you the simple assembly program. In
this part, you will demonstrate a more complex TinyRV1 program that you
write on your own. The program must use all three arithmetic instructions
(`addi`, `add`, `mul`), must use both memory instructions (`lw`, `sw`),
and must use the `bne` instruction. The program should get at least one
input value from the switches using a `csrr` instruction and should write
one to three output values using `csrw` instructions. We recommend ending
your assembly program with an infinite loop like this:

```
  asm( 'h00c, "jal  x0, 0x00c"   );
```

Note that you will obviously need to use a different instruction address.
This way your processor will stop executing new instructions when it
reaches the infinite loop. Students can experiment with smaller programs
if they like, but the final program for the lab check-off needs to follow
the above rules.

### 4.1. Complete Worksheet for Complex TinyRV1 Program

You will start by filling out a paper worksheet with your complex TinyRV1
program. _Remember that our FPGA prototype will only have a 256-byte
memory which means there are only 64 32-bit memory locations for both
instructions and data!_ The worksheet should show the assembly for each
instruction, the values of `in0`, `in1`, and `in2` at the start of the
program, the location of the instructions in memory, and any initial data
values in memory. You should then go ahead and "execute" the assembly
program by hand using the worksheet. Use checks next to each instruction
and update the register/memory values. Make sure that `out0`, `out1`, and
`out2` reflect the expected outputs at the end of the program. You want
to make sure that you use either `in0` or `in1` to provide input data and
that this input data should influence the final output data that is
written to `out0`, `out1`, or `out2`.

!!! note "Lab Report Task 1: Complex TinyRV1 Program Worksheet"

    Make sure to keep your worksheet. You will need to include it in your
    lab report. Feel free to ask for another copy if you want to clean up
    your worksheet before including it in your lab report.

!!! success "Lab Check-Off Task 6: Complex TinyRV1 Program Worksheet"

    Show a TA your worksheet. Explain how each instruction executes
    step-by-step. Explain the final expected values for `out0`, `out1`,
    and `out2`. Explain how the final expected values for `out0`, `out1`,
    and `out2` would change if we modify `in0` or `in1`.

### 4.2. Simulate Complex TinyRV1 Program

Now write your complex TinyRV1 program in the `sim/proc-sim-prog2.v`
file. You can use the `data()` task to initialize values in memory just
like in your tests. Once you have written your program, you can test it
on the FL processor simulator like this:

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-fl-sim
% ./proc-fl-sim +prog-num=2 +in0-switches=00000 +in1-switches=00000
```

If this does not produce the expected behavior, then you need to revisit
your worksheet or possibly use waveforms to understand the execution of
your program. Try different values for `in0` and `in1`. Once you are
happy with your program, you can test it on the single-cycle processor
simulator like this:

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-scycle-sim
% ./proc-scycle-sim +prog-num=2 +in0-switches=00000 +in1-switches=00000
```

If the FL processor simulator produces the expected behavior but the
single-cycle processor simulator does not then there might be a bug in
your single-cycle processor. Ideally, an effective testing strategy will
ensure that your single-cycle processor is fully correct by the time we
start using these interactive simulators. However, if the behavior is not
as expected then you will have no choice but to try and debug what has
gone wrong. You will need to use waveforms to carefully examine each
cycle. You can dump waveforms using the `+dump-vcd=waves.vcd` command
line option. You should probably use what you learn to add more directed
tests cases.

Once your program is producing the expected behavior on both the FL
processor and single-cycle processor simulators, you are ready to put the
program into the memory which will actually be programmed on the FPGA.
Use the `+dump-bin` command line option like this:

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-fl-sim
% ./proc-fl-sim +prog-num=2 +dump-bin
```

Then copy-and-paste the corresponding code into `hw/ProcMem.v`. Then
verify that your single-cycle processor still produces the expected
output when using this memory image.

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-scycle-sim
% ./proc-scycle-sim +prog-num=0 +in0-switches=00000 +in2-switches=00000
```

!!! success "Lab Check-Off Task 7: Complex TinyRV1 Program Simulation"

    Show a TA your assembly program producing the expected output (based
    on your worksheet) as follows:

    ```bash
    % cd ${HOME}/ece2300/groupXX/lab4-proc/build
    % make proc-fl-sim
    % make proc-scycle-sim
    % ./proc-fl-sim     +prog-num=2 +in0-switches=00000 +in1-switches=00000
    % ./proc-scycle-sim +prog-num=2 +in0-switches=00000 +in1-switches=00000
    % ./proc-scycle-sim +prog-num=0 +in0-switches=00000 +in1-switches=00000
    ```

    Then you must show the TA how the output changes when you modify the
    input switches using either `+in0-switches` and/or `+in1-switches`.

### 4.3. Synthesize, Analyze, and Configure Single-Cycle TinyRV1 Processor

Copy-and-paste your new program from `ecelinux` to the `hw/ProcMem.v`
file on the lab workstation. Keep the same top-level design and timing
constraints as before. Choose _Processing > Start Compilation_ from the
menu to synthesize your design. Confirm your design still meetings timing
with the 80ns clock constraint.

Now we can configure the FPGA:

 - Choose _Tools > Programmer_ from the menu
 - Click _Hardware Setup_
 - Currently selected hardware: _USB-Blaster [USB-0]_
 - Click _Close_
 - Click _Start_

You probably need to press the reset button on the FPGA board to start
the execution of the assembly program. Confirm that the seven-segment
displays show the exact same output as the simulation.

!!! success "Lab Check-Off Task 8: Demonstrate TinyRV1 Processor Running Program"

    First, show the TA the same simulation you did earlier on ecelinux
    like this:

    ```bash
    % cd ${HOME}/ece2300/groupXX/lab4-proc/build
    % make proc-scycle-sim
    % ./proc-scycle-sim +prog-num=0 +in0-switches=00000 +in1-switches=00000
    ```

    Then press the reset button the FPGA board to show the TA that your
    FPGA prototype produces the expected output. Then modify the input
    switches in simulation and also on the FPGA board and demonstrate
    that the FPGA prototype produces the same output as the simulation.

!!! success "Lab Check-Off Task 9: Turn In FPGA Board"

    When you are finished with your demo, pack up your FPGA development
    board. Neatly put the board, power cable, and USB cable back in the
    box. Return the box to a TA who will then record the board number on
    your check-off sheet, initial the final check-off, and then collect
    your check-off sheet.

