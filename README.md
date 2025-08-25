# RISC-V RV32I Processor Design and Implementation (ASIC, FPGA, Calculator Demo)

## Introduction

This project focuses on the design, implementation, and demonstration of a 32-bit RISC-V (RV32I) processor using both ASIC (VLSI) design flow and FPGA prototyping. The final processor is showcased through a Calculator Demonstration, highlighting the integration of hardware design, verification, and real-world application.

The work follows an end-to-end methodology:

* RTL coding in Verilog
* ASIC design flow with open-source EDA tools
* FPGA synthesis and implementation
* Application demonstration on a Calculator system

This repository combines all three aspects into a unified RISC-V learning and implementation project.

---

## Objectives

* Implement the RV32I RISC-V processor using open-source tools such as Icarus Verilog, Yosys, Magic VLSI, NGSpice, and Netgen
* Map the processor design onto an FPGA for real-time execution and verification
* Deploy the processor to execute calculator functions such as addition, subtraction, multiplication, and division
* Cover both digital design (RTL, simulation, verification) and physical design (synthesis, floorplanning, place and route, DRC, LVS)

---

## Table of Contents

1. Tools and Environment Setup
2. RISC-V Processor Overview
3. ASIC Design Flow

   * RTL Design and Simulation
   * Synthesis (Yosys)
   * Floorplanning and Placement
   * Routing and Layout (Magic VLSI)
   * LVS and DRC Checks
4. FPGA Implementation

   * Synthesis and Mapping
   * FPGA Board Setup
   * Test Program Execution
5. Calculator Demonstration

   * Calculator Architecture
   * Operation Flow
   * Demo Results
6. Results and Key Learnings
7. References

---

## Tools and Environment Setup

* ASIC Design Tools: Icarus Verilog, GTKWave, Yosys, Xschem, NGSpice, Magic VLSI, Netgen
* FPGA Tools: Xilinx Vivado or Intel Quartus (depending on FPGA board)
* Languages: Verilog HDL, Python (for testbenches)
* Hardware: FPGA board such as Nexys A7 or Basys 3, Calculator I/O peripherals

---

## RISC-V Processor Overview

### Why RISC-V (Comparison with ARM and x86)

RISC-V is an open standard instruction set architecture (ISA), unlike ARM (licensed) and x86 (proprietary).
<img src="https://github.com/Shubham210204/RISC-V-RV32I-Processor-Design-Implementation/blob/main/images/RISCV_vs_ARM_vs_x86.png" height=600 width=900>

* **Openness**: RISC-V is free and open-source, allowing academic, research, and commercial use without licensing fees.
* **Simplicity**: RISC-V has a reduced instruction set, making it easier to implement compared to ARM and x86.
* **Flexibility**: Supports custom extensions, enabling domain-specific processors (AI, IoT, embedded).
* **Scalability**: Works across small microcontrollers to high-performance processors.
* **Community Support**: Backed by a rapidly growing ecosystem of open-source tools and hardware.

### RISC-V Specifications

The implemented processor follows the **RV32I** base ISA (32-bit Integer instructions):

* Word length: 32 bits
* 32 general-purpose registers (x0–x31)
* Instructions: Arithmetic, Logic, Load/Store, Branch, Immediate, Jumps
* No floating-point or advanced extensions (kept modular for simplicity)
* Suitable for embedded applications, FPGA prototyping, and ASIC design

### RISC-V Architecture

The processor follows a single-cycle design for clarity and ease of verification:
<img src="https://github.com/Shubham210204/RISC-V-RV32I-Processor-Design-Implementation/blob/main/images/architecture.png">

* **Program Counter (PC):** Tracks instruction execution sequence
* **Instruction Memory:** Stores program instructions
* **Register File:** Provides 32 general-purpose registers
* **ALU (Arithmetic Logic Unit):** Executes arithmetic and logical operations
* **Control Unit:** Decodes instructions and generates control signals
* **Immediate Generator:** Extracts and formats immediate values
* **Data Memory:** Handles load and store instructions
* **ALU Control:** Determines ALU operation based on opcode and function codes

### RISC-V 5 Execution Stages

RISC-V processors typically follow a **5-stage pipeline** execution model. Each instruction goes through the following stages:

<img src="https://github.com/Shubham210204/RISC-V-RV32I-Processor-Design-Implementation/blob/main/images/pipeline.png">

1. **Instruction Fetch (IF)**

   * The Program Counter (PC) fetches the instruction from Instruction Memory.
   * PC is incremented to point to the next instruction.

2. **Instruction Decode (ID)**

   * The instruction is decoded to identify the operation type.
   * The Register File is accessed to read the required source operands.
   * Immediate values are generated if needed.

3. **Execute (EX)**

   * The Arithmetic Logic Unit (ALU) performs computations (e.g., addition, subtraction, logical operations).
   * For branch instructions, the target address is calculated.

4. **Memory Access (MEM)**

   * If the instruction is a load/store, Data Memory is accessed.
   * Load reads data from memory, Store writes data to memory.

5. **Write Back (WB)**

   * The result of the computation or data from memory is written back to the destination register.

