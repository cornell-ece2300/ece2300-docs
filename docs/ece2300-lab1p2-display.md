
Lab 1.2: Five-Bit Numeric Display FPGA
==========================================================================

Lab 1 is a warmup designed to give you experience designing,
implementing, testing, and prototyping a simple Verilog hardware design.
This lab will leverage the concepts from lecture across two key
abstraction layers: logic gates and boolean equations.

You will continue to explore a five-bit numeric display that takes as
input a five-bit binary value and displays this value as a decimal number
using two seven-segment displays. Your implementation should exclusively
used combinational logic gates and/or Boolean equations. This five-bit
numeric display will be reused extensively across all of the remaining
labs. You will also gain experience optimizing your design. Lab 1.1
focuses on using simulation to test your design, while Lab 1.2 will
explore integrating, synthesizing, analyzing, and configuring your design
for an FPGA prototype. **Lab 1.1 must be done individually without a
partner; Lab 1.2 must be done with a randomly assigned partner.**

This handout assumes that you have read and understand the course
tutorials and that you have attended the discussion sections. This
handout assumes you have completed the FPGA development primer, and you
have successfully completed all parts of Lab 1.1. For Lab 1.1, both
students worked individually, so for Lab 1.2, you should choose one
student's code to use for the FPGA prototype. Obviously only choose code
which is fully functional, but also choose whichever student's
implementation of `BinaryToSevenSegOpt_GL` seems to be more aggressively
optimized.

Here are the steps to get started:

 - Step 1. Check Canvas for your randomly assigned lab partner (Click
    on _People_, then _Groups_, then search for your name to find your
    Lab 1.2 group)
 - Step 2. Find your randomly assigned lab partner
 - Step 3. Find a free workstation
 - Step 4. Ask the TAs for a lab check-off sheet (each student needs
    their own check-off sheet)

Throughout this handout you will see two kinds tasks: lab check-off tasks
and lab report tasks.

For each _lab check-off task_ you must raise your hand and have a TA come
to check-off your work. The TA will ask you the questions included as
part of the lab check-off task and the assess your understanding using
the following rubric: mastery; accomplished; emerging; beginning. If the
TA and students together feel the students have not mastered the lab
check-off task, the students are encouraged to take a few minutes and try
again.

For each _lab report task_ you must take some notes, save a screen shot,
and/or record some data for your lab report. Students can start working
on their lab report during their lab session, but will likely need to
continue working on their lab report after the lab session. The lab
report is due on Thursday at 11:59pm.

!!! success "Lab Check-Off Task 1: Setup FPGA Board"

    Request an FPGA board from the TAs. The TAs will record the board
    number on your check-off sheet. Use the power cord to plug the FPGA
    board into an outlet, and use the USB cable to plug the FPGA board
    into the workstation.

1. Simulation of Five-Bit Numeric Display
--------------------------------------------------------------------------

Before starting to work on an FPGA prototype, you must make sure you have
a working Verilog hardware design that has been _thoroughly_ tested in
simulation. The student whose code will be used for Lab 1.2, should start
VS Code on the workstation, log into an `ecelinux` server, source the
setup script, and make sure their individual remote repository is up to
date.

```bash
% source setup-ece2300.sh
% cd ${HOME}/ece2300
% git pull
% tree
```

Now run all of the tests to ensure your design is fully functional.

```bash
% cd ${HOME}/ece2300/netid

% verilator -Wall --lint-only BinaryToSevenSegUnopt_GL.v
% iverilog -Wall -g2012 -o BinaryToSevenSegUnopt_GL-test BinaryToSevenSegUnopt_GL-test.v
% ./BinaryToSevenSegUnopt_GL-test

% verilator -Wall --lint-only BinaryToSevenSegOpt_GL.v
% iverilog -Wall -g2012 -o BinaryToSevenSegOpt_GL-test BinaryToSevenSegOpt_GL-test.v
% ./BinaryToSevenSegUnopt_GL-test

% verilator -Wall --lint-only BinaryToBinCodedDec_GL.v
% iverilog -Wall -g2012 -o BinaryToBinCodedDec_GL-test BinaryToBinCodedDec_GL-test.v
% ./BinaryToBinCodedDec_GL-test

% verilator -Wall --lint-only DisplayUnopt_GL.v
% iverilog -Wall -g2012 -o DisplayUnopt_GL-test DisplayUnopt_GL-test.v
% ./DisplayUnopt_GL-test

% verilator -Wall --lint-only DisplayOpt_GL.v
% iverilog -Wall -g2012 -o DisplayOpt_GL-test DisplayOpt_GL-test.v
% ./DisplayOpt_GL-test
```

