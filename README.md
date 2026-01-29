# QEMU & RISC-V
This repository contains a series of hands-on training tasks designed to get you up and running with QEMU for RISC-V development.

## Table of Contents
1.  [Prerequisites](#prerequisites)
2.  [Module 1: Getting Started with QEMU User Mode](#module-1-getting-started-with-qemu-user-mode)
3.  [Module 2: The Role of the Sysroot](#module-2-the-role-of-the-sysroot)
4.  [Module 3: Dynamic vs. Static Linking](#module-3-dynamic-vs-static-linking)

---

## Prerequisites

Before you begin, you will need a Linux-based environment (Ubuntu 22.04 recommended) with the following tools installed:

-   `git`
-   `wget`
-   `tar`
-   Standard build tools (`build-essential`)
-   `ninja-build`

---

## Module 1: Getting Started with QEMU User Mode
**Goal:** To compile the QEMU emulator, download a RISC-V toolchain, and successfully run a cross-compiled "Hello World" program.

### Task 1.1: Build QEMU
QEMU is a powerful open-source emulator. We will build the "User Mode" version, which is designed to run single applications for a different architecture.

1.  Clone the QEMU repository:
    ```bash
    git clone https://github.com/qemu/qemu.git
    cd qemu
    ```
2.  Initialize the necessary submodules:
    ```bash
    git submodule init && git submodule update --recursive
    ```
3.  Create a build directory and configure QEMU to build only the RISC-V User Mode emulators:
    ```bash
    mkdir build && cd build
    ../configure --target-list=riscv32-linux-user,riscv64-linux-user
    ```
4.  Start the compilation process. This will use all available CPU cores.
    ```bash
    ninja -j`nproc`
    ```
    Upon completion, you will have a `qemu-riscv64` binary inside the `build` directory.

### Task 1.2: Download the RISC-V Toolchain
A "toolchain" is a set of tools (compiler, linker, libraries) that allows you to create programs for a specific target architecture.

1.  Download the pre-built RISC-V GNU Toolchain:
    ```bash
    wget https://github.com/riscv-collab/riscv-gnu-toolchain/releases/download/2026.01.23/riscv64-glibc-ubuntu-22.04-gcc.tar.xz
    ```
2.  Extract the archive:
    ```bash
    tar -xvf riscv64-glibc-ubuntu-22.04-gcc.tar.xz
    ```
    This will create a directory named `riscv`, which contains the compiler and all necessary libraries.

### Task 1.3: Compile and Run Your First Program
1.  Create a new file named `add.c` with the following content:
    ```c
    #include <stdio.h>

    int main() {
        int a = 3;
        int b = 5;
        int c = a + b;
        printf("sum = %d\n", c);
        return 0;
    }
    ```
2.  Use the toolchain's `gcc` to compile your C code into a RISC-V executable:
    ```bash
    ./riscv/bin/riscv64-unknown-linux-gnu-gcc add.c -o add
    ```
3.  Run the executable using the QEMU you built.
    -   **-L**: This flag points to the **Sysroot**, which is the directory containing the libraries your program needs to run.
    -   **-cpu**: This flag defines the features of the virtual CPU.

    *Note: Replace `<Full/Path/To/Your/Folder>` with the actual absolute path to your riscv directory.*
    ```bash
    ./qemu/build/qemu-riscv64 \
        -L <Full/Path/To/Your/Folder>/riscv/sysroot \
        -cpu rv64,v=true \
        ./add
    ```

**Deliverable:** A screenshot showing the output `sum = 8`.

---

## Module 2: The Role of the Sysroot
**Goal:** Understand why the `-L` (Sysroot) flag is critical for running dynamically linked programs.

### Task 2.1: The "Missing Library" Error
1.  Take the working command from Task 1.3.
2.  **Remove** the entire `-L <path/to/sysroot>` flag.
3.  Run the command again. Observe the error.

**Questions:**
1.  What error message did you see?
2.  Even though the `add` binary exists, why does QEMU fail to run it?

---

## Module 3: Dynamic vs. Static Linking
**Goal:** Learn how to create a self-contained executable that does not depend on external libraries.

### Task 3.1: Building a Static Binary
1.  Modify your compilation command by adding the `-static` flag. This tells the compiler to copy all library code directly into your executable.
    ```bash
    ./riscv/bin/riscv64-unknown-linux-gnu-gcc -static add.c -o add_static
    ```
2.  Run this new `add_static` binary using QEMU, but **do not** include the `-L` (Sysroot) flag.
    ```bash
    ./qemu/build/qemu-riscv64 -cpu rv64 ./add_static
    ```

**Questions**
1.  Why does `add_static` run successfully without the `-L` flag, while `add` failed?
2.  Check the file sizes of `add` and `add_static`. Which one is larger, and why?
