# Digital Logic Design
> *The bedrock of all computing. From True/False to CPUs.*

---

## 0. MOSFET Fundamentals (The Building Blocks)
Before discussing logic gates, we must understand the physical switch: the **MOSFET** (Metal-Oxide-Semiconductor Field-Effect Transistor). In digital logic, we treat these as nearly ideal voltage-controlled switches.

### 0.1. Terminal Definitions
*   **Gate (G)**: The control terminal. Voltage applied here determines if current flows.
*   **Source (S)**: The terminal where charge carriers (electrons or holes) enter the channel.
*   **Drain (D)**: The terminal where carriers leave the channel.
*   **Body/Bulk (B)**: The substrate. Usually tied to GND for NMOS and VCC for PMOS to prevent the "Body Effect."

### 0.2. NMOS (N-channel MOSFET)
The "workhorse" of modern electronics.
*   **Switching Logic**: Turns **ON** when the Gate is **High** (Logic 1).
*   **Charge Carriers**: Electrons ($\text{e}^-$).
*   **Logic Role**: Used in the **Pull-Down Network (PDN)**.
*   **Strength**: Excellent at passing a strong **GND (0)**, but poor at passing a full **VCC (1)** because the source-to-gate voltage drops as the source rises.

### 0.3. PMOS (P-channel MOSFET)
The "complement" to NMOS.
*   **Switching Logic**: Turns **ON** when the Gate is **Low** (Logic 0).
*   **Charge Carriers**: Holes ($h^+$).
*   **Logic Role**: Used in the **Pull-Up Network (PUN)**.
*   **Strength**: Excellent at passing a strong **VCC (1)**, but poor at passing a full **GND (0)**.

### 0.4. CMOS (Complementary MOS) - Why use PMOS?
If PMOS has lower mobility, higher resistance, and higher capacitance, why not use only NMOS? The answer lies in **Signal Integrity** and **Static Power**.

1.  **"Strong" vs. "Weak" Logic**:
    *   **NMOS** is a **Strong 0 / Weak 1**. It can pull a node perfectly to GND, but as it pulls a node toward VCC, the $V_{gs}$ (gate-to-source voltage) decreases. Once the node reaches $VCC - V_{th}$, the transistor turns off. You can never reach a full 1 with just NMOS.
    *   **PMOS** is a **Strong 1 / Weak 0**. It can pull a node perfectly to VCC.
    *   By using both, we get **full-swing** logic (GND to VCC), which maximizes **Noise Margins**.
2.  **Zero Static Power (The "Killer App" of CMOS)**:
    *   Before CMOS, we used **NMOS Logic** (NMOS pull-down + a resistor pull-up).
    *   When the output was '0', current would flow continuously from VCC through the resistor to GND.
    *   In **CMOS**, the PMOS (PUN) and NMOS (PDN) are controlled by the same signal but react oppositely. In steady-state (either 0 or 1), one network is always fully OFF. **No DC current flows**, meaning your phone battery lasts all day instead of minutes.
3.  **Ratioed vs. Ratioless Logic**:
    *   In NMOS-only logic, you have to carefully size the resistor vs. the transistor to get the right output voltage.
    *   CMOS is **ratioless**; the logic works regardless of the size of the transistors (though sizing affects *speed*).

### 0.5. The "Threshold" Secret (Strong vs. Weak)
To understand why CMOS uses both, we must look at the **Threshold Voltage ($V_{th}$)**. Think of $V_{th}$ as the "toll" the transistor takes to stay open.

#### 0.5.1. The Control Logic (Gate)
First, let's clarify how we turn the switch ON or OFF:

| Transistor Type | Gate Voltage ($V_G$) | Switch State | Logic Interpretation |
| :--- | :--- | :--- | :--- |
| **NMOS** | **High (5V)** | **CLOSED (ON)** | Conducts when you give it "power" |
| **NMOS** | **Low (0V)** | OPEN (OFF) | Isolated |
| **PMOS** | **Low (0V)** | **CLOSED (ON)** | Conducts when you "ground" the gate |
| **PMOS** | **High (5V)** | OPEN (OFF) | Isolated |

