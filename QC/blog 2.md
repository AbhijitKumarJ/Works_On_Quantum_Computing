### Blog 2: Spinning Around the Unit Circle — Complex Numbers, Euler’s Formula, and the Secret Math of Phase

In Blog 1, we saw that the true engine of quantum speedup is **wave interference**—sculpting probability amplitudes so that wrong answers cancel out and right answers amplify each other.

Wait a second: in standard probability theory, probabilities are always positive real numbers between $0$ and $1$. How can two probabilities ever "cancel each other out" or point in different directions around a 3D sphere?

They can't—unless the underlying state of your system is built not on raw probabilities, but on **complex probability amplitudes** that carry both a magnitude and a rotation angle (phase). If you open any quantum computing textbook or inspect a statevector in Qiskit, you will immediately see expressions like $\frac{1}{\sqrt{2}}e^{i\pi/4}$. At first glance, raising the number $e$ to an imaginary power multiplied by $\pi$ looks like mathematical wizardry designed to intimidate newcomers. In reality, it is the cleanest, most visual way to describe **spinning a dial around a circle**.

In this post, we will build your mathematical intuition for imaginary and complex numbers from scratch, work through the exact exercises from our course material, and uncover why quantum computing relies almost exclusively on **exponential form ($re^{i\theta}$)** to model quantum states.

---

#### 1. From Impossible Square Roots to Imaginary Numbers

Let's start with a simple algebra warm-up involving square roots. Suppose we want to solve the equation:

$$x^2 = 4$$

That is easy enough: both $2^2 = 4$ and $(-2)^2 = 4$, which means $x = \pm 2$.

Now, what happens if we flip the sign on the right-hand side?

$$x^2 = -4$$

In the realm of ordinary real numbers, we hit a brick wall: since any positive or negative real number squared is always greater than or equal to zero, how can $x^2$ ever be a negative number?

To break through this wall, mathematicians defined a new fundamental unit, **$i$**, to represent the square root of negative one:

$$i = \sqrt{-1} \quad \Longleftrightarrow \quad i^2 = -1$$

Once we allow $i = \sqrt{-1}$ into our toolkit, $\sqrt{-4}$ is no longer impossible. We can factor out $\sqrt{-1}$ and write:

$$\sqrt{-4} = \sqrt{4 \times (-1)} = \sqrt{4} \cdot \sqrt{-1} = \pm 2i$$

Any number that contains a factor of $\sqrt{-1}$ (or $i$) is called an **imaginary number**. Despite the unfortunate historical name "imaginary," there is nothing fake about $i$—as we are about to see, multiplying by $i$ has a concrete geometric meaning: it is a $90^\circ$ ($\frac{\pi}{2}$ radian) rotation in space.

---

#### 2. Stepping Up to Complex Numbers & Their Arithmetic

Now we move one level up from pure imaginary numbers to **complex numbers**. All a complex number is is a number that combines a real number plus an imaginary number. A standard complex number $z$ is written in **Cartesian (rectangular) form** as:

$$z = a + ib$$

where both $a$ and $b$ are ordinary real numbers ($a, b \in \mathbb{R}$):

* **$a$** is called the **real part** of the number.


* **$b$** is called the **imaginary part** of the number.



At first glance, glueing a real number and an imaginary number together might not seem very useful for building a computer. However, Complex Numbers have three algebraic properties that make them tailor-made for quantum mechanics. Let's walk through how they behave under basic operations:

##### A. Addition and Subtraction

Adding and subtracting complex numbers is completely straightforward: all we do is add or subtract the real parts together and add or subtract the imaginary parts together:

$$(a + ib) + (c + id) = (a + c) + i(b + d)$$

$$(a + ib) - (c + id) = (a - c) + i(b - d)$$

* **Quick Practice:** Let's add $(3 + 4i)$ and $(5 - 2i)$:



$$(3 + 4i) + (5 - 2i) = (3 + 5) + i(4 - 2) = 8 + 2i$$



##### B. Multiplication

