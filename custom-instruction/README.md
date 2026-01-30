# **Adding a Custom Instruction in QEMU for RISC-V**

**Goal:** Implement a custom arithmetic instruction in QEMU's TCG (Tiny Code Generator) to understand the full path from assembly to execution.

**Instruction to Implement:** `FACTORIAL rd, rs1`

*   **Logic:** Calculates the factorial of the input register (`rs1`) and writes the result to the destination register (`rd`).
*   **Format:** R-Type (Unary).

---

## **Phase 1: Setup and The Core Logic**

### Task 1.1: The Custom Function

The factorial loop must be implemented in standard C code. This function will be called by QEMU's TCG.

1.  **Define the Helper:** Locate the correct header file and declare a TCG Helper named **`factorial`** that takes one `target_long` argument and returns one `target_long` value.
2.  **Implement the Logic:** In the appropriate file (`op_helper.c`), write the C function that performs the factorial calculation.

### Task 1.2: The TCG Bridge

The function's signature must be registered for the TCG to know how to generate a call to it.

1.  **Helper Registration:** In the relevant header file (likely `helper.h`), add the macro that registers the **`factorial`** helper.
2.  **Naming Convention:** Research the `DEF_HELPER_FLAGS_1` macro and ensure you use the correct syntax (`tl` for `target_long`).

---

## **Phase 2: Decoding and Translation**

The goal of this phase is to make the assembler emit the correct bits and QEMU's decoder recognize those bits.

### Task 2.1: Instruction Encoding

1.  **Choose the Opcode:** Select the standard **`CUSTOM-0`** opcode (`0b0001011` / `0x0B`) and a unique **`Funct3`** value (e.g., `0b001`) for the instruction.
2.  **Decoder Definition:** In **`target/riscv/insn32.decode`**, add the definition for the `factorial` instruction. Use the R-Type format shorthand to auto-map the `rs1` and `rd` registers.

### Task 2.2: The Translation Function

The core logic of the instruction must be defined.

1.  **Function Definition:** Create a new file, **`target/riscv/insn_trans/trans_rvcustom.c.inc`**, and define the function **`trans_factorial`**.
2.  **Linking:** Ensure the definition of `trans_factorial` is included in **`target/riscv/translate.c`** so the linker can find it.
3.  **TCG Logic:** Inside `trans_factorial`, use `get_gpr`, `gen_helper_factorial`, and `gen_set_gpr` to map the RISC-V registers to the TCG helper call.

### Task 2.3: Finalize Translation and Build

The goal of this task is to integrate your custom translation logic and successfully compile QEMU.

1.  **Attempt to Compile:** Run the build command.

2.  **Resolution:** If the build fails, it indicates an error in the custom code's function definition or linking. Resolve any compilation errors displayed by the terminal.

3.  **Final Status:** If the build finishes successfully, you can proceed directly to the testing phase (Task 3.1).
---

## **Phase 3: Final Verification**

### Task 3.1: Testing

Since the standard assembler does not know the word `factorial`, we must manually inject the bit pattern. You can use the below test file:

```c
#include <stdio.h>

int main() {
    long input = 5;
    long result = 0;

    printf("Calculating Factorial of %ld...\n", input);

    // .insn r opcode, func3, func7, rd, rs1, rs2
    asm volatile (
        ".insn r 0xB, 1, 0, %0, %1, x0"
        : "=r"(result)   // Output (rd)
        : "r"(input)     // Input (rs1)
        // We pass x0 for rs2 implicitly in the assembly string
    );

    printf("Result: %ld\n", result);

    if (result == 120) {
        printf("SUCCESS: 5! == 120\n");
    } else {
        printf("FAILURE: Expected 120, got %ld\n", result);
    }

    return 0;
}
```
1. **Compile:** Compile the test program with the standard toolchain.
```bash
riscv64-unknown-linux-gnu-gcc test.c -o test
```

2. **Run:** Run the compiled binary using
```bash
build/qemu-riscv64 -L riscv/sysroot ./test
```
3.  **Result:** The output must display the correct result: `Result: 120` and a success message.
