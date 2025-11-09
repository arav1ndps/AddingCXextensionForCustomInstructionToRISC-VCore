# Adding CX extension for Custom Instruction to RISC-VCore
**Master’s Thesis Project – Chalmers University of Technology, MPEES**

## Overview

This work presents the **design and implementation of a Composable Extension (CX)** framework for integrating **custom instructions** into the **MicroBlaze-V** core (soft-core). 
The work demonstrates how the CX interface enables seamless integration of user-defined hardware accelerators into a RISC-V core architecture to enhance performance for compute-intensive applications.

In this implementation, pre-existing **Xilinx IP cores** were employed as hardware accelerators and tested on **Field-Programmable Gate Array (FPGA)**. To assess the system’s efficiency, equivalent accelerator functions were also implemented in **C software** and executed on the same MicroBlaze-V core without extension.  

> **Note:**  
> This repository does not contain the source code, as it is the **intellectual property of AMD**.  
> However, it provides a detailed overview of the **design methodology**, **implementation process**, and **evaluation results** to illustrate how the project was conducted and verified.

##  Specification

This work follows the **Composable Custom Extension (CX)** specifications defined by the **RISC-V community** and standards of custom instructions execution in the RISC-V ISA. 

This specification enables unlimited, independent, efficient, and robust composition of diverse RISC-V composable extensions, hardware composable extension units (CXUs), and software libraries.

For detailed information about the CX specification, please refer to:
- *RISC-V Composable Custom Extension (CX) Specification* — available in the project documentation.  
- **Gray Research GitHub Repository:** [https://github.com/grayresearch](https://github.com/grayresearch) — containing reference implementations, testbenches, and CX interface documentation.

## Implemented Functions

In this work, following functions are implemented
-Interface handling the execution of a custom function in the RISC-V core
-Interconnect for the selection of CX Units.
-FFT and CORDIC IP cores for the acceleration
-Wrapper modules for the accelerators
-MicroBlaze-V software support for executing custom instructions


## Future Work

The current CX extension in the **MicroBlaze-V** core supports FFT acceleration with specific software-defined execution constraints.  

- **Hardware-Managed Instruction Sequencing:**  
  Implement hardware-level control logic to automatically handle the chronological execution order (`configuration → input → output`), removing constraints in the software.

- **Standalone Interconnect IP Core:**  
  Develop and release the **interconnect block** as an independent **IP core** with peripheral support for MicroBlaze-V.  
  This would enable seamless interfacing of multiple accelerators through a standardised extension port.

- **Vivado Wrapper Function Integration:**  
  Predefine and package the **wrapper module functions** within **Xilinx Vivado**, enabling users to easily instantiate and connect independent accelerators to the MicroBlaze-V core through the CX interface.

- **Extended Accelerator Library:**  
  Support for various reusable accelerator functions.

- **Automation & Toolflow Support:**  
  Integrate automation scripts for Vivado to streamline accelerator generation, mapping, and verification directly from a graphical or command-line interface.

These enhancements will strengthen the CX ecosystem for MicroBlaze-V, providing a **fully composable, user-friendly, and scalable acceleration framework** for future RISC-V-based SoC designs.








