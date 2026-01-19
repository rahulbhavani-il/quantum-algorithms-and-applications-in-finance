# Quantum Fourier Transform (QFT)

## 🎯 Learning Objectives

By the end of this section, you will be able to:

1. ✅ Explain the Discrete Fourier Transform (DFT) and its significance
2. ✅ Derive the quantum circuit for QFT from the DFT formula
3. ✅ Understand the role of controlled phase rotations
4. ✅ Implement QFT and inverse QFT in Qiskit
5. ✅ Recognize QFT as the key subroutine in Shor's algorithm

---

## 📚 Prerequisites

| Concept | Why Needed | Quick Review |
|---------|-----------|--------------|
| Complex exponentials | Phase rotations | $e^{i\theta} = \cos\theta + i\sin\theta$ |
| Tensor products | Multi-qubit states | $\|ab\rangle = \|a\rangle \otimes \|b\rangle$ |
| Controlled gates | Circuit building blocks | $CU\|1\rangle\|x\rangle = \|1\rangle U\|x\rangle$ |
| Hadamard gates | QFT is generalized Hadamard | $H = \frac{1}{\sqrt{2}}\begin{pmatrix} 1 & 1 \\ 1 & -1 \end{pmatrix}$ |

---

## 1️⃣ WHY: The Motivation

### The Power of Fourier Analysis

The Fourier Transform is one of the most important mathematical tools ever discovered. It reveals hidden periodic structure in data—whether it's:
- Audio signals (revealing musical notes)
- Images (enabling JPEG compression)
- Time series (detecting patterns)

### From Classical to Quantum

| Classical FFT | Quantum QFT |
|---------------|-------------|
| $O(N \log N)$ operations | $O((\log N)^2)$ operations |
| $N$ classical bits | $\log_2 N$ qubits |
| Direct access to output | Encoded in amplitudes |

**Exponential speedup in circuit depth!**

But there's a catch: The QFT output is encoded in quantum amplitudes, which we can't directly read. However, this is exactly what algorithms like Shor's and phase estimation need!

### The Key Insight

QFT transforms between two bases:
- **Computational basis**: States point to North/South poles
- **Fourier basis**: States point along the equator at different angles

This transformation enables **period finding**—the heart of Shor's factoring algorithm.

---

## 2️⃣ WHAT: The Core Definitions

### Discrete Fourier Transform (DFT)

Given a vector $\vec{x} = (x_0, x_1, \ldots, x_{N-1})$, the DFT produces $\vec{\tilde{x}} = (\tilde{x}_0, \tilde{x}_1, \ldots, \tilde{x}_{N-1})$ where:

$$\tilde{x}_k = \frac{1}{\sqrt{N}} \sum_{j=0}^{N-1} x_j \cdot \omega_N^{jk}$$

where $\omega_N = e^{2\pi i / N}$ is the **primitive N-th root of unity**.

### The DFT Matrix

$$F_N = \frac{1}{\sqrt{N}} \begin{pmatrix}
1 & 1 & 1 & \cdots & 1 \\
1 & \omega & \omega^2 & \cdots & \omega^{N-1} \\
1 & \omega^2 & \omega^4 & \cdots & \omega^{2(N-1)} \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
1 & \omega^{N-1} & \omega^{2(N-1)} & \cdots & \omega^{(N-1)^2}
\end{pmatrix}$$

### Quantum Fourier Transform

The QFT maps computational basis states to Fourier basis states:

$$\text{QFT}|j\rangle = \frac{1}{\sqrt{N}} \sum_{k=0}^{N-1} \omega_N^{jk} |k\rangle$$

For an n-qubit system ($N = 2^n$):

$$\text{QFT}|j\rangle = \frac{1}{2^{n/2}} \sum_{k=0}^{2^n-1} e^{2\pi i jk/2^n} |k\rangle$$

### The Product Formula (Key Insight!)

The QFT can be written as a **tensor product**:

$$\text{QFT}|j_1 j_2 \cdots j_n\rangle = \frac{1}{2^{n/2}} \bigotimes_{l=1}^{n} \left(|0\rangle + e^{2\pi i \cdot 0.j_{n-l+1}\cdots j_n}|1\rangle\right)$$

where $0.j_l j_{l+1} \cdots j_n = \sum_{m=l}^{n} j_m 2^{-(m-l+1)}$ is binary fraction notation.

**This product form shows that each qubit transforms independently (with phase kickback from others)!**

---

## 3️⃣ ANALOGY: The Roots of Unity Clock