We now need to get the files for your design from `ecelinux` onto the
workstation. This requires multiple steps.

 - Step 1. Click _Microsoft Edge_ on the desktop to open a web-browser on
   the workstation to log into GitHub and then find your repository

 - Step 2. Start PowerShell by clicking the _Start_ menu then searching
   for _Windeos PowerShell_

 - Step 3. Clone your repo onto the workstation by using this command in
   PowerShell (where `netid` is your Cornell NetID, **notice we are using
   https!**):

```
% git clone https://github.com/cornell-ece2300/netid
```

 - Step 4. In the _Connect to GitHub_ pop-up, click _Sign in with your
   browser_

 - Step 5. You may be asked for your GitHub username again and you may be
   asked to authorize the Git Credential Manager; click _authorize
   git-ecosystem_

 - Step 6. Verify that you have successfully cloned your repo by changing
   into your repo and using `tree` on the workstation:

```
% cd netid
% tree
```

!!! success "Lab Check-Off Task 2: Verify Test Simulations, Discuss Optimizations"

    Show a TA that your hardware design is passing all five tests. The TA
    will ask _both students_ to explain how they optimized their designs
    for the binary-to-seven-segment converter. Explain which outputs you
    ended up optimizing and how you used either Boolean theorems or a
    Karnaugh map to simplify the logic. Hypothesize how much you think
    these optimizations will reduce the overall area of the design and/or
    the critical path delay.

2. Setup Quartus Project
--------------------------------------------------------------------------

Click _Quartus (Quartus Prime 19.1)_ on the desktop to start Quartus, and
click _Run the Quartus Prime software_. You might need to try starting
Quartus twice. Setup a new quartus project using the _New Project
Wizard_:

 - Directory, Name, Top-Level Entity
    + Working directory: `C:\Users\netid\lab1`
    + Name of this project: `lab1`
    + Name of top-level design entity: `lab1`
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

3. Integrate, Synthesize, Analyze Unoptimized Five-Bit Numeric Display
--------------------------------------------------------------------------

We will start by integrating, synthesizing, and analyzing the unoptimized
version of your five-bit numeric display. Refer back to the FPGA
development primer for more details on each step.

### 3.1. Integrate

Start by spending a few minutes identifying the location of the input
switches and the seven-segment displays on the board.

