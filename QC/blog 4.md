### Blog 4: Anatomy of a Qubit — Dirac Notation, Born’s Rule, and Navigating the Bloch Sphere

In Blogs 2 and 3, we built our mathematical engine: we saw that complex numbers in exponential form ($re^{i\theta}$) are really 2D structs holding a `(magnitude, angle)` pair, and that matrices are reversible geometry transformations acting on column vectors.

Now it is time to connect that math directly to physical reality. When someone says a quantum computer uses **qubits** instead of classical bits, what is a qubit physically, how do we represent it mathematically without drowning in 7 different notations, what actually happens when you measure it, and how does every single-qubit state map onto a 3D globe called the **Bloch Sphere**?

In this post, we will unpack the complete anatomy of a qubit from the ground up, work through every exercise from our course material, and show in our **Software Engineer’s Unified View** how column vectors, Dirac kets ($\vert{}\psi\rangle$), Born’s rule, and 3D Bloch sphere coordinates ($\theta, \phi$) are all just views of the exact same underlying data structure.

---

#### 1. The Software Engineer's Rosettastone: Why Do Qubits Have Three Different Notations?

Your earlier observation about quantum documentation is sharpest right here: when you read about a single qubit, textbooks jump back and forth between **Column Vectors**, **Dirac (Ket) Notation**, and **3D Bloch Sphere Angles**. Why do all three exist, and how do they unify into one software model?

| Representation | Textbook Syntax | Why Physicists Invented It | Unified Software Engineering Equivalent |
| --- | --- | --- | --- |
| **1. Dense Array (Column Vector)** | $\begin{bmatrix} \alpha \\ \beta \end{bmatrix}$

 | Useful for multiplying small $2 \times 2$ matrices by hand on a chalkboard.

 | **Dense Array (`List[AmplitudePolar]`):** Index `0` holds the `"0"` amplitude; Index `1` holds the `"1"` amplitude. Wastes memory when $n$ grows to $2^n$.

 |
| **2. Sparse Key-Value Map (Dirac / Ket Notation)** | $\Vert{}\psi\rangle = \alpha\Vert{}0\rangle + \beta\Vert{}1\rangle$

 | Writing a $2^{10} = 1024$-row vector on a blackboard is impossible. Kets ($\Vert{}0\rangle, \Vert{}1\rangle$) let you write *only* the non-zero keys.

 | **Sparse Dictionary (`Dict[str, AmplitudePolar]`):** `{"0": (mag_0, angle_0), "1": (mag_1, angle_1)}`. Zero-amplitude keys are simply omitted!

 |
| **3. Normalized 3D Spherical Coordinates (Bloch Sphere)** | $\Vert{}\psi\rangle = \cos\left(\frac{\theta}{2}\right)\Vert{}0\rangle + e^{i\phi}\sin\left(\frac{\theta}{2}\right)\Vert{}1\rangle$

 | Because probabilities must add up to $100\%$ ($\vert{}\alpha\vert{}^2 + \vert{}\beta\vert{}^2 = 1$) and global phase is ignored, a 1-qubit state only has **2 real degrees of freedom**: latitude $\theta$ and longitude $\phi$.

 | **Constrained 2-Parameter Constructor (`from_bloch_angles(theta_deg, phi_deg)`):** Computes `mag_0 = cos(theta/2)`, `mag_1 = sin(theta/2)`, and sets key `"1"`'s angle to `phi_deg`.

 |

Once you see that **Column Vectors are Dense Arrays**, **Dirac Kets are Sparse Dictionaries**, and **Bloch Angles are a 2-parameter factory method that enforces the $100\%$ probability rule automatically**, the fragmentation vanishes. Let's walk through each one in detail.

---

#### 2. From Classical Bits to Physical Qubits

Classical computers store and process all data using binary bits—discrete switches that are strictly `0` or `1`. Quantum computers, by contrast, use **qubits** (quantum bits).

##### What Is a Qubit Physically?

Physically, a qubit is not a mystical math equation—it can be **any two-level quantum particle that exists in two distinct, distinguishable physical states**:

* **A Photon of Light:** Its electric field can be polarized **horizontally** ($\leftrightarrow$, which we label state `0`) or **vertically** ($\updownarrow$, which we label state `1`).