Imagine a clock with $N$ equally spaced marks around its face. The primitive root of unity $\omega_N = e^{2\pi i/N}$ represents a rotation by $\frac{1}{N}$ of a full circle.

**Example: N = 8 (3 qubits)**

```
        |1⟩ = ω⁰
        
    ω⁷     ω¹
      \   /
   ω⁶  ●  ω²      Unit circle in complex plane
      /   \       Each point is a power of ω₈
    ω⁵     ω³
    
        ω⁴
```

The QFT spreads each basis state around this clock:
- $|0\rangle \rightarrow$ uniform superposition (all clock positions)
- $|1\rangle \rightarrow$ phases rotate around clock
- Higher $|j\rangle \rightarrow$ faster rotations around clock

### Bloch Sphere Interpretation

| Computational Basis | Fourier Basis |
|---------------------|---------------|
| Points at North/South poles | Points on equator |
| Discrete jumps (0 or 1) | Continuous rotations |
| $\|0\rangle, \|1\rangle$ | $\|+\rangle, \|-\rangle$ (for 1 qubit) |

---

## 4️⃣ FORMAL: Mathematical Foundation

### Building the QFT Circuit

Let's derive the circuit from the product formula.

For input $|j\rangle = |j_1 j_2 \cdots j_n\rangle$ (where $j_1$ is MSB):

$$\text{QFT}|j\rangle = \frac{1}{2^{n/2}} \left(|0\rangle + e^{2\pi i \cdot 0.j_n}|1\rangle\right) \otimes \left(|0\rangle + e^{2\pi i \cdot 0.j_{n-1}j_n}|1\rangle\right) \otimes \cdots$$

### The Controlled Rotation Gates

Define the rotation gate $R_k$:

$$R_k = \begin{pmatrix} 1 & 0 \\ 0 & e^{2\pi i/2^k} \end{pmatrix}$$

This adds a phase $\frac{2\pi}{2^k}$ to the $|1\rangle$ state.

### Circuit Construction

**For qubit 1 (MSB):**
1. Apply Hadamard: $|j_1\rangle \rightarrow |0\rangle + e^{i\pi j_1}|1\rangle = |0\rangle + e^{2\pi i \cdot 0.j_1}|1\rangle$
2. Apply controlled-$R_2$ from qubit 2: adds phase $e^{2\pi i \cdot 0.0j_2}$ to $|1\rangle$
3. Apply controlled-$R_3$ from qubit 3: adds phase $e^{2\pi i \cdot 0.00j_3}$ to $|1\rangle$
4. Continue for all qubits...

**Result for qubit 1:** $|0\rangle + e^{2\pi i \cdot 0.j_1 j_2 \cdots j_n}|1\rangle$

**Repeat for each qubit**, then **swap the order** (QFT reverses bit order).

### Complexity Analysis

| Component | Count |
|-----------|-------|
| Hadamard gates | $n$ |
| Controlled rotations | $\frac{n(n-1)}{2}$ |
| SWAP gates | $\lfloor n/2 \rfloor$ |
| **Total gates** | $O(n^2)$ |
| **Circuit depth** | $O(n^2)$ |

For $N = 2^n$: Classical FFT is $O(N \log N)$, QFT is $O((\log N)^2)$.

---

## 5️⃣ VISUAL: Circuit Diagram

### QFT Circuit for 3 Qubits

```
         ┌───┐                                           
q₀: ─────┤ H ├──■──────■──────────────────────────────×──
         └───┘  │      │                              │  
         ┌──────┴──────┼───────┐┌───┐                 │  
q₁: ─────┤     R₂      │       ├┤ H ├──■──────────────┼──
         └─────────────┼───────┘└───┘  │              │  
         ┌─────────────┴───────┐┌──────┴──────┐┌───┐  │  
q₂: ─────┤         R₃         ├┤     R₂      ├┤ H ├──×──
         └─────────────────────┘└─────────────┘└───┘     
```

Where:
- $R_k = \begin{pmatrix} 1 & 0 \\ 0 & e^{2\pi i/2^k} \end{pmatrix}$
- $R_2 = e^{i\pi/2}$ phase (T gate)
- $R_3 = e^{i\pi/4}$ phase

### State Evolution Example

Input: $|101\rangle$ (5 in decimal)

1. **Initial**: $|101\rangle$
2. **After H on q₀**: Superposition on first qubit
3. **After controlled rotations**: Phase information encoded
4. **After H on q₁, q₂**: Full Fourier transform
5. **After SWAP**: Bit order corrected

Output: $\frac{1}{\sqrt{8}}\sum_{k=0}^{7} e^{2\pi i \cdot 5k/8}|k\rangle$

