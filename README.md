# RISC-V Emulation & Cross-Development Training

This repository serves as a guide and reference architecture for working with the RISC-V architecture using QEMU. It is designed to familiarize engineers with the toolchains, emulation modes, and workflows required for embedded development and high-performance computing on RISC-V platforms.

## Repository Intent

The primary goal of this repository is to bridge the gap between development on x86 host machines and deployment on RISC-V targets. By completing these modules, engineers will transition from running simple cross-compiled binaries to managing full system-mode emulated environments.

**Key Learning Outcomes:**
*   **Cross-Compilation:** Understanding how to build software on x86_64 for execution on RISC-V 64-bit (`riscv64`).
*   **Emulation Architectures:** Distinguishing between Application Level (User Mode) and Full System Level (System Mode) emulation.
*   **Systems Engineering:** Managing Sysroots, dynamic linking, bootloaders (U-Boot), and kernel interactions.
*   **Debugging:** analyzing instruction sets, memory maps, and boot processes without physical hardware.

## Repository Structure

The training is divided into two distinct phases, progressing from application-level logic to system-level integration.

### [Phase 1: User Mode Emulation](./user-mode/README.md)
**Focus:** Application Logic, Toolchains, and Libraries.

In this phase, the focus is on `qemu-riscv64`. This mode emulates a single process, translating RISC-V instructions to host instructions in real-time. It is the preferred method for compiling code, running scripts, and executing unit tests due to its speed and direct access to host resources.

**Topics Covered:**
*   Building QEMU from source.
*   Setting up the RISC-V GNU Toolchain.
*   The importance of the Sysroot (`-L` flag).
*   Static vs. Dynamic linking mechanisms.

### [Phase 2: System Mode Emulation](./system-mode/README.md)
**Focus:** Operating Systems, Networking, and Virtual Hardware.

In this phase, the focus shifts to `qemu-system-riscv64`. This mode emulates a complete motherboard, including the CPU, RAM, MMU, and peripheral devices. It is required for kernel development, driver testing, and validating boot sequences.

**Topics Covered:**
*   Cross-compiling firmware (U-Boot).
*   Configuring User-Mode Networking (Slirp).
*   Booting a full Linux distribution (Ubuntu Server).
*   Managing virtual disk images and partitioning.

### [Phase 3: Custom Instruction Implementation](./03_Custom_ISA)
**Focus:** QEMU Internals, Instruction Set Architecture (ISA) Extension,.

This advanced phase involves modifying the QEMU source code to implement a new, non-standard instruction. This task explores the entire instruction pipeline from binary decoding to execution via the TCG helper system.

**Topics Covered:**
*   Instruction format encoding.
*   QEMU's TCG Helper architecture (`DEF_HELPER_FLAGS`).
*   Creating a C-code execution backend for complex instructions.
*   Bypassing the assembler for testing using the `.insn` directive.