![](https://www.terasic.com.tw/attachment/archive/921/image/image_68_thumb.jpg)

The ten switches are numbered from right to left. Switch `SW[0]` is the
right-most switch, and switch `SW[9]` is the left-most switch. The
seven-segment displays are also numbered from right to levet.
Seven-segment display `HEX0` is the right-most display, and seven-segment
display `HEX5` is the left-most display.

The _New Project Wizard_ creates a top-level Verilog module for us which
has ports for all of the switches, LEDs, seven-segment displays, and pins
on the FPGA development board. You need to instantiate whatever design
you want to synthesize and analyze in this top-level Verilog module and
connect the ports appropriately.

 - Double-click on _DE0_CV_golden_top_
 - Instantiate _DisplayUnopt_GL_ in the top-level module
 - Connect the ports as shown below
 - Choose _File > Save_ from the menu

```verilog
DisplayUnopt_GL display
(
  .in       (SW[4:0]),
  .seg_tens (HEX1),
  .seg_ones (HEX0)
);
```

!!! success "Lab Check-Off Task 3: Explain Top-Level Connections"

    Show a TA the location of switches `SW[0]` through `SW[4]` on the
    board. Show the TA the location of seven-segment displays `HEX0` and
    `HEX1` on the board. Show the TA your top-level connections in
    Verilog, and clearly explain how the inputs and outputs of your
    `DisplayUnopt_GL` Verilog module will be hooked up to the physical
    switches and seven-segment displays on the board. Explain to the TA
    what is the expected behavior (i.e., when we flip these switches, we
    expect this to happen).

### 3.2. Synthesize and Analyze

Before we synthesize and analyze the unoptimized display unit, we need to
create a _timing constraint_ file. It is critical to understand that the
FPGA tools do not synthesize the design to just run as fast as possible.
The way the FPGA tools work, is that the designer provides a _timing
constraint_ on the critical path delay, and the tools work as hard as
they can (but no harder!) to meet this critical path delay constraint.
When finished, the design will either "meet timing" (i.e., the actual
critical path delay is less than the constraint) or "not meet timing"
(i.e., the actual critical path delay is greater than the contraint).

Here are the steps to create a timing constraint file:

 - Choose _File > New_ from the menu
 - Click _Synopsys Design Constraints File_
 - Click _OK_
 - Enter the constraints shown below
 - Click _File > Save_ from the menu
 - Name the file _timing.sdc_
 - Save the file in the _lab1_ directory

We will use the following initial constraints:

```
set_time_format -unit ns -decimal_places 3
create_clock -period 20 [get_ports {CLOCK_50}]

set_input_delay -add_delay -clock { CLOCK_50 } -max 0 [get_ports SW*]
set_input_delay -add_delay -clock { CLOCK_50 } -min 0 [get_ports SW*]

set_output_delay -add_delay -clock { CLOCK_50 } -max 0 [get_ports HEX*]
set_output_delay -add_delay -clock { CLOCK_50 } -min 0 [get_ports HEX*]
```

These constraints tell the FPGA tools that our critical path delay
constraint is 20ns and that the FPGA tools should analyze all paths from
every input switches (`SW*`) to every seven-segment display (`HEX*`).

Now use the following steps to synthesize your design and then look at
the RTL viewer, technology map viewer, and chip planner.

 - Choose _Processing > Start Compilation_ from the menu
 - Wait 2-3 minutes for synthesis to complete
 - RTL Viewer
    + Choose _Tools > Netlist Viewer > RTL Viewer_ from the menu
    + Drill down in the hierarchy to see the netlist for `BinaryToSevenSegUnopt_GL`
    + Choose _File > Close_ from menu to close the RTL viewer
 - Technology Map Viewer
    + Choose _Tools > Netlist Viewer > Technology Map Viewer (Post-Fitting)_
    + Click + for a new tab
    + In _Netlist Navigator_ choose _DE0_CV_golden_top > Instances > DisplayUnopt_GL_
    + Drag _DisplayUnopt_GL_ into the empty tab
    + Drill down in the hierarchy to see the implementation of `BinaryToSevenSegUnopt_GL`
    + Choose _File > Close_ from the menu to close the technology map viewer
 - Chip Planner
    + Choose _Tools > Chip Planner_ from the menu
    + Identify where the logic used to implement your design is located in the FPGA
    + Choose _File > Close_ from the menu to close the chip planner

!!! note "Lab Report Task 1: RTL Viewer for Unoptimized Design"

    Save a screenshot of the RTL viewer for just
    `BinaryToSevenSegUnopt_GL` for your lab report.

The next step is to analyze the area of your design.

 - In _Table of Contents_ choose _Fitter > Resource Section > Resource Usage Summary_
 - Look through the report to determine the number of combinational ALUs
   (configurable look-up tables) are used for your design

The final step is to analyze the timing (i.e., the critical path delay)
of your design. We will analyze timing for the **Slow 1100mV 85C Model** which is the default choice in the Timing Analyzer.

 - Choose _Tools > Timing Analyzer_ from the menu
 - Double-click _Update Timing Netlist_
 - Choose _Reports > Custom Reports > Report Timing_ from the menu
 - Report Timing
    + From: _[get_keepers SW*]_
    + To: _[get_keepers HEX*]_
    + Report number of paths: _100_
    + Click _Report Timing_
 - Identify the propagation delay of the longest path
 - Look at the actual critical path (i.e., _Data Arrival Path_) which
    shows the longest path from one of the input switches through your
    design to one of the seven-segment displays
 - Choose _File > Close_ from the menu to close the timing analyzer

!!! note "Lab Report Task 2: Critical Path for Unoptimized Design"

    Save a screenshot of just the critical path of your unoptimized
    design for your lab report. The screenshot should clearly show the
    total delay, incremental delay, location, and element for each gate
    along the critical path.

!!! success "Lab Check-Off Task 4: Discuss RTL Viewer and Critical Path for Unoptimized Design"

    Show a TA your screenshot of the RTL viewer for just
    `BinaryToSevenSegUnopt_GL`. Explain how the RTL viewer connects back
    to the Verilog for your design. Show a TA your screenshot of the
    critical path. Explain how the critical path connects back to the
    Verilog code for your design (i.e., where does the critical path
    start and end? what modules does the critical path go through?).
    Explain what is the actual delay of every single gate in the real
    FPGA along the critical path. What kind of delay model (i.e., a
    zero-delay model? a constant-delay model? a more complex delay
    model?) are the FPGA tools using to analyze the delay?

### 3.3. Iterate

Your design will almost certainly meet timing with a critical path delay
constraint of 20ns. We are interested in the limit on the critical path
delay (i.e., what is the true minimum critical path delay) so we can
compare our unoptimized and optimized designs. To find the limit, we need
to iteratively reduce the critical path timing constraint until we no
longer meet timing. We can consider the shortest critical path delay
while still meeting timing as the "true minimum critical path delay").

