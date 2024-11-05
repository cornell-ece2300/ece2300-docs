
TinyRV1 ISA
==========================================================================

In Lab 4, you will be implementing the TinyRV1 ISA on your processor. The
TinyRV1 ISA is a very limited subset of the 
[RISC-V ISA](https://web.csl.cornell.edu/courses/ece2300/resources/waterman-riscv-isa-vol1-2p1.pdf),
an open-source instruction set architecture that has gained significant 
popularity in the last decade.

1. Architectural State
--------------------------------------------------------------------------

### 1.1: Data Formats

TinyRV1 only supports 4B (four-byte) signed and unsigned integer values.
There are no byte nor half-word values and no floating-point.

### 1.2: General-Purpose Registers

There are 31 general-purpose registers (GPRs) `x1`-`x31` (called x 
registers), which hold integer values. Register `x0` is hardwired to the
constant zero. Each register is 32 bits wide. TinyRV1 uses the same calling
convention and symbolic register names as RISC-V:

```
+----------+----------+--------------------------------------------------------+
| Register | ABI Name | Description                                            |
|----------|----------|--------------------------------------------------------|
| x0       | zero     | The constant value 0                                   |
| x1       | ra       | Return address (caller saved)                          |
| x2       | sp       | Stack pointer (callee saved)                           |
| x3       | gp       | Global pointer                                         |
| x4       | tp       | Thread pointer                                         |
| x5       | t0       | Temporary registers (caller saved)                     |
| x6       | t1       | ↑                                                      |
| x7       | t2       | ↑                                                      |
| x8       | s0/fp    | Saved register or frame pointer (callee saved)         |
| x9       | s1       | Saved register (callee saved)                          |
| x10      | a0       | Function arguments and/or return values (caller saved) |
| x11      | a1       | ↑                                                      |
| x12      | a2       | Function arguments (caller saved)                      |
| x13      | a3       | ↑                                                      |
| x14      | a4       | ↑                                                      |
| x15      | a5       | ↑                                                      |
| x16      | a6       | ↑                                                      |
| x17      | a7       | ↑                                                      |
| x18      | s2       | Saved registers (callee saved)                         |
| x19      | s3       | ↑                                                      |
| x20      | s4       | ↑                                                      |
| x21      | s5       | ↑                                                      |
| x22      | s6       | ↑                                                      |
| x23      | s7       | ↑                                                      |
| x24      | s8       | ↑                                                      |
| x25      | s9       | ↑                                                      |
| x26      | s10      | ↑                                                      |
| x27      | s11      | ↑                                                      |
| x28      | t3       | Temporary registers (caller saved)                     |
| x29      | t4       | ↑                                                      |
| x30      | t5       | ↑                                                      |
| x31      | t6       | ↑                                                      |
+----------+----------+--------------------------------------------------------+
```

### 1.3: Memory

TinyRV1 only supports a 1MB virtual memory address space from
`0x00000000` to `0x000fffff`. The result of memory accesses to addresses
larger than `0x000fffff` are undefined. TinyRV1 uses a 
[little endian memory system](https://en.wikipedia.org/wiki/Endianness).

2. TinyRV1 Instruction and Immediate Encoding
--------------------------------------------------------------------------

The TinyRV1 ISA uses the same instruction encoding as RISC-V. There
are four instruction types and four immediate encodings. Each instruction
has a specific instruction type, and if that instruction includes an
immediate, then it will also have an immediate type.

### 2.1: TinyRV1 Instruction Formats

#### R-type

<script type="WaveDrom">
  {reg: [
    {bits: 7, name: "opcode"},
    {bits: 5, name: "rd",      type: 2},
    {bits: 3, name: "funct3"},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 5, name: "rs2",     type: 7},
    {bits: 7, name: "funct7"}
  ]}
</script>

#### I-type

<script type="WaveDrom">
  {reg: [
    {bits: 7,  name: "opcode"},
    {bits: 5,  name: "rd",      type: 2},
    {bits: 3,  name: "funct3"},
    {bits: 5,  name: "rs1",     type: 3},
    {bits: 12, name: "imm",     type: 5}
  ]}
</script>

#### S-type

<script type="WaveDrom">
  {reg: [
    {bits: 7, name: "opcode"},
    {bits: 5, name: "imm",     type: 5},
    {bits: 3, name: "funct3"},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 5, name: "rs2",     type: 7},
    {bits: 7, name: "imm",     type: 5}
  ]}
