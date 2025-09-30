# **RISC-V RV32I Processor VLSI Design Flow**

## **Introduction**
This project focuses on the **RTL to GDSII design** of a **5 stages pipelined RISC-V processor RV32I** using **Verilog and QFLOW toolchain**. The entire design flow is executed using **open-source VLSI tools**, ensuring an efficient and fabrication-ready implementation.  

## **Key Features**

* **RV32I Base Instruction Set:** Implements the complete 32-bit integer instruction set.
* **Pipelined Architecture:** Instructions are executed in an overlapping manner across the five stages to improve throughput.
* **Hazard Detection and Resolution:**
    * **Data Hazards:** Resolved using a **Forwarding Unit** to forward results from later stages to the Execute stage.
    * **Control Hazards:** Handled by flushing the pipeline on taken branches.
    * **Load-Use Hazards:** Managed by a **Stalling Unit** that inserts bubbles into the pipeline.
* **Modular Design:** The RTL is organized into clean, stage-specific modules for better readability and maintainability.  

## **Contents**
1. [Tools Setup](#1-tools-setup)
2. [RISC-V Processor Overview](#2-risc-v-processor-overview)
3. [VLSI Design Flow](#3-vlsi-design-flow)  
   - [3.1 RTL Design & Simulation](#31-rtl-design--simulation)  
   - [3.2 Synthesis & Netlist Generation](#32-synthesis--netlist-generation)  
   - [3.3 Placement](#33-placement)  
   - [3.4 Static Timing Analysis](#34-static-timing-analysis)
   - [3.5 Routing](#35-routing)
   - [3.6 Post-route STA](#36-post-route-sta)
   - [3.7 Migration](#37-migration)
   - [3.8 LVS & DRC](#38-lvs--drc)
   - [3.9 GDS Generation](#39-gds-generation)
4. [Results & Discussion](#4-results--discussion)  

## **1. Tools Setup**  

### **1.1 VS Code**
**VS Code** is a **lightweight and powerful code editor** used for writing, debugging, and managing Verilog and VLSI design files.  
It offers **extensions for syntax highlighting, simulation, and debugging** to enhance the HDL design workflow.

### **Installation Steps**
```bash
sudo apt update
sudo apt install code
```

### **1.2 Icarus Verilog (Iverilog)**
**Icarus Verilog (Iverilog)** is an **open-source Verilog simulator** used for compiling and simulating digital designs.  
It is widely used in **VLSI design, FPGA development, and digital logic verification**.

### **Installation Steps**
```bash
git clone https://github.com/steveicarus/iverilog.git
cd iverilog
sh autoconf.sh
./configure
make
sudo make install
```

### **1.3 GTKWave**
**GTKWave** is an **open-source waveform viewer** used for analyzing **digital simulation outputs** from Verilog and VHDL simulations.  
It supports **VCD (Value Change Dump), LXT, LXT2, FST**, and other common waveform formats.

### **Installation Steps**
```bash
git clone https://github.com/gtkwave/gtkwave.git
cd gtkwave
./configure
make
sudo make install
```

### **1.4 Qflow toolchain**
**Qflow** is an **open-source framework for Physical Design**.  
It is widely used for **logic synthesis, verification, layout design, timing analysis, etc**.
It comprises of various tools: **Yosys, Graywolf, Vesta, Qrouter, Magic, Netgen and Klayout**.

### **Installation Steps**
```bash
git clone https://github.com/kunalg123/vsdflow.git
cd vsdflow
chmod 777 opensource_eda_tool_install.sh
./opensource_eda_tool_install.sh
```
---

## **2. RISC-V Processor Overview**

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

Below is a block diagram of a single-cycle RISC-V processor:
<img src="https://github.com/Shubham210204/RISC-V-RV32I-Processor-Design-Implementation/blob/main/images/architecture.png" width=900>

* **Program Counter (PC):** Tracks instruction execution sequence
* **Instruction Memory:** Stores program instructions
* **Register File:** Provides 32 general-purpose registers
* **ALU (Arithmetic Logic Unit):** Executes arithmetic and logical operations
* **Control Unit:** Decodes instructions and generates control signals
* **Immediate Generator:** Extracts and formats immediate values
* **Data Memory:** Handles load and store instructions
* **ALU Control:** Determines ALU operation based on opcode and function codes

### RISC-V 5 Execution Stages

While the block diagram illustrates the components of a **single-cycle processor**, practical RISC-V implementations adopt a **5-stage pipelined architecture** to achieve higher throughput. Each instruction progresses through Instruction Fetch, Decode, Execute, Memory Access, and Write-Back.

<img src="https://github.com/Shubham210204/RISC-V-RV32I-Processor-Design-Implementation/blob/main/images/pipeline.png" width=900>

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

### Hazards in a Pipelined RISC-V Processor

In a pipelined architecture, multiple instructions overlap in execution. While this improves throughput, it also introduces **hazards**—situations that may cause incorrect execution or require stalling. The main types of hazards are:

#### 1. **Data Hazards**

* Occur when an instruction depends on the result of a previous instruction that has not yet completed.

**Example:**

```assembly
ADD x3, x1, x2   # produces result in x3
SUB x4, x3, x5   # needs value of x3 immediately
```

* **Problem:** The second instruction requires the updated value of `x3`, but it is not yet written back.
* **Solutions:** Forwarding (data bypassing) or inserting stalls (pipeline bubbles).

#### 2. **Control Hazards**

* Arise from branch and jump instructions, where the next instruction’s address depends on the outcome of a condition.

**Example:**

```assembly
BEQ x1, x2, LABEL   # branch decision depends on comparison
ADD x3, x4, x5      # this instruction may be wrongly fetched
```

* **Problem:** The processor may fetch the wrong instruction if the branch outcome is not yet known.
* **Solutions:** Branch prediction, delayed branching, or flushing incorrect instructions.

#### 3. **Structural Hazards**

* Happen when two instructions require the same hardware resource in the same cycle.

**Example:**

* If a single memory unit is shared for **both instruction fetch (IF)** and **data load/store (MEM)**:

  ```assembly
  LW  x3, 0(x1)   # needs memory access in MEM stage
  ADD x4, x5, x6  # next instruction also needs memory access for fetch
  ```
* **Problem:** Both instructions compete for memory at the same cycle.
* **Solutions:** Use separate instruction and data memories (Harvard architecture) or multi-ported hardware.

### Base ISA Instructions (RV32I – 39 Instructions)

The RV32I base instruction set contains 39 fundamental instructions, grouped into different types based on their format and purpose.
<img src="https://github.com/Shubham210204/RISC-V-RV32I-Processor-Design-Implementation/blob/main/images/instruction_format.png">

#### 1. R-Type Instructions (Register-Register Operations)

* Format: `funct7 | rs2 | rs1 | funct3 | rd | opcode`
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

* Format: `imm[11:0] | rs1 | funct3 | rd | opcode`
* Used for immediate arithmetic and memory load.
* Examples:

  * `ADDI rd, rs1, imm` → rd = rs1 + imm
  * `ANDI rd, rs1, imm` → rd = rs1 & imm
  * `ORI rd, rs1, imm` → rd = rs1 | imm
  * `XORI rd, rs1, imm` → rd = rs1 ^ imm
  * `SLTI rd, rs1, imm` → Set if less than (signed)
  * `LW rd, offset(rs1)` → Load Word from memory

#### 3. S-Type Instructions (Store Operations)

* Format: `imm[11:5] | rs2 | rs1 | funct3 | imm[4:0] | opcode`
* Used for storing data from register to memory.
* Examples:

  * `SW rs2, offset(rs1)` → Store Word
  * `SH rs2, offset(rs1)` → Store Halfword
  * `SB rs2, offset(rs1)` → Store Byte

#### 4. B-Type Instructions (Branch Operations)

* Format: `imm[12|10:5] | rs2 | rs1 | funct3 | imm[4:1|11] | opcode`
* Used for conditional branching.
* Examples:

  * `BEQ rs1, rs2, offset` → Branch if Equal
  * `BNE rs1, rs2, offset` → Branch if Not Equal
  * `BLT rs1, rs2, offset` → Branch if Less Than (signed)
  * `BGE rs1, rs2, offset` → Branch if Greater or Equal (signed)
  * `BLTU rs1, rs2, offset` → Branch if Less Than (unsigned)
  * `BGEU rs1, rs2, offset` → Branch if Greater or Equal (unsigned)

#### 5. U-Type Instructions (Upper Immediate)

* Format: `imm[31:12] | rd | opcode`
* Used for loading large immediate values.
* Examples:

  * `LUI rd, imm` → Load Upper Immediate
  * `AUIPC rd, imm` → Add Upper Immediate to PC

#### 6. J-Type Instructions (Jump Operations)

* Format: `imm[20|10:1|11|19:12]| rd | opcode`
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

## **3. VLSI Design Flow**

### **3.1 RTL Design & Simulation**

### **3.2 Synthesis & Netlist Generation**

### **3.3 Placement**

### **3.4 Static Timing Analysis**

### **3.5 Routing**

### **3.6 Post-route STA**

### **3.7 Migration**

### **3.8 LVS & DRC**

### **3.9 GDS Generation**

---

## **4. Results & Discussion**

* Completed end-to-end flow from RTL to ASIC Layout to FPGA Implementation to Calculator Demo
* Gained experience with open-source VLSI EDA tools and FPGA prototyping
* Developed understanding of RISC-V ISA, digital design, and physical design flows
* Demonstrated a working RISC-V powered Calculator


