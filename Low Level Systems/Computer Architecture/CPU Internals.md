# CPU Internals: The Engine of Performance
> *How a processor extracts maximum speed from silicon.*

## 1. Pipelining
Pipelining is the process of overlapping the execution of multiple instructions. Think of it like a car assembly line: while one car is having its engine installed, the next car is being painted.

### 1.1. The Standard 5-Stage Pipeline (RISC)
1.  **IF (Instruction Fetch)**: Get the instruction from memory (via Program Counter).
2.  **ID (Instruction Decode)**: Figure out what the instruction wants to do and read registers.
3.  **EX (Execute)**: The ALU performs the math or calculates an address.
4.  **MEM (Memory Access)**: Read from or write to Data Cache (if needed).
5.  **WB (Write Back)**: Store the result into the register file.

*   **Throughput**: Pipelining increases throughput (instructions finished per second) but slightly increases latency for a single instruction due to overhead.

---

## 2. Pipeline Hazards
Hazards are situations that prevent the next instruction in the instruction stream from executing in its designated clock cycle.

### 2.1. Structural Hazards
Resource conflict. Two instructions need the same hardware unit at the same time (e.g., a single memory port for both instruction fetch and data access).
*   **Solution**: Duplicate hardware (separate I-Cache and D-Cache).

### 2.2. Data Hazards
An instruction depends on the result of a previous instruction that hasn't finished yet.
*   **RAW (Read After Write)**: The most common. `ADD R1, R2, R3` followed by `SUB R4, R1, R5`. R1 isn't ready for the `SUB`.
*   **Solution**: **Forwarding (Bypassing)**. Sending the ALU result directly to the next instruction's ALU input before it is written back to the register file.

### 2.3. Control Hazards
Caused by branches (`JMP`, `BEQ`). The processor doesn't know which instruction to fetch next until the branch is evaluated in the EX stage.
*   **Solution**: **Branch Prediction** or **Stalling (Bubbles)**.

---

## 3. Branch Prediction & Speculative Execution
Modern CPUs are so fast that they cannot afford to wait for a branch to finish. They "guess" which way the branch will go.

### 3.1. Static Prediction
Simple rules like "Always assume backward branches are taken" (used for loops).

### 3.2. Dynamic Prediction
The CPU uses a **Branch History Table (BHT)** to remember how a specific branch behaved in the past.
*   **BTB (Branch Target Buffer)**: Stores the target address of the branch so the CPU can start fetching immediately.

### 3.3. Speculative Execution
The CPU starts executing instructions based on its guess. 
*   **If Correct**: Massive speedup. 
*   **If Wrong**: The **Pipeline Flush**. The CPU must throw away all "speculated" work and restart from the correct address. This is the source of the **Spectre** security vulnerability.

---

## 4. Superscalar & ILP (Instruction Level Parallelism)
A Superscalar CPU has multiple "Execution Pipelines." It can start and finish more than one instruction per clock cycle.

*   **IPC (Instructions Per Cycle)**: In a superscalar CPU, IPC > 1.
*   **Execution Ports**: A CPU might have 4 ALU ports, 2 Load/Store ports, and 1 Vector port, allowing 7 µops to fire simultaneously.

---

## 5. Out-of-Order (OoO) Execution
The CPU analyzes the instruction stream and executes instructions as soon as their **data is ready**, even if they appear later in the code.

### 5.1. The OoO Workflow
1.  **In-Order Front-End**: Fetch and Decode instructions in order.
2.  **Register Renaming**: Map architectural registers (`RAX`) to a larger pool of physical registers to remove "false" dependencies (WAR/WAW).
3.  **Dispatch (Issue)**: Place instructions into **Reservation Stations** (Wait rooms).
4.  **Execute**: When operands are ready, the instruction fires.
5.  **Commit (Retire)**: Results are put back in order into a **Reorder Buffer (ROB)** so the software thinks everything happened sequentially.

### 5.2. Tomasulo's Algorithm
The classic algorithm for OoO execution using reservation stations and a common data bus (CDB) to broadcast results to waiting instructions.

---

## 6. Summary: The Performance Stack
| Technique | Problem Solved | Efficiency Gain |
| :--- | :--- | :--- |
| **Pipelining** | Single-step execution is slow | Overlaps stages |
| **Forwarding** | Data RAW hazards | Reduces stalls |
| **Branch Prediction** | Control hazards | Prevents pipeline flushes |
| **Superscalar** | 1 IPC limit | Executes instructions in parallel |
| **OoO Execution** | Stalls due to slow data (Cache miss) | Keeps the ALU busy by reordering work |