Multiplying two complex numbers works just like expanding two binomial brackets in high-school algebra, with one crucial rule: **whenever we get $i^2$, we replace it with $-1$** since $i = \sqrt{-1}$:

$$(a + ib)(c + id) = ac + iad + ibc + i^2 bd = (ac - bd) + i(ad + bc)$$

* **Quick Practice:** Multiply $(1 + 2i)$ by $(3 + i)$:



$$(1 + 2i)(3 + i) = 3 + i + 6i + 2i^2 = 3 + 7i + 2(-1) = 1 + 7i$$



---

#### 3. The Complex Conjugate ($z^*$): Turning Complex Amplitudes into Real Probabilities

Here is the first superpower of complex numbers. Suppose a qubit's state has a complex amplitude like $\beta = \frac{1}{\sqrt{2}}i$. When we measure the qubit in the real physical world, the probability of measuring a $1$ cannot be an imaginary number—probabilities in the lab must be real numbers between $0$ and $1$. How does nature convert a complex amplitude into a real-world probability?

Through an operation called the **Complex Conjugate**.

To find the complex conjugate of any complex number $z = a + ib$, all you do is **flip the sign (negate) of the imaginary part**. We denote the complex conjugate with an asterisk ($z^*$):

$$z = a + ib \quad \xrightarrow{\text{Conjugate}} \quad z^* = a - ib$$

Here are a few quick examples of taking the complex conjugate:

* If $z = 2 + 3i$, then $z^* = 2 - 3i$.


* If $z = 4 - 5i$, then $z^* = 4 + 5i$.


* If $z = i$, then $z^* = -i$.


* If $z = 7$ (purely real, i.e., $7 + 0i$), then $z^* = 7$.



##### The "Always Real" Property

One of the coolest and most important properties in all of quantum mathematics happens when you **multiply any complex number by its own complex conjugate ($z \cdot z^*$)**:

$$z \cdot z^* = (a + ib)(a - ib) = a^2 - iab + iab - i^2 b^2 = a^2 - (-1)b^2 = a^2 + b^2$$

Notice what just happened: the cross-terms ($-iab$ and $+iab$) canceled out completely, and $-i^2$ became $+1$! **Multiplying any complex number by its complex conjugate always yields a non-negative real number**.

> **Worked Notebook Exercise:** *Multiply $2 + 3i$ with its complex conjugate $2 - 3i$ and confirm that the result is a real number*.
> 
> 
> $$(2 + 3i)(2 - 3i) = 2(2) - 2(3i) + (3i)(2) - (3i)(3i)$$
> 
> 
> $$= 4 - 6i + 6i - 9i^2$$
> 
> 
> $$= 4 - 9(-1) = 4 + 9 = 13$$
> 
> 
> The imaginary part vanishes completely, leaving the pure real number **$13$**.
> 
> 

---

#### 4. Complex Numbers as 2D Vectors & Magnitude ($\vert{}z\vert{}$)

Instead of thinking of $a + ib$ as an abstract algebraic sum, we can visualize it geometrically as a **2D vector** on the **complex plane** (also called the Argand diagram), where:

* The **Real Axis ($\text{Re}$)** runs along the horizontal ($x$-axis).


* The **Imaginary Axis ($\text{Im}$)** runs along the vertical ($y$-axis).



For example, the complex number $z = -2 + 3i$ is simply a vector starting at the origin $(0, 0)$ and ending at the coordinate point $(-2, 3)$:

```text
                      Imaginary Axis (Im)
                              ▲
         (-2, 3)              │
       z = -2 + 3i  ●─────────┤ 3i
                     \        │
                      \  |z|  │
                       \      │   θ
                        \     │  ╭─╮
      ───────────────────┴────┼──┴──► Real Axis (Re)
                        -2    0

```

When we graph a complex number on the plane like this, we immediately uncover a fundamental geometric property: its **magnitude** (or absolute value), which is the length of the vector—its straight-line distance from the origin $(0,0)$.

