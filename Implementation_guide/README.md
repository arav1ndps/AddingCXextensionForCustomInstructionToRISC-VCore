# 🧠 CX Integration for MicroBlaze-V — Complete Workflow

> **Note:** This repository does **not** contain AMD proprietary source code. It documents the entire **workflow**, **methodology**, and **integration process** for the Composable Extension (CX) implemented on the MicroBlaze-V RISC-V core using FPGA.  
> The repository includes verification testbenches, software macros, build artifacts, and scripts to reproduce the design flow — from **C code to Vitis**, **Vivado system integration**, and **bitstream generation**.

---

## 🧩 1️⃣ Project Overview

This project presents the **design and implementation of a Composable Extension (CX)** for the **MicroBlaze-V** core to enable custom instruction acceleration using hardware IP cores.  
The CX interface allows the seamless integration of accelerators such as **FFT** and **CORDIC**, significantly improving performance compared to pure software execution.

All integration and verification were done using **Xilinx Vivado** for hardware design and **Vitis IDE** for software compilation and deployment.

---

## ⚙️ 2️⃣ Step-by-Step Project Workflow

### 2.1 — Configure Accelerator IP Cores

**Folder:** [`IP_configuration/`](./IP_configuration/)  
**Tool:** Xilinx Vivado

- Select FFT or CORDIC IPs from the Vivado IP catalog.  
- Configure data widths, streaming modes, and parameters.  
- Export IP configuration files (`.xci` / `.tcl`) to this folder.

Output: Synthesizable IP cores with configuration metadata.

---

### 2.2 — Create CX Wrappers for Accelerators

**Folder:** [`wrapper/`](./wrapper/)  
**Purpose:** Interface accelerator IPs with the CX interconnect.

Each wrapper translates AXI-Stream signals to accelerator-specific I/O.  
It includes:
- `TVALID`, `TREADY`, and `TDATA` handshake management.  
- Packing and unpacking of CF_ID and CXU_ID fields.  
- Optional parameter registers for configuration.

---

### 2.3 — Implement the CX Interface in MicroBlaze-V

**Folder:** [`microblaze/CX_interface/`](./microblaze/CX_interface/)  
The CX interface connects the MicroBlaze-V execution pipeline to external accelerators.

**Tasks:**
- Integrate the CX instruction decode logic.  
- Manage pipeline stalls and handshakes.  
- Extend CSR registers for status and control.  
- Handle request (M_AXIS) and response (S_AXIS) channels.

---

### 2.4 — Design the AXI Interconnect

**Folder:** [`interconnect/`](./interconnect/)  
Implements the interconnection network between the CX interface and accelerators.

Features:
- AXI-Stream-based routing (round-robin or priority).  
- Arbitration logic for multi-accelerator systems.  
- Error/status propagation to the CX interface.

---

### 2.5 — Verification Workflow

**Folder:** [`verification/`](./verification/)

A **top-level testbench** verifies the CX interface, interconnect, and accelerators.  
The verification mimics software instruction behavior using **Python-generated test vectors**.

**Verification process:**
- Testbench imitates software issuing CX instructions.  
- Python script generates instruction streams (CXU_ID + CF_ID + data).  
- Self-checking system compares accelerator outputs with reference data from IP libraries.

**Results:**
- Functional equivalence between hardware and reference verified.  
- Assertions validated AXI-Stream handshakes and pipeline stalls.

---

### 2.6 — MicroBlaze-V Pipeline Modifications

**Folder:** [`microblaze/pipeline/`](./microblaze/pipeline/)

Pipeline stages updated to support CX instruction execution:
- Added EX_CX_Instr and EX_CX_Stall signals.  
- Controlled pipeline stalls while accelerators process data.  
- Updated instruction decode and CSR logic.

---

### 2.7 — System Integration in Vivado

**Folder:** [`vivado/`](./vivado/)

Integrate all hardware components:
- MicroBlaze-V core with CX interface.  
- AXI Interconnect.  
- Wrappers and accelerator IPs.  
- Memory and peripheral interfaces.

**Vivado Steps:**
1. Launch Vivado → Create a new project.  
2. Import IPs, wrappers, and CX modules.  
3. Connect via AXI interfaces (Stream & Lite).  
4. Apply clock, reset, and memory constraints.  
5. Validate design and save the block diagram.

**Output:** Synthesized hardware platform.

---

### 2.8 — Export Hardware to Vitis

**Command:**
```bash
vivado -mode batch -source vivado/export_hw.tcl
```
This generates:
- `.xsa` — hardware definition for Vitis  
- `.bit` — bitstream for FPGA programming

Files saved to `vivado/hw/`.

---

### 2.9 — Vitis Software Integration and C Code Compilation

**Folder:** [`software/`](./software/)

1. Launch **Vitis IDE** → Create New Application Project.  
2. Import the `.xsa` from `vivado/hw/`.  
3. Add C source files from `software/examples/`.  
4. Include CX macros from `software/macros/`.

**Macros defined using GCC inline assembly:**
```c
#define CX_INSTR(opcode, rd, rs1, rs2, cfid, cxuid)     asm volatile (".word ((" #opcode " << 25) | (" #cxuid " << 20) | (" #cfid " << 15) | (" #rs1 " << 10) | (" #rs2 " << 5) | (" #rd "))")
```

Build in Release or Debug mode.  
Output ELF binaries: `build/elf/`

---

### 2.10 — Hardware–Software Linking and Bitstream Generation

**Tool:** Vivado

**Command:**
```bash
vivado -mode batch -source vivado/generate_bitstream.tcl
```

This performs:
- Synthesis and Implementation  
- Timing and Power Analysis  
- Bitstream (`.bit`) generation

**Result:** Bitstream available in `vivado/bitstream/`

---

### 2.11 — Programming the FPGA

**Option 1 — Using Vivado Hardware Manager:**
```bash
open_hw
connect_hw_server
open_hw_target
set_property PROGRAM.FILE {vivado/bitstream/system.bit} [current_hw_device]
program_hw_devices [current_hw_device]
```

**Option 2 — Using Vitis:**
- Launch on Hardware (Single Application Debug)  
- Select `.xsa` and `.elf` files to program the device

---

### 2.12 — Hardware Validation

After programming, verify CX instruction behavior via UART logs in Vitis Serial Terminal.  
Compare hardware accelerator results with software-computed references.

---

## 📦 3️⃣ Output Artifacts

| Artifact | Description | Location |
|-----------|-------------|-----------|
| `.xsa` | Hardware definition file | `vivado/hw/` |
| `.bit` | FPGA bitstream | `vivado/bitstream/` |
| `.elf` | Software binary | `build/elf/` |
| `.log` | Simulation and synthesis logs | `process/` |

---

## ✅ 4️⃣ Summary

This repository demonstrates the complete **hardware/software co-design flow** for a CX-extended MicroBlaze-V processor.  
It includes:
- Hardware design in **Vivado**  
- Software integration in **Vitis**  
- Verification using **assertion-based testbenches**  
- End-to-end FPGA deployment with generated **ELF** and **bitstream** files

---

© 2025 — CX Integration for MicroBlaze-V | Chalmers University of Technology & AMD Collaboration