</script>

#### U-type

<script type="WaveDrom">
  {reg: [
    {bits: 7,  name: "opcode"},
    {bits: 5,  name: "rd",      type: 2},
    {bits: 20, name: "imm",     type: 5}
  ]}
</script>

### 2.2: TinyRV1 Immediate Formats

RISC-V has an asymmetric immediate encoding which means that the
immediates are formed by concatenating different bits in an asymmetric
order based on the specific immediate formats. Note that in RISC-V, all
immediates are always sign extended, and the sign-bit for the immediate
is always in bit 31 of the instruction.

For this, let's separate out the different sections of an
instruction:

<script type="WaveDrom">
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
</script>

The following diagrams represent how to re-arrange these sections to
create different immediates, based on the encoding for the particular
instruction. Here:

 - `0` is used to indicate that a particular bit is 0
 - `<-[n]->` is used to indicate that bit `n` should be replicated for
   all bits in the field

#### I-immediate

<script type="WaveDrom">
  {reg: [
    {bits: 1, name: '[20]'            },
    {bits: 4, name: '[24:21]', type: 6},
    {bits: 6, name: '[30:25]', type: 7},
    {bits: 21, name: '<-[31]->'       }
  ]}
</script>

#### S-immediate

<script type="WaveDrom">
  {reg: [
    {bits: 1, name: '[7]',     type: 2},
    {bits: 4, name: '[11:8]',  type: 4},
    {bits: 6, name: '[30:25]', type: 7},
    {bits: 21, name: '<-[31]->'       }
  ]}
</script>

#### J-immediate

<script type="WaveDrom">
  {reg: [
    {bits: 1, name: '0',       type: 1},
    {bits: 4, name: '[24:21]', type: 6},
    {bits: 6, name: '[30:25]', type: 7},
    {bits: 1, name: '[20]'            },
    {bits: 8, name: '[19:12]', type: 5},
    {bits: 12, name: '<-[31]->'       }
  ]}
</script>

#### B-immediate

<script type="WaveDrom">
  {reg: [
    {bits: 1, name: '0',       type: 1},
    {bits: 4, name: '[11:8]',  type: 4},
    {bits: 6, name: '[30:25]', type: 7},
    {bits: 1, name: '[7]',     type: 2},
    {bits: 20, name: '<-[31]->'       }
  ]}
</script>

3. TinyRV1 Instruction Details
--------------------------------------------------------------------------

For each instruction we include a brief summary, assembly syntax,
instruction semantics, instruction and immediate encoding format, and the
actual encoding for the instruction. We use the following conventions
when specifying the instruction semantics:

 - $R[\texttt{rx}]$: general-purpose register value for register specifier `rx`
 - $CSR[\texttt{csr}]$: control/status register value for register specifier `csr`
 - $\text{sext}$: sign extend to 32 bits
 - $M[\texttt{addr}]$ : 4-byte memory value at address `addr`
 - $PC$: current program counter
 - $\texttt{imm}$: immediate according to the immediate type

<style>
  .semantics_list {
    list-style: none;
    margin: 0;
    padding: 0;
  }
</style>

#### CSRR

 - Summary: Move value in control/status register to GPR
 - Assembly: `csrr rd, csr`
 - Format: I-type, I-immediate
 - Semantics: 
 - $R[\texttt{rd}] \leftarrow CSR[\texttt{csr}]$
   {: .semantics_list }