---

## 6️⃣ CODE: Qiskit 2.x Implementation

```python
from qiskit import QuantumCircuit, QuantumRegister
from qiskit.circuit.library import QFT
import numpy as np

def qft_rotations(circuit: QuantumCircuit, n: int):
    """
    Apply QFT rotations to first n qubits of circuit.
    Does NOT include SWAP gates.
    """
    if n == 0:
        return circuit
    
    n -= 1  # Qiskit uses 0-indexing
    
    # Apply Hadamard to the most significant qubit
    circuit.h(n)
    
    # Apply controlled rotations
    for qubit in range(n):
        # R_k gate where k = n - qubit + 1
        k = n - qubit + 1
        circuit.cp(2 * np.pi / (2**k), qubit, n)
    
    # Recursively apply to remaining qubits
    qft_rotations(circuit, n)


def swap_registers(circuit: QuantumCircuit, n: int):
    """
    Swap qubits to reverse bit order after QFT rotations.
    """
    for qubit in range(n // 2):
        circuit.swap(qubit, n - qubit - 1)


def qft(circuit: QuantumCircuit, n: int):
    """
    Apply n-qubit QFT to the first n qubits of circuit.
    
    Args:
        circuit: Quantum circuit
        n: Number of qubits for QFT
    
    Returns:
        Circuit with QFT applied
    """
    qft_rotations(circuit, n)
    swap_registers(circuit, n)
    return circuit


def inverse_qft(circuit: QuantumCircuit, n: int):
    """
    Apply inverse QFT.
    
    The inverse QFT is simply the QFT with all gates reversed.
    """
    # Create QFT circuit
    qft_circuit = QuantumCircuit(n)
    qft(qft_circuit, n)
    
    # Get inverse
    inverse_qft_circuit = qft_circuit.inverse()
    
    # Append to main circuit
    circuit.compose(inverse_qft_circuit, inplace=True)
    return circuit


# Example: 3-qubit QFT
n = 3
qc = QuantumCircuit(n)

# Prepare input state |101⟩
qc.x(0)  # q0 = 1
qc.x(2)  # q2 = 1

qc.barrier(label="input |101⟩")

# Apply QFT
qft(qc, n)

print("3-Qubit QFT Circuit:")
print(qc.draw())

# Using Qiskit's built-in QFT
from qiskit.circuit.library import QFT as QiskitQFT

qc_builtin = QuantumCircuit(3)
qc_builtin.x(0)
qc_builtin.x(2)
qc_builtin.barrier()
qc_builtin.append(QiskitQFT(3), range(3))

print("\nUsing Qiskit's QFT library:")
print(qc_builtin.decompose().draw())
```

---

## 7️⃣ TRAPS: Common Mistakes

### ⚠️ Trap 1: Forgetting the SWAP Gates

**Wrong:**
```python
# Only apply rotations, no SWAP
def wrong_qft(circuit, n):
    for j in range(n):
        circuit.h(j)
        for k in range(j+1, n):
            circuit.cp(2*np.pi/2**(k-j+1), k, j)
```

**Right:**
```python
def correct_qft(circuit, n):
    qft_rotations(circuit, n)
    swap_registers(circuit, n)  # Don't forget!
```

**Why:** The QFT naturally outputs qubits in reversed order. Without SWAP, your $|j_1 j_2 \cdots j_n\rangle$ output is actually $|j_n \cdots j_2 j_1\rangle$.

### ⚠️ Trap 2: Incorrect Rotation Angles

**Wrong:**
```python
# Using wrong angle
circuit.cp(np.pi / 2**k, control, target)  # Missing factor of 2
```

**Right:**
```python
circuit.cp(2 * np.pi / 2**k, control, target)  # Correct!
```

The $R_k$ gate has angle $\frac{2\pi}{2^k}$, not $\frac{\pi}{2^k}$.

### ⚠️ Trap 3: Thinking QFT Enables Exponential Parallelism

**Misconception:** "QFT computes the Fourier transform of $2^n$ values in polynomial time"

**Reality:** QFT transforms quantum amplitudes, which we cannot directly read. It's useful as a **subroutine** in algorithms like Shor's and phase estimation, but you cannot directly extract all Fourier coefficients.

### ⚠️ Trap 4: Control Qubit Direction

In QFT, the controlled rotation goes from **lower-indexed** qubits to the **current** qubit:

```
Correct:                Wrong:
q₀ ──■──                q₀ ──R₂──
     │                       │
q₁ ──R₂──               q₁ ──■──
```

### ⚠️ Trap 5: Confusing QFT with Classical FFT

