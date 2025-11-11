# CX extension Workflow

> **Important note:** This repository does **not** contain AMD proprietary source code. It documents the full **methodology**, configuration files, scripts, IP wrappers, testbenches, and build artifacts (ELF files) used to integrate a **Composable Extension (CX)** into the **MicroBlaze-V** RISC-V core. The IP sources and MicroBlaze-V core modifications are AMD/third-party intellectual property and are therefore **not included** here. Where required, pointers to repository folders and configuration artifacts are provided.

---
<!--
## Repository layout (relevant folders)

```
<repo-root>/
├── IP_configuration/          
├── wrapper/                   
├── microblaze/
│   ├── CX_interface/          
│   └── pipeline/              
├── interconnect/              
├── vivado/                    
├── verification/
│   ├── top/                   
│   └── assertions/            
├── software/
│   ├── macros/                
│   ├── inline_asm/            
│   └── examples/              
├── build/
│   └── elf/                   
├── process/                   
└── README.md                  
```

> If your repository structure differs, adapt the folder names above to match your tree.
-->
---
## 1 - Define the specifications
The work follows specifications for the execution of the Custom Instruction defined by the RISC-V community. As the specifications were still being updated during this thesis period, we have made slight modifications to the spec documents.  

## 2 - Design

CX extension was demonstrated using CORDIC and FFT IP cores from Xilinx's Library. These IP cores were configured to manage the instruction-based data flow. In order to manage the data and verify the integrity, CX-compliant wrapper modules were defined.

The Advanced eXtensible Interface (AXI) Stream protocol has been implemented for the high-speed and efficient communication of data between the processor and accelerators.![AXI protocol]() 

Added an interface and the necessary modifications to MicroBlaze-V to support CX extension. These modifications include CSR definitions and pipeline changes. 

The inteconnect module enables the connection of multiple accelerators to the MicroBlaze-V.

For detailed information about the design: [Design](https://github.com/arav1ndps/AddingCXextensionForCustomInstructionToRISC-VCore/tree/main/Derification)

---

## 3 - Top-Level Testbench Integration

The overall testbench structure and simulation setup are provided in the [verification](https://github.com/arav1ndps/AddingCXextensionForCustomInstructionToRISC-VCore/tree/main/Verification) folder.  
It includes stimulus generation, DUT integration, and self-checking result comparison.

---

## 4 - MicroBlaze-V Modifications

Implemented modifications to perform the following functions in MicroBlaze-V:<br>
- CSR definitions (CX_status, CX_mcx_selector)
- Instruction decode
- Enable CX execution
- Pipeline stall
- Result write-back

---

## 5 — Software Library definitions:

The custom CX instructions used in this project were implemented using **GCC inline assembly macros**.  
These macros provide a software abstraction layer that allows C programs to invoke the CX-based hardware accelerators directly through instruction-level encoding.<br>

The macros serve two main purposes:
- To **encode custom instructions** following the CX specification format.  
- To provide a **high-level software interface** that allows programmers to execute hardware accelerators from C without modifying compiler internals.

## 6 — System Integration in Vivado

Build the complete system using the MicroBlaze support in Vivado, following scripts in the [vivado](./vivado/) folder.  
This includes resets, clock generation, and memory configuration files required to implement the final system.

---

## 7 — Software and Vitis Integration

Export the hardware platform to Vitis and integrate the software layer from [software](./software/).  
Define instruction macros, add CX instruction support, and compile C applications that communicate with hardware accelerators.  
The ELF generation process is documented in the [process](./process/) folder.

---

© 2025 — CX Integration for MicroBlaze-V | Chalmers University of Technology & AMD Collaboration
