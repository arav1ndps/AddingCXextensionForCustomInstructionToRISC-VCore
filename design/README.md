# Design Overview — Composable Extension (CX) Integration in MicroBlaze-V

This document explains the architectural design of the **Composable Extension (CX)** framework implemented in the **MicroBlaze-V RISC-V soft-core**.  
It provides a detailed description of each hardware component, its purpose, and its interaction within the system.  

## 1 - System Architecture

The CX framework extends the MicroBlaze-V processor by introducing a **Composable Extension Interface** that allows the integration of multiple hardware accelerators.  
It enables the CPU to issue **custom instructions** through the CX interface, which are routed to the appropriate **Composable Extension Unit (CXU)** via a shared **AXI4-Stream interconnect**.

Top-Level CX Integration Architecture*  
![CX System Architecture](./figures/cx_system_architecture.png)

**Main design modules:**
- CX Interface (integrated into MicroBlaze-V)
- CX Interconnect
- Wrapper modules for accelerators
- Hardware accelerators (FFT, CORDIC)
- CSR extensions for CX configuration and status

---

## 2 - CX Interface
The CX interface designed in MicroBlaze-V handles the execution of custom instructions through a series of the following operations
*CX interface functions*
![cx_interface_flow](./figures/cx_interface_flow.png)

### **Functionality**
- Version Check: Ensures the MicroBlaze-V version matches the version of the CX extension
- Data Ending: Encodes CXU_ID, CF_ID, and operand values for the AXI master payload.
- Transmission of AXI-Stream, stall the pipeline in case of write-back, else continue the pipeline 
- Receives the response and decodes the result and status of execution.
- Updates the registers and the CSRs

---

## - CX Interconnect

**Location:** `interconnect/`

The interconnect has been designed to act as a bridge between the MicroBlaze-V core and the accelerators. Interconnect has two phases of execution: the write phase and the read phase.

### - Write Phase
The write phase of the interconnect contains the broadcaster, error handling unit, and CXU Error accumulator
- Broadcaster: Manages the transfer of AXI-stream from MicroBlaze-V to the accelerators
- Error handling unit: Manages the CXU error in case if there has been a mismatch in the CXU identifiers or if the accelerator is missing

*CX Interconnect Write Phase*  
![CX Interconnect Write_Phase](./figures/cx_interconnect_write_phase.png)

### - Read Phase
The Read phase of the interconnect contains the mixer, evaluator, and error handling unit.
- Mixer: Manages the AXI stream containing the result and the status to be updated in the CSR (data path).
- Evaluator: Manages Write-back to the processor (control path).
  
*CX Interconnect Read Phase*  
![CX Interconnect Read_Phase](./figures/cx_interconnect_read_phase.png)

---

### - IP core configuration

Two existing Xilinx IP cores were integrated as accelerators:
- **FFT Core** — computes Fast Fourier Transform.
- **CORDIC Core** — performs trigonometric and vector operations.
  
> **Note:**  
> In our system, IP cores function as independent accelerator units, each assigned a unique identifier. The execution of every IP core is controlled through distinct instruction formats.

### **CORDIC**
 The CORDIC IP core can be used to implement many different mathematical func
tions, including trigonometric functions, square root, hyperbolic and also rectangular
polar conversions. The respective function can be configured in the IP core and the
 operands can be selected accordingly, whether it is a phase or Cartesian operand.
The detailed information can be found in AMD's official documentation: 
 
*CORDIC PIN diagram*  
![CORDIC pin Diagram](./figures/cordic_pin.png)


#### CORDIC IP Core Configuration Comparison

| Parameter                             | Vector Configuration | Trigonometric Configuration |
|--------------------------------------|-----------------------|-----------------------------|
| Function                              | Vector                | Trigonometric               |
| Input Width                           | 16 bits               | 16 bits                     |
| Phase Width                           | 16 bits               | 16 bits                     |
| Output Width                          | 16 bits               | 16 bits                     |
| Data Format                           | Signed Fractional     | Signed Fractional           |
| Rounding Mode                         | Truncate              | Truncate                    |
| Iterations                            | 16                    | 16                          |
| Pipelining                            | Maximum               | Maximum                     |
| Scale Compensation                    | Enabled               | Enabled                     |
| Angle Format                          | Radians               | Radians                     |
| Cordic Function Type                  | Vector                | Rotation                    |
| Implementation Architecture            | Parallel              | Parallel                    |
| Phase Quantization                    | Enabled               | Enabled                     |
| Latency                               | 16 cycles             | 16 cycles                   |

### **FFT**

The FFT IP core was configured to have independent stages to configure, write, and read.
Each stage will be managed by unique instruction formats with the same CXU ID.

#### FFT IP Core Configuration:

| Parameter                              | FFT Configuration Value     |
|---------------------------------------|-----------------------------|
| Function                              | FFT                         |
| Architecture Type                     | Burst I/O                   |
| Implementation Architecture            | Radix-2                     |
| Transform Length                      | 256                         |
| Number of Stages                      | 8                           |
| Input Width                           | 16 bits                     |
| Output Width                          | 16 bits                     |
| Phase Factor Width                    | 16 bits                     |
| Data Format                           | Signed Fractional           |
| Scaling Option                        | Scaled                      |
| Rounding Mode                         | Truncate                    |
| Butterfly Type                        | Radix-2                     |
| Scale Compensation                    | Enabled                     |
| Phase Quantization                    | Enabled                     |
| Latency                               | Dependent on 8 pipeline stages |
| Throughput                            | Moderate                    |
| Resource Utilization                  | Medium                      |

#### FFT Timing Diagram:
![FFT Timing Diagram](./figures/fft_timing.png)

## IP core wrapper

 ### **Wrapper Module Functions**

- Manages the execution of custom instructions for each IP core.  
- Samples and verifies incoming data (`tvalid`, `tdata`) for integrity.  
- Validates `CXU_ID` and determines **CXU hit** or **mismatch**.  
- Forwards parameters to the IP core after **CF_ID** and **operand** checks.  
- Handles errors and sends appropriate responses to **MicroBlaze**.  
- Waits for the IP core to complete computation and encodes the result with status.  
- Connects IP cores via **AXI master/slave interfaces** to manage handshake and data signals.  
- Supports **write-back** operations for instructions expecting a result.  
- Skips result write-back for instructions not requiring output.  
- Accumulates and reports **CF** and **operand error** statuses during read operations.  

![Wrapper Block Diagram](./figures/wrapper_block.png)

### **CORDIC IP Wrapper Functions**
- Executes **read instructions** from MicroBlaze.  
- Waits for the IP core to complete computation.  
- Collects and returns the computed result.  

---

### **FFT IP Wrapper Functions**
- Manages three distinct phases of FFT operation.  
- Includes **additional control logic** to handle these phases efficiently.  
- Performs extra functions to coordinate data flow and synchronization between phases.  
- Ensures correct sequencing of input, computation, and output stages.  

![FFT Wrapper Diagram](./figures/fft_wrapper.png)

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
