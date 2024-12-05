
Lab 4 (Parts E & F): TinyRV1 Processor - FPGA Analysis/Prototype and Report
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
components, your single-processor, and your accumulate accelerator in
Lab 4A, 4B, and 4C. You should also have already finished your initial
single-cycle FPGA prototype in Lab 4D. In Lab 4E, we will first implement
a two-function calculator assembly program before extending this program
to support subtraction. We will then implement an accumulate assembly
program and quantify the area and performance of this kernel running on
your single-cycle processor. Finally, we will quantitatively compare the
area and performance of this software implementation to a specialized
accumulate accelerator.

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
and/or record some data for your lab report. Students can start working
on their lab report during their lab session, but will likely need to
continue working on their lab report after the lab session. The lab
report is due on Monday, Dec 9th at 11:59pm for all groups regardless of
your lab session.

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

1. Verifying Single-Cycle TinyRV1 Processor and Accumulate Accelerator
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
% source ../scripts/lab4c-run-tests.sh
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
    Show the TA running a single test case for your accumulate
    accelerator like this:

    ```bash
    % cd ${HOME}/ece2300/groupXX/lab4-proc/build
    % make AccumXcel-test
    % ./AccumXcel-test +test-case=5 +dump-vcd=waves.vcd
    % code waves.vcd
    ```

    The final result should be 36. The TA will first ask the students to
    explain _why_ the correct answer is 36. The students need to display
    the `clk`, `rst`, `go`, `size`, `result_val`, and `result` ports as
    well as the internal state register for the accumulate accelerator
    FSM in the waveforms. The TA will ask the students to show where in
    the waveforms the accelerator is producing the value 36. The TA will
    then ask the students to explain how many cycles it takes to
    calculate this result and to justify why it takes this many cycles
    using the waveform.

2. Setup Quartus Project
--------------------------------------------------------------------------

Click _Quartus (Quartus Prime 19.1)_ on the desktop to start Quartus, and
click _Run the Quartus Prime software_. You might need to try starting
Quartus twice. Setup a new Quartus project using the _New Project
Wizard_:

 - Directory, Name, Top-Level Entity
    + Working directory: `C:\Users\netid\lab4e`
    + Name of this project: `lab4e`
    + Name of top-level design entity: `lab4e`
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

3. Three-Function Calculator TinyRV1 Program
--------------------------------------------------------------------------

In this part, we will implement a TinyRV1 assembly program that has the
same behavior as the two-function calculator we implemented in Lab 2, and
then we will extend the calculator to support subtraction.

### 3.1. Simulate Simple TinyRV1 Program

We will be using the same simulators we used in the Lab 4D to emulate
what will happen when your processor is configured on the FPGA. Recall
that these simulators and the actual FPGA will use the following
connections:

 - `in0[4:0]` is connected to first five switches
 - `in1[4:0]` is connected to second five switches
 - `in2[3:0]` is connected to the four push-buttons
 - `out0[4:0]` is connected to the two seven-segment displays
 - `out1[4:0]` is connected to the two seven-segment displays
 - `out2[4:0]` is connected to the two seven-segment displays

Recall that we provided you a very simple TinyRV1 program in the
`sim/proc-sim-prog1.v` file. Take a look at this file on `ecelinux` using
VS Code.

```verilog
task proc_sim_prog1();

  asm( 'h000, "addi x1, x0, 2"   );
  asm( 'h004, "addi x2, x1, 2"   );
  asm( 'h008, "csrw out0, x2"    );
  asm( 'h00c, "jal  x0, 0x00c"   );

endtask
```

Go ahead and run this program on the FL processor simulator on `ecelinux`
like this:

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

### 3.2. Implement Two-Function Calculator TinyRV1 Program

We want our two-function calculator TinyRV1 program to exactly emulate
the behavior of the specialized two-function calculator we implemented in
Lab 2. The two-function calculator should take two inputs: a five-bit
value specified with the first five switches and a five-bit value
specified with the second five switches. The calculator should then
display these two input values on the seven-segment displays. The
calculator should perform addition if the push button is _not_ pressed
and should perform multiplication if the push button is pressed. The
calculator should output the result on the final seven-segment displays.
The pseudo-code for our two-function calculator is shown below.

```python
# read switches and buttons

in0     = read_in0()
in1     = read_in1()
buttons = read_in2()

# display inputs

write_out0(in0)
write_out1(in1)

# addition

if buttons == 0b0000:
  result = in0 + in1

# multiply

else:
  result = in0 * in1

# display result

write_out2( result )
```

Implement the two-function calculator in assembly in the
`sim/proc-sim-prog2.v` file. We recommend taking an incremental design
approach. Start by implementing a calculator that _only_ performs
addition. Once this is working, think critically about how to implement
an if/else conditional operator in assembly and then add support for
multiplication.

In general, we suggest writing out all of the assembly instructions but
leave the actual instruction address values until the end. Use `???` as
place holders for the branch and jump target addresses. Once you have all
of the assembly instructions finished, then go through and update the
address for each assembly instruction. The final step would be to go back
and update the branch and jump target addresses.