| Aspect | QFT | Classical FFT |
|--------|-----|---------------|
| Input | Quantum amplitudes | Data array |
| Output | Quantum amplitudes | Transformed array |
| Readout | Measurement (probabilistic) | Direct access |
| Use case | Subroutine in quantum algorithms | Standalone transform |

---

## 🔗 Connections

### Backward Links
- **Deutsch-Jozsa:** Hadamard as 1-qubit QFT
- **Simon's:** Inspired the periodic structure detection in Shor's
- **Single-Qubit Gates:** R gates and phase rotations

### Forward Links
- **Phase Estimation:** Uses QFT to extract eigenvalue phases
- **Shor's Algorithm:** QFT enables efficient period finding
- **VQE:** Some variational ansätze use QFT-like structures

### Cross-Cutting Themes
- **Basis change:** QFT transforms computational ↔ Fourier bases
- **Exponential compression:** $2^n$ amplitudes, $O(n^2)$ gates
- **Hybrid algorithms:** QFT output read via measurement

---

## ✅ Quick Knowledge Check

**Q1:** Why does the QFT circuit have $O(n^2)$ gates for n qubits?

<details>
<summary>Click for answer</summary>

Each qubit requires:
- 1 Hadamard gate
- Controlled rotations from all subsequent qubits

This gives: $n + (n-1) + (n-2) + \cdots + 1 = \frac{n(n+1)}{2} = O(n^2)$ gates.

Plus $n/2$ SWAP gates, still $O(n^2)$ total.
</details>

**Q2:** What is the 1-qubit QFT?

<details>
<summary>Click for answer</summary>

The 1-qubit QFT is exactly the **Hadamard gate**!

$H|0\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$

$H|1\rangle = \frac{1}{\sqrt{2}}(|0\rangle - |1\rangle) = \frac{1}{\sqrt{2}}(|0\rangle + e^{i\pi}|1\rangle)$

This matches the QFT formula with $N=2$: $\text{QFT}|j\rangle = \frac{1}{\sqrt{2}}\sum_{k=0}^{1} e^{i\pi jk}|k\rangle$
</details>

**Q3:** Why can't we use QFT to compute Fourier transforms faster than classical FFT?

<details>
<summary>Click for answer</summary>

Two reasons:

1. **Input problem:** Loading classical data into quantum amplitudes requires $O(N)$ operations.

2. **Output problem:** The transformed amplitudes are in superposition. Reading all $N$ values requires exponentially many measurements.

QFT is powerful as a **subroutine** where we don't need to read all amplitudes—just use interference to extract specific information (like period in Shor's algorithm).
</details>

---

## 📝 Exercises

### Exercise 1: Build QFT Step by Step (Beginner)
Manually construct the 2-qubit QFT circuit using only H and CP (controlled-phase) gates. Verify by comparing to Qiskit's built-in QFT.

### Exercise 2: Verify QFT Matrix (Intermediate)
Compute the unitary matrix for 3-qubit QFT using Qiskit's `Operator` class. Verify it matches the DFT matrix $F_8$.

### Exercise 3: Input/Output Verification (Intermediate)
For 3-qubit QFT, verify that $\text{QFT}|5\rangle = \frac{1}{\sqrt{8}}\sum_{k=0}^{7} e^{2\pi i \cdot 5k/8}|k\rangle$ by computing amplitudes.

### Exercise 4: Inverse QFT (Intermediate)
Implement inverse QFT and verify that $\text{QFT}^{-1} \cdot \text{QFT} = I$ for n=4.

### Exercise 5: Approximate QFT (Advanced)
Implement "approximate QFT" that drops controlled rotations with small angles (< threshold). How does truncation affect fidelity?

---

## 📖 Summary

| Concept | Key Point |
|---------|-----------|
| **Definition** | Transforms computational basis to Fourier basis |
| **Formula** | $\text{QFT}\|j\rangle = \frac{1}{\sqrt{N}}\sum_k e^{2\pi ijk/N}\|k\rangle$ |
| **Circuit** | H gates + controlled phase rotations + SWAPs |
| **Complexity** | $O(n^2)$ gates for n qubits |
| **1-qubit case** | Hadamard gate |
| **Key application** | Subroutine in Shor's algorithm, phase estimation |
| **Limitation** | Cannot directly read all Fourier coefficients |

---

## 🎯 What's Next?

**Quantum Phase Estimation** - We'll use QFT as a subroutine to extract eigenvalue phases of unitary operators. This is the key component that powers Shor's factoring algorithm and many quantum chemistry algorithms.
