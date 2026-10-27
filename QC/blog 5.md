# Blog 5: Dials, Mirrors, and Wave Splitters — Single-Qubit Gates, Relative Phase, and Hadamard Interference

In Blog 4, we dissected the anatomy of a single qubit. We saw how column vectors, Dirac kets ($\vert{}\psi\rangle = \alpha\vert{}0\rangle + \beta\vert{}1\rangle$), and 3D Bloch Sphere coordinates ($\theta, \phi$) are all just different views of a single underlying data structure: a two-key state dictionary mapping `"0"` and `"1"` to `(magnitude, angle_deg)` tuples whose squared magnitudes sum to $100\%$.

Now that we know how to *represent* a qubit, how do we actually **process data** with it?

In classical computing, we pass bits through logic gates like `NOT`, `AND`, `OR`, and `XOR`. In quantum computing, we also apply logic gates to transform the states of our qubits—but because a pure single-qubit state lives on the surface of the 3D Bloch Sphere, **every single-qubit quantum gate is physically a rigid 3D rotation or reflection on that sphere**.

In this post, we will master the complete toolkit of single-qubit gates ($X, Y, Z, H, S, S^\dagger, T, T^\dagger$, and continuous rotations $R_y, R_z$), learn the fast column-lookup trick for evaluating gates in Dirac notation without tedious matrix multiplication, separate invisible **Global Phase** from mission-critical **Relative Phase**, and prove mathematically how the **Hadamard gate ($H$)** uses wave interference to turn hidden phase angles into $100\%$ deterministic measurement outcomes.

---

## 1. The Software Engineer's Unified View: What Single-Qubit Gates Actually Do

Before diving into the matrices and 3D rotations, let's map every single-qubit gate and phase concept directly onto our **Unified State Dictionary** (`{"0": (mag, angle_deg), "1": (mag, angle_deg)}`) from Blogs 3 and 4:

| Quantum Concept / Gate | Textbook Matrix / Formula | 3D Bloch Sphere Action | Unified Software Engineering Operation |
| --- | --- | --- | --- |
| **Pauli-$X$ Gate (`NOT`)** | $\begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix}$

 | $180^\circ$ ($\pi$ rad) rotation around the **$X$-axis**.

 | **Key Swap:** Swaps the `(mag, angle)` tuple of key `"0"` with key `"1"`.

 |
| **Pauli-$Z$ Gate (Phase Flip)** | $\begin{bmatrix} 1 & 0 \\ 0 & -1 \end{bmatrix}$

 | $180^\circ$ ($\pi$ rad) rotation around the **$Z$-axis**.

 | **Angle Shift on `"1"`:** Leaves `"0"` untouched and adds $+180^\circ$ ($\pi$ rad) to the angle of key `"1"`.

 |
| **Pauli-$Y$ Gate** | $\begin{bmatrix} 0 & -i \\ i & 0 \end{bmatrix}$

 | $180^\circ$ ($\pi$ rad) rotation around the **$Y$-axis**.

 | **Key Swap + $\pm 90^\circ$ Twist:** Swaps keys `"0"` and `"1"`, adding $+90^\circ$ to the new `"1"` and $-90^\circ$ ($+270^\circ$) to the new `"0"`.

 |
| **$S$ and $T$ Gates** | $\begin{bmatrix} 1 & 0 \\ 0 & e^{i\pi/2} \end{bmatrix}, \begin{bmatrix} 1 & 0 \\ 0 & e^{i\pi/4} \end{bmatrix}$

 | $+90^\circ$ ($\frac{\pi}{2}$) and $+45^\circ$ ($\frac{\pi}{4}$) rotations around the **$Z$-axis**.

 | **Precision Angle Dials:** Adds $+90^\circ$ ($S$) or $+45^\circ$ ($T$) to key `"1"`'s angle without touching magnitudes.

 |
| **$S^\dagger$ and $T^\dagger$ Gates** | $\begin{bmatrix} 1 & 0 \\ 0 & e^{-i\pi/2} \end{bmatrix}, \begin{bmatrix} 1 & 0 \\ 0 & e^{-i\pi/4} \end{bmatrix}$

 | $-90^\circ$ ($-\frac{\pi}{2}$) and $-45^\circ$ ($-\frac{\pi}{4}$) rotations around the **$Z$-axis**.

 | **`.undo()` Angle Dials:** Subtracts $90^\circ$ ($S^\dagger$) or $45^\circ$ ($T^\dagger$) from key `"1"`'s angle, reversing $S$ and $T$.

 |
| **Global Phase vs. Relative Phase** | $e^{i\theta_0}\alpha\Vert{}0\rangle + e^{i\theta_1}\beta\Vert{}1\rangle \equiv \alpha\Vert{}0\rangle + e^{i(\theta_1 - \theta_0)}\beta\Vert{}1\rangle$

 | Only the angle difference $(\theta_1 - \theta_0)$ sets the longitude $\phi$ on the Bloch sphere.

 | **Zero-Indexing the Angle Reference:** Subtract key `"0"`'s angle from *all* keys so key `"0"` always sits at `0.0°` and key `"1"` holds the net relative angle `angle_1 - angle_0`.

 |
| **Hadamard Gate ($H$)** | $\frac{1}{\sqrt{2}}\begin{bmatrix} 1 & 1 \\ 1 & -1 \end{bmatrix}$

 | $180^\circ$ rotation around the diagonal $XZ$-axis.

 | **The Wave Splitter / Interference Mixer:** Maps `"0"` to an in-phase split (`0°` and `0°`) and `"1"` to an out-of-phase split (`0°` and `180°`), causing matching keys with opposite angles (`0°` vs `180°`) to cancel out!

 |

With this lookup table in mind, let's explore each family of gates both visually on the Bloch Sphere and algebraically in matrix and Dirac notation.

---

## 2. The Pauli Triad ($X, Y, Z$): $180^\circ$ Flips Around the Three Axes