Always simulate your assembly program on the FL processor simulator
first as follows:

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-fl-sim
% ./proc-fl-sim +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0000
% ./proc-fl-sim +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0001
```

You can single-step through each assembly instruction one at a time
using the `+step` command line option. Press enter to execute the next
instruction, enter `r` and then press enter to finish the program, and
enter `q` and then press enter to quit.

Once you know your assembly program is working on the FL processor
simulator, then try it on the single-cycle processor simulator like this.

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-scycle-sim
% ./proc-scycle-sim +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0000
% ./proc-scycle-sim +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0001
```

Ideally, an effective testing strategy will ensure that your single-cycle
processor is fully correct by the time we start using these interactive
simulators. However, if the behavior is not as expected then you will
have no choice but to try and debug what has gone wrong. You will need to
use waveforms to carefully examine each cycle. You can dump waveforms
using the `+dump-vcd=waves.vcd` command line option. You should probably
use what you learn to add more directed tests cases.

!!! success "Lab Check-Off Task 3: Simulate Two-Function Calculator Program"

    Show a TA your two-function calculator program running on (1) the FL
    processor simulator; and (2) the single-cycle processor simulator.
    The TA will ask you to try some different input data. Here are the
    steps you need to show the TA.

    ```bash
    % cd ${HOME}/ece2300/groupXX/lab4-proc/build
    % make proc-fl-sim
    % make proc-scycle-sim

    % ./proc-fl-sim     +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0000
    % ./proc-fl-sim     +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0001

    % ./proc-scycle-sim +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0000
    % ./proc-scycle-sim +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0001
    ```

### 3.3. Implement Three-Function Calculator TinyRV1 Program

The two-function calculator program running on the single-cycle TinyRV1
processor requires significantly more area and has a much longer
execution time compared to the specialized two-function calculator
implemented in Lab 2. The real power of a general-purpose processor is
the ability to easily program new capabilities without adding any
hardware. Modify your two-function calculator program to add support for
subtraction. If the user does not press any push buttons the calculator
should perform addition. If the user presses the first push button the
calculator should perform multiplication. If the user presses the second
push button the calculator should perform subtraction. Note that the
TinyRV1 instruction set does not include a subtract instruction, so you
will need to implement subtraction using just the available arithmetic
instructions. Make sure your program works on the FL processor simulator
and then verify it works on the single-cycle processor simulator.

We also need to take an extra step to choose this program to actually run
on the processor once it has been configured to the FPGA. The program
which will run on the processor once it has been configured to the FPGA
is located in the `hw/ProcMem.v` module. Go ahead and take a look at this
file on `ecelinux` using VS Code. You will see a region of the module
that looks like this although it might look different based on your work
in Lab 4D.

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
% ./proc-fl-sim +prog-num=2 +dump-bin
```

So once you have verified one of your assembly programs works, use
`+dump-bin` and then copy-and-paste the resulting lines into
`hw/ProcMem.v`. You can use program number 0 to verify that the program
currently stored in `hw/ProcMem.v` works as expected:

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-scycle-sim
% ./proc-scycle-sim +prog-num=0 +in0-switches=00011 +in1-switches=00010 +buttons=0000
```

!!! note "Lab Report Task 1: Three-Function Calculator Assembly Program"

    Save your three-function calculator assembly program so you can
    include it in your lab report. All assembly code should be formatted
    using a fixed-width font.

!!! success "Lab Check-Off Task 4: Simulate Three-Function Calculator Program"

    Show a TA your thre-function calculator program running on (1) the FL
    processor simulator; (2) the single-cycle processor simulator; and
    (3) the single-cycle processor simulator with the `ProcMem`. Here are
    the steps you need to show the TA.

    ```bash
    % cd ${HOME}/ece2300/groupXX/lab4-proc/build
    % make proc-fl-sim
    % make proc-scycle-sim

    % ./proc-fl-sim     +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0000
    % ./proc-fl-sim     +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0001
    % ./proc-fl-sim     +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0010

    % ./proc-scycle-sim +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0000
    % ./proc-scycle-sim +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0001
    % ./proc-scycle-sim +prog-num=2 +in0-switches=00011 +in1-switches=00010 +buttons=0010

    % ./proc-scycle-sim +prog-num=0 +in0-switches=00011 +in1-switches=00010 +buttons=0000
    % ./proc-scycle-sim +prog-num=0 +in0-switches=00011 +in1-switches=00010 +buttons=0001
    % ./proc-scycle-sim +prog-num=0 +in0-switches=00011 +in1-switches=00010 +buttons=0010
    ```

### 3.4. Synthesize, Analyze, and Configure Single-Cycle TinyRV1 Processor

Now that we know our single-cycle processor can successfully execute the
three-function calculator assembly program in simulation, we want to see
if we can verify the same program running on the processor FPGA
prototype. As in previous labs, the _New Project Wizard_ creates a
top-level Verilog module for us which has ports for all of the switches,
LEDs, seven-segment displays, and pins on the FPGA development board.
Here is the code you can use for your top-level design.