<script type="WaveDrom">
  {reg: [
    {bits: 7, name: 0b1110011},
    {bits: 5, name: "rd",      type: 2},
    {bits: 3, name: 0b1010},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 12, name: "csr",    type: 5}
  ]}
</script>

The control/status register read instruction is used to read a CSR and
write the result to a GPR. The CSRs supported in TinyRV1 are listed in
Section 4. Note that in RISC-V, `CSRR` is really a pseudo-instruction for a
specific usage of `CSRRS`, but in TinyRV1 we only support the subset of
`CSRRS` captured by `CSRR`. See Section 5 for more details about
pseudo-instructions.

#### CSRW

 - Summary: Move value in GPR to control/status register
 - Assembly: `csrw csr, rs1`
 - Format: I-type, I-immediate
 - Semantics: 
 - $CSR[\texttt{csr}] \leftarrow R[\texttt{rs1}]$
   {: .semantics_list }

<script type="WaveDrom">
  {reg: [
    {bits: 7, name: 0b1110011},
    {bits: 5, name: "rd",      type: 2},
    {bits: 3, name: 0b1001},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 12, name: "csr",    type: 5}
  ]}
</script>

The control/status register write instruction is used to read a GPR and
write the result to a CSR. The CSRs supported in TinyRV1 are listed in
Section 4. Note that in RISC-V, `CSRW` is really a pseudo-instruction for a
specific usage of `CSRRW`, but in TinyRV1 we only support the subset of
`CSRRW` captured by `CSRW`. See Section 5 for more details about
pseudo-instructions.

#### ADD

 - Summary: Addition with 3 GPRs (no overflow)
 - Assembly: `add rd, rs1, rs2`
 - Format: R-type
 - Semantics: 
 - $R[\texttt{rd}] \leftarrow R[\texttt{rs1}] + R[\texttt{rs2}]$
   {: .semantics_list }
 - $PC \leftarrow PC + 4$
   {: .semantics_list }

<script type="WaveDrom">
  {reg: [
    {bits: 7, name: 0b0110011},
    {bits: 5, name: "rd",      type: 2},
    {bits: 3, name: 0b1000},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 5, name: "rs2",     type: 7},
    {bits: 7, name: 0b10000000}
  ]}
</script>

#### ADDI

 - Summary: Add constant
 - Assembly: `addi rd, rs1, imm`
 - Format: I-type, I-immediate
 - Semantics:
 - $R[\texttt{rd}] \leftarrow R[\texttt{rs1}] + \text{sext}(\texttt{imm})$
   {: .semantics_list }
 - $PC \leftarrow PC + 4$
   {: .semantics_list }

<script type="WaveDrom">
  {reg: [
    {bits: 7,  name: 0b0010011},
    {bits: 5,  name: "rd",      type: 2},
    {bits: 3,  name: 0b1000},
    {bits: 5,  name: "rs1",     type: 3},
    {bits: 12, name: "imm",     type: 5}
  ]}
</script>

#### MUL

 - Summary: Signed multiplication with 3 GPRs (no overflow)
 - Assembly: `mul rd, rs1, imm`
 - Format: R-type
 - Semantics: 
 - $R[\texttt{rd}] \leftarrow R[\texttt{rs1}] \times R[\texttt{rs2}]$
   {: .semantics_list }
 - $PC \leftarrow PC + 4$
   {: .semantics_list }

<script type="WaveDrom">
  {reg: [
    {bits: 7, name: 0b0110011},
    {bits: 5, name: "rd",      type: 2},
    {bits: 3, name: 0b1000},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 5, name: "rs2",     type: 7},
    {bits: 7, name: 0b10000001}
  ]}
</script>

