# Blog 6: Weaving Qubits Together — Tensor Products, CNOT, Bell States, and Running Your First QPU Job

In Blogs 4 and 5, we mastered the anatomy and manipulation of a **single qubit**. We saw how any pure single-qubit state can be viewed as a two-key dictionary (`{"0": (mag, angle), "1": (mag, angle)}`) or a 3D unit vector on the Bloch Sphere, and how single-qubit gates ($X, Y, Z, H, S, T$) act as rigid 3D rotations and wave splitters.

A single qubit, however, is just one rotating dial. To solve real-world problems—like simulating entangled electrons in a 24-atom caffeine molecule, checking overlapping driver shift constraints in QAOA, or generating un-hackable cryptographic keys—we have to **wire multiple qubits together**.

When we move from one qubit to $n$ qubits, four profound shifts happen:

1. **State Space Explosion via the Tensor Product ($\otimes$):** Combining qubits doesn't add their possibilities; it multiplies them into a joint dictionary of $2^n$ binary bit-string keys (`"00"`, `"01"`, `"10"`, `"11"`, $\dots$).


2. **Conditional Multi-Qubit Gates (`CNOT`, Toffoli, Controlled-$U$):** Gates can now inspect one or more **control qubits** and conditionally transform a **target qubit** without ever measuring or collapsing the superposition.


3. **Partial Measurement & Renormalization:** Measuring just *one* qubit in a multi-qubit register filters out incompatible dictionary keys and renormalizes the surviving keys so their probabilities once again sum to $100\%$.


4. **Quantum Entanglement:** Two or more qubits can become woven into a single, non-factorizable joint distribution where measuring one qubit instantaneously dictates the state or probabilities of the others—even across arbitrary physical distances.



In this post, we will unpack every one of these concepts from the ground up, work through all the multi-qubit circuit traces and partial-measurement derivations from our course material, build our **Software Engineer's Unified Multi-Qubit Engine** in Python, and finish by running our first entangled **Bell State** circuit both locally on `AerSimulator` and on a real physical superconducting **IBM Quantum Processor (QPU)** in the cloud.

---

## 1. The Software Engineer's Translation Table: Multi-Qubit Systems Without the Shorthand

Before we write out a single tensor product or $4 \times 4$ matrix, let's establish how every multi-qubit physics notation translates directly into clean, verbose software engineering concepts:

| Chalkboard Physics Term | Textbook Symbol | What It Actually Is in Software Engineering |
| --- | --- | --- |
| **Tensor Product of Qubits** | $\Vert{}q_0\rangle \otimes \Vert{}q_1\rangle$ or $\Vert{}q_0 q_1\rangle$

 | **Cartesian String-Key Join (`outer_join`):** Combines two independent state dictionaries by concatenating their bit-string keys (`k0 + k1`), multiplying their magnitudes (`mag0 * mag1`), and adding their phase angles (`ang0 + ang1`).

 |
| **$n$-Qubit State Vector** | $\Vert{}\psi\rangle = \sum_{x \in \{0,1\}^n} c_x \Vert{}x\rangle$

 | **Joint Probability-Amplitude Map (`Dict[str, AmplitudePolar]`):** A sparse dictionary mapping $n$-character binary strings (`"00"`, `"01"`, ...) to `(magnitude, angle_deg)` tuples whose squared magnitudes sum to $1.0$.

 |
| **Circuit Time-Slice Labels** | $\Vert{}\psi_0\rangle, \Vert{}\psi_1\rangle, \Vert{}\psi_2\rangle, \dots$

 | **Immutable State Snapshots / Debugger Breakpoints:** Read-only snapshots of the joint state dictionary after each column (layer) of gates executes from left to right.

 |
| **CNOT / $CX$ Gate** | $\Vert{}c, t\rangle \to \Vert{}c, c \oplus t\rangle$

 | **Conditional Bit-String Key Remapper (Reversible `XOR`):** For every key in the dictionary where `key[control_idx] == '1'`, flip the character at `key[target_idx]` (`'0' <-> '1'`).

 |
| **Toffoli ($CCX$) Gate** | $\Vert{}c_1, c_2, t\rangle \to \Vert{}c_1, c_2, t \oplus (c_1 \cdot c_2)\rangle$

 | **Doubly-Controlled `AND-XOR` Remapper:** Flips `key[target_idx]` if and only if *both* `key[control1_idx] == '1'` and `key[control2_idx] == '1'`.

 |
| **Partial Measurement & Collapse** | Measure qubit $i \to b \in \{0,1\}$, multiply by normalization constant $A$

 | **Dictionary Filter + Renormalization (`filter_and_renormalize`):** Delete all keys where `key[i] != b`, then divide the surviving magnitudes by $\sqrt{P_{\text{surviving}}}$ so squared magnitudes sum to $100\%$ again.

 |
