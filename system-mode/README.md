# QEMU System Mode

This module focuses on **System Mode** emulation (`qemu-system-riscv64`). Unlike User Mode, which runs a single binary, System Mode emulates a complete hardware platform, including the CPU, memory, peripheral devices, and networking. This allows for booting full operating systems like Ubuntu Server.

## Table of Contents
1.  [Prerequisites](#prerequisites)
2.  [Module 1: Building the Infrastructure](#module-1-building-the-infrastructure)
3.  [Module 2: Booting the Operating System](#module-2-booting-the-operating-system)
4.  [Module 3: Verification & Analysis](#module-3-verification--analysis)

---

## Prerequisites

Ensure the following dependencies are installed on the host Linux system to allow compiling QEMU and its dependencies:

```bash
sudo apt-get install meson git libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev ninja-build
```

---

## Module 1: Building the Infrastructure
**Goal:** Compile the U-Boot bootloader and the QEMU System Mode emulator with user-mode networking support.

### Task 1.1: Setup the Toolchain
U-Boot must be cross-compiled. We will use a pre-built RISC-V GNU toolchain.

1.  Download the toolchain:
    ```bash
    wget https://github.com/riscv-collab/riscv-gnu-toolchain/releases/download/2024.09.03/riscv64-glibc-ubuntu-22.04-gcc-nightly-2024.09.03-nightly.tar.gz
    ```
2.  Extract the archive and export the path:
    ```bash
    tar -xvf riscv64-glibc-ubuntu-22.04-gcc-nightly-2024.09.03-nightly.tar.gz

    # Set the variable for future commands
    export RISCV_GNU_TOOLCHAIN=$(pwd)/riscv
    ```

### Task 1.2: Cross-Compile U-Boot
U-Boot acts as the firmware/BIOS for the virtual machine. It initializes hardware and loads the operating system kernel.

1.  Clone the repository:
    ```bash
    git clone https://github.com/qemu/u-boot.git
    cd u-boot
    ```
2.  Configure for QEMU RISC-V 64-bit Supervisor Mode (`smode`):
    ```bash
    make qemu-riscv64_smode_defconfig CROSS_COMPILE=$RISCV_GNU_TOOLCHAIN/bin/riscv64-unknown-linux-gnu-
    ```
3.  Build the binary:
    ```bash
    make CROSS_COMPILE=$RISCV_GNU_TOOLCHAIN/bin/riscv64-unknown-linux-gnu-
    ```
    **Result:** A file named `u-boot.bin` is generated. Note the absolute path to this file.

### Task 1.3: Build QEMU with Slirp
`libslirp` is required to enable user-mode networking, allowing the VM to access the internet via the host connection.

1.  Build and install `qemu-slirp`:
    ```bash
    git clone https://github.com/openSUSE/qemu-slirp.git
    cd qemu-slirp
    meson build
    ninja -C build install
    cd ..
    ```
2.  Clone and Build QEMU:
    ```bash
    git clone https://github.com/qemu/qemu.git
    cd qemu
    git submodule init && git submodule update --recursive

    mkdir build && cd build

    # Configure for System Mode, enabling Slirp
    ../configure --target-list=riscv64-linux-user,riscv64-softmmu --enable-slirp

    make -j$(nproc)
    ```

---

## Module 2: Booting the Operating System
**Goal:** Launch a full Ubuntu Server instance inside the emulated RISC-V environment.

### Task 2.1: Prepare the Disk Image
1.  Download the pre-installed Ubuntu Server image for RISC-V:
    [Download Ubuntu for RISC-V](https://ubuntu.com/download/risc-v/canonical-built#qemu)
2.  If the file is compressed (`.img.xz`), decompress it:
    ```bash
    unxz ubuntu-24.04.3-preinstalled-server-riscv64.img.xz
    ```

### Task 2.2: Launch the VM
Execute the following command to boot the system.

*Note: Replace `<PATH_TO_UBOOT>` and `<PATH_TO_IMAGE>` with your actual paths.*

```bash
./qemu/build/qemu-system-riscv64 \
    -machine virt \
    -cpu rv64,zba=true,zbb=true,v=true,vlen=256,zfh=true,zvfh=true,zifencei=true,vext_spec=v1.0,zawrs=true \
    -semihosting \
    -nographic \
    -m 16G \
    -smp 8 \
    -kernel <PATH_TO_UBOOT>/u-boot.bin \
    -device virtio-net-device,netdev=eth0 \
    -netdev user,id=eth0 \
    -drive file=<PATH_TO_IMAGE>/ubuntu-24.04.3-preinstalled-server-riscv64.img,format=raw,if=virtio
```

**Questions:**
1.  **Kernel vs Bootloader:** Why does the `-kernel` flag point to `u-boot.bin` instead of a Linux kernel image (`vmlinuz`)? What advantage does this provide when running `apt upgrade` inside the VM?
2. **Virtual Hardware:** What does `-machine virt` represent? Is it modelling a specific physical board (like a Raspberry Pi) or a generic platform?
---

## Module 3: Verification & Analysis
**Goal:** Confirm the environment capabilities and networking status.

### Task 3.1: Login and Hardware Check
1.  Login with the default credentials (usually `ubuntu` / `ubuntu`).
2.  Verify the CPU capabilities:
    ```bash
    cat /proc/cpuinfo
    ```
    Confirm that the output lists the extensions (`v`, `zba`, `zbb`) specified in the boot command.

### Task 3.2: Network Verification
1.  Test internet connectivity:
    ```bash
    sudo apt update
    ```