We denote the magnitude of a complex number with vertical bars, $\vert{}z\vert{}$, and calculate it using **Pythagoras's Theorem** on the right-angled triangle formed by $a$ and $b$:

$$\vert{}a + ib\vert{} = \sqrt{a^2 + b^2}$$

Look closely at the inside of that square root ($a^2 + b^2$) and compare it to our complex conjugate formula above ($z \cdot z^* = a^2 + b^2$). They are identical! That means the **magnitude squared** of any complex number is simply the number multiplied by its complex conjugate:

$$\vert{}z\vert{}^2 = z \cdot z^* = a^2 + b^2$$

When we get to Blog 4 and learn **Born's Rule**, you will see that the probability of measuring a qubit in state $\vert{}0\rangle$ or $\vert{}1\rangle$ is the **magnitude squared** of its amplitude ($\vert{}\alpha\vert{}^2$ and $\vert{}\beta\vert{}^2$). That is why complex conjugates and magnitudes are everywhere in quantum computing!

> **Worked Notebook Exercise:** *Find the magnitude of the complex number $4 - 3i$*.
> 
> 
> Here, the real part is $a = 4$ and the imaginary part is $b = -3$. Applying Pythagoras's theorem:
> 
> 
> 
> $$\vert{}4 - 3i\vert{} = \sqrt{4^2 + (-3)^2} = \sqrt{16 + 9} = \sqrt{25} = 5$$
> 
> 
> 
> The vector $4 - 3i$ has a length (magnitude) of exactly **$5$**.
> 
> 

---

#### 5. The Three Faces of a Complex Number: Rectangular, Polar, and Exponential

Using the horizontal real part ($a$) and vertical imaginary part ($b$) as $(x, y)$ coordinates is not the only way to pinpoint a vector on a 2D plane.

Imagine giving someone directions from the origin $(0,0)$ to a point on a map. Instead of saying *"walk $a$ meters East and $b$ meters North"* (**Cartesian/Rectangular form**), you could say *"turn to angle $\theta$ and walk straight ahead for a distance of $r$ meters"*.

```text
                Imaginary Axis (Im)
                        ▲
                        │          ● z = a + ib
                        │         /│     = r(cos θ + i sin θ)
                        │        / │     = r e^(iθ)
                        │     r /  │
                        │      /   │ b = r sin θ
                        │     /    │
                        │    / θ   │
                        └───┴──────┴────► Real Axis (Re)
                        0    a = r cos θ

```

Let's connect these two descriptions using basic right-triangle trigonometry:

* Let **$r = \vert{}z\vert{} = \sqrt{a^2 + b^2}$** be the **magnitude** of the complex number (the hypotenuse).


* Let **$\theta$** be the **angle** (in radians) that the vector makes with the positive horizontal ($x$) axis.



From standard trigonometry ($\cos\theta = \frac{\text{adjacent}}{\text{hypotenuse}} = \frac{a}{r}$ and $\sin\theta = \frac{\text{opposite}}{\text{hypotenuse}} = \frac{b}{r}$), we can express the real and imaginary coordinates as:

$$a = r\cos\theta, \qquad b = r\sin\theta$$

And to find the angle $\theta$ from $a$ and $b$, we use the tangent ratio ($\tan\theta = \frac{\text{opposite}}{\text{adjacent}}$):

$$\tan\theta = \frac{b}{a} \quad \Longrightarrow \quad \theta = \arctan\left(\frac{b}{a}\right)$$

##### Form 2: Polar Form

Substituting $a = r\cos\theta$ and $b = r\sin\theta$ into $z = a + ib$ and factoring out $r$ gives us the **Polar Form** of a complex number:

$$z = r(\cos\theta + i\sin\theta)$$

##### Form 3: Exponential Form (Euler's Formula)

Here comes one of the most celebrated equations in all of mathematics—**Euler's Formula**—which proves that the trigonometric wave combination $\cos\theta + i\sin\theta$ is mathematically identical to raising Euler's constant $e$ to the imaginary power $i\theta$:

$$e^{i\theta} = \cos\theta + i\sin\theta$$

