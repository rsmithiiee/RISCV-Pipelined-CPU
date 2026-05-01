# 5-Stage Pipelined RV32I CPU

A fully-featured, 5-stage pipelined RISC-V RV32I processor implemented in SystemVerilog. Built incrementally across five labs for CMPE 140 Computer Architecture at San Jose State University, the design progresses from a basic pipeline skeleton to a complete CPU supporting arithmetic, memory access, hazard resolution, and control flow.

Authors: Haydon Behl · Mihir Phadke · Ryan Smith

---

## Table of Contents

- [What It Does](#what-it-does)
- [Features](#features)
- [Architecture Overview](#architecture-overview)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Running the Simulation](#running-the-simulation)
- [Instruction Support](#instruction-support)
- [Lab Progression](#lab-progression)
- [Where to Get Help](#where-to-get-help)
- [Contributing](#contributing)

---

## What It Does

This project implements an RV32I-compatible CPU — the base 32-bit integer instruction set of the RISC-V open standard architecture. The processor:

- Executes instructions through a classic 5-stage pipeline: IF → ID → EX → MEM → WB
- Handles data hazards via forwarding (EX→EX and MEM→EX paths) and stall-based detection
- Resolves control hazards by flushing the pipeline on taken branches and jumps
- Supports byte-granular memory access for load/store operations
- Produces cycle-accurate trace files (`pc.txt`, `data.txt`) for post-simulation analysis

The `Processor/` directory contains the polished, unified implementation. The `CMPE140/` directory contains the five incremental lab snapshots showing how the design evolved.

---

## Features

- Full RV32I base integer instruction set
- 5-stage pipeline with IF/ID, ID/EX, EX/MEM, and MEM/WB registers
- Data forwarding (EX→EX, MEM→EX) to minimize pipeline stalls
- Load-use hazard detection and stall insertion
- Branch resolution in the EX stage with pipeline flushing (BEQ, BNE, BLT, BGE, BLTU, BGEU)
- Unconditional jumps: JAL and JALR
- Sign- and zero-extended loads: LB, LH, LBU, LHU, LW
- Byte-granular stores: SB, SH, SW
- LUI and AUIPC (U-type instructions)
- Separate instruction ROM and data RAM with configurable init files
- Cycle counter and file-based trace output for debugging

---

## Architecture Overview

```
        ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐   ┌─────┐
Fetch → │IF/ID│ → │ID/EX│ → │EX/MEM│→ │MEM/WB│→  │ WB  │
        └─────┘   └─────┘   └─────┘   └─────┘   └─────┘
                     ↑  Forwarding paths  ↑
                  EX→EX              MEM→EX

Branch/Jump decision made in EX stage → flushes IF/ID and ID/EX
```

Key Modules (in `Processor/`):

| File | Description |
|---|---|
| `cpu.sv` | Top-level pipeline, hazard logic, forwarding, control |
| `instruction_decoder.sv` | Decodes all RV32I instruction formats (R/I/S/B/U/J) |
| `ALU.sv` | Arithmetic/logic unit for all RV32I ALU operations |
| `PC.sv` | Program counter with stall and branch/jump override |
| `register_file.sv` | 32×32 general-purpose register file |
| `rom.sv` | Instruction memory (initialized from `.dat` file) |
| `ram.sv` | Data memory with byte-enable write support |
| `tb.sv` | Testbench connecting CPU, ROM, and RAM |

---

## Repository Structure

```
RISCV-Proc-main/
├── Processor/               # Final unified implementation
│   ├── cpu.sv               # Top-level CPU
│   ├── instruction_decoder.sv
│   ├── ALU.sv
│   ├── PC.sv
│   ├── register_file.sv
│   ├── rom.sv
│   ├── ram.sv
│   ├── tb.sv                # Testbench
│   ├── line.asm             # Sample RISC-V assembly program
│   ├── line.dat             # Instruction ROM image (hex)
│   └── dmem.dat             # Initial data RAM image (hex)
│
└── CMPE140/                 # Incremental lab snapshots
    ├── Lab-3/               # Basic pipeline + stall-based RAW hazard detection
    ├── Lab-4/               # R- and I-type ALU + forwarding
    ├── Lab-5/               # Load/store + data memory + forwarding
    ├── Lab-6/               # Sign/zero-extended loads + load-use hazards
    └── Lab-7/               # Branch + jump + pipeline flushing (complete)
```

Each lab directory contains its SystemVerilog sources and a PDF design report.

---

## Getting Started

### Requirements

- A SystemVerilog simulator. Any of the following work:
  - Vivado Simulator (Xilinx) — recommended for waveform debugging
  - ModelSim / Questa (Mentor/Siemens)
  - Icarus Verilog (`iverilog`) — free and open source
  - VCS (Synopsys)

No special libraries or dependencies are required beyond a standard SV simulator.

### Clone the Repository

```bash
git clone https://github.com/<your-username>/RISCV-Proc.git
cd RISCV-Proc/Processor
```

---

## Running the Simulation

### With Icarus Verilog (iverilog)

```bash
cd Processor/

# Compile all source files and the testbench
iverilog -g2012 -o sim \
  tb.sv cpu.sv instruction_decoder.sv ALU.sv \
  PC.sv register_file.sv rom.sv ram.sv

# Run the simulation
vvp sim
```

After the run completes, two trace files are generated:

| File | Contents |
|---|---|
| `pc.txt` | Program counter value at each clock cycle |
| `data.txt` | Write-back result and destination register per cycle |

### With Vivado

1. Create a new RTL project and add all `.sv` files from `Processor/` as sources.
2. Set `tb.sv` as the simulation top.
3. In simulation settings, set the working directory to `Processor/` so the `.dat` init files are found.
4. Run Behavioral Simulation.

### Using a Different Program

To simulate a custom program, replace `line.dat` with your own instruction ROM image (one 32-bit hex word per line) and update `dmem.dat` as needed. The testbench picks up the filenames from the `rom` and `ram` instantiation parameters in `tb.sv`:

```systemverilog
rom #(.init_file("line.dat")) imem (...);
ram #(.init_file("dmem.dat")) dmem (...);
```

The included `line.asm` demonstrates a software integer multiply routine using only RV32I instructions (no `MUL` extension required).

---

## Instruction Support

### R-Type
`ADD`, `SUB`, `SLL`, `SLT`, `SLTU`, `XOR`, `SRL`, `SRA`, `OR`, `AND`

### I-Type (Arithmetic)
`ADDI`, `SLTI`, `SLTIU`, `XORI`, `ORI`, `ANDI`, `SLLI`, `SRLI`, `SRAI`

### I-Type (Loads)
`LB`, `LH`, `LW`, `LBU`, `LHU`

### S-Type (Stores)
`SB`, `SH`, `SW`

### B-Type (Branches)
`BEQ`, `BNE`, `BLT`, `BGE`, `BLTU`, `BGEU`

### U-Type
`LUI`, `AUIPC`

### J-Type (Jumps)
`JAL`, `JALR`

---

## Lab Progression

The `CMPE140/` directory documents the complete design evolution. Each lab builds on the previous:

| Lab | Key Additions | Report |
|---|---|---|
| Lab 3 | Basic 5-stage skeleton, IF/ID/EX/MEM/WB registers, stall-based RAW hazard detection | `Lab-3/CMPE140_lab3_report.pdf` |
| Lab 4 | Full R- and I-type ALU, forwarding to reduce stalls | `Lab-4/CMPE140_lab4_report.pdf` |
| Lab 5 | Load/store (S-type, I-type memory), data memory interface, byte-enable writes, two-operand forwarding | `Lab-5/CMPE140_lab5_report.pdf` |
| Lab 6 | Sign/zero-extended loads (LB/LH/LBU/LHU), load-use hazard fix | `Lab-6/CMPE140_lab6_report.pdf` |
| Lab 7 | All branch types (BEQ–BGEU), JAL/JALR, pipeline flushing, complete control flow | `Lab-7/CMPE140_lab7_report.pdf` |

The PDF reports cover design rationale, implementation details, and cycle-accurate waveform/trace analyses.

---

## Where to Get Help

- Lab reports — each `CMPE140/Lab-N/` directory contains a detailed PDF explaining design decisions, timing diagrams, and test results for that stage of development.
- RISC-V ISA Specification — [riscv.org/technical/specifications](https://riscv.org/technical/specifications/) — the authoritative reference for RV32I encoding and semantics.
- RARS (RISC-V Assembler and Runtime Simulator) — [github.com/TheThirdOne/rars](https://github.com/TheThirdOne/rars) — useful for writing and testing assembly programs before simulating in RTL.
- Vivado documentation — available through the Xilinx/AMD support portal.

---

## Contributing

This project was developed as academic coursework. Please respect academic integrity policies at your institution if you are a student using this project as a reference.