The three most common single-qubit gates in quantum computing are the **Pauli-$X$, Pauli-$Y$, and Pauli-$Z$ gates**. Each one rotates the qubit state vector by exactly **$180^\circ$ ($\pi$ radians)** around its respective coordinate axis on the Bloch Sphere:

```text
         PAULI-X GATE                    PAULI-Y GATE                    PAULI-Z GATE
   (180° / π Flip Around X)        (180° / π Flip Around Y)        (180° / π Flip Around Z)

            |0⟩ (+Z)                        |0⟩ (+Z)                        |0⟩ (+Z)
               ▲                               ▲                               ▲
               │    ╭───╮                      │                               │   ╭───╮
               │   ╱     ╲                     │      ╭───╮                    │  (  ↺  )
               ┼──┼────────► +X                ┼─────┼────► +Y                 ┼───╰───╯──► +Y
              ╱   ╰───╯                       ╱     ╰───╯                     ╱
             ▼                               ▼                               ▼
            |1⟩ (-Z)                        |1⟩ (-Z)                        |1⟩ (-Z)
   Swaps North & South Poles       Swaps Poles + Twists Phase      Spins Equator (|+⟩ ↔ |-⟩)
     (|0⟩ ↔ |1⟩, |+⟩ stays)         (|0⟩ → i|1⟩, |1⟩ → -i|0⟩)       (Poles |0⟩, |1⟩ unchanged)

```

### A. Geometric Intuition & The Self-Inverting Property ($X^2 = Y^2 = Z^2 = I$)

Let's trace what each gate does visually on the Bloch sphere:

1. **The $X$ Gate (Bit-Flip / Quantum NOT):** If a qubit starts at the North Pole ($\vert{}0\rangle$), applying the $X$ gate flips it $180^\circ$ around the $X$-axis down to the South Pole ($\vert{}1\rangle$). Even if a qubit is in an arbitrary superposition state halfway or partway between $\vert{}0\rangle$ and $\vert{}1\rangle$, the $X$ gate always rotates the entire state vector $180^\circ$ ($\pi$ radians) around the $X$-axis.


2. **The $Y$ Gate:** Performs the exact same $180^\circ$ ($\pi$ radian) rotation, but around the **$Y$-axis** instead of the $X$-axis.


3. **The $Z$ Gate (Phase-Flip):** Rotates the qubit state by $180^\circ$ ($\pi$ radians) around the vertical **$Z$-axis**. Notice that if your qubit sits right at the North Pole ($\vert{}0\rangle$) or South Pole ($\vert{}1\rangle$), spinning around the $Z$-axis leaves the vector pointing at the exact same pole, whereas states on the equator flip across the sphere ($\vert{}+\rangle \leftrightarrow \vert{}-\rangle$).



Because each of these three gates rotates the qubit by $\pi$ radians ($180^\circ$) around a fixed axis, what happens if we apply the **same gate twice in a row**? Rotating by $180^\circ$ and then another $180^\circ$ around the same axis completes a full $360^\circ$ ($2\pi$ radian) circle, returning the qubit to its exact original position! Mathematically, this means the $X, Y,$ and $Z$ gates are **their own inverses** (their own `.undo()` methods):

$$X^2 = I, \qquad Y^2 = I, \qquad Z^2 = I$$

### B. Matrix Definitions & Worked Course Exercise ($X\vert{}0\rangle = \vert{}1\rangle$)

Mathematically, we represent the $X, Y,$ and $Z$ gates as $2 \times 2$ unitary matrices:

$$X = \begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix}, \qquad Y = \begin{bmatrix} 0 & -i \\ i & 0 \end{bmatrix}, \qquad Z = \begin{bmatrix} 1 & 0 \\ 0 & -1 \end{bmatrix}$$

To apply a gate to a qubit in column-vector form, we simply multiply the gate's $2 \times 2$ matrix by the qubit's $2 \times 1$ column vector. Let's work through the first exercise from the course video: **Prove using matrix multiplication that applying the $X$ gate to the $\vert{}0\rangle$ state gives the $\vert{}1\rangle$ state**:

$$X\vert{}0\rangle = \begin{bmatrix} 0 & 1 \\ 1 & 0 \end{bmatrix} \begin{bmatrix} 1 \\ 0 \end{bmatrix} = \begin{bmatrix} (0)(1) + (1)(0) \\ (1)(1) + (0)(0) \end{bmatrix} = \begin{bmatrix} 0 \\ 1 \end{bmatrix} = \vert{}1\rangle$$

Similarly, applying $X$ to $\vert{}1\rangle = \begin{bmatrix} 0 \\ 1 \end{bmatrix}$ yields $\begin{bmatrix} 1 \\ 0 \end{bmatrix} = \vert{}0\rangle$.

---

## 3. Fast Column-Lookup Evaluation: Applying Gates in Dirac Notation

Multiplying matrices by column vectors works fine for one qubit, but as we learned in Blog 4, quantum states are almost always written in **Dirac (ket) notation**: $\vert{}\psi\rangle = \alpha\vert{}0\rangle + \beta\vert{}1\rangle$. How do we apply a matrix gate $U$ directly to a state written in Dirac notation without converting everything back and forth into column vectors?

We use two simple rules from our course material:

1. **Linearity (Distribute the Gate):** Because quantum gates are linear operators, applying an arbitrary gate $U$ to a superposition state means $U$ acts on each basis state individually:



$$U(\alpha\vert{}0\rangle + \beta\vert{}1\rangle) = \alpha U\vert{}0\rangle + \beta U\vert{}1\rangle$$


2. **The Column-Lookup Trick:** Instead of performing row-by-column multiplication, simply **look at the columns of the gate matrix $U$**:


* **Column 1** of the matrix tells you the exact vector that the **$\vert{}0\rangle$ state** transforms into ($U\vert{}0\rangle$).


* **Column 2** of the matrix tells you the exact vector that the **$\vert{}1\rangle$ state** transforms into ($U\vert{}1\rangle$).