You can iteratively reduce the critical path delay, by changing _20_ in
the timing constraint file to something smaller. So the iterative process
will look like this:

 - Edit the timing constraints file to reduce the critical path delay constraint
 - Choose _Processing > Start Compilation_ from the menu
 - Wait 2-3 minutes for synthesis to complete
 - Analyze the area of your design
 - Analyze the timing of your design

You will be working to fill in this table:

 - <https://docs.google.com/spreadsheets/d/1lJkyLqEPCzKxX4zA0fiVSmTuxLOED7eDWVi5F_9yAC0/edit?gid=0#gid=0>

Make a copy of this table, and enter in the data for your unoptimized
design with a 20ns critical path delay constraint. You can find the
number of 7-input ALUts, 6-input ALUts, etc in the area report. The
critical path delay is just the _Data Delay_ of the slowest path in the
timing report.

Then iteratively reduce the timing constraint until your design no longer
meets timing. If the data delay is 14ns with a timing constraint of 20ns,
we recommend you reduce the timing constraint to 14ns and try again. You
want to find the a timing constraint where the design meets timing, but
if we reduce the timing constraint by 1ns the design would no longer meet
timing. The data delay for the final experiment where the design meets
timing is the "true minium critical path delay". You should enter at
least four rows into the table, but you can enter more if you need to.

!!! note "Lab Report Task 3: Save Unoptimized Analysis Data Table"

    Save your completed data table with your analysis of the unoptimized
    design and include it in your report.

!!! success "Lab Check-Off Task 5: Discuss Area and Delay Analysis for Unoptimized Design"

    Show a TA your completed data table with your analysis of the
    unoptimized design. Explain if the number ALUTs (i.e., configurable
    truth tables) either does or does not match your expectation given
    your Verilog code. Discuss what you found for the "true minimum
    critical path delay" of your unoptimized design.

4. Integrate, Synthesize, Analyze Optimized Five-Bit Numeric Display
--------------------------------------------------------------------------

Now we will integrate, synthesize, and analyze the optimized version of
your five-bit numeric display. Refer back to the FPGA development primer
for more details on each step.

### 4.1. Integrate

We will not create a new project, but we will instead simply change which
module is being instantiated in the top-level Verilog module provided for
us by the _New Project Wizard_.

 - Double-click on _DE0_CV_golden_top_
 - Instantiate _DisplayOpt_GL_ in the top-level module
 - Connect the ports as shown below
 - Choose _File > Save_ from the menu

```verilog
DisplayOpt_GL display
(
  .in       (SW[4:0]),
  .seg_tens (HEX1),
  .seg_ones (HEX0)
);
```

### 4.2. Synthesize and Analyze

We will repeat the steps we did to synthesize and analyze the unoptimized
design, except now for the optimized design. Start by resetting the
timing constraint file to use a 20ns critical path delay constraint:

```
set_time_format -unit ns -decimal_places 3
create_clock -period 20 [get_ports {CLOCK_50}]

set_input_delay -add_delay -clock { CLOCK_50 } -max 0 [get_ports SW*]
set_input_delay -add_delay -clock { CLOCK_50 } -min 0 [get_ports SW*]

set_output_delay -add_delay -clock { CLOCK_50 } -max 0 [get_ports HEX*]
set_output_delay -add_delay -clock { CLOCK_50 } -min 0 [get_ports HEX*]
```

Now use the following steps to synthesize your design and then look at
the RTL viewer, technology map viewer, and chip planner.

 - Choose _Processing > Start Compilation_ from the menu
 - Wait 2-3 minutes for synthesis to complete
 - RTL Viewer
    + Choose _Tools > Netlist Viewer > RTL Viewer_ from the menu
    + Drill down in the hierarchy to see the netlist for `BinaryToSevenSegOpt_GL`
    + **How does this compare to your unoptimized design?**
    + Choose _File > Close_ from menu to close the RTL viewer
 - Technology Map Viewer
    + Choose _Tools > Netlist Viewer > Technology Map Viewer (Post-Fitting)_
    + Click + for a new tab
    + In _Netlist Navigator_ choose _DE0_CV_golden_top > Instances > DisplayOpt_GL_
    + Drag _DisplayOpt_GL_ into the empty tab
    + Drill down in the hierarchy to see the implementation of `BinaryToSevenSegOpt_GL`
    + **How does this compare to your unoptimized design?**
    + Choose _File > Close_ from the menu to close the technology map viewer
 - Chip Planner
    + Choose _Tools > Chip Planner_ from the menu
    + Identify where the logic used to implement your design is located in the FPGA
    + **How does this compare to your unoptimized design?**
    + Choose _File > Close_ from the menu to close the chip planner

