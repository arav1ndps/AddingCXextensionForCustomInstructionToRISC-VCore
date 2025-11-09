# Project README — CX Integration for MicroBlaze-V (No source code included)

> **Important note:** This repository does **not** contain AMD proprietary source code. It documents the full **methodology**, configuration files, scripts, IP wrappers, testbenches, and build artifacts (ELF files) used to integrate a **Composable Extension (CX)** into the **MicroBlaze-V** RISC-V core. The IP sources and MicroBlaze-V core modifications are AMD/third-party intellectual property and are therefore **not included** here. Where required, pointers to repository folders and configuration artifacts are provided.

This README describes, in detail and step-by-step, how to reproduce the project environment, how each component is organized in this repository, and the verification & build flow used to validate and generate the final system (including ELF files). No figures or images are included — the text is written to be professional, self-contained, and reproducible.

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

## 1 — Configure the IP core

Configure the accelerator IPs (FFT, CORDIC, etc.) as described in the [IP_configuration](./IP_configuration/) folder.  
Follow the parameter files and Vivado scripts located there to instantiate the IPs.

---

## 2 — Create the wrapper functions

Create CX-compliant wrappers for each accelerator using the modules and documentation in the [wrapper](./wrapper/) folder.  
These wrappers define the translation logic between AXI-Stream and accelerator-specific interfaces.

---

## 3 — Implement the CX interface in MicroBlaze-V

Add the CX front-end and signal management as described in [microblaze/CX_interface](./microblaze/CX_interface/).  
This integrates the CX request and response logic into the decode/execute pipeline stages.

---

## 4 — Implement the Interconnect

Develop the AXI-Stream based interconnect from [interconnect](./interconnect/), connecting multiple accelerators to the CX interface.

---

## 5 — AXI Connectivity

All components communicate via AXI-Stream channels:  

```
MicroBlaze (CX_interface) --[CX_M_AXIS]--> Interconnect Router --> Wrapper(s) --> IP Core
MicroBlaze (CX_interface) <--[CX_S_AXIS]-- Interconnect Router <-- Wrapper(s) <-- IP Core
```

Ensure clock and data-width consistency across all connected components.

---

## 6 — Verification using Assertion-Based Testbench

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
