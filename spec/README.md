# RISC-V Composable Custom Extensions (CX) Specification

## 📘 Overview

This repository provides a detailed overview of the **RISC-V Composable Custom Extensions (CX)** specification — a framework that defines hardware-software and hardware-hardware interfaces, enabling flexible integration of independently developed **custom instruction extensions** within RISC-V processors.

Composable Custom Extensions introduce a unified mechanism to **extend the RISC-V ISA** with user-defined hardware accelerators and software libraries, while preserving interoperability, scalability, and forward compatibility.  

The specification defines how to integrate diverse **Composable Extension Units (CXUs)**, **libraries**, and **software runtimes** into a cohesive system that allows developers to rapidly prototype and deploy hardware-accelerated compute functions.

---

## 🧩 Key Concepts

- **Composable Extension (CX):**  
  A fixed, named set of custom functions and control/status registers (CSRs), which may be stateless or stateful.

- **Composable Extension Unit (CXU):**  
  A hardware unit that implements one or more CXs. It communicates with the CPU through a standardized logic interface (CXU-LI).

- **CXU Logic Interface (CXU-LI):**  
  Defines the **hardware-to-hardware signaling protocol** between CPUs and CXUs, supporting fixed, variable, and reordering latency accelerators.

- **CX-ISA:**  
  The composable ISA extension introduces standard CSRs such as:
  - `mcx_selector` – selects the current CXU and state context  
  - `cx_status` – accumulates CXU error and execution status  
  - `mcx_table`, `cx_index` – access control and indexing mechanisms  

- **CX-API & CX-ABI:**  
  Define software-level interfaces for managing CX interactions, instruction selection, and context handling at runtime and binary levels.

- **IStateContext:**  
  A standardized extension that defines functions to read/write status and state context for serializable, stateful composable extensions.

---

## ⚙️ Features and Scope

- Enables seamless composition of hardware and software extensions across CPUs, accelerators, and libraries.  
- Provides **collision-free multiplexing** of custom instructions and CSRs, removing dependency on central opcode allocation.  
- Supports **stateful and stateless** accelerators, each operating in isolated and composable contexts.  
- Designed for scalability — from **microcontrollers** to **multicore Linux-class systems**.  
- Defines uniform metadata and manifest files to enable **automatic system composition** by design tools.  

---

## 🧠 Design Philosophy

The CX framework promotes **open, modular, and interoperable design** principles by allowing independent teams to:

- Define new extensions without central coordination.  
- Reuse existing CXUs and libraries across different CPU architectures.  
- Achieve robust, predictable system integration via **strict isolation** of extension state and behavior.  
- Build systems that evolve seamlessly through **versioned metadata** and extensible specifications.  

---

## 🧾 Specification Details

This repository references the draft document:  
**_Draft Proposed RISC-V Composable Custom Extensions Specification (v0.95.240403)_**,  
authored by Jan Gray and contributors.

Key sections include:
- Introduction to the CX ecosystem  
- CX hardware-software interface definitions  
- CXU Logic Interface (CXU-LI)  
- CX Application Binary Interface (CX-ABI)  
- CX Metadata and System Composition  
- Versioning and future direction  

Full details are available in the provided `spec.pdf`.  

For upstream updates and discussions, see the original **Gray Research** GitHub repository:  
🔗 [https://github.com/grayresearch](https://github.com/grayresearch)

---

## 📄 License

This work is distributed under the **Apache License, Version 2.0**.  
You may obtain a copy at:  
[www.apache.org/licenses/LICENSE-2.0](https://www.apache.org/licenses/LICENSE-2.0)
