
Lesson: The Geometry and Conventions of Qubit States


Introduction

In quantum mechanics, we describe the state of a single qubit using mathematical shorthand (Dirac notation), matrices, and 3D geometry (the Bloch Sphere). This lesson explores how these representations connect, why we use certain conventions, and how the imaginary unit (\(i\)) gets mapped to a physical spatial direction.

1. The Starting Point: Vector Spaces and Constraints

A general qubit state is written as a linear combination of two basis states, \(\vert{}0\rangle\) and \(\vert{}1\rangle\):
\(|{}\psi \rangle =c_{0}|{}0\rangle +c_{1}|{}1\rangle \)
In matrix form, this is represented as a two-element column vector:
\(|{}\psi \rangle =\left(\begin{matrix}c_{0}\\ c_{1}\end{matrix}\right)=\left(\begin{matrix}x_{0}+iy_{0}\\ x_{1}+iy_{1}\end{matrix}\right)\)
At first glance, this state possesses four real degrees of freedom (\(x_0, y_0, x_1, y_1\)). However, two fundamental constraints reduce these four parameters down to a 3D sphere:

Constraint A: Total Probability Must Equal 1

Because the qubit must exist in some state when measured, the total probability must equal 1:
\(|{}c_{0}|{}^{2}+|{}c_{1}|{}^{2}=1\)
This drops the system from four dimensions to three.

Constraint B: Global Phase Freedom

Multiplying an entire quantum state by a shared phase factor (\(e^{i\theta }\)) shifts the mathematical values but leaves all physical, measurable properties completely unchanged.
By universal convention, we use this freedom to eliminate the imaginary part of the first coefficient (\(c_{0}\)), forcing it to be a purely real, positive number. This simplifies the state to:
\(|{}\psi \rangle =\cos \left(\frac{\theta }{2}\right)|{}0\rangle +e^{i\phi }\sin \left(\frac{\theta }{2}\right)|{}1\rangle \)
Key Takeaway: The \(\vert{}1\rangle\) state component is not inherently the imaginary part simply because it is written second. Rather, because we use global phase freedom to make the \(\vert{}0\rangle\) coefficient real, all complex phase behavior (\(e^{i\phi }\)) is mathematically forced into the coefficient of \(\vert{}1\rangle\).

2. Navigating the Sphere: \(\theta \) vs. \(\phi \)

A qubit uses two distinct angles to describe two separate physical properties:
• \(\theta \) (Polar angle): Measures the vertical tilt down from the \(+Z\)-axis. It purely controls the probability balance of measuring a \(0\) versus a \(1\).
• \(\phi \) (Azimuthal angle): Measures the circular rotation around the vertical axis. It purely controls the relative phase.

Mapping the Planes

These angles define trajectories across the standard Cartesian planes:

Trajectory / Constraint	Plane	Physical Meaning
Changing \(\theta \) (when \(\phi = 0\))	\(XZ\) Plane	Modulates real-valued superpositions (e.g., \(\vert{}0\rangle \rightarrow \vert{}+\rangle \rightarrow \vert{}1\rangle\)).
Changing \(\theta \) (when \(\phi = \frac{\pi}{2}\))	\(YZ\) Plane	Modulates imaginary-valued superpositions (e.g., \(\vert{}0\rangle \rightarrow \vert{}+i\rangle \rightarrow \vert{}1\rangle\)).
Changing \(\phi \) (when \(\theta = \frac{\pi}{2}\))	\(XY\) Plane	Traverses the equator; changes relative phase while keeping probabilities strictly 50/50.


Why We Isolate Phase to the \(XY\) Plane

To isolate and study phase without altering measurement probabilities, we must hold \(\theta \) constant at \(\frac{\pi }{2}\) (a perfect 50/50 split). On a sphere, this locks the state vector to the equator, which sits entirely on the \(XY\) plane.
When \(\theta = \frac{\pi}{2}\), the spatial coordinates simplify to:
\(x=\cos \phi ,\quad y=\sin \phi ,\quad z=0\)
Using Euler's formula (\(e^{i\phi} = \cos\phi + i\sin\phi\)), the phase factor maps directly to this flat circle:
• The Real component (\(\cos\phi\)) tracks along the spatial \(X\)-axis.
• The Imaginary component (\(\sin\phi\)) tracks along the spatial \(Y\)-axis.

3. Physical Justification: The Role of Measurement

The connection between the "second row of a vector" and a "spatial direction" is a standardized mathematical framework, but it is justified by the physics of quantum operators.
In quantum mechanics, a physical measurement along a real-world spatial axis corresponds to a matrix called a Pauli operator. The coordinates \((x, y, z)\) on the Bloch sphere are calculated using expectation values:
\(x=\langle \psi |{}\sigma _{x}|{}\psi \rangle ,\quad y=\langle \psi |{}\sigma _{y}|{}\psi \rangle ,\quad z=\langle \psi |{}\sigma _{z}|{}\psi \rangle \)
The Pauli-\(Y\) matrix, which dictates measurements along the lab's physical \(Y\)-axis, explicitly requires the imaginary unit \(i\):
\(\sigma _{y}=\left(\begin{matrix}0&-i\\ i&0\end{matrix}\right)\)

Mathematical Proof

If we insert a state with an imaginary relative phase factor (\(e^{i\phi }\)) on the equator into the expectation value formula for the \(Y\)-axis:
\(|{}\psi \rangle =\left(\begin{matrix}\frac{1}{\sqrt{2}}\\ \frac{1}{\sqrt{2}}e^{i\phi }\end{matrix}\right)\)
\(y=\left(\begin{matrix}\frac{1}{\sqrt{2}}&\frac{1}{\sqrt{2}}e^{-i\phi }\end{matrix}\right)\left(\begin{matrix}0&-i\\ i&0\end{matrix}\right)\left(\begin{matrix}\frac{1}{\sqrt{2}}\\ \frac{1}{\sqrt{2}}e^{i\phi }\end{matrix}\right)\)
Multiplying the matrices yields:
\(y=\frac{1}{2}\left(-ie^{i\phi }+ie^{-i\phi }\right)\)
Substituting Euler's identity (\(e^{i\phi} = \cos\phi + i\sin\phi\) and \(e^{-i\phi} = \cos\phi - i\sin\phi\)):
\(y=\frac{1}{2}\Big(-i(\cos \phi +i\sin \phi )+i(\cos \phi -i\sin \phi )\Big)\)
\(y=\frac{1}{2}\Big(-i\cos \phi +\sin \phi +i\cos \phi +\sin \phi \Big)\)
\(y=\sin (\phi )\)

Summary

1. The \(\vert{}1\rangle\) state vector itself points to the South Pole (negative \(Z\)-axis), not the \(Y\)-axis.
2. The column vector structure is an index layout where all complex numbers are grouped into the bottom row by phase convention.
3. Multiplying the \(\vert{}1\rangle\) component by an imaginary phase (\(i\)) physically rotates the combined state vector onto the geometric \(Y\)-axis.
4. This convention functions perfectly because the underlying mathematical framework was precisely engineered to preserve our three-dimensional spatial intuition.
If you want to continue exploring, we can look at how quantum logic gates like the Phase (S) gate or Hadamard (H) gate physically manipulate these vectors across these planes.
Ready to create a study guide?
Use Canvas to save, edit, and share your guide
Get started

