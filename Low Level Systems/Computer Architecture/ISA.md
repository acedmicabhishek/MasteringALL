# Instruction Set Architecture (ISA)
> *The fundamental contract between software and hardware.*

## 1. What is an ISA?
The ISA is the part of the computer architecture that is visible to the programmer or the compiler writer. It serves as the boundary where the Software (Code) meets the Hardware (Microarchitecture).

*   **Software view**: A set of registers, instructions, and memory models.
*   **Hardware view**: A specification that must be implemented using decoders, ALUs, and control units.

### 1.1. CISC (Complex Instruction Set Computer)
*   **Philosophy**: "Make the hardware do more."
*   **ALU & Components**:
    *   **Micro-ops (µops)**: Modern CISC CPUs (like Intel Core) don't actually execute CISC instructions. They have a **complex decoder** that breaks one instruction (e.g., `ADD [MEM], RAX`) into 3-4 tiny, RISC-like µops.
    *   **Microcode ROM**: Complex instructions are like "macros." The processor has a small internal memory (Microcode ROM) that tells the Control Unit how to sequence multiple steps for one instruction.
    *   **Multi-step ALU**: The ALU might be tied to a cycle where it waits for memory to arrive before performing the addition.
*   **Traits**: Variable-length instructions, many addressing modes.

### 1.2. RISC (Reduced Instruction Set Computer)
*   **Philosophy**: "Make the instructions simple and fast."
*   **ALU & Components**:
    *   **Hardwired Control**: Since instructions are simple, there is no need for a Microcode ROM. The control logic is physically "wired" into the chip for maximum speed.
    *   **Register-to-Register ALU**: The ALU **never** touches memory. It only performs math on registers. This keeps the ALU pipeline extremely fast and predictable.
    *   **Uniform Pipeline**: Every instruction goes through the same Fetch-Decode-Execute-Memory-Writeback stages.
*   **Traits**: Fixed-length (32-bit) instructions, Load/Store architecture.

### 1.3. The Implementation Flow
Imagine the calculation: `Memory[Address] = Memory[Address] + 5`

| Step | CISC Implementation (`ADD [ADDR], 5`) | RISC Implementation (`LDR`, `ADD`, `STR`) |
| :--- | :--- | :--- |
| **Decode** | Complex Decoder breaks it into µops. | Hardwired logic immediately knows the task. |
| **ALU** | Wait for memory read, then add, then write. | ALU only handles the `ADD R1, R1, #5` part. |
| **Pipeline** | The pipeline "stalls" or stretches to handle the complexity. | Pipeline flows perfectly; each step completes in 1 cycle. |

---

## 2. x86_64 (The CISC Giant)
The 64-bit extension of the Intel x86 architecture.

### 2.1. Key Characteristics
*   **Backward Compatibility**: x86_64 can run 16-bit and 32-bit code (Real Mode, Protected Mode).
*   **Variable Instruction Length**: Instructions range from **1 to 15 bytes**. This makes decoding extremely difficult for hardware but saves memory space.
*   **Two-Operand Format**: Most instructions are `OP DST, SRC` (e.g., `ADD EAX, EBX` results in $EAX = EAX + EBX$).

### 2.2. Register File
*   **General Purpose (GPRs)**: 16 registers (`RAX`, `RBX`, `RCX`, `RDX`, `RSI`, `RDI`, `RBP`, `RSP`, `R8`-`R15`).
*   **Specialized**: `RIP` (Instruction Pointer), `RFLAGS` (Status flags like Zero, Carry, Sign).
*   **SIMD**: `XMM` (128-bit), `YMM` (256-bit), `ZMM` (512-bit) for AVX.

### 2.3. Memory Model
*   **Strong Memory Consistency**: Typically adheres to **Total Store Ordering (TSO)**. Hardware ensures that writes from a single core appear in order to other cores (with some exceptions).

---

## 3. ARMv8-A (AArch64)
The ubiquitous architecture for mobile, and increasingly, servers and PCs (Apple Silicon).

### 3.1. Key Characteristics
*   **Fixed Instruction Length**: Every instruction is exactly **32 bits (4 bytes)**. This allows for very fast, parallel decoding.
*   **Load/Store Architecture**: You cannot `ADD` a value from memory directly. You must `LDR` (Load) into a register, `ADD`, then `STR` (Store).
*   **Three-Operand Format**: `OP DST, SRC1, SRC2` (e.g., `ADD X0, X1, X2` results in $X0 = X1 + X2$). X1 and X2 remain unchanged.

### 3.2. Register File
*   **General Purpose**: 31 registers (`X0` to `X30`). `X30` is the **Link Register (LR)**.
*   **Fixed Zero Register**: `XZR` always returns 0.
*   **Stack Pointer**: `SP` is separate from GPRs.

### 3.3. Advanced Features
*   **Conditional Execution**: Reduced in v8 compared to v7, but still present via `CSEL` (Conditional Select).
*   **Energy Efficiency**: Designed for high performance-per-watt.

---

## 4. RISC-V (The Open Standard)
A clean-slate, modular, open-source ISA.

### 4.1. Philosophy: Modularity
RISC-V is not a single giant ISA. It consists of a **Base** and several **Extensions**.
*   **RV64I**: The base 64-bit integer instruction set.
*   **M Extension**: Integer Multiplication and Division.
*   **A Extension**: Atomic instructions.
*   **F/D Extension**: Single/Double precision Floating Point.
*   **C Extension**: Compressed instructions (16-bit) to reduce code size.
*   **V Extension**: Vector processing.

### 4.2. Registers
*   32 registers (`x0` to `x31`).
*   `x0` is hardwired to zero.
*   Program Counter (`pc`) is separate.

### 4.3. Privilege Levels
Designed for everything from tiny embedded sensors to supercomputers.
*   **U-Mode** (User): For applications.
*   **S-Mode** (Supervisor): For Operating Systems (handles Virtual Memory).
*   **M-Mode** (Machine): Highest privilege (Firmware/Bootloader).

---

## 5. ISA Comparison Summary

| Feature | x86_64 | ARMv8 (AArch64) | RISC-V |
| :--- | :--- | :--- | :--- |
| **Logic Type** | CISC | RISC | RISC |
| **Instruction Length** | Variable (1-15B) | Fixed (4B) | Fixed (4B) / Compressed (2B) |
| **Reg/Mem math?** | Yes | No (Load/Store) | No (Load/Store) |
| **GPR Count** | 16 | 31 + XZR | 31 + x0 |
| **Endianness** | Little-Endian | Bi-Endian (Default LE) | Bi-Endian (Default LE) |
| **Control** | Proprietary (Intel/AMD) | Licensed (ARM Ltd) | **Open Source** |

---

## 6. Memory Consistency Models
*How different cores see each other's memory writes.*

1.  **Sequential Consistency (SC)**: The strictest. Every core sees all operations in exactly the same global order. Very slow to implement.
2.  **Total Store Ordering (TSO)**: (x86). Stores from one core are seen in order, but a Load can bypass an older Store to a different address.
3.  **Weak Consistency**: (ARM, RISC-V). Hardware can reorder almost anything for speed. Software must use **Memory Barriers** (`DMB`, `FENCE`) to enforce order when needed.