Replacing $(\cos\theta + i\sin\theta)$ with $e^{i\theta}$ gives us the **Exponential Form** of a complex number:

$$z = r e^{i\theta}$$

Let's compare all three forms side by side:

| Representation | Formula | Parameters | Best Used For... |
| --- | --- | --- | --- |
| **1. Cartesian (Rectangular) Form** | $z = a + ib$

 | Real part $a$, Imaginary part $b$

 | Adding and subtracting complex numbers

 |
| **2. Polar Form** | $z = r(\cos\theta + i\sin\theta)$

 | Magnitude $r$, Angle $\theta$

 | Bridging geometry and trigonometry

 |
| **3. Exponential Form** | $z = r e^{i\theta}$

 | Magnitude $r$, Phase angle $\theta$

 | **Quantum Computing!** Rotating states and multiplying amplitudes

 |

> **Worked Notebook Exercise:** *Convert the complex number $z = 1 + i$ into Polar Form and then into Exponential Form*.
> 
> 
> 1. **Find the magnitude $r$:** Here $a = 1$ and $b = 1$. Using Pythagoras's theorem:
> 
> 
> 
> $$r = \sqrt{1^2 + 1^2} = \sqrt{2}$$
> 
> 
> 2. **Find the angle $\theta$:** Using the inverse tangent:
> 
> 
> 
> $$\theta = \arctan\left(\frac{1}{1}\right) = \arctan(1) = \frac{\pi}{4}\text{ radians } (45^\circ)$$
> 
> 
> 3. **Write in Polar Form:**
> 
> 
> $$1 + i = \sqrt{2}\left(\cos\frac{\pi}{4} + i\sin\frac{\pi}{4}\right)$$
> 
> 
> 4. **Write in Exponential Form:**
> 
> 
> $$1 + i = \sqrt{2}\,e^{i\pi/4}$$
> 
> 
> 
> 

---

#### 6. Why Quantum Computing Exclusively Uses Exponential Form ($r e^{i\theta}$)

In our course material, a key principle is emphasized: **in quantum computing, we almost exclusively use complex numbers in exponential form ($re^{i\theta}$)**.

Why? Because when you write a complex number as $re^{i\theta}$, **changing $\theta$ rotates the vector around a circle**.

Specifically, consider what happens when we take two complex numbers in exponential form whose magnitude is $1$ ($r = 1$, meaning they lie on the **unit circle**), say $z_1 = e^{i\theta_1}$ and $z_2 = e^{i\theta_2}$, and multiply them together. By standard exponent rules ($e^x \cdot e^y = e^{x+y}$), their angles simply **add together**:

$$e^{i\theta_1} \cdot e^{i\theta_2} = e^{i(\theta_1 + \theta_2)}$$

Graphically, multiplying a state by $e^{i\phi}$ is nothing more than **grabbing the vector on the unit circle and rotating it counter-clockwise by $\phi$ radians**!

##### The Four Landmark Rotations on the Complex Unit Circle

Let's evaluate Euler's formula $e^{i\theta} = \cos\theta + i\sin\theta$ at the four main compass points around the unit circle ($r = 1$):

```text
                         Im (+i)
                      e^(iπ/2) = i
                           ▲
                           │     ╭─── Multiplying by e^(iφ)
                           │    /     rotates by angle φ!
      e^(iπ) = -1  ◄───────┼───────►  e^(i0) = +1   Re
      (180° / π rad)       │          (0° / 0 rad)
                           │
                           ▼
                     e^(i3π/2) = -i
                       (or e^(-iπ/2))

```

1. **Angle $\theta = 0$ radians ($0^\circ$):**

$$e^{i\cdot 0} = \cos(0) + i\sin(0) = 1 + 0i = +1$$



No rotation at all.


2. **Angle $\theta = \frac{\pi}{2}$ radians ($90^\circ$):**

$$e^{i\pi/2} = \cos\left(\frac{\pi}{2}\right) + i\sin\left(\frac{\pi}{2}\right) = 0 + i(1) = i$$