* **An Electron’s Spin:** Spin-up ($\uparrow$, state `0`) vs. spin-down ($\downarrow$, state `1`).
* **A Superconducting Transmon Circuit (like IBM's QPUs):** The ground energy level ($E_0$, state `0`) vs. the first excited energy level ($E_1$, state `1`) cooled inside a dilution refrigerator at $\approx 15\text{ mK}$ ($-273^\circ\text{C}$).



##### Defining `0` and `1` as Orthogonal Column Vectors

Just like classical computers, we still use the labels `0` and `1` in quantum computation. However, instead of treating them as single numbers, we define the `0` state and the `1` state as **two-element column vectors**:

$$\vert{}0\rangle = \begin{bmatrix} 1 \\ 0 \end{bmatrix}, \qquad \vert{}1\rangle = \begin{bmatrix} 0 \\ 1 \end{bmatrix}$$

In general, any single-qubit state—conventionally denoted by the Greek letter **$\psi$ ("psi")**—is represented as a column vector with two elements:

$$\vert{}\psi\rangle = \begin{bmatrix} \alpha \\ \beta \end{bmatrix}$$

What do those two positions in the column vector mean?

* **The Top Element ($\alpha$):** Indicates **how much the qubit is in the $\vert{}0\rangle$ state**.


* **The Bottom Element ($\beta$):** Indicates **how much the qubit is in the $\vert{}1\rangle$ state**.



Now look back at how we defined $\vert{}0\rangle = \begin{bmatrix} 1 \\ 0 \end{bmatrix}$ and $\vert{}1\rangle = \begin{bmatrix} 0 \\ 1 \end{bmatrix}$:

* If a qubit is in the state $\begin{bmatrix} 1 \\ 0 \end{bmatrix}$, it has $1$ in the top slot and $0$ in the bottom slot—meaning it is **all in the $\vert{}0\rangle$ state and none in the $\vert{}1\rangle$ state**.


* Similarly, if a qubit is in the state $\begin{bmatrix} 0 \\ 1 \end{bmatrix}$, it has $0$ in the top slot and $1$ in the bottom slot—meaning it is **all in the $\vert{}1\rangle$ state and none in the $\vert{}0\rangle$ state**.



##### What Superposition Really Means (Without the Hand-Waving)

In quantum mechanics, **superposition** means a quantum particle is in a linear combination of two distinct states at the same time—like a photon of light being both horizontally and vertically polarized simultaneously.

Mathematically, there is nothing vague about it: **a qubit is in a superposition of $\vert{}0\rangle$ and $\vert{}1\rangle$ whenever both the top element ($\alpha$) and the bottom element ($\beta$) of its column vector are non-zero numbers**! Furthermore, as highlighted in our course material, because we can never directly observe a superposition without collapsing it, the most precise engineering way to think about a superposition is as **a mathematical description of the quantum state whose complex amplitudes ($\alpha, \beta$) both determine the probabilities of measurement outcomes and can interfere with each other like overlapping waves**.

---

#### 3. Measurement, Wave-Function Collapse, and Born’s Rule

So we have a qubit in a superposition $\vert{}\psi\rangle = \begin{bmatrix} \alpha \\ \beta \end{bmatrix}$ where both $\alpha$ and $\beta$ are non-zero. How do we read information out of that qubit?

Here we hit one of the foundational laws of quantum mechanics: **when you measure a quantum system, the act of measurement permanently forces (collapses) the system into the exact classical state you measured**.

```text
  BEFORE MEASUREMENT                         ACT OF MEASUREMENT                  AFTER MEASUREMENT
  (Quantum Superposition)                    (Random Collapse)                   (Permanent Classical State)

  |ψ⟩ = α|0⟩ + β|1⟩        ───────────►      Ask Qubit: "0 or 1?"   ─────┬────►  |0⟩ = [1, 0]^T  (with Prob |α|²)
  (Both α ≠ 0 and β ≠ 0)                     (Irreversible!)             │       Every future measurement = 0!
                                                                         └────►  |1⟩ = [0, 1]^T  (with Prob |β|²)
                                                                                 Every future measurement = 1!

```

Let's unpack what this means using our photon polarization example from the course:

1. **Only Binary Outcomes Are Ever Observed:** Even if a photon is in a superposition of horizontally and vertically polarized, when you pass it through a polarizing detector, **you will only ever measure it as horizontally polarized (`0`) OR vertically polarized (`1`)—never "halfway" or both**. You do *not* directly read out the numbers $\alpha$ and $\beta$ in a single measurement.


2. **Irreversible State Collapse:** Immediately after you measure the qubit, its superposition is destroyed. If the qubit was in a superposition and your detector clicks `0`, the qubit **collapses into the $\vert{}0\rangle = \begin{bmatrix} 1 \\ 0 \end{bmatrix}$ state permanently**. If you measure that same qubit a second, third, or hundredth time right afterward, it will return `0` every single time because it is no longer in a superposition!



##### Born’s Rule: What Are $\alpha$ and $\beta$ Good For?

If measuring a qubit only ever gives us a classic `0` or `1`, what is the point of the amplitudes $\alpha$ and $\beta$ telling us "how much" the qubit is in the $\vert{}0\rangle$ or $\vert{}1\rangle$ state?

They tell us the **probabilities** of measuring `0` or `1`! By **Born’s Rule**:

* **The probability of measuring `0` is the magnitude squared of the $\vert{}0\rangle$ amplitude:** $P(0) = \vert{}\alpha\vert{}^2$.


* **The probability of measuring `1` is the magnitude squared of the $\vert{}1\rangle$ amplitude:** $P(1) = \vert{}\beta\vert{}^2$.



And because there are only two possible outcomes (`0` or `1`) when you measure a single qubit, **the probability of measuring `0` plus the probability of measuring `1` must always add up to $1$ ($100\%$)**:

$$\vert{}\alpha\vert{}^2 + \vert{}\beta\vert{}^2 = 1$$

This equation is the **Normalization Constraint** of quantum computing. Every valid single-qubit state vector in the universe must satisfy $\vert{}\alpha\vert{}^2 + \vert{}\beta\vert{}^2 = 1$ (which geometrically means the state vector always has unit length $1$).

##### Worked Course Examples: Checking Valid vs. Invalid States & Computing Probabilities

Let's walk through the exact examples from the course material to see Born's Rule in action:

1. **The $75\% / 25\%$ Superposition State:**
Suppose we have a qubit in the state:



$$\vert{}\psi\rangle = \begin{bmatrix} \frac{\sqrt{3}}{2} \\ \frac{1}{2} \end{bmatrix}$$


* **Probability of measuring `0`:** Square the magnitude of the top element: $P(0) = \left\vert{}\frac{\sqrt{3}}{2}\right\vert{}^2 = \frac{3}{4} = 0.75$ (**$75\%$ chance**).


* **Probability of measuring `1`:** Square the magnitude of the bottom element: $P(1) = \left\vert{}\frac{1}{2}\right\vert{}^2 = \frac{1}{4} = 0.25$ (**$25\%$ chance**).


* **Validity Check:** Do the probabilities sum to $1$? $\frac{3}{4} + \frac{1}{4} = 1$ ($100\%$). Yes—this is a valid quantum state! And if we measure it once and happen to get `0`, the vector immediately changes from $\begin{bmatrix} \sqrt{3}/2 \\ 1/2 \end{bmatrix}$ to $\begin{bmatrix} 1 \\ 0 \end{bmatrix}$, so every subsequent measurement is guaranteed to be `0`.




2. **Why $\vert{}0\rangle = \begin{bmatrix} 1 \\ 0 \end{bmatrix}$ and $\vert{}1\rangle = \begin{bmatrix} 0 \\ 1 \end{bmatrix}$ Make Complete Sense:**
For $\vert{}0\rangle = \begin{bmatrix} 1 \\ 0 \end{bmatrix}$, the probability of measuring `0` is $\vert{}1\vert{}^2 = 1$ ($100\%$) and the probability of measuring `1` is $\vert{}0\vert{}^2 = 0$ ($0\%$)—so it will *always* be measured as `0`. The exact same logic applies to $\vert{}1\rangle = \begin{bmatrix} 0 \\ 1 \end{bmatrix}$.


3. **Spotting Valid vs. Invalid Qubit States:**
* **Is $\begin{bmatrix} \frac{1}{\sqrt{2}} \\ -\frac{1}{\sqrt{2}} \end{bmatrix}$ a valid qubit state?** Let's check: $\left\vert{}\frac{1}{\sqrt{2}}\right\vert{}^2 + \left\vert{}-\frac{1}{\sqrt{2}}\right\vert{}^2 = \frac{1}{2} + \frac{1}{2} = 1$. **Valid!** Each outcome has a $50\%$ probability.


* **Is $\begin{bmatrix} \frac{1}{\sqrt{2}} \\ \frac{i}{\sqrt{2}} \end{bmatrix}$ a valid qubit state?** Remember from Blog 2 that $\vert{}i\vert{} = 1$, so $\left\vert{}\frac{i}{\sqrt{2}}\right\vert{}^2 = \frac{\vert{}i\vert{}^2}{2} = \frac{1}{2}$. Thus $\frac{1}{2} + \frac{1}{2} = 1$. **Valid!**


* **Is $\begin{bmatrix} 1 \\ 1 \end{bmatrix}$ a valid qubit state?** Let's check: $\vert{}1\vert{}^2 + \vert{}1\vert{}^2 = 1 + 1 = 2 \neq 1$. **Invalid!** Because the probabilities add up to more than $1$ ($200\%$), $\begin{bmatrix} 1 \\ 1 \end{bmatrix}$ cannot exist unless it is multiplied by the normalization constant $\frac{1}{\sqrt{2}}$.





---

#### 4. Demystifying Dirac (Bra-Ket) Notation: Why Physicists Ditched Column Vectors

Normally, when we write out the quantum state of a quantum computer on paper or in documentation, we don't write column vectors—we use **Dirac notation** (also called **ket notation**).

Let's derive Dirac notation step by step from a standard column vector $\begin{bmatrix} \alpha \\ \beta \end{bmatrix}$ using basic linear algebra from Blog 3:

1. **Step 1 — Split the vector into the sum of two column vectors:**


$$\vert{}\psi\rangle = \begin{bmatrix} \alpha \\ \beta \end{bmatrix} = \begin{bmatrix} \alpha \\ 0 \end{bmatrix} + \begin{bmatrix} 0 \\ \beta \end{bmatrix}$$


2. **Step 2 — Factor out the scalars $\alpha$ and $\beta$:**


$$\vert{}\psi\rangle = \alpha \begin{bmatrix} 1 \\ 0 \end{bmatrix} + \beta \begin{bmatrix} 0 \\ 1 \end{bmatrix}$$


3. **Step 3 — Replace the basis vectors with $\vert{}0\rangle$ and $\vert{}1\rangle$:**
Look at the two column vectors left behind: $\begin{bmatrix} 1 \\ 0 \end{bmatrix}$ is just the $\vert{}0\rangle$ state, and $\begin{bmatrix} 0 \\ 1 \end{bmatrix}$ is just the $\vert{}1\rangle$ state! Substituting those in gives us a **linear combination of the $\vert{}0\rangle$ and $\vert{}1\rangle$ states**:



$$\vert{}\psi\rangle = \alpha\vert{}0\rangle + \beta\vert{}1\rangle$$



Those angled brackets $\vert{}\cdot\rangle$ around $0$, $1$, and $\psi$ are called **kets**. All a ket tells you is: *"the thing inside these brackets is a quantum state vector"*.

##### Worked Course Exercise: Converting Matrix Form to Dirac Notation

*Convert the column vector $\begin{bmatrix} \frac{1}{\sqrt{2}} \\ -\frac{i}{\sqrt{2}} \end{bmatrix}$ from matrix notation into Dirac notation*:

$$\begin{bmatrix} \frac{1}{\sqrt{2}} \\ -\frac{i}{\sqrt{2}} \end{bmatrix} = \begin{bmatrix} \frac{1}{\sqrt{2}} \\ 0 \end{bmatrix} + \begin{bmatrix} 0 \\ -\frac{i}{\sqrt{2}} \end{bmatrix} = \frac{1}{\sqrt{2}}\begin{bmatrix} 1 \\ 0 \end{bmatrix} - \frac{i}{\sqrt{2}}\begin{bmatrix} 0 \\ 1 \end{bmatrix} = \frac{1}{\sqrt{2}}\vert{}0\rangle - \frac{i}{\sqrt{2}}\vert{}1\rangle$$

##### The Real Engineering Reason Dirac Notation Exists

Why not just stick with column vectors forever? As noted in the course, for 1 qubit a column vector has $2^1 = 2$ rows. For 2 qubits, it has $2^2 = 4$ rows. For 10 qubits, it has $2^{10} = 1,024$ rows; and for 50 qubits, it has $2^{50} \approx 1.12 \times 10^{15}$ rows!

Suppose a 10-qubit register is in a Bell-like state where only the all-zeros state and all-ones state have non-zero probability. In column-vector notation, you would have to write a matrix with $1,024$ rows where $1,022$ of the rows are zeroes. In Dirac notation, you simply write the two non-zero dictionary keys:

$$\frac{1}{\sqrt{2}}\vert{}0000000000\rangle + \frac{1}{\sqrt{2}}\vert{}1111111111\rangle$$

Dirac notation is literally just **Sparse Dictionary Notation** where keys with amplitude `0` are omitted!

---

#### 5. Navigating the Bloch Sphere: The Complete 3D Geometric Map of a Single Qubit

While sparse dictionaries scale to any number of qubits, for a **single qubit** we have a wonderful visual tool: we can plot any pure single-qubit state as a 3D unit vector pointing from the center of a sphere to its surface—the **Bloch Sphere**.

```text
                         +Z Axis (North Pole)
                            |0⟩ = [1, 0]^T
                              (θ = 0)
                                 ▲
                                 │      ↗ |ψ⟩ = cos(θ/2)|0⟩ + e^(iφ)sin(θ/2)|1⟩
                                 │    /
                                 │  /  θ (Polar Latitude: controls 0 vs 1 odds)
                     |-i⟩        │/
                      ╲          ┼──────┬────────► +Y Axis: |i⟩ = (|0⟩ + i|1⟩)/√2
       -X Axis: |-⟩ ◄────────────┼──────┼────────► +X Axis: |+⟩ = (|0⟩ + |1⟩)/√2
                                /│  φ   │
                              /  │ (Azimuthal Longitude: controls Relative Phase)
                            ↙    │
                       +X        ▼
                         -Z Axis (South Pole)
                            |1⟩ = [0, 1]^T
                              (θ = π)

```

##### A. The General Bloch Sphere Equation ($\theta$ and $\phi$)

Why does a complex vector $\begin{bmatrix} \alpha \\ \beta \end{bmatrix}$ fit onto a 3D unit sphere parameterized by just two angles?

1. Because $\vert{}\alpha\vert{}^2 + \vert{}\beta\vert{}^2 = 1$, the magnitudes of $\alpha$ and $\beta$ always satisfy the trigonometric identity $\cos^2\left(\frac{\theta}{2}\right) + \sin^2\left(\frac{\theta}{2}\right) = 1$.


2. Because overall **global phase** (multiplying the entire qubit by $e^{i\gamma}$) has zero physical effect on any measurement, we can factor out the phase of $\vert{}0\rangle$ so that only the $\vert{}1\rangle$ state carries the **relative phase** factor $e^{i\phi}$.



That gives us the master equation of the Bloch Sphere:

$$\vert{}\psi\rangle = \cos\left(\frac{\theta}{2}\right)\vert{}0\rangle + e^{i\phi}\sin\left(\frac{\theta}{2}\right)\vert{}1\rangle$$

Let's understand what the two angles control geometrically:

* **Polar Angle / Latitude $\theta \in [0, \pi]$ ($0^\circ$ to $180^\circ$):** Measures how far the arrow tilts down from the North Pole ($+Z$ axis) toward the South Pole ($-Z$ axis). **$\theta$ strictly controls the measurement probabilities**:



$$P(0) = \cos^2\left(\frac{\theta}{2}\right), \qquad P(1) = \sin^2\left(\frac{\theta}{2}\right)$$



The closer up the qubit vector is to the North Pole ($\vert{}0\rangle$), the higher the probability of measuring `0`; the lower the qubit vector is toward the South Pole ($\vert{}1\rangle$), the higher the probability of measuring `1`.


* **Azimuthal Angle / Longitude $\phi \in [0, 2\pi)$ ($0^\circ$ to $360^\circ$):** Measures how far the arrow spins horizontally around the vertical $Z$-axis in the $XY$-plane. **$\phi$ is the relative phase** of the qubit. Spinning around the sphere horizontally changes $\phi$ without changing the qubit's height—so the probabilities $P(0)$ and $P(1)$ stay completely unchanged!



> **💡 Why $\frac{\theta}{2}$ Instead of $\theta$?** Notice that on the Bloch Sphere, $\vert{}0\rangle$ is at the North Pole ($\theta = 0$) and $\vert{}1\rangle$ is at the South Pole ($\theta = \pi = 180^\circ$). In linear algebra, $\vert{}0\rangle = \begin{bmatrix} 1 \\ 0 \end{bmatrix}$ and $\vert{}1\rangle = \begin{bmatrix} 0 \\ 1 \end{bmatrix}$ are orthogonal ($90^\circ$ / $\frac{\pi}{2}$ apart). Dividing $\theta$ by $2$ stretches a $90^\circ$ Hilbert-space angle into a $180^\circ$ pole-to-pole angle on the sphere so that opposite basis states sit at opposite poles!
> 
> 

##### B. The 6 Cardinal Landmarks on the Bloch Sphere

Let's tour the 6 standard states located at the poles and along the equator of the Bloch sphere:

| Bloch Sphere Position | Shorthand Ket | Expanded Dirac Form | Column Vector | Angles $(\theta, \phi)$ | Measurement Odds $(P_0 / P_1)$ |
| --- | --- | --- | --- | --- | --- |
| **North Pole ($+Z$)** | $\Vert{}0\rangle$

 | $1\Vert{}0\rangle + 0\Vert{}1\rangle$

 | $\begin{bmatrix} 1 \\ 0 \end{bmatrix}$

 | $\theta = 0^\circ, \phi = \text{any}$

 | **$100\%$ `0` / $0\%$ `1**`

 |
| **South Pole ($-Z$)** | $\Vert{}1\rangle$

 | $0\Vert{}0\rangle + 1\Vert{}1\rangle$

 | $\begin{bmatrix} 0 \\ 1 \end{bmatrix}$

 | $\theta = 180^\circ, \phi = \text{any}$

 | **$0\%$ `0` / $100\%$ `1**`

 |
| **Equator $+X$ Axis** | $\Vert{}+\rangle$

 | $\frac{1}{\sqrt{2}}\Vert{}0\rangle + \frac{1}{\sqrt{2}}\Vert{}1\rangle$

 | $\frac{1}{\sqrt{2}}\begin{bmatrix} 1 \\ 1 \end{bmatrix}$

 | $\theta = 90^\circ, \phi = 0^\circ$

 | **$50\%$ `0` / $50\%$ `1**` (Phase $0^\circ$)

 |
| **Equator $-X$ Axis** | $\Vert{}-\rangle$

 | $\frac{1}{\sqrt{2}}\Vert{}0\rangle - \frac{1}{\sqrt{2}}\Vert{}1\rangle$

 | $\frac{1}{\sqrt{2}}\begin{bmatrix} 1 \\ -1 \end{bmatrix}$

 | $\theta = 90^\circ, \phi = 180^\circ$

 | **$50\%$ `0` / $50\%$ `1**` (Phase $180^\circ$)

 |
| **Equator $+Y$ Axis** | $\Vert{}i\rangle$

 | $\frac{1}{\sqrt{2}}\Vert{}0\rangle + \frac{i}{\sqrt{2}}\Vert{}1\rangle$

 | $\frac{1}{\sqrt{2}}\begin{bmatrix} 1 \\ i \end{bmatrix}$

 | $\theta = 90^\circ, \phi = 90^\circ$

 | **$50\%$ `0` / $50\%$ `1**` (Phase $90^\circ$)

 |
| **Equator $-Y$ Axis** | $\Vert{}-i\rangle$

 | $\frac{1}{\sqrt{2}}\Vert{}0\rangle - \frac{i}{\sqrt{2}}\Vert{}1\rangle$

 | $\frac{1}{\sqrt{2}}\begin{bmatrix} 1 \\ -i \end{bmatrix}$

 | $\theta = 90^\circ, \phi = 270^\circ$

 | **$50\%$ `0` / $50\%$ `1**` (Phase $270^\circ$)

 |

Notice how all four equatorial states ($\vert{}+\rangle, \vert{}-\rangle, \vert{}i\rangle, \vert{}-i\rangle$) lie halfway between the North and South Poles ($\theta = 90^\circ = \frac{\pi}{2}$), so they all have an exact $50/50$ chance of being measured as `0` or `1`—differing *only* in their relative phase $\phi$ ($0^\circ, 180^\circ, 90^\circ, 270^\circ$)!

##### C. Worked Course Exercise: Locating $\begin{bmatrix} \frac{1}{2} \\ \frac{\sqrt{3}}{2} \end{bmatrix}$ on the Bloch Sphere

Let's solve the Bloch Sphere puzzle posed in the course video: *Where does the qubit state $\begin{bmatrix} \frac{1}{2} \\ \frac{\sqrt{3}}{2} \end{bmatrix}$ lie on the Bloch Sphere?*

1. **Calculate the Measurement Probabilities:**
* Probability of measuring `0`: $P(0) = \left\vert{}\frac{1}{2}\right\vert{}^2 = \frac{1}{4}$ (**$25\%$ chance**).


* Probability of measuring `1`: $P(1) = \left\vert{}\frac{\sqrt{3}}{2}\right\vert{}^2 = \frac{3}{4}$ (**$75\%$ chance**).




2. **Find the Exact Latitude Angle $\theta$:**
Using $\cos\left(\frac{\theta}{2}\right) = \frac{1}{2}$ and $\sin\left(\frac{\theta}{2}\right) = \frac{\sqrt{3}}{2}$:



$$\frac{\theta}{2} = \arccos\left(\frac{1}{2}\right) = \frac{\pi}{3}\text{ radians } (60^\circ) \quad \Longrightarrow \quad \theta = \frac{2\pi}{3}\text{ radians } (120^\circ)$$


3. **Geometric Location:**
Since the Equator is at $\theta = 90^\circ$ ($\frac{\pi}{2}$) and the South Pole ($\vert{}1\rangle$) is at $\theta = 180^\circ$ ($\pi$), an angle of $\theta = 120^\circ$ ($\frac{2\pi}{3}$) places the qubit in the **Southern Hemisphere, tilted down below the Equator toward the $\vert{}1\rangle$ South Pole** (with $\phi = 0^\circ$ along the $+X$ meridian), reflecting its $\frac{3}{4}$ ($75\%$) preference for collapsing to `1`!



---

#### 6. The Software Engineer's Unified View: One Explicit Class Bridging Dictionaries, Vectors, and Bloch Coordinates

Let's bring all of Blog 4 together into a clean, verbose, self-documenting Python architecture. Notice how our `UnifiedQubitState` class below eliminates the notation fragmentation:

* Internally, it stores **one canonical representation**: our sparse dictionary mapping `"0"` and `"1"` to `AmplitudePolar(magnitude, angle_degrees)`.


* It provides explicit, self-documenting converters to construct or inspect the state from **Column Vectors**, **Dirac strings**, or **Bloch Sphere angles $(\theta, \phi)$**, plus an explicit `measure_and_collapse()` method demonstrating irreversible wave-function collapse:



```python
import math
import random
from dataclasses import dataclass
from typing import Dict, List, Tuple


@dataclass(frozen=True)
class AmplitudePolar:
    """Explicit (magnitude, angle_degrees) representation of a complex amplitude."""
    magnitude: float
    angle_degrees: float

    @classmethod
    def from_complex(cls, value: complex) -> "AmplitudePolar":
        mag = float(abs(value))
        if mag < 1e-12:
            return cls(magnitude=0.0, angle_degrees=0.0)
        angle_deg = (math.degrees(math.atan2(value.imag, value.real)) + 360.0) % 360.0
        return cls(magnitude=round(mag, 6), angle_degrees=round(angle_deg, 2))

    def to_complex(self) -> complex:
        rad = math.radians(self.angle_degrees)
        return self.magnitude * complex(math.cos(rad), math.sin(rad))


class UnifiedQubitState:
    """
    A single unified data structure for a Qubit that bridges:
      1. Sparse Dictionary (Dirac Ket notation): {'0': (mag, angle), '1': (mag, angle)}
      2. Dense Array (Column Vector): [[alpha], [beta]]
      3. 3D Bloch Sphere Coordinates: (theta_degrees, phi_degrees) and (x, y, z)
    """
    def __init__(self, state_dictionary: Dict[str, AmplitudePolar]):
        # Enforce Born's Rule normalization constraint: |alpha|^2 + |beta|^2 == 1.0
        total_prob = sum(amp.magnitude ** 2 for amp in state_dictionary.values())
        if not math.isclose(total_prob, 1.0, rel_tol=1e-4):
            raise ValueError(
                f"Invalid Qubit State! Total probability is {total_prob:.4f}, "
                f"but Born's Rule requires sum(magnitude^2) == 1.0."
            )
        self.state_map: Dict[str, AmplitudePolar] = {
            "0": state_dictionary.get("0", AmplitudePolar(0.0, 0.0)),
            "1": state_dictionary.get("1", AmplitudePolar(0.0, 0.0)),
        }

    @classmethod
    def from_column_vector(cls, alpha: complex, beta: complex) -> "UnifiedQubitState":
        """Constructs the unified state from a 2x1 column vector [alpha, beta]^T."""
        return cls({
            "0": AmplitudePolar.from_complex(alpha),
            "1": AmplitudePolar.from_complex(beta),
        })

    @classmethod
    def from_bloch_angles(cls, theta_degrees: float, phi_degrees: float) -> "UnifiedQubitState":
        """
        Constructs the unified state from 3D Bloch Sphere coordinates:
          |psi> = cos(theta/2)|0> + e^(i*phi) * sin(theta/2)|1>
        Automatically satisfies Born's Rule because cos^2 + sin^2 == 1!
        """
        half_theta_rad = math.radians(theta_degrees / 2.0)
        mag_zero = math.cos(half_theta_rad)
        mag_one = math.sin(half_theta_rad)
        return cls({
            "0": AmplitudePolar(magnitude=round(mag_zero, 6), angle_degrees=0.0),
            "1": AmplitudePolar(
                magnitude=round(mag_one, 6),
                angle_degrees=round(phi_degrees % 360.0, 2) if mag_one > 1e-12 else 0.0
            ),
        })

    def to_bloch_coordinates(self) -> Tuple[float, float, Tuple[float, float, float]]:
        """
        Extracts (theta_deg, phi_deg, (x, y, z)) on the 3D Bloch Sphere after
        factoring out global phase from key '0'.
        """
        amp0 = self.state_map["0"]
        amp1 = self.state_map["1"]

        # Clamp magnitude to [0, 1] for numerical safety in acos
        clamped_mag0 = max(0.0, min(1.0, amp0.magnitude))
        theta_rad = 2.0 * math.acos(clamped_mag0)
        theta_deg = round(math.degrees(theta_rad), 2)

        # Relative phase phi is (angle_of_1 - angle_of_0)
        if amp0.magnitude > 1e-9 and amp1.magnitude > 1e-9:
            phi_deg = round((amp1.angle_degrees - amp0.angle_degrees + 360.0) % 360.0, 2)
        else:
            phi_deg = 0.0

        phi_rad = math.radians(phi_deg)
        x = round(math.sin(theta_rad) * math.cos(phi_rad), 4)
        y = round(math.sin(theta_rad) * math.sin(phi_rad), 4)
        z = round(math.cos(theta_rad), 4)
        return theta_deg, phi_deg, (x, y, z)

    def measure_and_collapse(self) -> str:
        """
        Executes Born's Rule measurement and PERMANENTLY collapses the state dictionary
        into the measured classical state ('0' or '1').
        """
        prob_zero = self.state_map["0"].magnitude ** 2
        random_draw = random.random()

        if random_draw < prob_zero:
            measured_bit = "0"
            self.state_map = {
                "0": AmplitudePolar(1.0, 0.0),
                "1": AmplitudePolar(0.0, 0.0),
            }
        else:
            measured_bit = "1"
            self.state_map = {
                "0": AmplitudePolar(0.0, 0.0),
                "1": AmplitudePolar(1.0, 0.0),
            }
        return measured_bit

    def inspect(self, label: str) -> None:
        theta_deg, phi_deg, (x, y, z) = self.to_bloch_coordinates()
        p0 = (self.state_map["0"].magnitude ** 2) * 100.0
        p1 = (self.state_map["1"].magnitude ** 2) * 100.0
        print(f"=== {label} ===")
        print(f"  1. Sparse Dict (Ket) : '0': (mag={self.state_map['0'].magnitude:.4f}, ang={self.state_map['0'].angle_degrees:5.1f}°), "
              f"'1': (mag={self.state_map['1'].magnitude:.4f}, ang={self.state_map['1'].angle_degrees:5.1f}°)")
        print(f"  2. Born's Rule Odds  : P('0') = {p0:5.1f}%  |  P('1') = {p1:5.1f}%")
        print(f"  3. 3D Bloch Sphere   : Latitude θ = {theta_deg:5.1f}°, Longitude φ = {phi_deg:5.1f}° -> 3D Vector (x={x}, y={y}, z={z})\n")


# =====================================================================
# Verify the 6 Cardinal Landmarks & Course Exercises
# =====================================================================
inv_sqrt2 = 1.0 / math.sqrt(2.0)

state_plus = UnifiedQubitState.from_bloch_angles(theta_degrees=90.0, phi_degrees=0.0)
state_plus_i = UnifiedQubitState.from_column_vector(inv_sqrt2, inv_sqrt2 * 1j)
state_course = UnifiedQubitState.from_column_vector(0.5, math.sqrt(3.0) / 2.0)

state_plus.inspect("Equator +X Landmark: |+> State")
state_plus_i.inspect("Equator +Y Landmark: |i> State")
state_course.inspect("Course Bloch Exercise: [1/2, sqrt(3)/2]^T")

# Demonstrate irreversible collapse on the Course Exercise state
random.seed(42)
outcome = state_course.measure_and_collapse()
print(f"--> Measured Qubit! Detector clicked: '{outcome}'")
state_course.inspect("Course State AFTER Measurement (Permanently Collapsed!)")

```

**Output:**

```text
=== Equator +X Landmark: |+> State ===
  1. Sparse Dict (Ket) : '0': (mag=0.7071, ang=  0.0°), '1': (mag=0.7071, ang=  0.0°)
  2. Born's Rule Odds  : P('0') =  50.0%  |  P('1') =  50.0%
  3. 3D Bloch Sphere   : Latitude θ =  90.0°, Longitude φ =   0.0° -> 3D Vector (x=1.0, y=0.0, z=0.0)

=== Equator +Y Landmark: |i> State ===
  1. Sparse Dict (Ket) : '0': (mag=0.7071, ang=  0.0°), '1': (mag=0.7071, ang= 90.0°)
  2. Born's Rule Odds  : P('0') =  50.0%  |  P('1') =  50.0%
  3. 3D Bloch Sphere   : Latitude θ =  90.0°, Longitude φ =  90.0° -> 3D Vector (x=0.0, y=1.0, z=0.0)

=== Course Bloch Exercise: [1/2, sqrt(3)/2]^T ===
  1. Sparse Dict (Ket) : '0': (mag=0.5000, ang=  0.0°), '1': (mag=0.8660, ang=  0.0°)
  2. Born's Rule Odds  : P('0') =  25.0%  |  P('1') =  75.0%
  3. 3D Bloch Sphere   : Latitude θ = 120.0°, Longitude φ =   0.0° -> 3D Vector (x=0.866, y=0.0, z=-0.5)

--> Measured Qubit! Detector clicked: '1'
=== Course State AFTER Measurement (Permanently Collapsed!) ===
  1. Sparse Dict (Ket) : '0': (mag=0.0000, ang=  0.0°), '1': (mag=1.0000, ang=  0.0°)
  2. Born's Rule Odds  : P('0') =   0.0%  |  P('1') = 100.0%
  3. 3D Bloch Sphere   : Latitude θ = 180.0°, Longitude φ =   0.0° -> 3D Vector (x=0.0, y=0.0, z=-1.0)

```

Look at line 3 of the output for `[1/2, sqrt(3)/2]^T`: the 3D Bloch vector has vertical coordinate `z = -0.5`—literally **halfway between the Equator (`z = 0.0`) and the South Pole (`z = -1.0`)**, matching the exact visual explanation from the course video!

And if you want to render any of these states visually in 3D using **Qiskit**, you can run these few lines from our notebook:

```python
import numpy as np
from qiskit.visualization import plot_bloch_vector

# Plot the course state [1/2, sqrt(3)/2]^T using spherical coordinates [r=1, theta=2*pi/3, phi=0]
bloch_spherical = [1.0, 2 * np.pi / 3, 0.0]
fig = plot_bloch_vector(bloch_spherical, coord_type="spherical", title="State [1/2, sqrt(3)/2]^T")
fig.show()

```

---

#### Up Next in the Series

Now that we know how to represent a qubit as a unified state dictionary and visualize it on the 3D Bloch Sphere, how do we **manipulate** it? In **Blog 5**, we will master the complete suite of **Single-Qubit Gates ($X, Y, Z, H, S, T, S^\dagger, T^\dagger$)**, see why **Global Phase** is physically invisible while **Relative Phase** is the secret weapon of quantum computing, and prove mathematically how the **Hadamard gate ($H$)** turns invisible phase differences into $100\%$ deterministic measurement outcomes via wave interference.

---

**Would you like to explore any detail from Blog 4 further, or are you ready for me to generate Blog 5:** ***Dials, Mirrors, and Wave Splitters — Single-Qubit Gates, Relative Phase, and Hadamard Interference*** **?**