#### 0.5.2. Why the signals are "Weak"
The "Strength" refers to the **voltage level being passed** across the switch (from Source to Drain).

*   **NMOS (Passing a 1)**:
    *   To stay open, an NMOS needs $V_{Gate} - V_{Source} \ge V_{th}$.
    *   If you put 5V on the Gate and try to pass 5V to the output (Source)...
    *   As the output rises, the difference ($V_{Gate} - V_{Source}$) gets smaller.
    *   When the output reaches **$VCC - V_{th}$** (around 4.3V), the transistor **shuts itself off**.
    *   It literally cannot reach 5V. It is a **Weak 1**.

*   **PMOS (Passing a 0)**:
    *   To stay open, a PMOS needs $V_{Source} - V_{Gate} \ge |V_{th}|$.
    *   If you put 0V on the Gate and try to pass 0V to the output (Source)...
    *   As the output drops, the difference ($V_{Source} - V_{Gate}$) gets smaller.
    *   When the output reaches **$|V_{th}|$** (around 0.7V), the transistor **shuts itself off**.
    *   It literally cannot reach 0V. It is a **Weak 0**.

**Summary**: NMOS is a champion at pulling down to 0V (Strong 0), and PMOS is a champion at pulling up to 5V (Strong 1). By using them as a team, we get a perfect 0V to 5V swing.

---

### 1.1. Core Axioms & Theorems
Digital logic usually operates on two States: **0 (Low/False/GND)** and **1 (High/True/VCC)**.
*   **Identity**: $A \cdot 1 = A$, $A + 0 = A$
*   **Null**: $A \cdot 0 = 0$, $A + 1 = 1$
*   **Idempotent**: $A \cdot A = A$, $A + A = A$
*   **Inverse**: $A \cdot \overline{A} = 0$, $A + \overline{A} = 1$
*   **De Morgan’s Laws** (Crucial for NAND/NOR logic):
    *   $\overline{A \cdot B} = \overline{A} + \overline{B}$ (Break the bar, change the sign)
    *   $\overline{A + B} = \overline{A} \cdot \overline{B}$

### 1.2. Universal Gates & CMOS Efficiency
A **universal gate** is a gate which can implement any Boolean function without the need for any other gate type. Both **NAND** and **NOR** are universal.

