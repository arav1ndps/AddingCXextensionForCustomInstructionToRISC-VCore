## Verification Workflow

A comprehensive **top-level testbench** was developed for the **functional verification** of the CX interface, the interconnect architecture, and the integrated accelerator modules.  
The verification environment was designed to accurately replicate the behavior of the **software layer**, effectively simulating how the MicroBlaze-V software libraries issue CX-based custom instructions.


![test_bench_flow](./figures/test_bench.png)

### Testbench Design

The testbench emulates the software driver behavior by generating and sending instruction streams to the CX interface.  
It manages the handshaking between the **CX master**, **interconnect**, and **accelerator modules**, ensuring that all data and control signals are properly synchronized.

**Key features:**
- Emulation of instruction flow from the software layer (configuration → input → output).
- Integration of all major hardware blocks: CX interface, interconnect, wrapper, and accelerator.
- Timing-accurate simulation of request and response handshakes using AXI-Stream protocols.

---

### Test Vector Generation

Test vectors were automatically generated using a **Python-based generator script** located in the `verification/scripts/` directory.  
The script concatenates:
- **CXU_ID** (Custom Execution Unit Identifier)  
- **CF_ID** (Custom Function Identifier)  
- **Input data** required by the accelerator

These fields are combined into valid CX instruction frames that are fed to the testbench through a stimulus driver.  
The **input data** for each accelerator was extracted directly from the **inbuilt testbench** provided in the respective **Xilinx IP core library**, ensuring that all test cases use verified and meaningful datasets.

---

### Self-Checking Mechanism

The verification setup includes a **self-checking system** that automatically validates the functional correctness of accelerator outputs.  
After each simulation run:
- The **computed results** from the CX-connected accelerators are captured.
- These results are **compared against reference outputs** obtained from the IP core’s original testbench or simulation model.
- Any mismatch triggers an assertion or failure log entry, providing precise debugging feedback.

This mechanism ensures that:
- All accelerator responses conform to expected numerical accuracy.
- The CX interconnect and data routing logic introduce no corruption or timing hazards.

---

### Verification Outcomes

Upon completion of simulations:
- All assertion checks on **AXI-Stream protocols**, **stalling logic**, and **CSR updates** passed successfully.
- Functional equivalence between the **CX-integrated accelerators** and the **standalone IP cores** was confirmed.
- The automated framework demonstrated reliable validation of new CX instruction integrations with minimal manual intervention.

---

**Summary:**  
The verification workflow establishes a robust and automated framework to validate the CX interface integration.  
It effectively bridges the gap between **software-issued instructions** and **hardware accelerator responses**, ensuring both **functional correctness** and **specification compliance** across the full CX pipeline.