!!! note "Lab Report Task 4: RTL Viewer for Optimized Design"

    Save a screenshot of the RTL viewer for just
    `BinaryToSevenSegOpt_GL` for your lab report.

The next step is to analyze the area of your design.

 - In _Table of Contents_ choose _Fitter > Resource Section > Resource Usage Summary_
 - Look through the report to determine the number of combinational ALUs
   (configurable look-up tables) are used for your design
 - **How does this compare to your unoptimized design?**

The final step is to analyze the timing (i.e., the critical path delay)
of your design. Once again, we analyze the timing for the **Slow 1100mV 85C Model** which is the default choice in the Timing Analyzer.


 - Choose _Tools > Timing Analyzer_ from the menu
 - Double-click _Update Timing Netlist_
 - Choose _Reports > Custom Reports > Report Timing_ from the menu
 - Report Timing
    + From: _[get_keepers SW*]_
    + To: _[get_keepers HEX*]_
    + Report number of paths: _100_
    + Click _Report Timing_
 - Identify the propagation delay of the longest path
 - Look at the actual critical path (i.e., _Data Arrival Path_) which
    shows the longest path from one of the input switches through your
    design to one of the seven-segment displays
 - **How does this compare to your unoptimized design?**
 - Choose _File > Close_ from the menu to close the timing analyzer

!!! note "Lab Report Task 5: Critical Path for Optimized Design"

    Save a screenshot of just the critical path of your optimized design
    for your lab report. The screenshot should clearly show the total
    delay, incremental delay, location, and element for each gate along
    the critical path.

!!! success "Lab Check-Off Task 6: Discuss RTL Viewer and Critical Path for Optimized Design"

    Show a TA your screenshot of the RTL viewer for just
    `BinaryToSevenSegOpt_GL`. Explain how the RTL viewer connects back to
    the Verilog for your design. Show a TA your screenshot of the
    critical path. Explain how the critical path connects back to the
    Verilog code for your design (i.e., where does the critical path
    start and end? what modules does the critical path go through?).
    Explain what is the actual delay of every single gate in the real
    FPGA along the critical path.

### 4.3. Iterate

We are beginning to be able to compare our unoptmized and optimized
designs, but need to find the "true minimum critical path delay" of the
optimized design for a rigorous comparison. As with the unoptimized
design, you can iteratively reduce the critical path delay, by changing
_20_ in the timing constraint file to something smaller. The iterative
process looks like this:

 - Edit the timing constraints file to reduce the critical path delay constraint
 - Choose _Processing > Start Compilation_ from the menu
 - Wait 2-3 minutes for synthesis to complete
 - Analyze the area of your design
 - Analyze the timing of your design

You will be working to fill out the rest of this table:

 - <https://docs.google.com/spreadsheets/d/1lJkyLqEPCzKxX4zA0fiVSmTuxLOED7eDWVi5F_9yAC0/edit?gid=0#gid=0>

Work on the same copy of this table you made earlier, and enter in the
data for your optimized design with a 20ns critical path delay
constraint. Then iteratively reduce the timing constraint until your
design no longer meets timing, updating the table as you go along. You
should enter at least four rows into the table, but you can enter more if
you need to.

!!! note "Lab Report Task 6: Save Optimized Analysis Data Table"

    Save your completed data table with your analysis of the optimized
    design and include it in your report.

!!! success "Lab Check-Off Task 7: Discuss Area and Delay Analysis for Optimized Design"

    Show a TA your completed data table with your analysis of the
    optimized design. Discuss in detail your findings comparing the
    unoptimized vs. optimized implementations. What conclusions can we
    draw in terms of how much effort we should spend trying to optimize
    the gate-level implementation and/or Boolean equations in our Verilog
    hardware designs?

5. Configure Five-Bit Numeric Display FPGA Prototype
--------------------------------------------------------------------------

