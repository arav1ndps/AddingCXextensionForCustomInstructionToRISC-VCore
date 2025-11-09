# Design Overview — Composable Extension (CX) Integration in MicroBlaze-V

This document explains the architectural design of the **Composable Extension (CX)** framework implemented in the **MicroBlaze-V RISC-V soft-core**.  
It provides a detailed description of each hardware component, its purpose, and its interaction within the system.  

## 1 - System Architecture

The CX framework extends the MicroBlaze-V processor by introducing a **Composable Extension Interface** that allows the integration of multiple hardware accelerators.  
It enables the CPU to issue **custom instructions** through the CX interface, which are routed to the appropriate **Composable Extension Unit (CXU)** via a shared **AXI4-Stream interconnect**.

🖼️ *Figure 1: Top-Level CX Integration Architecture*  
![CX System Architecture](./figures/cx_system_architecture.png)

**Main design modules:**
- CX Interface (integrated into MicroBlaze-V)
- CX Interconnect
- Wrapper modules for accelerators
- Hardware accelerators (FFT, CORDIC)
- CSR extensions for CX configuration and status

---

## 2 - CX Interface

**Location:** `microblaze/CX_interface/`
![cx_interface_flow](./figures/cx_interface_flow.PNG)
The CX interface forms the **communication bridge** between the MicroBlaze-V pipeline and the external accelerator system.

### **Functionality**
- Receives **custom instruction opcodes** decoded from the MicroBlaze-V instruction stream.
- Extracts the **CF_ID (Custom Function ID)** and **CXU_ID (Custom Execution Unit ID)** fields.
- Sends these fields, along with operand data, to the interconnect using the **AXI4-Stream (M_AXIS)** interface.
- Waits for the accelerator response via **S_AXIS** interface.
- Handles **pipeline stalling** and **data forwarding** during execution.

**Signals:**
| Signal | Direction | Description |
|---------|------------|-------------|
| `CX_M_AXIS_*` | Output | Request channel to the interconnect |
| `CX_S_AXIS_*` | Input | Response channel from the interconnect |
| `CX_STALL` | Output | Indicates instruction stall to MicroBlaze-V pipeline |
| `CX_DONE` | Input | Accelerator operation complete |
| `CX_ERROR` | Input | Error flag from CXU |

---

## 🧩 3️⃣ CX Interconnect

**Location:** `interconnect/`

The interconnect is an **AXI4-Stream-based routing unit** that connects the CX interface to multiple accelerator wrappers.

### **Functionality**
- Routes CX requests to the correct accelerator based on the **CXU_ID**.
- Performs **round-robin arbitration** when multiple accelerators request access.
- Supports **scalable connections** (parameterized for number of accelerators).
- Handles **AXI4-Stream handshaking** between source and destination.
- Ensures deterministic latency by maintaining valid/ready synchronization.

🖼️ *Figure 2: CX Interconnect Architecture*  
![CX Interconnect Diagram](./figures/cx_interconnect.png)

---

## ⚙️ 4️⃣ Wrapper Modules

**Location:** `wrapper/`

Each accelerator is enclosed in a **wrapper module** that adapts the accelerator’s native interface to the CX framework.

### **Functionality**
- Acts as a protocol converter between **AXI4-Stream** and **accelerator-specific I/O**.
- Manages signal packing/unpacking of `CF_ID`, data operands, and control words.
- Synchronizes the accelerator operation with `TVALID`, `TREADY`, and `TLAST`.
- Optionally contains configuration registers for runtime parameterization.

🖼️ *Figure 3: Wrapper Module Interface*  
![CX Wrapper Diagram](./figures/cx_wrapper.png)

---

## 🧮 5️⃣ Accelerator Modules

**Location:** `accelerators/`

Two existing Xilinx IP cores were integrated as accelerators:
- **FFT Core** — computes Fast Fourier Transform.
- **CORDIC Core** — performs trigonometric and vector operations.

### **Integration Process**
- Instantiated directly from Vivado IP catalog.
- Wrapped using CX-compliant interfaces for operand and control data.
- Configured with bit-width and latency parameters to match MicroBlaze-V datapath.

**Key Behavior:**
- Accelerator operation begins when CX instruction is issued.
- Results are sent back through the S_AXIS response channel.
- Latency determined by the internal pipeline of each accelerator.

🖼️ *Figure 4: Accelerator Integration Example*  
![CX Accelerator Integration](./figures/cx_accelerator.png)

---

## 🧱 6️⃣ MicroBlaze-V Pipeline Modifications

**Location:** `microblaze/pipeline/`

The **decode and execute stages** of the MicroBlaze-V pipeline were modified to recognize and handle CX instructions.

### **Changes made**
- Added a **CX decode unit** to detect custom instruction opcodes.
- Introduced a **stall mechanism** during accelerator execution.
- Updated **CSR register set** (`mcx_selector`, `cx_status`) for tracking accelerator state.
- Integrated feedback logic to resume execution after accelerator completion.

🖼️ *Figure 5: MicroBlaze-V CX Pipeline Extension*  
![CX Pipeline Diagram](./figures/cx_pipeline.png)

---

## 🧰 7️⃣ Control and Status Registers (CSR)

New CSRs were introduced to manage and monitor CX operations:

| CSR Name | Description |
|-----------|--------------|
| `mcx_selector` | Selects current CXU and operation context |
| `cx_status` | Holds status flags and error codes |
| `mcx_table` | Stores CXU configuration metadata |
| `cx_index` | Tracks accelerator instruction index |

These CSRs provide the software with direct access to the hardware extension status through Vitis runtime.

---

## 🔍 8️⃣ Verification Environment

**Location:** `verification/`

A **top-level testbench** was developed to verify the integration of:
- CX interface
- Interconnect
- Wrapper modules
- Accelerators

### **Features**
- Testbench emulates software instruction flow.
- Test vectors generated using Python (`verification/scripts/`).
- Self-checking system compares accelerator results with golden reference from IP testbench.
- Assertions used to validate AXI protocol correctness.

🖼️ *Figure 6: CX Verification Setup*  
![CX Verification Diagram](./figures/cx_verification.png)

---

## 🧪 9️⃣ Integration in Vivado and Vitis

- Hardware synthesized and implemented in **Vivado**.  
- Exported `.xsa` file imported into **Vitis** for C software development.  
- ELF binaries generated to test CX instructions through C macros using GCC inline assembly.

🖼️ *Figure 7: Vivado–Vitis Integration Flow*  
![Vivado-Vitis Flow Diagram](./figures/vivado_vitis_flow.png)

---

## ✅ Summary

The design successfully extends MicroBlaze-V with a **scalable CX interface** that allows seamless integration of multiple accelerators.  
Through AXI4-Stream interconnect and standardized CSR mapping, the framework achieves:
- Modularity  
- Scalability  
- Low latency communication between CPU and accelerators  

This framework serves as a foundation for **future RISC-V SoC designs** supporting composable hardware acceleration.

---

© 2025 — CX Integration for MicroBlaze-V | Chalmers University of Technology & AMD Collaboration