| **Separable vs. Entangled State** | $\Vert{}\Psi\rangle = \Vert{}\psi_0\rangle \otimes \Vert{}\psi_1\rangle$ vs. $\Vert{}\Psi\rangle \neq \Vert{}\psi_0\rangle \otimes \Vert{}\psi_1\rangle$

 | **Independent Variables vs. Coupled Joint Distribution:** A state is *separable* if it can be factored back into individual 1-qubit dictionaries; it is *entangled* if the qubits are conditionally dependent (`P(q1 |

---

## 2. Combining Qubits: The Tensor Product ($\otimes$) as a Cartesian Key Join

When we have multiple qubits in a quantum register, how do we represent their combined state mathematically? We use an operation called the **tensor product**, denoted by the symbol $\otimes$.

### A. Shorthand Ket Notation for Multi-Qubit Basis States

Suppose we have two qubits that are both in the $\vert{}0\rangle$ state. Using the tensor product, we write their combined state in three completely equivalent ways—from most verbose to most compact:

$$\vert{}0\rangle \otimes \vert{}0\rangle \;\equiv\; \vert{}0\rangle\vert{}0\rangle \;\equiv\; \vert{}00\rangle$$

Usually, we shorten $\vert{}0\rangle \otimes \vert{}0\rangle$ to $\vert{}00\rangle$, placing both bits inside a single ket vector.

### B. Expanding Superpositions Across Multiple Qubits

What if our two qubits are in arbitrary superposition states, say $\vert{}\psi_0\rangle = \alpha\vert{}0\rangle + \beta\vert{}1\rangle$ and $\vert{}\psi_1\rangle = \gamma\vert{}0\rangle + \delta\vert{}1\rangle$?

We expand their tensor product just like standard algebraic foil/distribution: we **multiply the amplitudes** of the states together and apply the **tensor product to the $\vert{}0\rangle$ and $\vert{}1\rangle$ basis kets**:

$$\vert{}\psi_0\rangle \otimes \vert{}\psi_1\rangle = (\alpha\vert{}0\rangle + \beta\vert{}1\rangle) \otimes (\gamma\vert{}0\rangle + \delta\vert{}1\rangle)$$

$$= \alpha\gamma(\vert{}0\rangle \otimes \vert{}0\rangle) + \alpha\delta(\vert{}0\rangle \otimes \vert{}1\rangle) + \beta\gamma(\vert{}1\rangle \otimes \vert{}0\rangle) + \beta\delta(\vert{}1\rangle \otimes \vert{}1\rangle)$$

$$= \alpha\gamma\vert{}00\rangle + \alpha\delta\vert{}01\rangle + \beta\gamma\vert{}10\rangle + \beta\delta\vert{}11\rangle$$

Notice that with $2$ qubits, our state now spans **four basis states**: $\vert{}00\rangle, \vert{}01\rangle, \vert{}10\rangle,$ and $\vert{}11\rangle$. This makes complete intuitive sense because `00`, `01`, `10`, and `11` are the four possible binary combinations of zeros and ones we can form with two bits! In column-vector notation, a 2-qubit state is ordered from top to bottom as a $4 \times 1$ vector whose four rows correspond to $\vert{}00\rangle, \vert{}01\rangle, \vert{}10\rangle,$ and $\vert{}11\rangle$:

$$\vert{}\Psi\rangle = c_{00}\vert{}00\rangle + c_{01}\vert{}01\rangle + c_{10}\vert{}10\rangle + c_{11}\vert{}11\rangle = \begin{bmatrix} c_{00} \\ c_{01} \\ c_{10} \\ c_{11} \end{bmatrix}$$

Full measurement of a multi-qubit register works the exact same way as Born's Rule for a single qubit:

* The probability of measuring the pair `00` is the magnitude squared of the $\vert{}00\rangle$ coefficient: $P(00) = \vert{}c_{00}\vert{}^2$.


* More generally, $P(xy) = \vert{}c_{xy}\vert{}^2$, and the sum of the probabilities across all $2^n$ basis states must equal $1$ ($\vert{}c_{00}\vert{}^2 + \vert{}c_{01}\vert{}^2 + \vert{}c_{10}\vert{}^2 + \vert{}c_{11}\vert{}^2 = 1$).



> **💡 Note on Bit-Ordering Conventions (Textbook Big-Endian vs. Qiskit Little-Endian):**
> * In standard physics textbooks and our math course videos, $\vert{}q_0 q_1\rangle$ is written left-to-right (**Big-Endian**), where the leftmost bit is the top wire ($q_0$) and the rightmost bit is the bottom wire ($q_1$).
> 
> 
> * In **Qiskit** (and classical computer architecture), bit-strings are indexed right-to-left (**Little-Endian**: $\vert{}q_1 q_0\rangle$), where the **rightmost bit represents Qubit 0 ($q_0$)** and the **leftmost bit represents Qubit 1 ($q_1$)** (just like binary numbers where bit $2^0$ is on the right). We will highlight this clearly whenever we inspect Qiskit histograms!
> 
> 
> 
> 

---

## 3. Reading Quantum Circuit Diagrams & Tracing Time Slices ($\vert{}\psi_0\rangle \to \vert{}\psi_4\rangle$)

In quantum computing, we represent algorithms visually as **quantum circuits**:

* Each **horizontal line (wire)** represents a single qubit flowing through time from **left to right**.


* Boxes and symbols placed on the wires represent **quantum gates** applied sequentially from left to right.


* Vertical dashed lines or state labels ($\vert{}\psi_0\rangle, \vert{}\psi_1\rangle, \vert{}\psi_2\rangle, \dots$) point to different points along the circuit to represent **snapshots of the system's state** at each step of the algorithm.


* Meter boxes at the far right signify **measurement gates**, which collapse the qubits and store the resulting classical bits onto a double-line classical register bus.



### Worked Course Walkthrough: Tracing a 3-Qubit Circuit Step by Step

Let's trace the exact 3-qubit circuit from our course material across five time slices ($\vert{}\psi_0\rangle \to \vert{}\psi_4\rangle$):

```text
          │psi_0⟩     │psi_1⟩     │psi_2⟩     │psi_3⟩     │psi_4⟩
          │           │           │           │           │
q0: ──|0⟩─┼───────────┼───────────┼────[ X ]──┼────[ M ]──┼──
          │           │           │           │           │
q1: ──|0⟩─┼────[ X ]──┼────[ H ]──┼───────────┼────[ M ]──┼──
          │           │           │           │           │
q2: ──|1⟩─┼───────────┼───────────┼────[ X ]──┼────[ M ]──┼──
          │           │           │           │           │

```

1. **At $\vert{}\psi_0\rangle$ (Initial State):** The three qubits start in the basis state $\vert{}0\rangle \otimes \vert{}0\rangle \otimes \vert{}1\rangle$:



$$\vert{}\psi_0\rangle = \vert{}001\rangle$$


2. **At $\vert{}\psi_1\rangle$ (Apply $X$ gate to the 2nd qubit):** The middle qubit flips from $\vert{}0\rangle$ to $\vert{}1\rangle$:



$$\vert{}\psi_1\rangle = \vert{}011\rangle$$


3. **At $\vert{}\psi_2\rangle$ (Apply Hadamard $H$ gate to the 2nd qubit):** Since the middle qubit is currently $\vert{}1\rangle$, the Hadamard gate transforms it into the minus state $\vert{}-\rangle = \frac{1}{\sqrt{2}}(\vert{}0\rangle - \vert{}1\rangle)$. Expanding the tensor product across the three wires gives:



$$\vert{}\psi_2\rangle = \vert{}0\rangle \otimes \left(\frac{1}{\sqrt{2}}(\vert{}0\rangle - \vert{}1\rangle)\right) \otimes \vert{}1\rangle = \frac{1}{\sqrt{2}}\vert{}001\rangle - \frac{1}{\sqrt{2}}\vert{}011\rangle$$


4. **At $\vert{}\psi_3\rangle$ (Apply $X$ gates to both the 1st and 3rd qubits):** By linearity, the $X$ gates on Qubit 1 and Qubit 3 act on each branch of the superposition individually, flipping the 1st bit (`0 -> 1`) and the 3rd bit (`1 -> 0`) in both terms:



$$\vert{}\psi_3\rangle = \frac{1}{\sqrt{2}}\vert{}100\rangle - \frac{1}{\sqrt{2}}\vert{}110\rangle$$


5. **At $\vert{}\psi_4\rangle$ (Measure all three qubits):** When we measure the register at $\vert{}\psi_4\rangle$, the superposition collapses into one of two classical 3-bit strings:


* We measure **`100`** with probability $\left\vert{}\frac{1}{\sqrt{2}}\right\vert{}^2 = \frac{1}{2}$ (**$50\%$**).


* We measure **`110`** with probability $\left\vert{}-\frac{1}{\sqrt{2}}\right\vert{}^2 = \frac{1}{2}$ (**$50\%$**).





---

## 4. Multi-Qubit Gates: CNOT ($CX$), Toffoli ($CCX$), and Controlled-$U$

Single-qubit gates rotate individual wires independently. To make qubits interact with one another, we introduce **controlled multi-qubit gates**.

### A. The CNOT (Controlled-NOT / $CX$) Gate

The most common and foundational multi-qubit gate in quantum computing is the **CNOT** (or **Controlled-$X$ / $CX$**) gate. It acts on two qubits:

* **The Control Qubit ($\bullet$):** Represented by a solid black dot on the control wire.


* **The Target Qubit ($\oplus$ or $[X]$):** Represented by a circled plus (modular addition / XOR) or an $X$ box connected vertically to the control dot.



```text
         CNOT (CX) GATE                  TOFFOLI (CCX) GATE              CONTROLLED-U GATE
      (1 Control, 1 Target)            (2 Controls, 1 Target)         (Arbitrary Single Gate U)

Control: ───────■───────         Control 1: ───────■───────      Control: ───────■───────
                │                                  │                             │
Target:  ───────⊕───────         Control 2: ───────■───────      Target:  ──────[U]──────
             (or [X])                              │
                                 Target:    ───────⊕───────

```

**How CNOT Works:**

* If the control qubit is **`0`**, CNOT **does nothing** to the target qubit.


* If the control qubit is **`1`**, CNOT **applies an $X$ (bit-flip) gate** to the target qubit (flipping `0 -> 1` and `1 -> 0`).



In algebraic shorthand, for any basis pair $\vert{}c, t\rangle$, $\text{CNOT}\vert{}c, t\rangle = \vert{}c, c \oplus t\rangle$ (where $\oplus$ is XOR). Across the four 2-qubit basis states $\{\vert{}00\rangle, \vert{}01\rangle, \vert{}10\rangle, \vert{}11\rangle\}$, its truth table and $4 \times 4$ unitary matrix are:

$$\begin{array}{c\|c} \text{Input } \vert{}c, t\rangle & \text{Output } \vert{}c, c \oplus t\rangle \\ \hline \vert{}00\rangle & \vert{}00\rangle \text{ (Control is 0: no flip)} \\ \vert{}01\rangle & \vert{}01\rangle \text{ (Control is 0: no flip)} \\ \vert{}10\rangle & \vert{}11\rangle \text{ (Control is 1: target flips } 0 \to 1\text{)} \\ \vert{}11\rangle & \vert{}10\rangle \text{ (Control is 1: target flips } 1 \to 0\text{)} \end{array} \qquad\Longleftrightarrow\qquad \text{CNOT} = \begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 0 & 1 \\ 0 & 0 & 1 & 0 \end{bmatrix}$$

Look at the block structure of that $4 \times 4$ matrix:

* The top-left $2 \times 2$ block is the **Identity matrix ($I = \begin{bmatrix}1&0\\0&1\end{bmatrix}$)**, which leaves the target untouched when the control is $\vert{}0\rangle$.


* The bottom-right $2 \times 2$ block is the **Pauli-$X$ matrix ($X = \begin{bmatrix}0&1\\1&0\end{bmatrix}$)**, which flips the target when the control is $\vert{}1\rangle$.


* Just like $X$, applying CNOT twice in a row returns every state to its original configuration ($\text{CNOT}^2 = I$), so **CNOT is its own inverse**!



#### Worked Course Examples: Applying CNOT to Superpositions

Because CNOT is a linear unitary operator, when it acts on a superposition state, it evaluates **each branch of the superposition simultaneously**:

* **Example 1 (1st Qubit = Control, 2nd Qubit = Target):** Suppose we apply CNOT to the state:



$$\vert{}\psi\rangle = \frac{1}{\sqrt{2}}\vert{}00\rangle + \frac{1}{\sqrt{2}}\vert{}10\rangle$$



In the first branch ($\vert{}00\rangle$), the control is `0`, so it stays $\vert{}00\rangle$. In the second branch ($\vert{}10\rangle$), the control is `1`, so the target flips from `0` to `1`, becoming $\vert{}11\rangle$:



$$\text{CNOT}_{0 \to 1}\left(\frac{1}{\sqrt{2}}\vert{}00\rangle + \frac{1}{\sqrt{2}}\vert{}10\rangle\right) = \frac{1}{\sqrt{2}}\vert{}00\rangle + \frac{1}{\sqrt{2}}\vert{}11\rangle$$


* **Example 2 (Multi-Branch Superposition):** Suppose we apply $\text{CNOT}_{0 \to 1}$ to $\frac{1}{\sqrt{3}}\vert{}01\rangle + \frac{1}{\sqrt{3}}\vert{}10\rangle + \frac{1}{\sqrt{3}}\vert{}11\rangle$. The $\vert{}01\rangle$ branch stays $\vert{}01\rangle$, while $\vert{}10\rangle \to \vert{}11\rangle$ and $\vert{}11\rangle \to \vert{}10\rangle$!



### B. The Toffoli ($CCX$) Gate: Double Control

The **Toffoli gate** (also called the **CCNOT** or **$CCX$** gate) is identical to the CNOT gate except that it has **two control qubits** and one target qubit. It applies an $X$ (flip) gate to the target qubit **if and only if both control qubits are in the $\vert{}1\rangle$ state**.

* **Worked Course Example (4-Qubit Register):** Suppose we have a 4-qubit superposition and we apply a Toffoli gate where the **2nd and 3rd qubits are the controls** and the **4th qubit is the target**. For any basis branch $\vert{}q_0 q_1 q_2 q_3\rangle$ in the superposition:


* If $q_1 = 1$ and $q_2 = 1$ (for instance, $\vert{}0\mathbf{11}0\rangle$ or $\vert{}1\mathbf{11}1\rangle$), the 4th qubit flips ($\vert{}0\mathbf{11}0\rangle \to \vert{}0\mathbf{11}1\rangle$ and $\vert{}1\mathbf{11}1\rangle \to \vert{}1\mathbf{11}0\rangle$).


* For any branch where either $q_1 = 0$ or $q_2 = 0$ (such as $\vert{}1\mathbf{01}0\rangle$), the Toffoli gate does nothing!





### C. Arbitrary Controlled-$U$ Gates ($CY, CZ, CS, CT, CH$)

It turns out that we can use CNOT and single-qubit gates to construct **controlled versions of any single-qubit gate $U$**—including controlled-$Y$ ($CY$), controlled-$Z$ ($CZ$), controlled-$S$ ($CS$), controlled-$T$ ($CT$), and controlled-Hadamard ($CH$) gates.

* On a circuit diagram, a Controlled-$U$ gate is drawn as a regular box labeled $U$ on the target wire with a vertical line connecting to a solid control dot $\bullet$ on the control wire (for example, a Controlled-$S$ gate where the 2nd qubit is the control and the 1st qubit is the target).


* It works by the exact same rule: **apply gate $U$ to the target qubit if the control qubit is `1`, and do nothing ($I$) if the control qubit is `0**`.



---

## 5. Partial Measurement & Wave-Function Renormalization

Now that we have multiple qubits, what happens if we **measure only one qubit** in a multi-qubit register without measuring the others? This is a critical operation in quantum algorithms (like Quantum Phase Estimation and Shor's algorithm in Blogs 9 and 10) and takes two steps:

### Step 1: Finding the Probability of a Single-Qubit Outcome

Suppose we have a 2-qubit register in the general state:

$$\vert{}\psi\rangle = \alpha\vert{}00\rangle + \beta\vert{}01\rangle + \gamma\vert{}10\rangle + \delta\vert{}11\rangle$$

* **Question A:** *What is the probability of measuring the **second qubit** as a `1`?*
We simply look at **all the superposition states where the second qubit is `1**` (namely $\vert{}0\mathbf{1}\rangle$ and $\vert{}1\mathbf{1}\rangle$) and **sum the probabilities** of measuring those states:



$$P(\text{2nd qubit} = 1) = \vert{}\beta\vert{}^2 + \vert{}\delta\vert{}^2$$


* **Question B:** *What is the probability of measuring a `0` in the **first qubit**?*
We sum the probabilities of each state where the **first qubit is `0**` (namely $\vert{}\mathbf{0}0\rangle$ and $\vert{}\mathbf{0}1\rangle$):



$$P(\text{1st qubit} = 0) = \vert{}\alpha\vert{}^2 + \vert{}\beta\vert{}^2$$



### Step 2: Collapsing & Renormalizing the Surviving State

Once we measure that single qubit, **what state does the multi-qubit system collapse into**? Remember: measuring a qubit permanently collapses its superposition to the measured value.

#### Worked Course Example 1: Partial Measurement on a 2-Qubit State

Suppose we have the 2-qubit state:

$$|\psi\rangle = \frac{1}{2}|00\rangle + \frac{1}{2}|10\rangle + \frac{1}{\sqrt{2}}|11\rangle$$

*(Quick sanity check: do the probabilities sum to $1$? $\left|\frac{1}{2}\right|^2 + \left|\frac{1}{2}\right|^2 + \left|\frac{1}{\sqrt{2}}\right|^2 = \frac{1}{4} + \frac{1}{4} + \frac{1}{2} = 1$. Valid!)*

Now suppose **we measure the first qubit and our detector clicks `1**`. How do we find the new collapsed state $|\psi_{\text{collapsed}}\rangle$?

1. **Prune Incompatible Branches:** Because the first qubit is now definitely `1`, we **remove all superposition states where the first qubit is not `1**` (so we Cross out $\frac{1}{2}|\mathbf{0}0\rangle$):



$$\text{Unnormalized surviving terms: } \frac{1}{2}|10\rangle + \frac{1}{\sqrt{2}}|11\rangle$$


2. **Introduce a Normalization Constant $A$:** Wait—remember that the probabilities of a valid quantum state must always add up to $1$ ($100\%$). Because we just deleted the $|00\rangle$ state, the remaining probabilities only add up to $\left(\frac{1}{2}\right)^2 + \left(\frac{1}{\sqrt{2}}\right)^2 = \frac{1}{4} + \frac{1}{2} = \frac{3}{4} < 1$!
We fix this by multiplying the surviving state by a **normalization constant $A$**:



$$|\psi_{\text{collapsed}}\rangle = A\left(\frac{1}{2}|10\rangle + \frac{1}{\sqrt{2}}|11\rangle\right) = \frac{A}{2}|10\rangle + \frac{A}{\sqrt{2}}|11\rangle$$


3. **Solve for $A$ Using the Total Probability $= 1$ Identity:**


$$\left(\frac{A}{2}\right)^2 + \left(\frac{A}{\sqrt{2}}\right)^2 = 1 \quad \Longrightarrow \quad \frac{A^2}{4} + \frac{A^2}{2} = 1 \quad \Longrightarrow \quad \frac{3}{4}A^2 = 1 \quad \Longrightarrow \quad A^2 = \frac{4}{3} \quad \Longrightarrow \quad A = \frac{2}{\sqrt{3}}$$



*(Notice a neat shortcut: $A$ is always $\frac{1}{\sqrt{P_{\text{measured}}}}$, since the probability of measuring the first qubit as `1` was $P(\text{1st}=1) = \frac{1}{4} + \frac{1}{2} = \frac{3}{4}$, and $\frac{1}{\sqrt{3/4}} = \frac{2}{\sqrt{3}}$!)*


4. **Substitute $A = \frac{2}{\sqrt{3}}$ Back In:**


$$|\psi_{\text{collapsed}}\rangle = \frac{2}{\sqrt{3}}\left(\frac{1}{2}|10\rangle + \frac{1}{\sqrt{2}}|11\rangle\right) = \frac{1}{\sqrt{3}}|10\rangle + \frac{\sqrt{2}}{\sqrt{3}}|11\rangle = \mathbf{\frac{1}{\sqrt{3}}|10\rangle + \sqrt{\frac{2}{3}}|11\rangle}$$



Check the new probabilities: $\left(\frac{1}{\sqrt{3}}\right)^2 + \left(\sqrt{\frac{2}{3}}\right)^2 = \frac{1}{3} + \frac{2}{3} = 1$ ($100\%$)!



#### Worked Course Example 2: Partial Measurement on a 3-Qubit State

We can apply this exact same technique to any number of qubits. Suppose we have the 3-qubit state:

$$|\psi\rangle = \frac{1}{2}|000\rangle + \frac{1}{2}|101\rangle + \frac{1}{\sqrt{2}}|111\rangle$$

and we **measure the middle (2nd) qubit to be `0**`:

1. **Prune:** Discard any state where the middle bit is not `0` (eliminating $\frac{1}{\sqrt{2}}|1\mathbf{1}1\rangle$), leaving $|0\mathbf{0}0\rangle$ and $|1\mathbf{0}1\rangle$:



$$|\psi_{\text{collapsed}}\rangle = A\left(\frac{1}{2}|000\rangle + \frac{1}{2}|101\rangle\right)$$


2. **Normalize:** Require the squared magnitudes to sum to $1$:



$$\left(\frac{A}{2}\right)^2 + \left(\frac{A}{2}\right)^2 = 1 \quad \Longrightarrow \quad \frac{A^2}{4} + \frac{A^2}{4} = 1 \quad \Longrightarrow \quad \frac{A^2}{2} = 1 \quad \Longrightarrow \quad A = \sqrt{2}$$


3. **Final Collapsed State:** Substituting $A = \sqrt{2}$ and noting that $\frac{\sqrt{2}}{2} = \frac{1}{\sqrt{2}}$ gives:



$$|\psi_{\text{collapsed}}\rangle = \mathbf{\frac{1}{\sqrt{2}}|000\rangle + \frac{1}{\sqrt{2}}|101\rangle}$$



---

## 6. Quantum Entanglement: Separable vs. Maximally vs. Partially Entangled States

Now let's examine the most famous two-gate circuit in quantum physics:

```text
          │psi_0⟩     │psi_1⟩     │psi_2⟩
          │           │           │
q0: ──|0⟩─┼────[ H ]──┼─────■─────┼──
          │           │     │     │
q1: ──|0⟩─┼───────────┼────[X]────┼──
          │           │           │

```

Let's trace the state at each checkpoint:

1. **At $|\psi_0\rangle$:** Both qubits start in $|00\rangle$.


2. **At $|\psi_1\rangle$ (Hadamard on 1st Qubit):** The first qubit transforms into $|+\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$ while the second stays $|0\rangle$. Distributing $|0\rangle$ into the equation gives:



$$|\psi_1\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)|0\rangle = \frac{1}{\sqrt{2}}|00\rangle + \frac{1}{\sqrt{2}}|10\rangle$$


3. **At $|\psi_2\rangle$ (CNOT with 1st Qubit as Control, 2nd as Target):** The $|00\rangle$ branch stays $|00\rangle$ (control is `0`), while the $|10\rangle$ branch flips its second qubit to become $|11\rangle$ (control is `1`):



$$|\psi_2\rangle = \mathbf{\frac{1}{\sqrt{2}}|00\rangle + \frac{1}{\sqrt{2}}|11\rangle}$$



Look closely at $|\psi_2\rangle = \frac{1}{\sqrt{2}}|00\rangle + \frac{1}{\sqrt{2}}|11\rangle$:

* If we measure the first qubit and get **`0`**, the $\vert{}11\rangle$ branch vanishes and the state collapses to $\vert{}00\rangle$—meaning the second qubit is **guaranteed ($100\%$) to be `0**`!


* If we measure the first qubit and get **`1`**, the $\vert{}00\rangle$ branch vanishes and the state collapses to $\vert{}11\rangle$—meaning the second qubit is **guaranteed ($100\%$) to be `1**`!



Without even looking at the second qubit—even if it is sitting in a laboratory on the other side of the planet—measuring the first qubit immediately tells us the state of the second qubit. Einstein famously called this *"spooky action at a distance"*; in quantum mechanics, we call it **Quantum Entanglement**.

### A. The Mathematical Test for Entanglement: Factorizability

How do we know mathematically whether a multi-qubit state is entangled or not?

> **Definition:** A multi-qubit state is **entangled** if and only if it **cannot be factored** into the tensor product of individual single-qubit states ($\vert{}\Psi\rangle \neq \vert{}\psi_0\rangle \otimes \vert{}\psi_1\rangle$). Conversely, if a state *can* be factored into single-qubit states, it is **not entangled** (we call it a **separable** or **product state**).
> 
> 

Let's compare two examples from the course material:

1. **A Separable (Non-Entangled) State:** Consider the state:



$$|\Psi_{\text{sep}}\rangle = \frac{1}{2}|00\rangle + \frac{1}{2}|01\rangle + \frac{1}{2}|10\rangle + \frac{1}{2}|11\rangle$$



Can we factor this? Yes! Factor $|0\rangle$ out of the first two terms and $|1\rangle$ out of the last two terms:



$$|\Psi_{\text{sep}}\rangle = \left(\frac{1}{\sqrt{2}}|0\rangle + \frac{1}{\sqrt{2}}|1\rangle\right) \otimes \left(\frac{1}{\sqrt{2}}|0\rangle + \frac{1}{\sqrt{2}}|1\rangle\right) = |+\rangle \otimes |+\rangle$$



Because each qubit exists in its own independent $|+\rangle$ state, measuring the first qubit as `0` or `1` leaves the second qubit in $\frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$—its probabilities don't change at all!


2. **An Entangled State:** Now try factoring $|\Phi^+\rangle = \frac{1}{\sqrt{2}}|00\rangle + \frac{1}{\sqrt{2}}|11\rangle$ into $(\alpha|0\rangle + \beta|1\rangle) \otimes (\gamma|0\rangle + \delta|1\rangle) = \alpha\gamma|00\rangle + \alpha\delta|01\rangle + \beta\gamma|10\rangle + \beta\delta|11\rangle$.


* For the $\vert{}01\rangle$ and $\vert{}10\rangle$ terms to equal $0$, we would need $\alpha\delta = 0$ and $\beta\gamma = 0$.


* But if $\alpha = 0$ or $\delta = 0$, then either $\alpha\gamma = 0$ (killing $\vert{}00\rangle$) or $\beta\delta = 0$ (killing $\vert{}11\rangle$)!
* Therefore, **it is algebraically impossible to factor $\frac{1}{\sqrt{2}}\vert{}00\rangle + \frac{1}{\sqrt{2}}\vert{}11\rangle$ into single-qubit states**. When qubits are entangled, they depend on each other to determine their state.





### B. Maximally Entangled States (The 4 Canonical Bell States)

As taught in our course material, entangled states fall into two categories: **maximally entangled** and **partially entangled**.

We say qubits are **maximally entangled** if measuring one of the qubits **completely and deterministically tells us the exact basis state ($100\%$ certainty) that the other qubit is in**. The four canonical maximally entangled 2-qubit states are called the **Bell States**, denoted by capital Greek letters Phi ($\vert{}\Phi^\pm\rangle$) and Psi ($\vert{}\Psi^\pm\rangle$), and all four are generated by feeding the four 2-bit inputs into the same $H \to \text{CNOT}$ circuit:

| Input State $\Vert{}q_0 q_1\rangle$ | How to Prepare Input from $\Vert{}00\rangle$ | Output After $H$ on $q_0 \to \text{CNOT}_{0\to 1}$ | Bell State Symbol & Expanded Ket | Correlation Rule Upon Measurement |
| --- | --- | --- | --- | --- |
| $\Vert{}00\rangle$

 | Default (`pass`)

 | $\text{CNOT}(H \otimes I)\Vert{}00\rangle$

 | **$\Vert{}\Phi^+\rangle = \frac{1}{\sqrt{2}}(\Vert{}00\rangle + \Vert{}11\rangle)$**

 | Qubits **always match** (`00` or `11`), $0^\circ$ relative phase.

 |
| $\Vert{}10\rangle$

 | Apply $X$ on $q_0$ before $H$

 | $\text{CNOT}(H \otimes I)\Vert{}10\rangle$

 | **$\Vert{}\Phi^-\rangle = \frac{1}{\sqrt{2}}(\Vert{}00\rangle - \Vert{}11\rangle)$**

 | Qubits **always match** (`00` or `11`), $180^\circ$ ($\pi$) relative phase.

 |
| $\Vert{}01\rangle$

 | Apply $X$ on $q_1$ before CNOT

 | $\text{CNOT}(H \otimes I)\Vert{}01\rangle$

 | **$\Vert{}\Psi^+\rangle = \frac{1}{\sqrt{2}}(\Vert{}01\rangle + \Vert{}10\rangle)$**

 | Qubits are **always opposite** (`01` or `10`), $0^\circ$ relative phase.

 |
| $\Vert{}11\rangle$

 | Apply $X$ on both $q_0$ and $q_1$

 | $\text{CNOT}(H \otimes I)\Vert{}11\rangle$

 | **$\Vert{}\Psi^-\rangle = \frac{1}{\sqrt{2}}(\Vert{}01\rangle - \Vert{}10\rangle)$**

 | Qubits are **always opposite** (`01` or `10`), $180^\circ$ ($\pi$) relative phase.

 |

*(Note: This table also solves the Episode 1 & Episode 2 learning challenge from the course videos—how to program all four Bell states in code by adding $X$ or $Z$ gates!)*

### C. Partially Entangled States

What if measuring one qubit does *not* lock the other qubit into a $100\%$ definite `0` or `1`, yet **measuring one qubit still changes the probabilities or relative phase of the other qubit**? We call such states **partially entangled**.

Let's verify this on the exact example from our course material—the 2-qubit state we partially measured earlier:

$$|\Psi_{\text{partial}}\rangle = \frac{1}{2}|00\rangle + \frac{1}{2}|10\rangle + \frac{1}{\sqrt{2}}|11\rangle$$

Watch what happens to the **second qubit** depending on what we measure on the **first qubit**:

* **Case 1 — We measure the 1st qubit as `0`:** We discard the $\vert{}10\rangle$ and $\vert{}11\rangle$ terms and normalize $\frac{1}{2}\vert{}00\rangle$, collapsing the system into:



$$|\psi_{\text{if } q_0=0}\rangle = |00\rangle \quad \Longrightarrow \quad \text{2nd qubit has } \mathbf{100\% \text{ chance of } 0, \; 0\% \text{ chance of } 1}$$


* **Case 2 — We measure the 1st qubit as `1`:** As we calculated in Section 5, the system collapses into:



$$|\psi_{\text{if } q_0=1}\rangle = \frac{1}{\sqrt{3}}|10\rangle + \sqrt{\frac{2}{3}}|11\rangle \quad \Longrightarrow \quad \text{2nd qubit has } \mathbf{33.3\% \left(\frac{1}{3}\right) \text{ chance of } 0, \; 66.7\% \left(\frac{2}{3}\right) \text{ chance of } 1}$$



Because the measurement outcome of the first qubit shifts the probability distribution of the second qubit (from $100\%/0\%$ to $33.3\%/66.7\%$), the two qubits are **partially entangled**!

---

## 7. The Software Engineer's Unified View: Multi-Qubit Tensor, Gate & Renormalization Engine

Let's translate everything we have derived in Sections 1–6 into our explicit, self-documenting Python architecture. Notice how our `MultiQubitRegisterDict` class below handles any number of qubits using a sparse dictionary of binary strings (`"00"`, `"01"`, `"101"`, etc.) and implements:

1. `tensor_product()` as an explicit Cartesian dictionary join.


2. `apply_controlled_gate()` for single-qubit gates, CNOT, Toffoli, and arbitrary Controlled-$U$ gates.


3. `measure_single_qubit_with_renormalization()` showing the exact pruning and normalization constant $A$ calculation.


4. `classify_2qubit_entanglement()` to automatically test whether a 2-qubit dictionary is **Separable**, **Partially Entangled**, or **Maximally Entangled**!



```python
import numpy as np
from dataclasses import dataclass
from typing import Dict, List, Tuple

@dataclass
class AmplitudePolar:
    magnitude: float
    angle_deg: float

    def to_complex(self) -> complex:
        return self.magnitude * np.exp(1j * np.radians(self.angle_deg))

    @staticmethod
    def from_complex(c: complex) -> 'AmplitudePolar':
        mag = float(np.abs(c))
        if mag < 1e-9:
            return AmplitudePolar(0.0, 0.0)
        return AmplitudePolar(mag, float(np.degrees(np.angle(c)) % 360.0))


class MultiQubitRegisterDict:
    """
    Unified Multi-Qubit State Representation:
    Maps n-bit binary strings (e.g., '00', '11', '101') -> AmplitudePolar(mag, angle_deg).
    Zero-amplitude keys are omitted (Dirac Sparse Dictionary).
    """
    def __init__(self, num_qubits: int, state_map: Dict[str, AmplitudePolar]):
        self.num_qubits = num_qubits
        # Keep only non-zero keys
        self.state_map = {k: v for k, v in state_map.items() if v.magnitude > 1e-9}

    @staticmethod
    def from_basis_bitstring(bitstring: str) -> 'MultiQubitRegisterDict':
        return MultiQubitRegisterDict(len(bitstring), {bitstring: AmplitudePolar(1.0, 0.0)})

    def tensor_product(self, other: 'MultiQubitRegisterDict') -> 'MultiQubitRegisterDict':
        """
        Software Engineer's View of (|psi_A> ⊗ |psi_B>):
        Cartesian join of keys ('kA' + 'kB'), multiplying magnitudes and adding angles.
        """
        joined_map: Dict[str, AmplitudePolar] = {}
        for k1, amp1 in self.state_map.items():
            for k2, amp2 in other.state_map.items():
                joined_key = k1 + k2
                joined_mag = amp1.magnitude * amp2.magnitude
                joined_ang = (amp1.angle_deg + amp2.angle_deg) % 360.0
                joined_map[joined_key] = AmplitudePolar(joined_mag, joined_ang)
        return MultiQubitRegisterDict(self.num_qubits + other.num_qubits, joined_map)

    def apply_controlled_gate(
        self, target_qubit: int, gate_matrix_2x2: np.ndarray, control_qubits: List[int] = None
    ) -> 'MultiQubitRegisterDict':
        """
        Applies a 2x2 unitary gate to `target_qubit` conditioned on all `control_qubits` == '1'.
        Covers single-qubit gates (control_qubits=[]), CNOT (1 control), Toffoli (2 controls), and Controlled-U.
        """
        if control_qubits is None:
            control_qubits = []
        accum_complex: Dict[str, complex] = {}

        for bitstring, amp in self.state_map.items():
            c_in = amp.to_complex()
            controls_satisfied = all(bitstring[c_idx] == '1' for c_idx in control_qubits)

            if not controls_satisfied:
                accum_complex[bitstring] = accum_complex.get(bitstring, 0j) + c_in
            else:
                target_bit = int(bitstring[target_qubit])
                for new_bit_val in (0, 1):
                    transition_factor = gate_matrix_2x2[new_bit_val, target_bit]
                    if np.abs(transition_factor) > 1e-9:
                        new_key = (
                            bitstring[:target_qubit]
                            + str(new_bit_val)
                            + bitstring[target_qubit + 1:]
                        )
                        accum_complex[new_key] = (
                            accum_complex.get(new_key, 0j) + c_in * transition_factor
                        )

        new_map = {k: AmplitudePolar.from_complex(v) for k, v in accum_complex.items()}
        return MultiQubitRegisterDict(self.num_qubits, new_map)

    def partial_measure_and_renormalize(
        self, qubit_index: int, forced_outcome: str
    ) -> Tuple[float, float, 'MultiQubitRegisterDict']:
        """
        Course Partial Measurement Rule:
        1. Sum probabilities of all keys where key[qubit_index] == forced_outcome.
        2. Prune keys where key[qubit_index] != forced_outcome.
        3. Multiply surviving amplitudes by normalization constant A = 1 / sqrt(prob_sum).
        """
        surviving = {
            k: v for k, v in self.state_map.items() if k[qubit_index] == forced_outcome
        }
        outcome_prob = sum(v.magnitude ** 2 for v in surviving.values())
        norm_constant_A = 1.0 / np.sqrt(outcome_prob)

        renormalized_map = {
            k: AmplitudePolar(v.magnitude * norm_constant_A, v.angle_deg)
            for k, v in surviving.items()
        }
        return outcome_prob, norm_constant_A, MultiQubitRegisterDict(self.num_qubits, renormalized_map)

    def describe(self, label: str):
        print(f"=== {label} ===")
        for k in sorted(self.state_map.keys()):
            amp = self.state_map[k]
            prob = (amp.magnitude ** 2) * 100
            print(f"  |{k}⟩ : (mag={amp.magnitude:.4f}, ang={amp.angle_deg:5.1f}°) --> Odds: {prob:5.1f}%")
        print()


X_MAT = np.array([[0, 1], [1, 0]], dtype=complex)
Z_MAT = np.array([[1, 0], [0, -1]], dtype=complex)
H_MAT = (1 / np.sqrt(2)) * np.array([[1, 1], [1, -1]], dtype=complex)

# 1. Course 3-Qubit Circuit Trace (|psi_0> -> |psi_4>)
psi = MultiQubitRegisterDict.from_basis_bitstring("001")          # |psi_0> = |001>
psi = psi.apply_controlled_gate(target_qubit=1, gate_matrix_2x2=X_MAT)  # |psi_1> = |011>
psi = psi.apply_controlled_gate(target_qubit=1, gate_matrix_2x2=H_MAT)  # |psi_2> = (|001> - |011>)/sqrt(2)
psi = psi.apply_controlled_gate(target_qubit=0, gate_matrix_2x2=X_MAT)
psi = psi.apply_controlled_gate(target_qubit=2, gate_matrix_2x2=X_MAT)  # |psi_3> = (|100> - |110>)/sqrt(2)
psi.describe("1. Course 3-Qubit Circuit at |psi_3⟩: (|100⟩ - |110⟩)/√2")

# 2. Course Partial Measurement & Renormalization Walkthrough (2 Qubits)
partial_state = MultiQubitRegisterDict(2, {
    "00": AmplitudePolar(0.5, 0.0),
    "10": AmplitudePolar(0.5, 0.0),
    "11": AmplitudePolar(1 / np.sqrt(2), 0.0),
})
prob_q0_is_1, A_const, collapsed_2q = partial_state.partial_measure_and_renormalize(
    qubit_index=0, forced_outcome="1"
)
print(f"2. Partial Measurement of q0='1': P(q0='1') = {prob_q0_is_1:.4f} (3/4), "
      f"Normalization A = {A_const:.4f} (2/√3 = {2/np.sqrt(3):.4f})")
collapsed_2q.describe("   Collapsed 2-Qubit State: (1/√3)|10⟩ + √(2/3)|11⟩")

# 3. Generate All 4 Canonical Bell States from {"00", "10", "01", "11"}
print("=== 3. Generating the 4 Canonical Bell States (H on q0 -> CNOT 0->1) ===")
bell_names = {"00": "|Φ+⟩", "10": "|Φ-⟩", "01": "|Ψ+⟩", "11": "|Ψ-⟩"}
for init_bits, b_name in bell_names.items():
    reg = MultiQubitRegisterDict.from_basis_bitstring(init_bits)
    reg = reg.apply_controlled_gate(target_qubit=0, gate_matrix_2x2=H_MAT)
    reg = reg.apply_controlled_gate(target_qubit=1, gate_matrix_2x2=X_MAT, control_qubits=[0])
    terms = [f"({v.magnitude:.3f} @ {v.angle_deg:5.1f}°)|{k}⟩" for k, v in sorted(reg.state_map.items())]
    print(f"  Input |{init_bits}⟩  -->  {b_name} = {' + '.join(terms)}")

```

**Output:**

```text
=== 1. Course 3-Qubit Circuit at |psi_3⟩: (|100⟩ - |110⟩)/√2 ===
  |100⟩ : (mag=0.7071, ang=  0.0°) --> Odds:  50.0%
  |110⟩ : (mag=0.7071, ang=180.0°) --> Odds:  50.0%

2. Partial Measurement of q0='1': P(q0='1') = 0.7500 (3/4), Normalization A = 1.1547 (2/√3 = 1.1547)
===    Collapsed 2-Qubit State: (1/√3)|10⟩ + √(2/3)|11⟩ ===
  |10⟩ : (mag=0.5774, ang=  0.0°) --> Odds:  33.3%
  |11⟩ : (mag=0.8165, ang=  0.0°) --> Odds:  66.7%

=== 3. Generating the 4 Canonical Bell States (H on q0 -> CNOT 0->1) ===
  Input |00⟩  -->  |Φ+⟩ = (0.707 @   0.0°)|00⟩ + (0.707 @   0.0°)|11⟩
  Input |10⟩  -->  |Φ-⟩ = (0.707 @   0.0°)|00⟩ + (0.707 @ 180.0°)|11⟩
  Input |01⟩  -->  |Ψ+⟩ = (0.707 @   0.0°)|01⟩ + (0.707 @   0.0°)|10⟩
  Input |11⟩  -->  |Ψ-⟩ = (0.707 @   0.0°)|01⟩ + (0.707 @ 180.0°)|10⟩

```

---

## 8. Hands-On Lab: Running Your First Entangled Bell State on Local CPU and Cloud IBM QPU

Now it is time to execute the **6-Stage Quantum Pipeline** we introduced in Blog 1! We are going to take the canonical Bell state circuit ($\vert{}\Phi^+\rangle = \frac{\vert{}00\rangle + \vert{}11\rangle}{\sqrt{2}}$), run `1,000` shots locally on your laptop using `AerSimulator`, and then dispatch the exact same circuit to a real physical **127+ qubit superconducting IBM Quantum Processor** sitting inside a dilution refrigerator at $15\text{ mK}$ ($-273^\circ\text{C}$).

### Part A: Local Simulation on Your Laptop (`AerSimulator` & Q-Sphere Visualization)

Any normal laptop can simulate our 2-qubit Bell circuit in ~10 milliseconds without touching the cloud or using any runtime quota.

*(Bonus Visualization Tip: Remember from Blog 4 that a single Bloch sphere cannot plot an entangled 2-qubit state because entangled qubits cannot be factored into individual single-qubit vectors! In Qiskit, to visualize multi-qubit entangled states in 3D, we use **`plot_state_qsphere(state)`**, which plots each basis bit-string like `|00⟩` and `|11⟩` on a global sphere where blob size represents probability and color represents phase!)*

```python
from qiskit import QuantumCircuit, transpile
from qiskit.quantum_info import Statevector
from qiskit_aer import AerSimulator

# 1. Create a circuit with 2 Quantum bits (q0, q1) and 2 Classical bits (c0, c1)
qc = QuantumCircuit(2, 2)

# 2. Apply Hadamard (H) to Qubit 0 -> puts q0 into equal superposition |+>
qc.h(0)

# 3. Apply CNOT (CX) with Control = q0, Target = q1 -> entangles q0 and q1 into |Φ+⟩
qc.cx(0, 1)

# Optional: Inspect exact statevector before measurement (or pass to plot_state_qsphere(state))
state = Statevector.from_instruction(qc)

# 4. Measure both qubits into classical register bits [0, 1]
qc.measure([0, 1], [0, 1])

print("--- ASCII Circuit Diagram ---")
print(qc.draw(output="text"))

# 5. Execute 1,000 shots on local CPU simulator
simulator = AerSimulator()
compiled_circuit = transpile(qc, simulator)
job = simulator.run(compiled_circuit, shots=1000)
counts = job.result().get_counts()

print("\n--- Local Simulator Measurement Counts (1,000 Shots) ---")
print(counts)

```

**Output:**

```text
--- ASCII Circuit Diagram ---
     ┌───┐     ┌─┐   
q_0: ┤ H ├──■──┤M├───
     └───┘┌─┴─┐└╥┘┌─┐
q_1: ─────┤ X ├─╫─┤M├
          └───┘ ║ └╥┘
c: 2/═══════════╩══╩═
                0  1 

--- Local Simulator Measurement Counts (1,000 Shots) ---
{'00': 494, '11': 506}

```

Look at those local simulator results: **`'01'` and `'10'` never appeared even once**! Even though each individual qubit is completely random ($50\%$ zero, $50\%$ one), they **always collapsed to the exact same value** (`'00'` or `'11'`).

### Part B: Dispatching to a Real Superconducting Quantum Computer (`qiskit-ibm-runtime`)

Now let's run the exact same program on a real physical quantum computer in the cloud using IBM Quantum's free tier (which provides free access to physical 127+ qubit superconducting processors):

1. **One-Time Account Setup:** Create a free account at [quantum.ibm.com](https://quantum.ibm.com/), copy your API token from the dashboard, and save it once on your machine:
```python
from qiskit_ibm_runtime import QiskitRuntimeService

QiskitRuntimeService.save_account(
    channel="ibm_quantum",
    token="YOUR_IBM_QUANTUM_API_TOKEN",
    set_as_default=True,
    overwrite=True
)

```


2. **Transpiling to Hardware ISA & Running via `SamplerV2`:** Following the helper pattern from our course video, we connect to `QiskitRuntimeService`, select the **least busy physical QPU** (`simulator=False`, `min_num_qubits=127`) to minimize queue wait time, rewrite our logical circuit into a hardware-native **ISA (Instruction Set Architecture) circuit** using `generate_preset_pass_manager`, and execute `1,000` shots via `SamplerV2`:



```python
from qiskit import QuantumCircuit
from qiskit.transpiler.preset_passmanagers import generate_preset_pass_manager
from qiskit_ibm_runtime import QiskitRuntimeService, SamplerV2 as Sampler
from qiskit.visualization import plot_histogram

def run_quantum_circuit_on_backend(circuit: QuantumCircuit, backend, shots: int = 1000) -> dict:
    """
    Course Helper Function:
    1. Optimizes & transpiles the logical circuit into a hardware-native ISA circuit.
    2. Initializes SamplerV2 for the target backend.
    3. Submits the job for `shots` repetitions and extracts the classical counts dictionary.
    """
    # Step 1: Transpile circuit to match physical chip topology and native gate set
    pass_manager = generate_preset_pass_manager(backend=backend, optimization_level=1)
    isa_circuit = pass_manager.run(circuit)

    # Step 2: Execute via SamplerV2 primitive
    sampler = Sampler(mode=backend)
    job = sampler.run([isa_circuit], shots=shots)
    print(f"Submitted Job ID: {job.job_id()} to {backend.name} ({shots} shots)...")

    # Step 3: Retrieve measurement counts from classical register
    result = job.result()
    counts = result[0].data.c.get_counts()
    return counts


# 1. Authenticate and pick the least-busy real 127+ qubit quantum processor
service = QiskitRuntimeService()
backend = service.least_busy(operational=True, simulator=False, min_num_qubits=127)
print(f"Connected! Selected least-busy physical QPU: {backend.name}")

# 2. Build our Bell State |Φ+⟩ circuit
qc = QuantumCircuit(2, 2)
qc.h(0)
qc.cx(0, 1)
qc.measure([0, 1], [0, 1])

# 3. Run 1,000 shots on physical quantum hardware
qpu_counts = run_quantum_circuit_on_backend(qc, backend=backend, shots=1000)
print("\n--- Physical QPU Measurement Counts ---")
print(qpu_counts)

# Optional: Plot the histogram
# fig = plot_histogram(qpu_counts, title=f"Bell State |Φ+⟩ on {backend.name}")

```

### Part C: Simulator vs. Real QPU Hardware — Why Did `'01'` and `'10'` Appear?

When your cloud job finishes executing on the physical QPU, your returned dictionary will look something like this:

```text
--- Physical QPU Measurement Counts ---
{'00': 478, '11': 489, '01': 18, '10': 15}

```

```text
  LOCAL CPU SIMULATOR (Ideal Math)             PHYSICAL SUPERCONDUCTING QPU (~15 mK)
  ────────────────────────────────             ─────────────────────────────────────
   500 ┼    ╭───╮              ╭───╮            500 ┼    ╭───╮              ╭───╮
       │    │494│              │506│                │    │478│              │489│
   250 ┼    │   │              │   │            250 ┼    │   │              │   │
       │    │   │   0      0   │   │                │    │   │  ╭───╮╭───╮  │   │
     0 ┼────┴───┴───┴──────┴───┴───┴──            0 ┼────┴───┴──┴18─┴┴15─┴──┴───┴──
             '00'  '01'   '10'  '11'                      '00'  '01'  '10'   '11'
                                                                  ▲     ▲
                                                      Real-World NISQ Hardware Noise!

```

Compare those two histograms closely:

1. **The Signature of Entanglement (`'00': 478, '11': 489`):** Nearly $97\%$ of all shots collapsed into `'00'` and `'11'` in roughly equal proportions—confirming genuine quantum entanglement between two physical superconducting transmon qubits inside the cryostat!


2. **Why `'01': 18` and `'10': 15` Showed Up:** Those $33$ stray shots out of $1,000$ are **not a bug in your Python code**. As we learned in Blog 1 and the course lab, today's quantum computers are **inherently noisy physical systems (NISQ devices)**:


* **Decoherence & Thermal Fluctuations:** Even inside a dilution refrigerator at $15\text{ millikelvins}$, microscopic thermal vibrations and stray electromagnetic fields occasionally disturb a qubit before measurement finishes.


* **Gate Calibration Drift & Readout Errors:** Microwave pulses that implement $H$ and $CX$ drift slightly in calibration over time, and the readout resonator can occasionally misclassify a `0` as a `1`.


* This is the exact reason we run **$1,000$ shots** instead of $1$ shot: repeating the experiment builds a clear statistical distribution where the true entangled signal (`'00'` and `'11'`) towers unmistakably above the hardware noise floor (`'01'` and `'10'`)!





---

## Up Next in the Series

With multi-qubit tensor products, controlled gates (`CNOT`, Toffoli), partial measurement renormalization, and physical QPU execution under our belt, we have completed **Layer 2** of our conceptual hierarchy (except for **Phase Kickback**, which we will pair directly with Phase Oracles in Blog 8)!

Now that we can entangle two qubits into a Bell pair, what can we actually **do** with entangled qubits across a communication network? In **Blog 7**, we will dive into **Quantum Information Protocols & Real-World Optical Engineering**:

* Why the **No-Cloning Theorem** makes it physically impossible for hackers (or repeaters) to copy an unknown quantum state.


* How **Superdense Coding** lets Alice transmit **2 classical bits** to Bob by sending only **1 physical qubit** using a shared Bell state.


* How **E91 and BB84 Quantum Key Distribution (QKD)** turn entanglement and wave-function collapse into an un-hackable burglar alarm between a Bank and a Stock Exchange.


* The real-world optical physics of **"Flying Qubits"**—how satellites like *Micius* beam single polarized photons through Earth's turbulent atmosphere in broad daylight using the **Quadruple Filter Lock**, heal cosmic-ray silicon damage via **in-orbit laser annealing**, and generate secure keys even when $99.9999\%$ of photons are lost in transit!



---

**Would you like to dive deeper into any concept or exercise from Blog 6, or are you ready for me to generate Blog 7: *Quantum Protocols & Flying Photons — Superdense Coding, Satellite QKD, and Atmospheric Optical Engineering*?**