```verilog
  logic clk;

  ClockDiv_RTL clock_div
  (
    .clk_in  (CLOCK_50),
    .clk_out (clk)
  );

  logic rst0;
  logic rst;
  always @(posedge clk) begin
    rst0 <= ~RESET_N;
    rst  <= rst0;
  end

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
    .rst             (rst),

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
    .in2             ({31'b0,~KEY[3:0]}),

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
    .rst             (rst),

    .imemreq_val     (imemreq_val),
    .imemreq_addr    (imemreq_addr),
    .imemresp_data   (imemresp_data),

    .dmemreq_val     (dmemreq_val),
    .dmemreq_type    (dmemreq_type),
    .dmemreq_addr    (dmemreq_addr),
    .dmemreq_wdata   (dmemreq_wdata),
    .dmemresp_rdata  (dmemresp_rdata)
  );

  // Out Displays

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
composition. Our timing analysis in Lab 4D showed that the single-cycle
processor cannot meet timing with a 50MHz clock (i.e., clock constraint
of 20ns). So we are using a clock divider which divides the 50MHz clock
by four to produce a 12.5MHz clock (i.e., clock constraint of 80ns). We
are now using a _reset synchronizer_ which should help address some of
the flakiness that students were seeing Lab 4D. Prof. Batten will talk
more about reset synchronizers in lecture, but you can also read about
synchronizers in Sections 3.5.4 and 3.5.5 of Harris and Harris.

Once you are happy with your understanding, you just need to copy this
code into the _DE0_CV_golden_top.v_. As in previous labs, we need to
create a _timing constraint_ file. Here are the steps to create an
initial timing constraint file:

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

create_clock -period 20 [get_ports {CLOCK_50}]
create_clock -name clk -period 80 [get_nets {ClockDiv_RTL:clock_div|count[1]}]

set_output_delay -add_delay -clock clk -max 0 [all_outputs]
set_output_delay -add_delay -clock clk -min 0 [all_outputs]

set_input_delay -add_delay -clock clk -max 0 [all_inputs]
set_input_delay -add_delay -clock clk -min 0 [all_inputs]
```

These constraints tell the FPGA tools that:

 - Our critical path delay constraint is `80ns` from all inputs to all
   outputs
 - We have a clock signal named `clk`
    - There should be no setup time violations with respect to
      `clk` when the period is `80ns`
    - There should be no hold time violations with respect to `clk`
 - The output ports have a setup time of 0 (max constraint) and a hold
   time of 0 (min constraint)
 - The input ports have clock-to-port propagation delay of 0 (max
   constraint) and a clock-to-port contamination delay of 0 (min
   constraint)

Make sure to copy-and-paste the three-function calculator program into
`hw/ProcMem.v` within Quartus. Choose _Processing > Start Compilation_
from the menu to synthesize your design. You will need to wait 5-10
minutes for synthesis to complete. Be patient! **Students should continue
on and start developing their accumulate program on `ecelinux` while
waiting for synthesis to complete.**