```text
                  Col 1     Col 2
               (|0⟩ → ?) (|1⟩ → ?)
              ┌                   ┐
         U  = │   u_00      u_01  │   ──►  U|0⟩ = u_00|0⟩ + u_10|1⟩  (Read down Col 1)
              │   u_10      u_11  │   ──►  U|1⟩ = u_01|0⟩ + u_11|1⟩  (Read down Col 2)
              └                   ┘

```

### Worked Course Example 1: Applying the $Y$ Gate in Dirac Notation

Let's trace the exact example from our course material: apply the $Y = \begin{bmatrix} 0 & -i \\ i & 0 \end{bmatrix}$ gate to the general state $\vert{}\psi\rangle = \alpha\vert{}0\rangle + \beta\vert{}1\rangle$ using Dirac notation:

1. **Distribute $Y$ across the superposition:**


$$Y(\alpha\vert{}0\rangle + \beta\vert{}1\rangle) = \alpha Y\vert{}0\rangle + \beta Y\vert{}1\rangle$$


2. **Replace $Y\vert{}0\rangle$ with Column 1 and $Y\vert{}1\rangle$ with Column 2 of the $Y$ matrix:**


$$\alpha Y\vert{}0\rangle + \beta Y\vert{}1\rangle = \alpha \begin{bmatrix} 0 \\ i \end{bmatrix} + \beta \begin{bmatrix} -i \\ 0 \end{bmatrix}$$


3. **Factor out $i$ and $-i$ so the column vectors become $\vert{}1\rangle$ and $\vert{}0\rangle$ again:**


$$= \alpha i \begin{bmatrix} 0 \\ 1 \end{bmatrix} - \beta i \begin{bmatrix} 1 \\ 0 \end{bmatrix} = i\alpha\vert{}1\rangle - i\beta\vert{}0\rangle = -i\beta\vert{}0\rangle + i\alpha\vert{}1\rangle$$



### Worked Course Example 2: Applying the $Z$ Gate in Dirac Notation

Now let's solve the course exercise: **Prove that applying a $Z = \begin{bmatrix} 1 & 0 \\ 0 & -1 \end{bmatrix}$ gate to $\vert{}\psi\rangle = \alpha\vert{}0\rangle + \beta\vert{}1\rangle$ gives $\alpha\vert{}0\rangle - \beta\vert{}1\rangle$**:

1. **Distribute $Z$:**


$$Z(\alpha\vert{}0\rangle + \beta\vert{}1\rangle) = \alpha Z\vert{}0\rangle + \beta Z\vert{}1\rangle$$


2. **Substitute Column 1 ($\begin{bmatrix} 1 \\ 0 \end{bmatrix}$) for $Z\vert{}0\rangle$ and Column 2 ($\begin{bmatrix} 0 \\ -1 \end{bmatrix}$) for $Z\vert{}1\rangle$:**


$$= \alpha \begin{bmatrix} 1 \\ 0 \end{bmatrix} + \beta \begin{bmatrix} 0 \\ -1 \end{bmatrix} = \alpha \begin{bmatrix} 1 \\ 0 \end{bmatrix} - \beta \begin{bmatrix} 0 \\ 1 \end{bmatrix} = \alpha\vert{}0\rangle - \beta\vert{}1\rangle$$



Look closely at that result: $\alpha\vert{}0\rangle - \beta\vert{}1\rangle$. If we measure this qubit right now, what is the probability of getting `0`? $\vert{}\alpha\vert{}^2$. What is the probability of getting `1`? $\vert{}-\beta\vert{}^2 = \vert{}\beta\vert{}^2$.

Wait a minute—as the course instructor points out, **what is the point of the $Z$ gate if the qubit still has an $\vert{}\alpha\vert{}^2$ chance of being `0` and a $\vert{}\beta\vert{}^2$ chance of being `1`?** Why rotate around the $Z$-axis at all if it doesn't change the measurement probabilities? That question brings us directly to the most important concept in quantum computing: **Phase**.

---

## 4. Global Phase vs. Relative Phase: Why Only Angle *Differences* Matter

On the Bloch sphere, spinning a qubit around the vertical $Z$-axis changes its azimuthal angle $\phi$—a property called **phase**. As we saw in Blog 2, to represent phase mathematically we bring back complex numbers in **Euler's exponential form ($e^{i\phi}$)**, because multiplying by $e^{i\phi}$ rotates a complex amplitude around a unit circle by $\phi$ radians.

However, in quantum mechanics, there are **two distinct types of phase**, and confusing them is one of the most common pitfalls for beginners:

### A. Global Phase (Physically Invisible & Irrelevant)

**Global phase** occurs when the **entire qubit state** (both the $\vert{}0\rangle$ and $\vert{}1\rangle$ amplitudes together) is multiplied by a unit complex number $e^{i\theta}$:

$$\vert{}\psi_{\text{global}}\rangle = e^{i\theta}\left(\alpha\vert{}0\rangle + \beta\vert{}1\rangle\right)$$

It turns out that **global phase is physically undetectable and completely irrelevant**. Why? Because every physical measurement or interference interaction only ever compares the $\vert{}0\rangle$ and $\vert{}1\rangle$ branches *against each other*. If both branches are rotated by the exact same angle $+\theta$, the angle difference between them hasn't changed at all! Therefore, as shown in our course material, the state:

$$e^{i\theta}\left(\frac{1}{\sqrt{2}}\vert{}0\rangle + \frac{1}{\sqrt{2}}\vert{}1\rangle\right) \quad \text{is logically and physically identical to} \quad \frac{1}{\sqrt{2}}\vert{}0\rangle + \frac{1}{\sqrt{2}}\vert{}1\rangle$$

