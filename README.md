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

The processor implements the RV32I base instruction set, including:

* Arithmetic and Logical Instructions
* Load and Store Instructions
* Control Flow Instructions
* Immediate Handling

Key modules include:

* Program Counter (PC)
* Instruction Memory
* Register File
* ALU
* Control Unit
* Immediate Generator
* Data Memory
* ALU Control

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


