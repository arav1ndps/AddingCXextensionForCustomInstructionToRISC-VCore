# CX extension Workflow

> **Important note:** This repository does **not** contain AMD proprietary source code. It documents the full **methodology**, configuration files, scripts, IP wrappers, testbenches, and build artifacts (ELF files) used to integrate a **Composable Extension (CX)** into the **MicroBlaze-V** RISC-V core. The IP sources and MicroBlaze-V core modifications are AMD/third-party intellectual property and are therefore **not included** here. Where required, pointers to repository folders and configuration artifacts are provided.

---

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

---
## 1 - Define the specifications
The work follows specifications for the execution of the Custom Instruction defined by the RISC-V community. As the specifications were still being updated during this thesis period, we have made slight modifications to the spec documents.  

## 2 — Design
CX extension was demonstrated using CORDIC and FFT IP cores from Xilinx's Library. The configurations for the IP cores can be found in (./IP_configuration/) folder.

 The Advanced eXtensible Interface (AXI) Stream protocol has been implemented for the high-speed and efficient communication of data between the processor and accelerators. 

CX-compliant wrappers for each IP core to perform CX checks and translation logic. The[wrapper](./wrapper/) folder

Add the CX front-end and signal management as described in [microblaze/CX_interface](./microblaze/CX_interface/).  
This module is responsible for the execution of a custom instruction 

Inorder to connect multiple accelerators to the MicroBlaze, Interconnect module has been defined


## 3 — Verification using Assertion-Based Testbench

Use the top-level testbench in [verification/top](./verification/top/) with assertions defined in [verification/assertions](./verification/assertions/).  
This validates handshake protocols, CSR updates, and accelerator responses using assertion-based verification techniques.

---

## 7 — Top-Level Testbench Integration

The overall testbench structure and simulation setup are provided in the [verification/top](./verification/top/) folder.  
It includes stimulus generation, DUT integration, and self-checking result comparison.

---

## 8 — MicroBlaze-V Pipeline Modifications

Implement the instruction decode and pipeline stalling logic as described in [microblaze/pipeline](./microblaze/pipeline/).  
These changes ensure that the CX interface can safely pause instruction flow during accelerator execution.

---

## 9 — System Integration in Vivado

Build the complete system using the MicroBlaze support in Vivado, following scripts in the [vivado](./vivado/) folder.  
This includes resets, clock generation, and memory configuration files required to implement the final system.

---

## 10 — Software and Vitis Integration

Export the hardware platform to Vitis and integrate the software layer from [software](./software/).  
Define instruction macros, add CX instruction support, and compile C applications that communicate with hardware accelerators.  
The ELF generation process is documented in the [process](./process/) folder.

---

## 11 — ELF and Build Artifacts

Generated ELF files and corresponding metadata are located under [build/elf](./build/elf/).  
These files represent compiled software applications targeting the CX-extended MicroBlaze-V system.

---

## 12 — Process and Reproduction

A step-by-step guide to the complete implementation workflow, including hardware export, software build, and verification, is provided in the [process](./process/) folder.

---

## 13 — Licensing and Restrictions

This repository excludes all AMD/Xilinx proprietary IP and RTL source files.  
It contains only configuration files, scripts, and methodological documentation necessary for reproducing the process if you have licensed access to the corresponding IP.

---

© 2025 — CX Integration for MicroBlaze-V | Chalmers University of Technology & AMD Collaboration