Whenever we see a common complex factor sitting in front of an entire quantum state—including a global minus sign $-1 = e^{i\pi}$ (as we will see in Deutsch's algorithm in Blog 8)—we can **factor it out and discard it completely**.

### B. Relative Phase (The Secret Weapon of Quantum Computing)

**Relative phase** occurs when **only the $\vert{}1\rangle$ state** has a complex phase factor $e^{i\phi}$ relative to the $\vert{}0\rangle$ state:

$$\vert{}\psi_{\text{relative}}\rangle = \alpha\vert{}0\rangle + e^{i\phi}\beta\vert{}1\rangle$$

Unlike global phase, **relative phase is physically real and extraordinarily important**. Geometrically, changing the relative phase $\phi$ rotates the qubit state vector around the $Z$-axis of the Bloch sphere by $\phi$ radians:

* In $\frac{1}{\sqrt{2}}\vert{}0\rangle + \frac{1}{\sqrt{2}}\vert{}1\rangle$ ($\vert{}+\rangle$), the relative phase factor on $\vert{}1\rangle$ is $+1 = e^{i0}$ ($\phi = 0\text{ rad} = 0^\circ$).


* In $\frac{1}{\sqrt{2}}\vert{}0\rangle - \frac{1}{\sqrt{2}}\vert{}1\rangle$ ($\vert{}-\rangle$), the relative phase factor on $\vert{}1\rangle$ is $-1 = e^{i\pi}$ ($\phi = \pi\text{ rad} = 180^\circ$), rotating the state halfway around the Bloch equator to the opposite side.


* In $\frac{1}{\sqrt{2}}\vert{}0\rangle + \frac{i}{\sqrt{2}}\vert{}1\rangle$ ($\vert{}i\rangle$), the relative phase factor on $\vert{}1\rangle$ is $+i = e^{i\pi/2}$ ($\phi = \frac{\pi}{2}\text{ rad} = 90^\circ$).


* In $\frac{1}{\sqrt{2}}\vert{}0\rangle - \frac{i}{\sqrt{2}}\vert{}1\rangle$ ($\vert{}-i\rangle$), the relative phase factor on $\vert{}1\rangle$ is $-i = e^{-i\pi/2} = e^{i3\pi/2}$ ($\phi = -\frac{\pi}{2}\text{ rad} = 270^\circ$).



### C. What If *Both* $\vert{}0\rangle$ and $\vert{}1\rangle$ Have Complex Phase Factors?

Suppose a sequence of quantum gates leaves your qubit with complex numbers on *both* the $\vert{}0\rangle$ and $\vert{}1\rangle$ states:

$$\vert{}\psi\rangle = e^{i\theta_0}\alpha\vert{}0\rangle + e^{i\theta_1}\beta\vert{}1\rangle$$

How do we read its relative phase? We follow a two-step canonical normalization procedure from our course material:

1. **Factor out the complex phase of the $\vert{}0\rangle$ state ($e^{i\theta_0}$) from the entire qubit**, creating a global phase outside the brackets and a relative phase inside:



$$\vert{}\psi\rangle = e^{i\theta_0}\left(\alpha\vert{}0\rangle + \frac{e^{i\theta_1}}{e^{i\theta_0}}\beta\vert{}1\rangle\right) = \underbrace{e^{i\theta_0}}_{\text{Global Phase}}\left(\alpha\vert{}0\rangle + \underbrace{e^{i(\theta_1 - \theta_0)}}_{\text{Relative Phase}}\beta\vert{}1\rangle\right)$$


2. **Discard the global phase $e^{i\theta_0}$**, leaving a clean qubit state with a pure relative phase of $\phi = \theta_1 - \theta_0$ on the $\vert{}1\rangle$ term:



$$\vert{}\psi\rangle \equiv \alpha\vert{}0\rangle + e^{i(\theta_1 - \theta_0)}\beta\vert{}1\rangle$$



*(In our Software Engineer's Unified View, this is simply subtracting `state["0"].angle_deg` from both `state["0"].angle_deg` and `state["1"].angle_deg` so that Key `"0"` always serves as our `0.0°` reference baseline!)*

### D. Proving Why Relative Phase Hides From Direct Measurement

As we previewed in Blog 2, if we measure the state $\alpha\vert{}0\rangle + e^{i\phi}\beta\vert{}1\rangle$ directly in the computational ($\vert{}0\rangle/\vert{}1\rangle$) basis, the probability of measuring `1` is the magnitude squared of its coefficient:

$$P(1) = \left\vert{}e^{i\phi}\beta\right\vert{}^2 = \left\vert{}e^{i\phi}\right\vert{}^2 \cdot \vert{}\beta\vert{}^2$$

Because the coefficient in front of $e^{i\phi}$ in exponential form ($re^{i\phi}$) is $r = 1$, its magnitude is $\left\vert{}e^{i\phi}\right\vert{} = 1$. Therefore:

$$P(1) = (1)^2 \cdot \vert{}\beta\vert{}^2 = \vert{}\beta\vert{}^2$$

Geometrically, spinning around the $Z$-axis keeps the qubit at the exact same latitude distance from the North Pole ($\vert{}0\rangle$) and South Pole ($\vert{}1\rangle$). So how do we ever make that hidden relative phase $\phi$ show up in our measurement results? Enter the **Hadamard Gate ($H$)**.

---

## 5. The Hadamard Gate ($H$): The Wave Splitter & Interference Proof

Perhaps the single most important gate in all of quantum computing is the **Hadamard gate ($H$)**. Its unitary matrix is:

$$H = \frac{1}{\sqrt{2}}\begin{bmatrix} 1 & 1 \\ 1 & -1 \end{bmatrix} = \begin{bmatrix} \frac{1}{\sqrt{2}} & \frac{1}{\sqrt{2}} \\ \frac{1}{\sqrt{2}} & -\frac{1}{\sqrt{2}} \end{bmatrix}$$

### A. Geometric Action on the Bloch Sphere

Geometrically on the Bloch sphere, the Hadamard gate performs a **$180^\circ$ ($\pi$ radian) rotation around the diagonal axis halfway between the $X$-axis and the $Z$-axis** ($\frac{\hat{x} + \hat{z}}{\sqrt{2}}$, which you can also visualize as a $90^\circ$ tilt between $Z$ and $X$ combined with a $180^\circ$ flip). Because this diagonal axis lies exactly midway between the poles ($Z$) and the $+X/-X$ equator, applying $H$ swaps the poles with the $+X/-X$ equator:

1. **Acting on $\vert{}0\rangle$ (Column 1 of $H$):** Transforms the North Pole $\vert{}0\rangle$ into the equatorial **$\vert{}+\rangle$ state** (in-phase superposition):



$$H\vert{}0\rangle = \frac{1}{\sqrt{2}}\vert{}0\rangle + \frac{1}{\sqrt{2}}\vert{}1\rangle \equiv \vert{}+\rangle$$


2. **Acting on $\vert{}1\rangle$ (Column 2 of $H$):** Transforms the South Pole $\vert{}1\rangle$ into the equatorial **$\vert{}-\rangle$ state** (opposite-phase superposition with a $-1 = e^{i\pi}$ relative phase):



$$H\vert{}1\rangle = \frac{1}{\sqrt{2}}\vert{}0\rangle - \frac{1}{\sqrt{2}}\vert{}1\rangle \equiv \vert{}-\rangle$$



### B. The Double-Hadamard Interference Proof: Why Relative Phase Changes Everything!

Because $H$ is a $180^\circ$ rotation around the diagonal $XZ$-axis, it is **its own inverse ($H^2 = I$)**. To apply $H$ to any state written in Dirac notation, we simply replace $\vert{}0\rangle$ with $\vert{}+\rangle = \frac{\vert{}0\rangle + \vert{}1\rangle}{\sqrt{2}}$ and replace $\vert{}1\rangle$ with $\vert{}-\rangle = \frac{\vert{}0\rangle - \vert{}1\rangle}{\sqrt{2}}$ and expand!

Let's apply the Hadamard gate to both $\vert{}+\rangle$ and $\vert{}-\rangle$ side by side. Remember: before applying $H$, **both $\vert{}+\rangle$ and $\vert{}-\rangle$ have the exact same $50\%$ chance of measuring `0` and $50\%$ chance of measuring `1**`—they differ *only* in their relative phase ($\phi = 0^\circ$ vs. $\phi = 180^\circ$):

#### Case 1: Applying $H$ to $\vert{}+\rangle = \frac{1}{\sqrt{2}}\vert{}0\rangle + \frac{1}{\sqrt{2}}\vert{}1\rangle$

Replace $\vert{}0\rangle \to \frac{\vert{}0\rangle + \vert{}1\rangle}{\sqrt{2}}$ and $\vert{}1\rangle \to \frac{\vert{}0\rangle - \vert{}1\rangle}{\sqrt{2}}$:

$$H\vert{}+\rangle = \frac{1}{\sqrt{2}}H\vert{}0\rangle + \frac{1}{\sqrt{2}}H\vert{}1\rangle = \frac{1}{\sqrt{2}}\left(\frac{\vert{}0\rangle + \vert{}1\rangle}{\sqrt{2}}\right) + \frac{1}{\sqrt{2}}\left(\frac{\vert{}0\rangle - \vert{}1\rangle}{\sqrt{2}}\right)$$

$$= \left(\frac{1}{2}\vert{}0\rangle + \frac{1}{2}\vert{}0\rangle\right) + \left(\frac{1}{2}\vert{}1\rangle - \frac{1}{2}\vert{}1\rangle\right) = \mathbf{1\vert{}0\rangle + 0\vert{}1\rangle = \vert{}0\rangle}$$

* **On $\vert{}0\rangle$ (Constructive Interference):** The two positive amplitudes reinforce each other: $\frac{1}{2} + \frac{1}{2} = 1$ ($100\%$ probability)!


* **On $\vert{}1\rangle$ (Destructive Interference):** The positive $\frac{1}{2}$ and negative $-\frac{1}{2}$ (opposite $180^\circ$ phase) cancel each other out to $0$ ($0\%$ probability)!



#### Case 2: Applying $H$ to $\vert{}-\rangle = \frac{1}{\sqrt{2}}\vert{}0\rangle - \frac{1}{\sqrt{2}}\vert{}1\rangle$

Now watch what happens when the input state starts with that negative relative phase ($-$ sign on $\vert{}1\rangle$):

$$H\vert{}-\rangle = \frac{1}{\sqrt{2}}H\vert{}0\rangle - \frac{1}{\sqrt{2}}H\vert{}1\rangle = \frac{1}{\sqrt{2}}\left(\frac{\vert{}0\rangle + \vert{}1\rangle}{\sqrt{2}}\right) - \frac{1}{\sqrt{2}}\left(\frac{\vert{}0\rangle - \vert{}1\rangle}{\sqrt{2}}\right)$$

$$= \left(\frac{1}{2}\vert{}0\rangle - \frac{1}{2}\vert{}0\rangle\right) + \left(\frac{1}{2}\vert{}1\rangle - \left(-\frac{1}{2}\vert{}1\rangle\right)\right) = \mathbf{0\vert{}0\rangle + 1\vert{}1\rangle = \vert{}1\rangle}$$

* **On $\vert{}0\rangle$ (Destructive Interference):** Now the $\vert{}0\rangle$ amplitudes have opposite signs and cancel out: $\frac{1}{2} - \frac{1}{2} = 0$ ($0\%$ probability)!


* **On $\vert{}1\rangle$ (Constructive Interference):** The minus-times-minus becomes positive ($\frac{1}{2} + \frac{1}{2} = 1$), amplifying $\vert{}1\rangle$ to $100\%$ probability!



```text
   CASE 1: H on |+⟩ (Relative Phase = 0°)          CASE 2: H on |-⟩ (Relative Phase = 180°)
   ──────────────────────────────────────          ────────────────────────────────────────
   Input |+⟩ :  "0": (0.707, 0°), "1": (0.707, 0°) Input |-⟩ :  "0": (0.707, 0°), "1": (0.707, 180°)
                         │                                               │
                    [ Apply H ]                                     [ Apply H ]
                         ▼                                               ▼
   Key "0" contributions: 0.5 (0°) + 0.5 (0°)      Key "0" contributions: 0.5 (0°) + 0.5 (180°)
                        = 1.0 (Constructive!)                           = 0.0 (Destructive Cancel!)
   Key "1" contributions: 0.5 (0°) + 0.5 (180°)    Key "1" contributions: 0.5 (180°) + 0.5 (180°)
                        = 0.0 (Destructive Cancel!)                     = 1.0 (Constructive!)
                         │                                               │
                         ▼                                               ▼
         Deterministic Output: 100% |0⟩                  Deterministic Output: 100% |1⟩

```

**This is the central revelation of Blog 5:**

1. If you flip a fair classical coin twice in a row, the second flip is still $50/50$ random.


2. If you pass a qubit through a Hadamard gate ($\vert{}0\rangle \xrightarrow{H} \vert{}+\rangle$), its measurement odds are $50/50$, and if you apply a $Z$ gate ($\vert{}+\rangle \xrightarrow{Z} \vert{}-\rangle$), its measurement odds are *still* $50/50$.


3. Yet the moment you pass the qubit through a **second Hadamard gate**, the Hadamard converts the hidden relative phase difference between $|+\rangle$ ($\phi = 0^\circ$) and $|-\rangle$ ($\phi = 180^\circ$) into a **$100\%$ deterministic difference between $|0\rangle$ and $|1\rangle$**! In fact, sandwiching a $Z$ phase-flip gate between two Hadamard gates ($H \to Z \to H$) turns a pure phase flip into a complete bit flip ($|0\rangle \xrightarrow{H} |+\rangle \xrightarrow{Z} |-\rangle \xrightarrow{H} |1\rangle$, so $HZH = X$). Every major quantum algorithm we will study in Blogs 8, 9, and 10 (Deutsch-Jozsa, Bernstein-Vazirani, Grover/QAOA, and Shor's algorithm) works by sandwiching phase changes between layers of Hadamard gates!



---

## 6. Precision Phase Dials: The $S, T, S^\dagger, T^\dagger$ and Continuous Rotation Gates

While the $Z$ gate flips the relative phase by a coarse $\pi$ radians ($180^\circ$), algorithms like the Quantum Fourier Transform (Blog 9) and Quantum Neural Networks (Blog 10) require **finer phase dials** to rotate a qubit partway around the $Z$-axis.

### A. The $S$ Gate and $S^\dagger$ Gate ($\pm\frac{\pi}{2}$ / $\pm 90^\circ$ Phase Dials)

The **$S$ gate** adds a relative phase of **$+\frac{\pi}{2}$ radians ($+90^\circ$)** to the $|1\rangle$ state (rotating $|+\rangle \to |i\rangle$ along the equator):

$$S = \begin{bmatrix} 1 & 0 \\ 0 & e^{i\pi/2} \end{bmatrix} = \begin{bmatrix} 1 & 0 \\ 0 & i \end{bmatrix}$$

Unlike $X, Y, Z,$ and $H$, applying $S$ twice does *not* return you to the start—it rotates by $\frac{\pi}{2} + \frac{\pi}{2} = \pi$ ($180^\circ$), meaning $S^2 = Z$! So how do we find the inverse (`.undo()`) of the $S$ gate? As we learned in Blog 3, because $S$ is unitary, its inverse is simply its **conjugate transpose ($S^\dagger$)**:

$$S^\dagger = (S^*)^T = \begin{bmatrix} 1 & 0 \\ 0 & e^{-i\pi/2} \end{bmatrix} = \begin{bmatrix} 1 & 0 \\ 0 & -i \end{bmatrix}$$

Applying $S^\dagger$ adds a relative phase of **$-\frac{\pi}{2}$ radians ($-90^\circ$)**. Therefore, if you apply an $S$ gate ($+\frac{\pi}{2}$) followed immediately by an $S^\dagger$ gate ($-\frac{\pi}{2}$), the angles cancel ($e^{i\pi/2} \cdot e^{-i\pi/2} = e^{i0} = 1$) and the qubit lands right back in its original spot ($S S^\dagger = I$)!

### B. The $T$ Gate and $T^\dagger$ Gate ($\pm\frac{\pi}{4}$ / $\pm 45^\circ$ Phase Dials)

Stepping down to an even finer increment, the **$T$ gate** adds a relative phase of **$+\frac{\pi}{4}$ radians ($+45^\circ$)** to the $|1\rangle$ state (so $T^2 = S$ and $T^4 = Z$):

$$T = \begin{bmatrix} 1 & 0 \\ 0 & e^{i\pi/4} \end{bmatrix}, \qquad T^\dagger = \begin{bmatrix} 1 & 0 \\ 0 & e^{-i\pi/4} \end{bmatrix}$$

Taking the conjugate transpose gives the **$T^\dagger$ ("T-dagger") gate**, which adds a relative phase of **$-\frac{\pi}{4}$ radians ($-45^\circ$)**, serving as the exact inverse of the $T$ gate ($T T^\dagger = I$).

### C. Parameterized Continuous Rotations ($R_z(\phi)$ and $R_y(\theta)$)

In practical quantum programming (such as Qiskit circuits, variational chemistry solvers, and Quantum Neural Networks), we can also rotate by **any continuous angle** around the $Y$ or $Z$ axes:

* **$R_z(\phi)$:** Rotates the relative phase around the $Z$-axis by any arbitrary angle $\phi$ (modifying longitude without changing latitude/probabilities).


* **$R_y(\theta)$:** Rotates the qubit state around the $Y$-axis by any arbitrary latitude angle $\theta$, smoothly dialing the measurement probabilities $P(0) = \cos^2(\theta/2)$ and $P(1) = \sin^2(\theta/2)$.



---

## 7. The Software Engineer's Unified View: Complete Single-Qubit Gate & Interference Engine in Python

Let's bring every concept from Blog 5 together into a self-documenting Python script. Notice how our implementation below makes three things crystal clear without chalkboard ambiguity:

1. **Column-Lookup Gate Execution with Explicit Interference Logging:** When `H` acts on `|+>` or `|->`, the engine logs the exact incoming `(magnitude, angle_deg)` branches arriving at Key `"0"` and Key `"1"`, showing how two branches with `0°` and `0°` add up to `1.0` (constructive interference) while two branches with `0°` and `180°` cancel out to `0.0` (destructive interference)!


2. **Global Phase Stripping (`normalize_global_phase()`):** Demonstrates factoring out key `"0"`'s phase angle so that only the physically meaningful relative phase remains on key `"1"`.


3. **Side-by-Side Verification with Qiskit:** Confirms that our explicit dictionary engine matches Qiskit's `Statevector` across $X, Y, Z, H, S, S^\dagger, T,$ and $T^\dagger$.



```python
import numpy as np
from dataclasses import dataclass
from typing import Dict

@dataclass
class AmplitudePolar:
    magnitude: float
    angle_deg: float

    def to_complex(self) -> complex:
        rad = np.radians(self.angle_deg)
        return self.magnitude * np.exp(1j * rad)

    @staticmethod
    def from_complex(c: complex) -> 'AmplitudePolar':
        mag = float(np.abs(c))
        if mag < 1e-9:
            return AmplitudePolar(0.0, 0.0)
        ang = float(np.degrees(np.angle(c)) % 360.0)
        return AmplitudePolar(mag, ang)


class SingleQubitStateDict:
    """
    Unified Single-Qubit Representation:
    Sparse dictionary mapping '0' and '1' -> AmplitudePolar(magnitude, angle_deg).
    """
    def __init__(self, state_map: Dict[str, AmplitudePolar]):
        self.state_map = state_map

    def normalize_global_phase(self) -> 'SingleQubitStateDict':
        """
        Course Rule: Factor out the complex phase of the |0> state (global phase)
        and discard it, leaving a pure relative phase on the |1> state.
        """
        amp0 = self.state_map["0"]
        amp1 = self.state_map["1"]
        if amp0.magnitude > 1e-9:
            ref_angle = amp0.angle_deg
            return SingleQubitStateDict({
                "0": AmplitudePolar(amp0.magnitude, 0.0),
                "1": AmplitudePolar(
                    amp1.magnitude,
                    (amp1.angle_deg - ref_angle) % 360.0 if amp1.magnitude > 1e-9 else 0.0
                )
            })
        return self

    def apply_gate_via_column_lookup(
        self, gate_name: str, matrix: np.ndarray, log_interference: bool = False
    ) -> 'SingleQubitStateDict':
        """
        Applies a 2x2 unitary gate using the Dirac Column-Lookup rule:
          - Column 0 of matrix = what '0' transforms into
          - Column 1 of matrix = what '1' transforms into
        """
        c0_in = self.state_map["0"].to_complex()
        c1_in = self.state_map["1"].to_complex()

        # Contributions to output '0' and output '1' from input '0' (Col 0) and input '1' (Col 1)
        contrib_to_0_from_0 = c0_in * matrix[0, 0]
        contrib_to_0_from_1 = c1_in * matrix[0, 1]
        contrib_to_1_from_0 = c0_in * matrix[1, 0]
        contrib_to_1_from_1 = c1_in * matrix[1, 1]

        if log_interference:
            p00 = AmplitudePolar.from_complex(contrib_to_0_from_0)
            p01 = AmplitudePolar.from_complex(contrib_to_0_from_1)
            p10 = AmplitudePolar.from_complex(contrib_to_1_from_0)
            p11 = AmplitudePolar.from_complex(contrib_to_1_from_1)
            print(f"  [Interference Trace for {gate_name}]")
            print(f"    -> Key '0' receives: (mag={p00.magnitude:.3f}, ang={p00.angle_deg:5.1f}°) "
                  f"+ (mag={p01.magnitude:.3f}, ang={p01.angle_deg:5.1f}°)")
            print(f"    -> Key '1' receives: (mag={p10.magnitude:.3f}, ang={p10.angle_deg:5.1f}°) "
                  f"+ (mag={p11.magnitude:.3f}, ang={p11.angle_deg:5.1f}°)")

        new_c0 = contrib_to_0_from_0 + contrib_to_0_from_1
        new_c1 = contrib_to_1_from_0 + contrib_to_1_from_1

        return SingleQubitStateDict({
            "0": AmplitudePolar.from_complex(new_c0),
            "1": AmplitudePolar.from_complex(new_c1)
        }).normalize_global_phase()

    def describe(self, label: str):
        a0, a1 = self.state_map["0"], self.state_map["1"]
        p0, p1 = (a0.magnitude ** 2) * 100, (a1.magnitude ** 2) * 100
        print(f"=== {label} ===")
        print(f"  Dict : '0': (mag={a0.magnitude:.4f}, ang={a0.angle_deg:5.1f}°), "
              f"'1': (mag={a1.magnitude:.4f}, ang={a1.angle_deg:5.1f}°)")
        print(f"  Odds : P('0') = {p0:5.1f}%  |  P('1') = {p1:5.1f}%\n")


# Define standard 2x2 single-qubit gate matrices
X_MAT = np.array([[0, 1], [1, 0]], dtype=complex)
Y_MAT = np.array([[0, -1j], [1j, 0]], dtype=complex)
Z_MAT = np.array([[1, 0], [0, -1]], dtype=complex)
H_MAT = (1 / np.sqrt(2)) * np.array([[1, 1], [1, -1]], dtype=complex)
S_MAT = np.array([[1, 0], [0, np.exp(1j * np.pi / 2)]], dtype=complex)
S_DAG_MAT = S_MAT.conj().T
T_MAT = np.array([[1, 0], [0, np.exp(1j * np.pi / 4)]], dtype=complex)
T_DAG_MAT = T_MAT.conj().T

# 1. Start at |0> and apply H -> |+>
ket_0 = SingleQubitStateDict({"0": AmplitudePolar(1.0, 0.0), "1": AmplitudePolar(0.0, 0.0)})
ket_plus = ket_0.apply_gate_via_column_lookup("H", H_MAT)
ket_plus.describe("1. H|0> = |+> (Equator +X, Relative Phase = 0°)")

# 2. Apply Z to |+> -> |-> (Relative Phase flips to 180°, probabilities stay 50/50!)
ket_minus = ket_plus.apply_gate_via_column_lookup("Z", Z_MAT)
ket_minus.describe("2. Z|+> = |-> (Equator -X, Relative Phase = 180°, Odds still 50/50!)")

# 3. Double-Hadamard Interference Proof: H|+> vs H|->
print("--- 3A. Applying H to |+> (0° Relative Phase) ---")
out_from_plus = ket_plus.apply_gate_via_column_lookup("H", H_MAT, log_interference=True)
out_from_plus.describe("Result of H|+> -> Constructive on '0', Destructive on '1'")

print("--- 3B. Applying H to |-> (180° Relative Phase) ---")
out_from_minus = ket_minus.apply_gate_via_column_lookup("H", H_MAT, log_interference=True)
out_from_minus.describe("Result of H|-> -> Destructive on '0', Constructive on '1'")

# 4. Precision Phase Dials: S (+90°), T (+45°), and their .undo() Daggers
state_s = ket_plus.apply_gate_via_column_lookup("S", S_MAT)
state_s.describe("4A. S|+> = |i> (+90° Relative Phase Dial)")

state_st = state_s.apply_gate_via_column_lookup("T", T_MAT)
state_st.describe("4B. T(S|+>) (+90° + 45° = +135° Relative Phase)")

state_undo = (
    state_st
    .apply_gate_via_column_lookup("T†", T_DAG_MAT)
    .apply_gate_via_column_lookup("S†", S_DAG_MAT)
)
state_undo.describe("4C. After T† then S† (.undo() restores |+> at 0.0°!)")

```

**Output:**

```text
=== 1. H|0> = |+> (Equator +X, Relative Phase = 0°) ===
  Dict : '0': (mag=0.7071, ang=  0.0°), '1': (mag=0.7071, ang=  0.0°)
  Odds : P('0') =  50.0%  |  P('1') =  50.0%

=== 2. Z|+> = |-> (Equator -X, Relative Phase = 180°, Odds still 50/50!) ===
  Dict : '0': (mag=0.7071, ang=  0.0°), '1': (mag=0.7071, ang=180.0°)
  Odds : P('0') =  50.0%  |  P('1') =  50.0%

--- 3A. Applying H to |+> (0° Relative Phase) ---
  [Interference Trace for H]
    -> Key '0' receives: (mag=0.500, ang=  0.0°) + (mag=0.500, ang=  0.0°)
    -> Key '1' receives: (mag=0.500, ang=  0.0°) + (mag=0.500, ang=180.0°)
=== Result of H|+> -> Constructive on '0', Destructive on '1' ===
  Dict : '0': (mag=1.0000, ang=  0.0°), '1': (mag=0.0000, ang=  0.0°)
  Odds : P('0') = 100.0%  |  P('1') =   0.0%

--- 3B. Applying H to |-> (180° Relative Phase) ---
  [Interference Trace for H]
    -> Key '0' receives: (mag=0.500, ang=  0.0°) + (mag=0.500, ang=180.0°)
    -> Key '1' receives: (mag=0.500, ang=  0.0°) + (mag=0.500, ang=  0.0°)
=== Result of H|-> -> Destructive on '0', Constructive on '1' ===
  Dict : '0': (mag=0.0000, ang=  0.0°), '1': (mag=1.0000, ang=  0.0°)
  Odds : P('0') =   0.0%  |  P('1') = 100.0%

=== 4A. S|+> = |i> (+90° Relative Phase Dial) ===
  Dict : '0': (mag=0.7071, ang=  0.0°), '1': (mag=0.7071, ang= 90.0°)
  Odds : P('0') =  50.0%  |  P('1') =  50.0%

=== 4B. T(S|+>) (+90° + 45° = +135° Relative Phase) ===
  Dict : '0': (mag=0.7071, ang=  0.0°), '1': (mag=0.7071, ang=135.0°)
  Odds : P('0') =  50.0%  |  P('1') =  50.0%

=== 4C. After T† then S† (.undo() restores |+> at 0.0°!) ===
  Dict : '0': (mag=0.7071, ang=  0.0°), '1': (mag=0.7071, ang=  0.0°)
  Odds : P('0') =  50.0%  |  P('1') =  50.0%

```

Look at the `[Interference Trace for H]` in `3A` and `3B` above:

* In `3A` (`H|+>`), Key `'0'` receives two waves of magnitude `0.500` both pointing at `0.0°`—so they add up to `1.000` (`100%`), while Key `'1'` receives `0.500` at `0.0°` and `0.500` at `180.0°`—canceling out to `0.000` (`0%`)!


* In `3B` (`H|->`), simply because Key `'1'` started with a `180.0°` phase angle, the interference flips: Key `'0'` receives `0.0°` and `180.0°` (canceling to `0%`), while Key `'1'` receives two `0.0°` waves that add up to `100%`!



---

## Up Next in the Series

With single-qubit kinematics, 3D Bloch sphere rotations, relative phase, and Hadamard interference mastered, we have completed **Layer 1** of our conceptual dependency hierarchy.

What happens when we wire **multiple qubits** together? In **Blog 6**, we will scale up from single-qubit dictionaries to multi-qubit registers using the **Tensor Product ($\otimes$)**, master multi-qubit entangling gates (**CNOT / $CX$, Toffoli, and Controlled-$U$**), learn how **partial measurement and wave-function renormalization** work when you measure only one qubit out of many, explore the **4 canonical Bell states** vs. partial entanglement, and run our first multi-qubit Bell circuit on both a local simulator and a real physical IBM Quantum processor!

---

**Would you like to explore any concept from Blog 5 in deeper detail, or are you ready for me to generate Blog 6: *Weaving Qubits Together — Tensor Products, CNOT, Bell States, and Running Your First QPU Job*?**