We now have a fully verified Verilog hardware design, and we have
finished a rigorous comparative analysis of the area and timing for both
an unoptimized and optimized implementation. The last step is to
configure the FPGA with our optimized design and demostrate the final
FPGA prototype!

 - Choose _Tools > Programmer_ from the menu
 - Click _Hardware Setup_
 - Currently selected hardware: _USB-Blaster [USB-0]_
 - Click _Close_
 - Click _Start_

!!! success "Lab Check-Off Task 8: Final Five-Bit Numeric Display Demo"

    Show a TA your final five-bit numeric display demo. The TA will ask
    you to enter at least two specific binary numbers using the switches.
    **You must determine the correct decimal value first, tell the TA
    what the correct decimal value should be, and only then set the
    switches to confirm correct operation.**

!!! success "Lab Check-Off Task 9: Turn In FPGA Board"

    When you are finished with your demo, pack up your FPGA development
    board. Neatly put the board, power cable, and USB cable back in the
    box. Return the box to a TA who will then record the board number on
    your check-off sheet, initial the final check-off, and then collect
    your check-off sheet.

6. Lab Report Submission
--------------------------------------------------------------------------

Students should work with their partner to prepare a short lab report
that conveys what they have learned in this lab assignment. Before
starting the lab report, pick one student's code and create a Verilog
data table using this template:

 - <https://docs.google.com/spreadsheets/d/1lJkyLqEPCzKxX4zA0fiVSmTuxLOED7eDWVi5F_9yAC0/edit?gid=446446324>

Count the number of logic gates with a specific number of inputs in your
Verilog code and enter these counts in the table. Count the number of
logic gates along the critical path and enter these counts in the table.
The provided template uses a very simple area/delay model where the
area/delay of a gate is equal to the number of inputs to that gate. This
data should enable you to make a very simplistic estimate of the total
area and critical path delay of the unoptimized and optimized designs.

The lab report should start with no more than two pages of text. Students
should include all figures, tables, and diagrams after these two pages in
an appendix. The appendix can be as many pages as necessary. Do not
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

#### Section 2: Optimizing a Binary-to-Seven-Segment Converter (one paragraph)

 - Choose one student's code to feature in this section
 - Include a figure in the appendix illustrating the K-maps or your
    Boolean simplifications
 - Include 2-3 sentences describing your K-maps or Boolean
    simplifications
 - Include a sentence describing _why_ the result of your K-maps or
    Boolean simplifications should hopefully result in a more
    optimized implementation

#### Section 3: Comparative Analysis (three paragraphs)

 - Paragraph 1: Verilog Analysis
    + Include the Verilog data table mentioned above in the appendix
    + Include the RTL Viewer screenshots for both the unoptimized and
       optimized designs in the appendix
    + Include a sentence referencing the Verilog data table
    + Include a sentence comparing the total area of the unoptimized
       vs optimized designs using the simple Verilog area model
    + Include a sentence comparing the total delay of the unoptimized
       vs optimized designs using the simple Verilog delay model
    + Include 2-3 sentences comparing and contrasting the unoptimized
       vs optimized designs based just on the original Verilog
    + Include a sentence discussing the RTL Viewer screenshots

 - Paragraph 2: FPGA Area Analysis
    + Include the FPGA data table in the appendix
    + Include a sentence referencing the area data in the FPGA data
       table
    + Include a sentence comparing the area of the unoptimized vs
       optimized designs using the FPGA area data
    + Include a sentence relating this FPGA comparison to your comparison
       based purely on the simple Verilog area model

 - Paragraph 3: FPGA Delay Analysis
    + Include the critical path screenshots for both the unoptimized
       and optimized design in the appendix
    + Include a sentence referencing the delay data in the FPGA data
       table
    + Include a sentence describing where the critical path goes
       through the design (at a high-level) for both the unoptimized
       and optimized designs
    + Include a sentence comparing the critical path delay of the
       unoptimized vs optimized designs using the FPGA delay data
    + Include a sentence relating this FPGA comparison to your comparison
       based purely on the simple Verilog delay model

#### Section 4: Conclusion (one paragraph)

 - Include 2-3 sentences that summarizes all of the data and analysis
    in this lab assignment
 - Include a sentence that draws a high-level conclusion; how will
    what you have learned impact your design work throughout the rest
    of the semester?

#### Appendix

 - Figure illustrating K-maps or Boolean simplifications
 - Verilog data table
 - Screenshot of RTL Viewer for unoptimized design
 - Screenshot of RTL Viewer for optimized design
 - FPGA data table
 - Screenshot of critical path for unoptimized design
 - Screenshot of critical path for optimized design