#### 1.2.1. Functonal Completeness
To prove universality, one must show that the gate can implement the fundamental set $\{NOT, AND, OR\}$:
*   **NOT**: $\overline{A} = \text{NAND}(A, A)$
*   **AND**: $A \cdot B = \text{NOT}(\text{NAND}(A, B))$
*   **OR**: $A + B = \text{NAND}(\text{NOT}(A), \text{NOT}(B))$ (via De Morgan's)

#### 1.2.2. Why NAND wins in CMOS
In modern VLSI, **NAND** is almost universally preferred over **NOR** for static logic implementation due to the following transistor-level physical properties:

1.  **Carrier Mobility ($\mu_n$ vs $\mu_p$)**:
    *   In a silicon substrate, electrons (majority carriers in **NMOS**) have roughly **2 to 3 times higher mobility** than holes (majority carriers in **PMOS**).
    *   $\mu_n \approx 3 \times \mu_p$.
2.  **Network Topology**:
    *   A CMOS gate consists of a **Pull-Down Network (PDN)** of NMOS and a **Pull-Up Network (PUN)** of PMOS.
    *   **NAND**: Has NMOS in **series** and PMOS in **parallel**.
    *   **NOR**: Has NMOS in **parallel** and PMOS in **series**.
3.  **The "Series PMOS" Problem**:
    *   To achieve equal rise and fall times (symmetrical switching), the resistance of the PUN and PDN must be matched.
    *   Because PMOS is slower, a PMOS transistor must be wider than an NMOS transistor (typically $W_p \approx 2W_n$).
    *   In a **NOR** gate, the PMOS are in **series**. The total resistance of two series PMOS transistors is $R_p + R_p = 2R_p$. To match a single NMOS resistance, each PMOS must be sized up even further.
    *   In a **NAND** gate, the NMOS are in series. Since NMOS is inherently fast, the penalty for series connection is much smaller.
4.  **Area & Parasitics**:
    *   The series PMOS chain in a NOR gate leads to **larger physical area** and **higher input capacitance** (due to the larger gates). Higher capacitance means lower switching speed ($t_{pd} \propto R \cdot C$).
    *   Therefore, a NAND gate is **faster, smaller, and has lower power consumption** than a NOR gate of equivalent drive strength.


### 1.3. Karnaugh Maps (K-Maps)
Used for manual simplification of boolean functions up to 4-5 variables.
*   **Gray Code Ordering**: 00, 01, 11, 10. (Only 1 bit changes between adjacent cells).
*   **Grouping**: Group 1s in powers of 2 (1, 2, 4, 8, 16).
*   **Implicants**:
    *   **Prime Implicant (PI)**: A group that is not a subset of another group.
    *   **Essential Prime Implicant (EPI)**: A PI that covers a '1' that no other PI covers. **Must** be included in the final SOP (Sum of Products).

### 1.4. Quine-McCluskey (Tabular) Method
Algorithmic minimization suitable for computer implementation (unlike visual K-Maps).
1.  List minterms and group by number of 1s (Hamming weight).
2.  Combine groups that differ by exactly 1 bit.
3.  Repeat until no further combination.
4.  Construct Prime Implicant Chart to select essential ones.

---

## 2. Combinational Logic
*Output depends ONLY on current Inputs.*

### 2.1. Arithmetic Circuits
*   **Half Adder**: Adds 2 bits. $Sum = A \oplus B$, $C_{out} = A \cdot B$.
*   **Full Adder**: Adds 3 bits (A, B, $C_{in}$).
    *   $Sum = A \oplus B \oplus C_{in}$
    *   $C_{out} = A \cdot B + C_{in} \cdot (A \oplus B)$
*   **Ripple Carry Adder (RCA)**: Chain of Full Adders. Slow. Delay = $N \times t_{FA}$.
*   **Carry Lookahead Adder (CLA)**:
    *   Calculates Carry *Generate* ($G = A \cdot B$) and *Propagate* ($P = A \oplus B$) signals.
    *   $C_{i+1} = G_i + P_i \cdot C_i$.
    *   Computes carries in parallel. Faster ($O(\log N)$) but higher area.
*   **Prefix Adders**: Kogge-Stone (Fastest, High Area), Brent-Kung (Lower Area, Higher Fan-out).

### 2.2. Multiplexers (MUX) & Decoders
*   **MUX**: "Many to One". Selects 1 of $2^n$ inputs based on $n$ select lines.
    *   Can implement *any* boolean function of $n+1$ variables using a $2^n:1$ MUX.
*   **Decoder**: "One to Many". $n$ inputs activates 1 of $2^n$ outputs (One-hot). Used in Memory Addressing (Row Decoders).
*   **Encoder**: $2^n$ inputs to $n$ binary code. **Priority Encoder** handles strictly multiple active inputs.

### 2.3. Hazards (Glitches)
Temporary unintended transitions due to unequal propagation delays.
*   **Static-1 Hazard**: Output should be 1, but glitches to 0 briefly. (Caused by adjacent 1s in K-Map not being grouped).
*   **Static-0 Hazard**: Output should be 0, but glitches to 1 briefly.
*   **Fix**: Add redundant terms (Consensus terms) to cover transitions between groups.

---

## 3. Sequential Logic (The Memory Layer)
*Combinational logic is "forgetful" (Output = f(Current Input)). Sequential logic introduces **State** (Output = f(Current Input, Previous State)).*

### 3.1. The Fundamental Problem: Timing Uncertainty
In a CPU, signals don't move instantly. They have "Propagation Delay." If you have a complex math circuit, the different bits of the answer might arrive at different times. If you immediately use that "half-finished" answer for the next step, your computer will calculate garbage.

We need a way to **"Capture"** data and hold it steady so the next part of the circuit can work with it safely.

### 3.2. Step 1: The Latch (Level-Triggered)
A latch is **Transparent**.
*   **Workflow**: When the Enable (EN) signal is High, the output (Q) follows the input (D). When EN goes Low, the output "freezes" (latches) the last value.
*   **The Problem (Transparency)**: If your EN signal stays High for too long, any noise or "glitches" from the math circuit will pass right through the latch. In a feedback loop (like a counter), a latch is dangerous because the signal could "race" around the loop multiple times while the clock is still high.
*   **Analogy**: A latch is like a **window**. When it's open, you see everything outside. When it's closed, you see a snapshot of the last thing you saw.

### 3.3. Step 2: The Flip-Flop (Edge-Triggered)
A Flip-Flop is **Opaque**. It only looks at the input for a literal fraction of a nanosecond during the **transition** (the edge) of the clock.
*   **Workflow**: 
    1.  **Rising Edge**: At the exact moment the clock goes from 0 $\to$ 1, the Flip-Flop samples the input.
    2.  **Snapshot**: It locks that value and ignores the input until the *next* rising edge.
*   **The Problem it Solves**: It eliminates "Race Conditions." Even if the input changes 100 times while the clock is High, the Flip-Flop doesn't care. It only saw the value at the edge.

### 3.4. Master-Slave Architecture (How it actually works)
To create an Edge-Triggered Flip-Flop, we actually put **two Latches** in series with an inverter on one of their Enable pins.
1.  **Master Latch**: Open when Clock is LOW. It "pre-fetches" the data.
2.  **Slave Latch**: Open when Clock is HIGH.
*   **Result**: 
    *   While Clock is 0: Master is open, Slave is closed. Data enters the Master but can't reach the output.
    *   While Clock is 1: Master closes (locking the data), Slave opens (releasing the data to the output).
*   **Execution**: This ensures data only "moves" once per clock cycle, at the exact moment the clock flips. This is the **heartbeat** of your CPU.

### 3.5. Timing Constraints (The GHz Bottleneck)
In a GHz-speed processor, a clock cycle might last only **1 nanosecond** ($10^{-9}$ s). During that time, the electricity has to travel through thousands of gates, flip them, and arrive at the next Flip-Flop.

#### 3.5.1. The "Setup/Hold" Window
A Flip-Flop is like a digital camera. It doesn't take a snapshot "instantly"; the sensor needs a tiny amount of time to register the light.

*   **Setup Time ($t_{su}$)**: The data must arrive and be **steady** (stable) for a certain amount of time **BEFORE** the clock edge. If the data is still "flickering" between 0 and 1 when the edge hits, the FF fails.
*   **Hold Time ($t_{h}$)**: The data must remain **steady** for a certain amount of time **AFTER** the clock edge to ensure the "shutter" has fully closed.

#### 3.5.2. Why "Stable" is hard at 5GHz
1.  **Gate Delay**: Every NAND gate takes about ~10-50 picoseconds to switch. If your logic path is 40 gates deep, that's up to 2.0ns of delay.
2.  **Clock Skew**: The clock signal itself takes time to travel across the chip. If the "start" FF gets the clock at $T=0$ but the "end" FF gets it at $T=0.1ns$, your window just shrank.
3.  **Metastability**: If data changes *exactly* during the Setup/Hold window, the Flip-Flop enters a "Metastable" state—it balances between 0 and 1 for a random amount of time. This causes crashes that are nearly impossible to debug.

#### 3.5.3. The Equation of Speed
To avoid failure, the total delay must fit within the clock period ($T$):
$$T_{clock} \ge t_{cq} + t_{logic} + t_{setup}$$
*   **$t_{cq}$**: Time for data to exit the first FF.
*   **$t_{logic}$**: Time for the math (Adders/MUXs) to finish.
*   **$t_{setup}$**: The pre-edge "stability" requirement.

> [!IMPORTANT]
> **If $t_{logic}$ is too high, the data arrives "unstable" (still changing).** This is why high-end CPUs have very "deep pipelines"—they break complex math into many small steps so that each $t_{logic}$ is tiny, allowing the clock frequency ($1/T$) to go higher.

---

## 4. Finite State Machines (FSM)

### 4.1. Models
*   **Mealy Machine**: Output $Y = F(State, Input)$. output changes immediately when input changes (asynchronous). Faster response, but prone to glitches.
*   **Moore Machine**: Output $Y = F(State)$. Output changes only on clock edges. Glitch-free, but 1 cycle latency.

### 4.2. State Encoding Strategies
How we map a "state" (like `IDLE`, `READ`, `WRITE`) to actual 0s and 1s in Flip-Flops. The choice of encoding significantly impacts the speed (maximum frequency) and power of your circuit.

#### 4.2.1. Binary Encoding (Compact)
Uses the mathematical minimum number of Flip-Flops: $n = \lceil \log_2(\text{number of states}) \rceil$.
*   **Workflow**: 4 states $\to$ 2 bits (`00, 01, 10, 11`). 
*   **Pros**: Uses the least amount of hardware (Flip-Flops).
*   **Cons**: Requires **complex decoding logic**. To determine if you are in state `11`, the circuit must check multiple bits ($Q_1 \text{ AND } Q_0$). This adds gate delay to your clock cycle.
*   **Best For**: Complex state machines where area (number of transistors) is the biggest concern.

#### 4.2.2. One-Hot Encoding (Fast)
Uses one Flip-Flop per state. Only **one** bit is 'High' at any time.
*   **Workflow**: 4 states $\to$ 4 bits (`1000, 0100, 0010, 0001`).
*   **Pros**: **Extreme Speed**. To check if you are in state 3, you just look at bit 3. No decoding gates needed! The "Next State" logic also becomes very simple (essentially just OR gates).
*   **Cons**: High area (uses $N$ Flip-Flops). 
*   **Best For**: **FPGAs** (which have thousands of spare Flip-Flops but limited logic gates) and ultra-high-speed control logic in CPUs.

#### 4.2.3. Gray Code (Low Power & Glitch-Free)
A binary sequence where only **one bit changes** at a time between any two sequential states.
*   **Workflow**: `000 \to 001 \to 011 \to 010 \to 110 \dots`
*   **Pros**:
    1.  **Low Power**: Every time a bit flips, it consumes energy ($CV^2 f$). Since only 1 bit flips per transition, switching power is minimized.
    2.  **Safe Transitions**: In standard binary, going from `01` to `10` requires two bits to change. If one bit is slightly faster, the circuit might briefly see `11` or `00` (a glitch). Gray code prevents this "Intermediate State" error.
*   **Best For**: Asynchronous interfaces, counters, and low-power mobile chips.

---

## 5. Logic Families & Electrical Characteristics

### 5.1. CMOS (Complementary MOS)
The dominant technology. Uses NMOS (Pull-Down) and PMOS (Pull-Up).
*   **Inverter**: PMOS connected to VDD, NMOS to GND. Gates connected to Input.
*   **Static Power**: Nearly zero (only leakage current $I_{leak}$).
*   **Dynamic Power**: Charging/Discharging Load Capacitance ($C_L$).
    $$P_{dynamic} = \alpha \cdot C_L \cdot V_{DD}^2 \cdot f$$
    *   $\alpha$: Activity factor (switching probability).
    *   To save power: Reduce Voltage (Quadratic savings!), Reduce Frequency (Linear), or Clock Gating ($\alpha \to 0$).

### 5.2. Transmission Gates & Pass Transistor Logic
*   Uses fewer transistors but suffers from voltage degradation (NMOS passes weak 1, PMOS passes weak 0).
*   Restoring logic (Buffers/Inverters) needed periodically.

### 5.3. Interfacing
*   **Fan-out**: Max number of standard loads a gate can drive without degrading noise margins or speed.
*   **Noise Margin**: $NM_L = V_{IL,max} - V_{OL,max}$, $NM_H = V_{OH,min} - V_{IH,min}$. High noise margin = robust design.
