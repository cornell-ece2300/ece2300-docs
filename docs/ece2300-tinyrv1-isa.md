
TinyRV1 ISA
==========================================================================

In Lab 4, you will be implementing the TinyRV1 ISA on your processor. The
TinyRV1 ISA is a very limited subset of the RISC-V ISA, an open-source
instruction set architecture that has gained significant popularity in the
last decade.

1. How Is The Data Represented?
--------------------------------------------------------------------------

2. Where Can The Data Be Stored?
--------------------------------------------------------------------------

3. How Can Data Be Accessed?
--------------------------------------------------------------------------

4. What Operations Can Be Done On The Data?
--------------------------------------------------------------------------

5. How Are Instructions Encoded?
--------------------------------------------------------------------------

TinyRV1 contains eight instructions; `ADD`, `ADDI`, `MUL`, `LW`, `SW`,
`JAL`, `JALR`, and `BNE`. The encoding of these instructions involves the
instructions themselves, as well as how to form immediates from the
instruction when needed.

### 5.1: Instruction Encoding

The following section details the assembly representation and semantics of
each instruction, as well as how the values for each instruction are
organized in their binary representations.

#### ADD

 - Summary: Addition with 3 general-purpose registers (no overflow)
 - Assembly: `add rd, rs1, rs2`
 - Format: R-type
 - Semantics: 

<center>
$R[\texttt{rd}] \leftarrow R[\texttt{rs1}] + R[\texttt{rs2}]$

$PC \leftarrow PC + 4$
</center>

wavedrom(
  {reg: [
    {bits: 7, name: 0b0110011},
    {bits: 5, name: "rd",      type: 2},
    {bits: 3, name: 0b1000},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 5, name: "rs2",     type: 7},
    {bits: 7, name: 0b10000000}
  ]}
)

#### ADDI

 - Summary: Add constant
 - Assembly: `addi rd, rs1, imm`
 - Format: I-type, I-immediate
 - Semantics: 

<center>
$R[\texttt{rd}] \leftarrow R[\texttt{rs1}] + \text{sext}(\texttt{imm})$

$PC \leftarrow PC + 4$
</center>

wavedrom(
  {reg: [
    {bits: 7,  name: 0b0110011},
    {bits: 5,  name: "rd",      type: 2},
    {bits: 3,  name: 0b1000},
    {bits: 5,  name: "rs1",     type: 3},
    {bits: 12, name: "imm",     type: 5}
  ]}
)

#### MUL

 - Summary: Signed multiplication with 3 general-purpose registers (no overflow)
 - Assembly: `mul rd, rs1, imm`
 - Format: R-type
 - Semantics: 

<center>
$R[\texttt{rd}] \leftarrow R[\texttt{rs1}] \times R[\texttt{rs2}]$

$PC \leftarrow PC + 4$
</center>

wavedrom(
  {reg: [
    {bits: 7, name: 0b0110011},
    {bits: 5, name: "rd",      type: 2},
    {bits: 3, name: 0b1000},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 5, name: "rs2",     type: 7},
    {bits: 7, name: 0b10000001}
  ]}
)

#### LW

 - Summary: Load word from memory
 - Assembly: `lw rd, imm(rs1)`
 - Format: I-type, I-immediate
 - Semantics: 

<center>
$R[\texttt{rd}] \leftarrow M\left[R[\texttt{rs1}] + \text{sext}(\texttt{imm})\right]$

$PC \leftarrow PC + 4$
</center>

wavedrom(
  {reg: [
    {bits: 7,  name: 0b0000011},
    {bits: 5,  name: "rd",      type: 2},
    {bits: 3,  name: 0b1010},
    {bits: 5,  name: "rs1",     type: 3},
    {bits: 12, name: "imm",     type: 5}
  ]}
)

#### SW

 - Summary: Store word in memory
 - Assembly: `sw rs2, imm(rs1)`
 - Format: S-type, S-immediate
 - Semantics: 

<center>
$M\left[R[\texttt{rs1}] + \text{sext}(\texttt{imm})\right] \leftarrow R[\texttt{rs2}]$

$PC \leftarrow PC + 4$
</center>