#### LW

 - Summary: Load word from memory
 - Assembly: `lw rd, imm(rs1)`
 - Format: I-type, I-immediate
 - Semantics:
 - $R[\texttt{rd}] \leftarrow M\left[R[\texttt{rs1}] + \text{sext}(\texttt{imm})\right]$
   {: .semantics_list }
 - $PC \leftarrow PC + 4$
   {: .semantics_list }

<script type="WaveDrom">
  {reg: [
    {bits: 7,  name: 0b0000011},
    {bits: 5,  name: "rd",      type: 2},
    {bits: 3,  name: 0b1010},
    {bits: 5,  name: "rs1",     type: 3},
    {bits: 12, name: "imm",     type: 5}
  ]}
</script>

#### SW

 - Summary: Store word in memory
 - Assembly: `sw rs2, imm(rs1)`
 - Format: S-type, S-immediate
 - Semantics: 
 - $M\left[R[\texttt{rs1}] + \text{sext}(\texttt{imm})\right] \leftarrow R[\texttt{rs2}]$
   {: .semantics_list }
 - $PC \leftarrow PC + 4$
   {: .semantics_list }

<script type="WaveDrom">
  {reg: [
    {bits: 7, name: 0b0100011},
    {bits: 5, name: "imm",     type: 5},
    {bits: 3, name: 0b1010},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 5, name: "rs2",     type: 7},
    {bits: 7, name: "imm",     type: 5}
  ]}
</script>

#### JAL

 - Summary: Jump to address, place return address in GPR
 - Assembly: `jal rd, addr`
 - Format: U-type, J-immediate
 - Semantics: 
 - $R[\texttt{rd}] \leftarrow PC + 4$
   {: .semantics_list }
 - $PC \leftarrow PC + \text{sext}(\texttt{imm})$
   {: .semantics_list }

<script type="WaveDrom">
  {reg: [
    {bits: 7,  name: 0b1101111},
    {bits: 5,  name: "rd",      type: 2},
    {bits: 20, name: "imm",     type: 5}
  ]}
</script>

The encoded immediate `imm` is calculated during assembly such that
$PC + \text{sext}(\texttt{imm}) = \texttt{addr}$

#### JR

 - Summary: Jump to address
 - Assembly: `jr rs1`
 - Format: I-type
 - Semantics: 
 - $PC \leftarrow R[\texttt{rs1}]$
   {: .semantics_list }

<script type="WaveDrom">
  {reg: [
    {bits: 7,  name: 0b1100111},
    {bits: 5,  name: 0b100000  },
    {bits: 3,  name: 0b1000    },
    {bits: 5,  name: "rs1",     type: 3},
    {bits: 12, name: 0b1000000000000   },
  ]}
</script>

#### BNE

 - Summary: Branch if two GPRs are not equal
 - Assembly: `bne rs1, rs2, addr`
 - Format: S-type, B-immediate
 - Semantics: 
 - $\text{if}(R[\texttt{rs1}] \neq R[\texttt{rs2}]) \ PC \leftarrow PC + \text{sext}(\texttt{imm})$
   {: .semantics_list }
 - $\text{else}\qquad\qquad\qquad\ \ \ \ \,           PC \leftarrow PC + 4$
   {: .semantics_list }

<script type="WaveDrom">
  {reg: [
    {bits: 7, name: 0b1100011},
    {bits: 5, name: "imm",     type: 5},
    {bits: 3, name: 0b1001},
    {bits: 5, name: "rs1",     type: 3},
    {bits: 5, name: "rs2",     type: 7},
    {bits: 7, name: "imm",     type: 5}
  ]}
</script>

The encoded immediate `imm` is calculated during assembly such that
$PC + \text{sext}(\texttt{imm}) = \texttt{addr}$

4. TinyRV1 Privileged ISA
--------------------------------------------------------------------------

TinyRV1 does not support any kind of distinction between user and
privileged mode. Using the terminology in the RISC-V vol 2 ISA manual,
TinyRV1 only supports M-mode.

#### Reset Vector