Notice that! The imaginary unit $i$ is simply a unit complex number at an angle of $\frac{\pi}{2}$ radians ($90^\circ$).


3. **Angle $\theta = \pi$ radians ($180^\circ$):**

$$e^{i\pi} = \cos(\pi) + i\sin(\pi) = -1 + 0i = -1$$



(This is the famous **Euler's Identity**: $e^{i\pi} = -1$). In quantum computing, whenever you see a minus sign ($-1$) in front of a qubit state, your brain should immediately read it as $e^{i\pi}$: **a $\pi$-radian ($180^\circ$) phase rotation around the $Z$-axis**!


4. **Angle $\theta = \frac{3\pi}{2}$ (or $-\frac{\pi}{2}$) radians ($270^\circ$ / $-90^\circ$):**

$$e^{-i\pi/2} = \cos\left(-\frac{\pi}{2}\right) + i\sin\left(-\frac{\pi}{2}\right) = 0 - i(1) = -i$$



---

#### 7. Sneak Peek: How $e^{i\phi}$ Powers Qubit Phase on the Bloch Sphere

Let's connect what we just learned directly to a qubit so you can see where this is going in Blogs 4 and 5.

When we write a general single-qubit state in Dirac notation with a **relative phase** of $\phi$ radians, it looks like this:

$$\vert{}\psi\rangle = \alpha\vert{}0\rangle + e^{i\phi}\beta\vert{}1\rangle$$

Look at the amplitude of the $\vert{}1\rangle$ state: it is multiplied by the unit-magnitude exponential complex number $e^{i\phi}$. Because of the math we just proved above, two things happen simultaneously:

1. **Geometrically (Spinning Around the $Z$-Axis):** Changing the value of $\phi$ smoothly rotates the qubit state vector around the vertical $Z$-axis of the Bloch sphere by $\phi$ radians:


* If we start with $\frac{1}{\sqrt{2}}\vert{}0\rangle + \frac{1}{\sqrt{2}}\vert{}1\rangle$ ($\phi = 0$) and multiply the $\vert{}1\rangle$ state by $-1 = e^{i\pi}$, we get $\frac{1}{\sqrt{2}}\vert{}0\rangle - \frac{1}{\sqrt{2}}\vert{}1\rangle$—rotating the qubit by **$\pi$ radians ($180^\circ$)** around the $Z$-axis.


* If instead we multiply the $\vert{}1\rangle$ state by $i = e^{i\pi/2}$, we get $\frac{1}{\sqrt{2}}\vert{}0\rangle + \frac{i}{\sqrt{2}}\vert{}1\rangle$—rotating the qubit by **$\frac{\pi}{2}$ radians ($90^\circ$)** around the $Z$-axis.


* Later, when we meet the **$S$ gate** and **$T$ gate**, all they do is multiply the $\vert{}1\rangle$ state by $e^{i\pi/2}$ and $e^{i\pi/4}$, acting as precision phase dials!




2. **Probabilistically (Phase is a "Hidden" Wave Direction):** What is the probability of measuring $\vert{}1\rangle$ for the state $\alpha\vert{}0\rangle + e^{i\phi}\beta\vert{}1\rangle$? As we previewed earlier, the probability is the **magnitude squared** of the $\vert{}1\rangle$ amplitude:



$$P(1) = \left\vert{}e^{i\phi}\beta\right\vert{}^2 = \left\vert{}e^{i\phi}\right\vert{}^2 \cdot \vert{}\beta\vert{}^2$$



What is the magnitude $\left\vert{}e^{i\phi}\right\vert{}$? In exponential form $re^{i\phi}$, the coefficient in front of $e^{i\phi}$ is $r = 1$, which states its magnitude! Since $1^2 = 1$, the $e^{i\phi}$ factor disappears completely during direct measurement:



$$P(1) = 1^2 \cdot \vert{}\beta\vert{}^2 = \vert{}\beta\vert{}^2$$



No matter how far we spin $\phi$ around the circle, the distance to the $\vert{}0\rangle$ and $\vert{}1\rangle$ poles stays unchanged—yet that hidden phase angle $\phi$ determines whether two quantum waves will constructively reinforce or destructively cancel when they pass through a Hadamard gate!



---

#### 8. Hands-On Python Lab: Verifying Complex Math & Phase Rotations

You don't have to calculate complex conjugates or polar conversions only by hand—Python has native support for complex numbers (using `1j` for $i = \sqrt{-1}$), and `numpy` makes converting between Cartesian, Polar, and Exponential forms effortless.

Run this script on your laptop to verify every exercise we solved in this blog:

```python
import numpy as np

# 1. Complex Conjugate & "Always Real" Property: (2 + 3i)(2 - 3i)
z1 = 2 + 3j
z1_conj = np.conj(z1)
product = z1 * z1_conj
print(f"1. z = {z1}, z* = {z1_conj}")
print(f"   z * z* = {product} (Imaginary part is {product.imag} -> Pure Real!)\n")

# 2. Magnitude via Pythagoras: |4 - 3i|
z2 = 4 - 3j
magnitude_z2 = np.abs(z2)
print(f"2. Magnitude of {z2}: |z| = {magnitude_z2}\n")

# 3. Cartesian (1 + i) -> Polar & Exponential Form (r * e^(i*theta))
z3 = 1 + 1j
r = np.abs(z3)
theta = np.angle(z3)  # returns angle in radians
print(f"3. Converting z = {z3}:")
print(f"   Magnitude r = {r:.4f} (sqrt(2) = {np.sqrt(2):.4f})")
print(f"   Angle theta = {theta:.4f} rad (pi/4 = {np.pi/4:.4f} rad = {np.degrees(theta):.1f} deg)")
print(f"   Reconstructed via r * e^(i*theta): {r * np.exp(1j * theta):.4f}\n")

# 4. Unit Circle Phase Rotations in Quantum Computing
phi_pi = np.exp(1j * np.pi)         # e^(i*pi) = -1 (Z gate phase)
phi_half_pi = np.exp(1j * np.pi/2)  # e^(i*pi/2) = i (S gate phase)
phi_quarter_pi = np.exp(1j * np.pi/4) # e^(i*pi/4) (T gate phase)

print("4. Landmark Quantum Phase Factors:")
print(f"   e^(i*pi)   = {np.round(phi_pi, 4)}")
print(f"   e^(i*pi/2) = {np.round(phi_half_pi, 4)}")
print(f"   e^(i*pi/4) = {np.round(phi_quarter_pi, 4)}")
print(f"   Magnitude of e^(i*pi/4) squared: {np.abs(phi_quarter_pi)**2:.4f} (Preserves probability!)")

```

**Output:**

```text
1. z = (2+3j), z* = (2-3j)
   z * z* = (13+0j) (Imaginary part is 0.0 -> Pure Real!)

2. Magnitude of (4-3j): |z| = 5.0

3. Converting z = (1+1j):
   Magnitude r = 1.4142 (sqrt(2) = 1.4142)
   Angle theta = 0.7854 rad (pi/4 = 0.7854 rad = 45.0 deg)
   Reconstructed via r * e^(i*theta): 1.0000+1.0000j

4. Landmark Quantum Phase Factors:
   e^(i*pi)   = (-1+0j)
   e^(i*pi/2) = 1j
   e^(i*pi/4) = (0.7071+0.7071j)
   Magnitude of e^(i*pi/4) squared: 1.0000 (Preserves probability!)

```

---

#### Up Next in the Series

Now that we can represent any single quantum amplitude as a rotating complex number $re^{i\theta}$, how do we package multiple amplitudes into a quantum state, and how do quantum logic gates transform those states without ever losing or creating extra probability? That brings us to the second pillar of our mathematical foundation: **Linear Algebra**.

---

**Would you like to explore any concept or exercise from Blog 2 in more detail, or are you ready for me to generate Blog 3: *Matrices as Geometry Engines — Linear Algebra, Unitary Rotations, and Eigenstates*?**