wavedrom(
  {reg: [
    {bits: 7, name: 0b0100011},
    {bits: 5, name: "imm",     type: 5},
    {bits: 3, name: 0b1010},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 5, name: "rs2",     type: 7},
    {bits: 7, name: "imm",     type: 5}
  ]}
)

#### JAL

 - Summary: Jump to address, place return address in general-purpose register
 - Assembly: `jal rd, imm`
 - Format: U-type, J-immediate
 - Semantics: 

<center>
$R[\texttt{rd}] \leftarrow PC + 4$

$PC \leftarrow PC + \text{sext}(\texttt{imm})$
</center>

wavedrom(
  {reg: [
    {bits: 7,  name: 0b1101111},
    {bits: 5,  name: "rd",      type: 2},
    {bits: 20, name: "imm",     type: 5}
  ]}
)

#### JR

 - Summary: Jump to address
 - Assembly: `jr rs1`
 - Format: I-type
 - Semantics: 

<center>
$PC \leftarrow R[\texttt{rs1}]$
</center>

wavedrom(
  {reg: [
    {bits: 7,  name: 0b1100111},
    {bits: 5,  name: 0b100000  },
    {bits: 3,  name: 0b1010    },
    {bits: 5,  name: "rs1",     type: 3},
    {bits: 12, name: 0b1000000000000   },
  ]}
)

#### BNE

 - Summary: Branch if two general-purpose registers are equal
 - Assembly: `bne rs1, rs2, imm`
 - Format: S-type, B-immediate
 - Semantics: 

\begin{align}
  & \text{if}(R[\texttt{rs1}] \neq R[\texttt{rs2}]) && PC \leftarrow PC + \text{sext}(\texttt{imm}) \\
  & \text{else}                                     && PC \leftarrow PC + 4
\end{align}

wavedrom(
  {reg: [
    {bits: 7, name: 0b1100011},
    {bits: 5, name: "rd",      type: 2},
    {bits: 3, name: 0b1001},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 5, name: "rs2",     type: 7},
    {bits: 7, name: "imm",     type: 5}
  ]}
)

### 5.2: Immediate Encoding

Immediates are literal values encoded as part of an instruction. These use
up space in the instruction; depending on the type of instruction, we may
wish to change which bits of the instruction are used to encode an
immediate. For this, let's separate out the different sections of an
instruction:

wavedrom(
  {reg: [
    {bits: 7, name: '',        type: 1},
    {bits: 1, name: '[7]',     type: 2},
    {bits: 4, name: '[11:8]',  type: 4},
    {bits: 8, name: '[19:12]', type: 5},
    {bits: 1, name: '[20]'            },
    {bits: 4, name: '[24:21]', type: 6},
    {bits: 6, name: '[30:25]', type: 7},
    {bits: 1, name: '[31]'            }
  ]}
)

The following diagrams represent how to re-arrange these sections to
create different immediates, based on the encoding for the particular
instruction. Here:

 - `0` is used to indicate that a particular bit is 0
 - `<-[n]->` is used to indicate that bit `n` should be replicated for
   all bits in the field

#### I-immediate

wavedrom(
  {reg: [
    {bits: 1, name: '[20]'            },
    {bits: 4, name: '[24:21]', type: 6},
    {bits: 6, name: '[30:25]', type: 7},
    {bits: 21, name: '<-[31]->'       }
  ]}
)

#### S-immediate

wavedrom(
  {reg: [
    {bits: 1, name: '[7]',     type: 2},
    {bits: 4, name: '[11:8]',  type: 4},
    {bits: 6, name: '[30:25]', type: 7},
    {bits: 21, name: '<-[31]->'       }
  ]}
)

#### J-immediate

wavedrom(
  {reg: [
    {bits: 1, name: '0',       type: 1},
    {bits: 4, name: '[24:21]', type: 6},
    {bits: 6, name: '[30:25]', type: 7},
    {bits: 1, name: '[20]'            },
    {bits: 8, name: '[19:12]', type: 5},
    {bits: 12, name: '<-[31]->'       }
  ]}
)

#### B-immediate

wavedrom(
  {reg: [
    {bits: 1, name: '0',       type: 1},
    {bits: 4, name: '[11:8]',  type: 4},
    {bits: 6, name: '[30:25]', type: 7},
    {bits: 1, name: '[7]',     type: 2},
    {bits: 20, name: '<-[31]->'       }
  ]}
)