This pipeline increases instruction throughput by overlapping execution, meaning multiple instructions are processed simultaneously in different stages.

### Base ISA Instructions (RV32I – 39 Instructions)

The RV32I base instruction set contains 39 fundamental instructions, grouped into different types based on their format and purpose.
<img src="https://github.com/Shubham210204/RISC-V-RV32I-Processor-Design-Implementation/blob/main/images/instruction_format.png">

#### 1. R-Type Instructions (Register-Register Operations)

* Format: `opcode | rd | funct3 | rs1 | rs2 | funct7`
* Used for arithmetic and logical operations between registers.
* Examples:

  * `ADD rd, rs1, rs2` → rd = rs1 + rs2
  * `SUB rd, rs1, rs2` → rd = rs1 - rs2
  * `AND rd, rs1, rs2` → rd = rs1 & rs2
  * `OR rd, rs1, rs2` → rd = rs1 | rs2
  * `XOR rd, rs1, rs2` → rd = rs1 ^ rs2
  * `SLL rd, rs1, rs2` → Shift Left Logical
  * `SRL rd, rs1, rs2` → Shift Right Logical
  * `SRA rd, rs1, rs2` → Shift Right Arithmetic
  * `SLT rd, rs1, rs2` → Set rd = 1 if rs1 < rs2 (signed)

#### 2. I-Type Instructions (Immediate and Load Operations)

* Format: `opcode | rd | funct3 | rs1 | imm[11:0]`
* Used for immediate arithmetic and memory load.
* Examples:

  * `ADDI rd, rs1, imm` → rd = rs1 + imm
  * `ANDI rd, rs1, imm` → rd = rs1 & imm
  * `ORI rd, rs1, imm` → rd = rs1 | imm
  * `XORI rd, rs1, imm` → rd = rs1 ^ imm
  * `SLTI rd, rs1, imm` → Set if less than (signed)
  * `LW rd, offset(rs1)` → Load Word from memory

#### 3. S-Type Instructions (Store Operations)

* Format: `opcode | imm[11:5] | rs2 | rs1 | funct3 | imm[4:0]`
* Used for storing data from register to memory.
* Examples:

  * `SW rs2, offset(rs1)` → Store Word
  * `SH rs2, offset(rs1)` → Store Halfword
  * `SB rs2, offset(rs1)` → Store Byte

#### 4. B-Type Instructions (Branch Operations)

* Format: `opcode | imm[12|10:5] | rs2 | rs1 | funct3 | imm[4:1|11]`
* Used for conditional branching.
* Examples:

  * `BEQ rs1, rs2, offset` → Branch if Equal
  * `BNE rs1, rs2, offset` → Branch if Not Equal
  * `BLT rs1, rs2, offset` → Branch if Less Than (signed)
  * `BGE rs1, rs2, offset` → Branch if Greater or Equal (signed)
  * `BLTU rs1, rs2, offset` → Branch if Less Than (unsigned)
  * `BGEU rs1, rs2, offset` → Branch if Greater or Equal (unsigned)

#### 5. U-Type Instructions (Upper Immediate)

* Format: `opcode | rd | imm[31:12]`
* Used for loading large immediate values.
* Examples:

  * `LUI rd, imm` → Load Upper Immediate
  * `AUIPC rd, imm` → Add Upper Immediate to PC

#### 6. J-Type Instructions (Jump Operations)

* Format: `opcode | rd | imm[20|10:1|11|19:12]`
* Used for unconditional jumps.
* Examples:

  * `JAL rd, offset` → Jump and Link
  * `JALR rd, offset(rs1)` → Jump and Link Register

### Summary of Instruction Categories

* **R-Type** → Arithmetic/Logic (register-register)
* **I-Type** → Arithmetic immediate, Loads
* **S-Type** → Stores
* **B-Type** → Branches
* **U-Type** → Upper immediate instructions
* **J-Type** → Jumps

---

## ASIC Design Flow

1. RTL Design and Simulation

   * Written in Verilog HDL
   * Verified using Icarus Verilog and GTKWave

2. Synthesis

   * RTL synthesized using Yosys into gate-level netlist

3. Floorplanning and Placement

   * Implemented using Magic VLSI

4. Routing and Layout

   * Signal routing and transistor-level layout generated

5. Verification (LVS and DRC)

   * Layout verified using Netgen and Magic VLSI

---

## FPGA Implementation

* The synthesized RISC-V processor was implemented on FPGA
* Test programs executed to validate arithmetic and logical operations
* Verified correct functionality with on-board input and output peripherals

---

## Calculator Demonstration

The processor powers a simple Calculator System that supports:

* Addition, Subtraction, Multiplication, and Division
* Inputs via switches or keypad
* Outputs via LEDs, seven segment display, or LCD

This real-world demonstration bridges the gap between processor design and application usage.

---

## Results and Key Learnings

* Completed end-to-end flow from RTL to ASIC Layout to FPGA Implementation to Calculator Demo
* Gained experience with open-source VLSI EDA tools and FPGA prototyping
* Developed understanding of RISC-V ISA, digital design, and physical design flows
* Demonstrated a working RISC-V powered Calculator