RISC-V specifies that on reset, $PC$ will reset to an 
implementation-defined value. TinyRV1 defines this to be at `0x00000000`.
This is where assembly tests should reside.

#### Control/Status Registers

TinyRV1 includes six non-standard CSRs. Here is the mapping:

<center>

| CSR Name | Privilege | Read/Write | CSR Num | Note         |
|----------|-----------|------------|---------|--------------|
| `in0`    | M         | R          | `0xFC2` | non-standard |
| `in1`    | M         | R          | `0xFC3` | non-standard |
| `in2`    | M         | R          | `0xFC4` | non-standard |
| `out0`   | M         | R/W        | `0x7C2` | non-standard |
| `out1`   | M         | R/W        | `0x7C3` | non-standard |
| `out2`   | M         | R/W        | `0x7C4` | non-standard |

</center>

These are chosen to conform to the guidelines in Section 2.1 of the
RISC-V vol 2 ISA manual. Here is a description of each of these six
CSRs.

 - `in0`/`in1`/`in2`
    - Used to communicate data to the processor from an external
      manager. These registers have register-mapped FIFO-dequeue
      semantics, meaning reading the registers essentially dequeues 
      the data from the head of a FIFO. Reading the registers will 
      stall if the FIFO has no valid data. Writing the registers is
      undefined.
 - `out0`/`out1`/`out2`
    - Used to communicate data from an external manager to the processor.
      These registers have register-mapped FIFO-enqueue semantics, meaning
      writing the registers essentially enqueues the data on the tail of a 
      FIFO. Writing the registers will stall if the FIFO is not ready. 
      Reading the registers is undefined.

#### Address Translation

TinyRV1 only supports the most basic form of address translation.
Every logical address is directly mapped to the corresponding physical
address. As mentioned above, TinyRV1 only supports a 1MB virtual
memory address space from `0x00000000` to `0x000fffff`, and thus TinyRV1
only supports a $1MB$ physical memory address space. In the RISC-V vol 2
ISA manual this is called a `Mbare` addressing environment.

5. TinyRV1 Pseudo-Instructions
--------------------------------------------------------------------------

It is very important to understand the relationship between the "real"
instructions presented in this manual, the "real" instructions in the
official RISC-V ISA manual, and pseudo-instructions. There are two
instructions we need to be careful with: `NOP` and `JR`. The
following table illustrates which ISAs contain which of these two
instructions, and whether or not the instruction is considered a "real"
instruction or a "pseudo-instruction".

<style> 
  thead { 
    background-color: var(--md-primary-fg-color); 
    color: var(--md-primary-bg-color); 
  } 
</style>

<center>

|          | TinyRV1 | RISC-V |
|----------|---------|--------|
| `NOP`    | pseudo  | pseudo |
| `JR`     | real    | pseudo |
| `CSRR`   | real    | pseudo |
| `CSRW`   | real    | pseudo |

</center>

`NOP` is always a pseudo-instruction. It is always equivalent to the
following use of the `ADDI` instruction:

\begin{align}
  \texttt{nop} \equiv \texttt{addi x0, x0, 0}
\end{align}

`JR` is a "real" instruction in TinyRV1, but it is a pseudo-instruction in
RISC-V for the following use of the `JALR` instruction:

\begin{align}
  \texttt{jr rs1} \equiv \texttt{jalr x0, rs1, 0}
\end{align}

`CSRR` and `CSSRW` are real instructions in TinyRV1 but they are
pseudo-instructions for the following use of the `CSRRS` and `CSRRW`:

\begin{align}
  \texttt{csrr rd, csr}  &\equiv \texttt{csrrs rd, csr, x0} \\
  \texttt{csrw csr, rs1} &\equiv \texttt{csrrw x0, csr, rs1}
\end{align}

None of this changes the encodings. In TinyRV1, `JR` is encoded the same
way as the corresponding use of the `JALR` instruction in RISC-V.