**Once synthesis is done, double check that your design does not have any
inferred latches!** The compilation will emit warnings not errors
regarding inferred latches, but you must remove all inferred latches.
These warnings are confusingly in green text. [Check out this Ed
post](https://edstem.org/us/courses/62248/discussion/5740183) for some
more information on how to fix common issues, including inferred latches,
we saw in Lab 4D.

Now we can configure the FPGA:

 - Choose _Tools > Programmer_ from the menu
 - Click _Hardware Setup_
 - Currently selected hardware: _USB-Blaster [USB-0]_
 - Click _Close_
 - Click _Start_

You probably need to press the reset buttons on the FPGA board to start
the execution of the assembly program. Confirm that the seven-segment
displays show the exact same output as the simulation.

!!! success "Lab Check-Off Task 5: Demonstrate TinyRV1 Processor Running Three-Function Calculator Program"

    First, show the TA the same simulation you did earlier on `ecelinux`
    like this:

    ```bash
    % cd ${HOME}/ece2300/groupXX/lab4-proc/build
    % make proc-scycle-sim
    % ./proc-scycle-sim +prog-num=0 +in0-switches=00011 +in0-switches=00010 +buttons=0000
    ```

    Then press the reset buttons the FPGA board to show the TA that your
    FPGA prototype produces the expected output. The TA will ask you to
    perform different functions on different input data and to compare
    the output between your simulator and the FPGA prototype.
    Qualitatively discuss the advantages and disadvantages of your
    software calculator running on the single-cycle processor compared to
    the specialized hardware calculator you implemented in Lab 2.

4. Accumulate TinyRV1 Program
--------------------------------------------------------------------------

In this part, we will implement a TinyRV1 assembly program that
accumulates values stored in an array.

### 4.1. Implement Accumulate TinyRV1 Program

Our program should wait for a buttons press and then read the number of
elements to accumulate from the first five switches. The program should
output the size to the seven-segment displays and output the bottom five
bits of the final result to the seven-segment displays. The pseudo-code
for our accumulate program is shown below.

```python
  # set out1 to zero

  write_out1(0)

  # wait for go

wait:
  size    = read_in0()
  buttons = read_in2()
  if buttons != 1:
    goto wait

  # display size

  write_out0(size)

  # calc

  sum = 0
  for i in range(size):
    sum = sum + a[i]

  # done

  write_out1(sum)

  while True:
    pass
```

Implement the accumulate program in assembly in the
`sim/proc-sim-prog3.v` file. Use the following template which takes care
of the wait loop, writing the result, and initializing the input data as
an array starting at address 0x080 with 32 elements. The comment next to
each element in the array specifies the value of the bottom five bits of
the result (i.e., what the seven-segment display should show for a
correct execution).

```
task proc_sim_prog3();

  // set out1 to zero

  asm( 'h000, "csrw out1, x0"      );

  // wait for go

  asm( 'h004, "csrr x1, in0"       );
  asm( 'h008, "csrr x2, in2"       );
  asm( 'h00c, "addi x3, x0, 1"     );
  asm( 'h010, "bne  x2, x3, 0x004" );

  // display size

  asm( 'h014, "csrw out0, x1" );

  // fill in the accumulate loop here

  // done (assumes result is in x4)

  asm( ?????, "csrw out1, x4"      ); // set address appropriately
  asm( ?????, "jal  x0, ?????"     ); // set address appropriately

  // Input array

                     //  size result seven_seg
  data( 'h080, 36 ); //     1     36  4
  data( 'h084, 26 ); //     2     62 30
  data( 'h088, 69 ); //     3    131  3
  data( 'h08c, 57 ); //     4    188 28
  data( 'h090, 11 ); //     5    199  7
  data( 'h094, 68 ); //     6    267 11
  data( 'h098, 41 ); //     7    308 20
  data( 'h09c, 90 ); //     8    398 14
  data( 'h0a0, 32 ); //     9    430 14
  data( 'h0a4, 76 ); //    10    506 26
  data( 'h0a8, 44 ); //    11    550  6
  data( 'h0ac, 19 ); //    12    569 25
  data( 'h0b0, 17 ); //    13    586 10
  data( 'h0b4, 59 ); //    14    645  5
  data( 'h0b8, 99 ); //    15    744  8
  data( 'h0bc, 49 ); //    16    793 25
  data( 'h0c0, 65 ); //    17    858 26
  data( 'h0c4, 12 ); //    18    870  6
  data( 'h0c8, 55 ); //    19    925 29
  data( 'h0cc,  0 ); //    20    925 29
  data( 'h0d0, 51 ); //    21    976 16
  data( 'h0d4, 42 ); //    22   1018 26
  data( 'h0d8, 82 ); //    23   1100 12
  data( 'h0dc, 23 ); //    24   1123  3
  data( 'h0e0, 21 ); //    25   1144 24
  data( 'h0e4, 54 ); //    26   1198 14
  data( 'h0e8, 83 ); //    27   1281  1
  data( 'h0ec, 31 ); //    28   1312  0
  data( 'h0f0, 16 ); //    29   1328 16
  data( 'h0f4, 76 ); //    30   1404 28
  data( 'h0f8, 21 ); //    31   1425 17
  data( 'h0fc,  4 ); //    32   1429 21

endtask
```

We recommend taking an incremental design approach. Start by ignoring the
wait loop. Simply write a loop to accumulate the first four elements in
the array, output the result to the seven-segment displays, and end with
an infinite loop. Use the `+step` command line option to ensure the
processor is executing the instructions as you expect. Once this is
working, add support for the initial wait loop. Use the `+buttons=00000`
and `+step` command line option to ensure the processor is executing the
instructions as you expect when it is waiting for a button to be pushed.

In general, we suggest writing out all of the assembly instructions but
leave the actual instruction address values until the end. Use `???` as
place holders for the branch and jump target addresses. Once you have all
of the assembly instructions finished, then go through and update the
address for each assembly instruction. The final step would be to go back
and update the branch and jump target addresses.

Always simulate your assembly program on the FL processor simulator
first as follows:

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-fl-sim
% ./proc-fl-sim +prog-num=3 +in0-switches=00100 +buttons=0000
% ./proc-fl-sim +prog-num=3 +in0-switches=00100 +buttons=0001
```

From the comment above, the result when accumulating the first four
elements should be 188 and the seven-segment display should shown 28.
Once you know your assembly program is working on the FL processor
simulator, then try it on the single-cycle processor simulator like this.

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-scycle-sim
% ./proc-scycle-sim +prog-num=3 +in0-switches=00100 +buttons=0000
% ./proc-scycle-sim +prog-num=3 +in0-switches=00100 +buttons=0001
```

Once you have verified your assembly program works, use `+dump-bin` and
then copy-and-paste the resulting lines into `hw/ProcMem.v`. You can use
program number 0 to verify that the program currently stored in
`hw/ProcMem.v` works as expected:

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-scycle-sim
% ./proc-scycle-sim +prog-num=0 +in0-switches=00100 +buttons=0000
% ./proc-scycle-sim +prog-num=0 +in0-switches=00100 +buttons=0001
```

Let's run a full experiment to accumulate 31 elements.

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-scycle-sim
% ./proc-scycle-sim +prog-num=0 +in0-switches=11111 +buttons=0001
```

The correct result is 1425 and the seven-segment display should show 17.
The simulator will print out the `cycle_count`. This is the number of
cycles it takes to execute the accumulate program. You will be working to
fill in this data table:

 - <https://docs.google.com/spreadsheets/d/1u30pax9gSaYgxfGzijZ16sxKd5bPucyl2BFsVvLCns4>

Make a copy of this table, and enter in the cycle count for your
single-cycle processor into the _fpga-perf-data_ tab. The cycle count
starts from the beginning of the program and stops once out1 is no longer
zero.

!!! note "Lab Report Task 2: Accumulate Assembly Program and Cycle Count"

    Save your accumulate assembly program so you can include it in your
    lab report. All assembly code should be formatted using a fixed-width
    font. Make sure to save your completed data table with the cycle
    count number.

!!! success "Lab Check-Off Task 6: Simulate Accumulate Program"

    Show a TA your accumulate program running on (1) the FL processor
    simulator; (2) the single-cycle processor simulator; and (3) the
    single-cycle processor simulator with the `ProcMem`. The TA will ask
    you why the cycle count is reasonable if the given size if 4. The TA
    will ask you to try a different size. Here are the steps you need to
    show the TA.

    ```bash
    % cd ${HOME}/ece2300/groupXX/lab4-proc/build
    % make proc-fl-sim
    % make proc-scycle-sim

    % ./proc-fl-sim     +prog-num=3 +in0-switches=00100 +buttons=0000
    % ./proc-fl-sim     +prog-num=3 +in0-switches=00100 +buttons=0001

    % ./proc-scycle-sim +prog-num=3 +in0-switches=00100 +buttons=0000
    % ./proc-scycle-sim +prog-num=3 +in0-switches=00100 +buttons=0001

    % ./proc-scycle-sim +prog-num=0 +in0-switches=00100 +buttons=0000
    % ./proc-scycle-sim +prog-num=0 +in0-switches=00100 +buttons=0001
    ```

### 4.2. Synthesize and Analyze Single-Cycle TinyRV1 Processor

Now that we know our single-cycle processor can successfully execute the
accumulate assembly program in simulation, we want to see if we can
verify the same program running on the processor FPGA prototype.
Copy-and-paste the accumulate program into `hw/ProcMem.v` within Quartus.
Choose _Processing > Start Compilation_ from the menu to synthesize your
design. You will need to wait 5-10 minutes for synthesis to complete. Be
patient! **Students should continue on and start experimenting with the
accumulate accelerator interactive simulator on `ecelinux` while waiting
for synthesis to complete.**

 - RTL Viewer
    + Choose _Tools > Netlist Viewer > RTL Viewer_ from the menu
    + Use the Netlist Navigator to gradually drill down in the hierarchy as follows:
       * ProcScycle
       * ProcScycleDpath
       * ALU_GL
       * Adder_32b_GL
       * AdderCarrySelect_8b_GL
       * AdderRippleCarry_4b_GL
       * FullAdder_GL
    + **Appreciate how far we have come this semester!**
    + Choose _File > Close_ from menu to close the RTL viewer
 - Chip Planner
    + Choose _Tools > Chip Planner_ from the menu
    + Identify where the logic used to implement your design is located
       in the FPGA
    + Choose _File > Close_ from the menu to close the chip planner

The next step is to analyze the area of your design.

 - Choose _Processing -> Compilation Report_ from the menu
 - Under _Table of Contents_ choose _Fitter > Resource Section > Resource
   Usage Summary_
 - Look through the report to determine the number of combinational ALUTs
   (configurable look-up tables) that are used for your design
 - Look through the report to determine the number of dedicated logic
   registers that are used for your design

Add the area data to the data table. You can find the number of 7-input
ALUTs, 6-input ALUTs, etc in the area report. You can find the dedicated
logic registers also in the area report.

The final step is to analyze the timing (i.e., the critical path delay)
of your design. We will analyze timing for the _Slow 1100mV 85C Model_
which is the default choice in the Timing Analyzer.

 - Choose _Tools > Timing Analyzer_ from the menu
 - Double-click _Update Timing Netlist_
 - Choose _Reports > Custom Reports > Report Timing_ from the menu
 - Report Timing
    + Clocks - From clock: _clk_
    + Clocks - To clock: _clk_
    + Targets - From: _[get_registers *]_
    + Targets - To: _[get_registers *]_
    + Report number of paths: _1_
    + Check next to _File name_ and enter _proc-critical-path.txt_
    + Click _Report Timing_
 - Identify the "slack" and the "data delay" of the displayed path
 - Look at the actual critical path (i.e., _Data Arrival Path_) which
    shows the longest path from one register to another register
 - Choose _File > Close_ from the menu to close the timing analyzer

!!! note "Lab Report Task 3: Collect Data for Single-Cycle Processor"

    Save your completed data table with your analysis of your
    single-cycle processor and include it in your lab report. Take a
    screenshot of the entire RTL viewer window; it must clearly show the
    Netlist Navigator with the full hierarchy from the top to the full
    adder on the left and the gate-level implementation of the full adder
    on the right. Save a screenshot of the chip planner clearly showing
    where the logic used to implement your design is located on the FPGA.
    Save the critical path report and use it to highlight the critical
    path on the processor datapath diagram; annotate the delays of the
    various components along the critical path. Remember, if you select
    multiple cells in the _Incr_ column of the timing report and hover
    your mouse it will display a pop-up showing the sum of the delays
    along that portion of the path.

!!! success "Lab Check-Off Task 7: Discuss Single-Cycle Processor"

    Show a TA your completed data table with the area and performance
    results. Show a TA the screenshot of the full adder and explain how
    the full adder fits into the complete single-cycle processor. Show a
    TA the single-cycle processor datapath with the highlighted critical
    path and annotated delays. Is the critical path as expected?

### 4.4. Configure Single-Cycle TinyRV1 Processor Prototype

Now we can configure the FPGA:

 - Choose _Tools > Programmer_ from the menu
 - Click _Hardware Setup_
 - Currently selected hardware: _USB-Blaster [USB-0]_
 - Click _Close_
 - Click _Start_

You probably need to press the reset button on the FPGA board to start
the execution of the assembly program. Confirm that the seven-segment
displays show the exact same output as the simulation. Don't forget to
actually press the push button to start the kernel!

!!! success "Lab Check-Off Task 8: Demonstrate TinyRV1 Processor Running Accumulate Program"

    First, show the TA the same simulation you did earlier on `ecelinux`
    like this:

    ```bash
    % cd ${HOME}/ece2300/groupXX/lab4-proc/build
    % make proc-scycle-sim
    % ./proc-scycle-sim +prog-num=0 +in0-switches=00100 +buttons=0001
    ```

    Then press the reset button the FPGA board to show the TA that your
    FPGA prototype produces the expected output. The TA will ask you to
    try a different size and to compare the output between your simulator
    and the FPGA prototype.

5. Accumulate Accelerator
--------------------------------------------------------------------------

In this part, we will simulate, synthesize, analyze, and configure your
accumulate accelerator. The accelerator is specialized so it should have
lower area and higher performance compared to the general-purpose
processor; but of course since it is specialized it can only do one
thing!

### 5.1. Simulate Accumulate Accelerator

We provide you an interactive accumulate accelerator simulator which
emulates the eventual FPGA prototype. You can run the interactive
simulator as follows:

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make accum-xcel-sim
% ./accum-xcel-sim +in0-switches=00100 +buttons=0000
% ./accum-xcel-sim +in0-switches=00100 +buttons=0001
```

The accumulate accelerator is setup to use the exact same data as the
accumulate assembly program so it should display the same values for a
given size. As with the accumulate assembly program, the result when
accumulating the first four elements should be 188 and the seven-segment
display should shown 28.

Let's run a full experiment to accumulate 31 elements.

```bash
% cd ${HOME}/ece2300/groupXX/lab4-proc/build
% make proc-scycle-sim
% ./proc-scycle-sim +prog-num=0 +in0-switches=11111 +buttons=0001
```

The correct result is 1425 and the seven-segment display should show 17.
The simulator will print out the `cycle_count`. This is the number of
cycles it takes for the accumulate accelerator to finish. Add the cycle
count to the data table.

!!! note "Lab Report Task 4: Accumulate Accelerator and Cycle Count"

    Make sure to save your completed data table with the cycle count
    number. You will also need to include a datapath diagram and a FSM
    diagram of your accumulate accelerator in your lab report.

!!! success "Lab Check-Off Task 9: Simulate Accumulate Accelerator"

    Start by showing a TA the datapath and FSM diagram for your
    accumulate accelerator. Clearly explain how your accumulate
    accelerator works by describing the interaction between the datapath
    and control unit. Then show a TA your accumulate accelerator working
    in simulation. The TA will ask you why the cycle count is reasonable
    if the given size is 4. Here are the steps you need to show the TA.

    ```bash
    % cd ${HOME}/ece2300/groupXX/lab4-proc/build
    % make accum-xcel-sim
    % ./accum-xcel-sim +in0-switches=00100 +buttons=0000
    % ./accum-xcel-sim +in0-switches=00100 +buttons=0001
    ```

### 5.2. Synthesize and Analyze Accumulator Accelerator

Now that we know our accumulate accelerator is fully functional, we can
synthesize and analyze the accumulate accelerator using the FPGA tools.
Here is the code you can use for your top-level design.

```verilog
  // replace this with the clock divider if you do not meet timing
  logic clk;
  assign clk = CLOCK_50;

  logic rst0;
  logic rst;
  always @(posedge clk) begin
    rst0 <= ~RESET_N;
    rst  <= rst0;
  end

  logic        xcel_go;
  logic [13:0] xcel_size;
  logic        xcel_result_val;
  logic [31:0] xcel_result;

  logic        memreq_val;
  logic [15:0] memreq_addr;
  logic [31:0] memresp_data;

  assign xcel_go = ~KEY[0];
  assign xcel_size = SW[9:5];

  AccumXcel xcel
  (
    .clk          (clk),
    .rst          (rst),
    .go           (xcel_go),
    .size         (xcel_size),
    .result_val   (xcel_result_val),
    .result       (xcel_result),
    .memreq_val   (memreq_val),
    .memreq_addr  (memreq_addr),
    .memresp_data (memresp_data)
  );

  AccumXcelMem mem
  (
    .clk          (clk),
    .rst          (rst),
    .memreq_val   (memreq_val),
    .memreq_addr  (memreq_addr),
    .memresp_data (memresp_data)
  );

  // Size Display

  Display_GL xcel_size_display
  (
    .in       (xcel_size[4:0]),
    .seg_tens (HEX5),
    .seg_ones (HEX4)
  );

  // Result Display

  Display_GL xcel_result_display
  (
    .in       (xcel_result[4:0]),
    .seg_tens (HEX3),
    .seg_ones (HEX2)
  );

  assign LEDR[0] = xcel_result_val;
```

Spend a few minutes making sure you understand this top-level
composition. Notice we are not using a clock divider since we should be
able to meet timing using a 50MHz clock. We are again using a _reset
synchronizer_ which should help address some of the flakiness that
students were seeing Lab 4D. We have connected the `xcel_result_val`
signal to one of the LEDs.

Once you are happy with your understanding, you just need to copy this
code into the _DE0_CV_golden_top.v_. We also need to update our
constraints to be as follows:

```
set_max_delay -from [all_inputs] -to [all_outputs] 20
set_min_delay -from [all_inputs] -to [all_outputs] 0

create_clock -name clk -period 20 [get_ports {CLOCK_50}]

set_output_delay -add_delay -clock clk -max 0 [all_outputs]
set_output_delay -add_delay -clock clk -min 0 [all_outputs]

set_input_delay -add_delay -clock clk -max 0 [all_inputs]
set_input_delay -add_delay -clock clk -min 0 [all_inputs]
```

This is similar to the processor except without the clock divider. Choose
_Processing > Start Compilation_ from the menu to synthesize your design.
You will need to wait 2-3 minutes for synthesis to complete. Be patient!

If your design does not meet timing then you have two options: (1) you
can change your design to try and reduce the critical path; or (2) you
can increase the clock constraint (i.e., your accelerator will run at a
lower clock frequency). For this lab let's go with option (2). You can
instantiate a clock divider at the top-level just like you did for the
processor earlier in the lab. The clock divider will increase the clock
constraint to 80ns (i.e., the target clock frequency will be 12.5MHz
instead of 50MHz). After instantiating the clock divider make sure you
also change the timing constraints to be the same as what you used with
the processor. Then try synthesizing your design again.

**Once synthesis is done, double check that your design does not have any
inferred latches!** The compilation will emit warnings not errors
regarding inferred latches, but you must remove all inferred latches.
These warnings are confusingly in green text. [Check out this Ed
post](https://edstem.org/us/courses/62248/discussion/5740183) for some
more information on how to fix common issues, including inferred latches,
we saw in Lab 4D.

The next step is to analyze the area of your design.

 - Choose _Processing -> Compilation Report_ from the menu
 - Under _Table of Contents_ choose _Fitter > Resource Section > Resource
   Usage Summary_
 - Look through the report to determine the number of combinational ALUTs
   (configurable look-up tables) that are used for your design
 - Look through the report to determine the number of dedicated logic
   registers that are used for your design

Add the area data to the data table. You can find the number of 7-input
ALUTs, 6-input ALUTs, etc in the area report. You can find the dedicated
logic registers also in the area report.

The final step is to analyze the timing (i.e., the critical path delay)
of your design. We will analyze timing for the _Slow 1100mV 85C Model_
which is the default choice in the Timing Analyzer.

 - Choose _Tools > Timing Analyzer_ from the menu
 - Double-click _Update Timing Netlist_
 - Choose _Reports > Custom Reports > Report Timing_ from the menu
 - Report Timing
    + Clocks - From clock: _clk_
    + Clocks - To clock: _clk_
    + Targets - From: _[get_registers *]_
    + Targets - To: _[get_registers *]_
    + Report number of paths: _1_
    + Check next to _File name_ and enter _xcel-critical-path.txt_
    + Click _Report Timing_
 - Identify the "slack" and the "data delay" of the displayed path
 - Look at the actual critical path (i.e., _Data Arrival Path_) which
    shows the longest path from one register to another register
 - Choose _File > Close_ from the menu to close the timing analyzer

!!! note "Lab Report Task 5: Collect Data for Accumulate Accelerator"

    Save your completed data table with your analysis of your
    single-cycle processor and include it in your lab report. Save the
    critical path report and use it to highlight the critical path on the
    accumulate accelerator datapath diagram; annotate the delays of the
    various components along the critical path. Remember, if you select
    multiple cells in the _Incr_ column of the timing report and hover
    your mouse it will display a pop-up showing the sum of the delays
    along that portion of the path.

!!! success "Lab Check-Off Task 10: Discuss Accumulate Accelerator"

    Show a TA your completed data table with the area and performance
    results. Show a TA your accumulate accelerator datapath with the
    highlighted critical path and annotated delays. Is the critical path
    as expected? Discuss the trade-off between more general-purpose
    hardware (e.g., our TinyRV1 processor) and more specialized hardware
    (e.g., our accumulate accelerator).

### 5.3. Configure Accumulator Accelerator Prototype

Now we can configure the FPGA:

 - Choose _Tools > Programmer_ from the menu
 - Click _Hardware Setup_
 - Currently selected hardware: _USB-Blaster [USB-0]_
 - Click _Close_
 - Click _Start_

You might need to press the reset button on the FPGA board. Confirm that
the seven-segment displays show the exact same output as the simulation.
Don't forget to actually press the push button to start the accelerator!

!!! success "Lab Check-Off Task 11: Demonstrate Accumulate Accelerator"

    First, show the TA the same simulation you did earlier on `ecelinux`
    like this:

    ```bash
    % cd ${HOME}/ece2300/groupXX/lab4-proc/build
    % make accum-xcel-sim
    % ./accum-xcel-sim +in0-switches=00100 +buttons=0001
    ```

    Then press the reset button the FPGA board to show the TA that your
    FPGA prototype produces the expected output. The TA will ask you to
    try a different size and to compare the output between your simulator
    and the FPGA prototype.

!!! success "Lab Check-Off Task 12: Turn In FPGA Board"

    When you are finished with your demo, pack up your FPGA development
    board. Neatly put the board, power cable, and USB cable back in the
    box. Return the box to a TA who will then record the board number on
    your check-off sheet, initial the final check-off, and then collect
    your check-off sheet.

6. Lab Report Submission
--------------------------------------------------------------------------

Students should work with their partner to prepare a short lab report
that conveys what they have learned in this lab assignment. The lab
report should start with no more than two pages of text. Students should
include all figures, tables, and diagrams after these two pages in an
appendix. The appendix can be as many pages as necessary. Do not
interleave the text, figures, tables, and diagrams. There should be two
pages of text and then the appendix with all of the text, figures,
tables, and diagrams.

There are no restrictions on font size, margins, or line spacing, but
please make sure your report is readable. We recommend using 10pt Times
or 10pt Palintino with 0.75in to 1in margins. Please make sure you
include a title, your names, and your NetIDs at the top of the first
page. Do not include a title page.

The lab report must include the following numbered sections. Please
number your sections and use these specific titles. Please follow the
guidelines on the number of paragraphs, the content of each paragraph,
and which figures/tables to include. Some paragraphs might just be 2-3
sentences.

#### Section 1. Introduction (one paragraph)

 - Include 2-3 sentences explaining what the lab involves
 - Include one sentence explaining the purpose of this lab (why are
    students doing this lab?)
 - Include one sentence explicitly connecting the lab to one or more
    lecture topics; be specific on which lecture topics this lab
    reinforces with experiential learning

#### Section 2. Single-Cycle TinyRV1 Processor (two paragraphs)

 - Paragraph 1: Accumulate Assembly Program
    + Include a sentence referencing your accumulate assembly program
       listing in the appendix
    + Include 2-3 sentences clearly describing how the accumulate
       assembly program works

 - Paragraph 2: Single-Cycle TinyRV1 Processor FPGA Implementation
    + Include a sentence referencing the data tables in the appendix
    + Include a sentence discussing the area of the processor
    + Include 2-3 sentences referencing your annotated processor datapath
       diagram in the appendix, clearly describing where the critical
       path is in your processor, and discussing if this is the expected
       path
    + Include a sentence discussing the number of cycles required for the
       processor to accumulate 13 elements; clearly justify this cycle
       count
    + Include a sentence discussing the total execution time in
       nanoseconds required for the processor to accumulate 13 elements

#### Section 3. Accumulate Accelerator (two paragraphs)

 - Paragraph 1: Accumulate Accelerator Design
    + Include a sentence referencing your datapath and FSM diagram
    + Include several sentences that clearly explain how your accumulate
      accelerator works by describing the interaction between the
      datapath and control unit; be sure to clearly explain how the
      accelerator starts and stops

 - Paragraph 2: Accumulate Accelerator FPGA implementation
    + Include a sentence referencing the data tables in the appendix
    + Include a sentence discussing the area of the accelerator
    + Include 2-3 sentences referencing your annotated accelerator
       datapath diagram in the appendix, clearly describing where the
       critical path is in your accelerator, and discussing if this is
       the expected path
    + Include a sentence discussing the number of cycles required for the
       accelerator to accumulate 13 elements; clearly justify this cycle
       count
    + Include a sentence discussing the total execution time in
       nanoseconds required for the accelerator to accumulate 13 elements

#### Section 5: Conclusion (one paragraph)

 - Include a sentence that provides a clear quantitative comparison in
    terms of area and performance between using a general-purpose
    processor vs. specialized hardware for this accumulate kernel
 - Include a sentence that provides a clear qualitative comparison in
    terms of design complexity and generality between using a
    general-purpose processor vs. specialized hardware
 - Include a sentence that draws a high-level conclusion; how has what
    you have learned impact your perspective of computer engineering

#### Appendix

 - Complex TinyRV1 program worksheet (from Lab 4D)
 - Three-function assembly program
 - Accumulate assembly program
 - FPGA Area and Performance Data Tables
 - RTL viewer showing complete hierarchy on left and full adder gate-level
    implementation on the right
 - Chip planner showing location of logic used to implement processor
 - Processor datapath diagram with highlighted critical path and
     annotated delays
 - Accumulate accelerator datapath diagram with highlighted critical path
     and annotated delays
 - Accumulate accelerator FSM diagram
 - **You do not need to include the actual critical path reports!**

