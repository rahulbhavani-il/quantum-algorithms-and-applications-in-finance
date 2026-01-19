# Quantum Phase Estimation (QPE)

## 🎯 Learning Objectives

By the end of this section, you will be able to:

1. ✅ Explain the eigenvalue problem for unitary operators
2. ✅ Understand why eigenvalues of unitaries are always phases
3. ✅ Construct the QPE circuit using controlled-U gates and inverse QFT
4. ✅ Analyze the precision-qubit tradeoff in QPE
5. ✅ Recognize QPE as the core subroutine in Shor's algorithm and HHL

---

## 📚 Prerequisites

| Concept | Why Needed | Quick Review |
|---------|-----------|--------------|
| Quantum Fourier Transform | QPE uses inverse QFT for readout | See previous section |
| Eigenvalues & Eigenvectors | Core mathematical framework | $U\|\psi\rangle = \lambda\|\psi\rangle$ |
| Phase Kickback | How phase information transfers | $CU\|1\rangle\|\psi\rangle = e^{i\phi}\|1\rangle\|\psi\rangle$ |
| Controlled Gates | Building block of QPE circuit | See earlier sections |

---

## 1️⃣ WHY: The Motivation

### The Problem: Extracting Eigenvalues

Many quantum algorithms need to extract information about the eigenvalues of a unitary operator. But there's a fundamental problem:

**Global phases are undetectable!**

If $U|\psi\rangle = e^{2\pi i\theta}|\psi\rangle$, directly measuring $|\psi\rangle$ tells us nothing about $\theta$.

### Why Eigenvalues Are Phases

**Theorem**: For any unitary operator $U$, all eigenvalues have the form $e^{2\pi i\theta}$ where $\theta \in [0, 1)$.

**Proof**:
1. Let $U|\psi\rangle = \lambda|\psi\rangle$
2. Since $U$ is unitary: $U^\dagger U = I$
3. Take the inner product: $\langle\psi|U^\dagger U|\psi\rangle = \langle\psi|\psi\rangle = 1$
4. This gives: $|\lambda|^2 \langle\psi|\psi\rangle = 1$
5. Therefore: $|\lambda| = 1$, meaning $\lambda = e^{2\pi i\theta}$

### The Key Insight: Convert Global to Relative Phase

QPE converts the undetectable **global phase** into a **relative phase** using:
1. **Ancilla qubits** - Create a reference state
2. **Controlled-U operations** - Kick the phase onto control qubits
3. **Inverse QFT** - Decode the phase into computational basis

### Applications

| Algorithm | How QPE is Used |
|-----------|----------------|
| **Shor's Algorithm** | Find period via eigenvalue of modular multiplication |
| **HHL Algorithm** | Invert eigenvalues for linear systems |
| **Quantum Chemistry** | Estimate ground state energy (VQE/QPE hybrid) |
| **Quantum Simulation** | Extract energy levels of Hamiltonians |

---

## 2️⃣ WHAT: The Core Problem

### Formal Problem Statement

**Given:**
- A unitary operator $U$ (as a quantum circuit)
- An eigenstate $|\psi\rangle$ of $U$

**Promise:**
$$U|\psi\rangle = e^{2\pi i\theta}|\psi\rangle$$

**Find:** The phase $\theta$ to $n$ bits of precision.

### The Output

After QPE, we obtain an $n$-bit approximation:
$$\tilde{\theta} = \frac{m}{2^n}$$

where $m$ is the integer measured from $n$ counting qubits.

### Precision vs Resources

| Counting Qubits (n) | Precision | Example for $\theta = 1/8$ |
|---------------------|-----------|---------------------------|
| 1 | 0 or 0.5 | Cannot distinguish |
| 2 | 0, 0.25, 0.5, 0.75 | Cannot distinguish |
| 3 | 0, 0.125, 0.25, ... | ✅ Exact: $1/8 = 1/2^3$ |
| 4 | 0, 0.0625, ... | ✅ Exact |

**Rule**: Need $n$ qubits to distinguish phases separated by $1/2^n$.

---

## 3️⃣ ANALOGY: The Tuning Fork Frequency Meter

Imagine you want to measure the exact frequency of a tuning fork, but you can only hear whether two sounds are "in sync" or not.

**Classical Approach:**
Strike the fork, compare with known reference frequencies one by one.

**Quantum Approach (QPE):**
1. Create a "reference oscillator" that vibrates at ALL frequencies simultaneously (superposition of counting qubits)
2. Let it interact with the tuning fork (controlled-U operations)
3. The interference pattern reveals the frequency (inverse QFT decodes)

**The Key Insight:**
- More reference oscillators (counting qubits) = finer frequency resolution
- The tuning fork "kicks" its frequency information onto the reference oscillators
- Fourier analysis extracts the frequency from the interference pattern

---

## 4️⃣ FORMAL: Mathematical Foundation

### The QPE Circuit

```
Counting       |0⟩ ─ H ─────────────────────────────────────── ╔═══╗
Qubits         |0⟩ ─ H ───────────────────────── • ─────────── ║QFT║
               |0⟩ ─ H ─────────── • ─────────────────────────  ║^-1║
                      │           │           │               ╚═══╝
                      │           │           │                 │
Eigenstate     |ψ⟩ ──────────── U^4 ──────── U^2 ─────────── U^1 ─────
```

### Step-by-Step Analysis

**Step 1: Initialize and Superpose**
$$|0\rangle^{\otimes n}|\psi\rangle \xrightarrow{H^{\otimes n}} \frac{1}{\sqrt{2^n}}\sum_{k=0}^{2^n-1}|k\rangle|\psi\rangle$$

**Step 2: Apply Controlled-U Powers**

Apply $\text{C-}U^{2^j}$ controlled by qubit $j$:
$$\frac{1}{\sqrt{2^n}}\sum_{k=0}^{2^n-1}|k\rangle U^k|\psi\rangle = \frac{1}{\sqrt{2^n}}\sum_{k=0}^{2^n-1}e^{2\pi i\theta k}|k\rangle|\psi\rangle$$

**Step 3: Recognize as QFT**

The state of counting qubits is exactly $\text{QFT}|2^n\theta\rangle$!

$$\frac{1}{\sqrt{2^n}}\sum_{k=0}^{2^n-1}e^{2\pi i\theta k}|k\rangle = \text{QFT}\left|2^n\theta\right\rangle$$

**Step 4: Apply Inverse QFT**
$$\text{QFT}^{-1}\left[\text{QFT}|2^n\theta\rangle\right] = |2^n\theta\rangle$$

**Step 5: Measure**

Measure counting qubits → get integer $m \approx 2^n\theta$ → compute $\tilde{\theta} = m/2^n$.

### The Phase Kickback Mechanism

When control qubit is $|1\rangle$ and target is eigenstate $|\psi\rangle$:

$$\text{C-}U|1\rangle|\psi\rangle = |1\rangle U|\psi\rangle = e^{2\pi i\theta}|1\rangle|\psi\rangle$$

The phase "kicks back" to the control qubit!

For superposition: 
$$\text{C-}U\frac{|0\rangle + |1\rangle}{\sqrt{2}}|\psi\rangle = \frac{|0\rangle + e^{2\pi i\theta}|1\rangle}{\sqrt{2}}|\psi\rangle$$

---

## 5️⃣ VISUAL: Circuit and State Evolution

### Complete QPE Circuit (3 counting qubits)

```
         ┌───┐                                          ┌───────┐
count_0: ┤ H ├──────────────────────────────────■───────┤       ├─ M
         ├───┤                            ┌─────┴─────┐ │       │
count_1: ┤ H ├────────────────■───────────┤           ├─┤ QFT^-1├─ M
         ├───┤          ┌─────┴─────┐     │           │ │       │
count_2: ┤ H ├────■─────┤           ├─────┤           ├─┤       ├─ M
         └───┘┌───┴───┐ │    U^2    │     │    U^4    │ └───────┘
    |ψ⟩: ─────┤   U   ├─┤           ├─────┤           ├───────────
              └───────┘ └───────────┘     └───────────┘
```

### State Evolution Diagram

```
Initial:     |000⟩|ψ⟩
                ↓ H⊗H⊗H
After H:     |+++⟩|ψ⟩ = (1/√8)∑_{k=0}^{7} |k⟩|ψ⟩
                ↓ Controlled-U powers
After CU:    (1/√8)∑_{k=0}^{7} e^{2πiθk}|k⟩|ψ⟩ = QFT|8θ⟩|ψ⟩
                ↓ QFT^{-1}
After QFT⁻¹: |8θ⟩|ψ⟩
                ↓ Measure
Output:      Integer m ≈ 8θ  →  θ̃ = m/8
```

### Example: T-gate Phase Estimation

The T gate: $T = \begin{pmatrix} 1 & 0 \\ 0 & e^{i\pi/4} \end{pmatrix}$

Eigenstate: $|1\rangle$ with eigenvalue $e^{i\pi/4} = e^{2\pi i \cdot (1/8)}$

So $\theta = 1/8 = 0.001_2$ (binary)

With 3 counting qubits: Measure $|001\rangle$ → $m = 1$ → $\theta = 1/8$ ✅

---

## 6️⃣ IMPLEMENTATION: Code Structure

```python
from qiskit import QuantumCircuit, QuantumRegister, ClassicalRegister
from qiskit.circuit.library import QFT
import numpy as np

def qpe_circuit(unitary_gate, num_counting_qubits, eigenstate_prep=None):
    """
    Construct Quantum Phase Estimation circuit.
    
    Args:
        unitary_gate: The unitary U to estimate eigenvalue of
        num_counting_qubits: Number of qubits for phase precision
        eigenstate_prep: Circuit to prepare eigenstate (optional)
    
    Returns:
        QuantumCircuit for QPE
    """
    n_count = num_counting_qubits
    n_target = unitary_gate.num_qubits
    
    # Registers
    count_reg = QuantumRegister(n_count, 'count')
    target_reg = QuantumRegister(n_target, 'target')
    classical_reg = ClassicalRegister(n_count, 'result')
    
    qc = QuantumCircuit(count_reg, target_reg, classical_reg)
    
    # Step 1: Prepare eigenstate (if provided)
    if eigenstate_prep:
        qc.compose(eigenstate_prep, target_reg, inplace=True)
    
    # Step 2: Hadamard on counting qubits
    qc.h(count_reg)
    
    qc.barrier(label='init')
    
    # Step 3: Controlled-U^(2^j) operations
    repetitions = 1
    for j in range(n_count):
        for _ in range(repetitions):
            # Control: count_reg[n_count - 1 - j], Target: target_reg
            controlled_u = unitary_gate.control(1)
            qc.compose(controlled_u, 
                      [count_reg[n_count - 1 - j]] + list(target_reg),
                      inplace=True)
        repetitions *= 2
    
    qc.barrier(label='CU')
    
    # Step 4: Inverse QFT on counting qubits
    qft_inverse = QFT(n_count, inverse=True)
    qc.compose(qft_inverse, count_reg, inplace=True)
    
    qc.barrier(label='QFT†')
    
    # Step 5: Measure counting qubits
    qc.measure(count_reg, classical_reg)
    
    return qc


# Example: QPE for T-gate
def t_gate_qpe_example():
    """Estimate phase of T-gate eigenvalue."""
    from qiskit.circuit.library import TGate
    
    # T-gate
    t_gate = TGate()
    
    # Eigenstate preparation: |1⟩
    eigenstate = QuantumCircuit(1)
    eigenstate.x(0)
    
    # Build QPE circuit with 3 counting qubits
    qpe = qpe_circuit(t_gate, num_counting_qubits=3, 
                      eigenstate_prep=eigenstate)
    
    return qpe
```

---

## 7️⃣ TRAP ALERT: Common Mistakes

### ⚠️ Trap 1: Wrong Order of Controlled-U Powers

❌ **Wrong**: Applying $U^1, U^2, U^4, \ldots$ from top to bottom
```python
# WRONG: Powers in wrong order
for j in range(n_count):
    power = 2**j
    apply_controlled_u(control=count_reg[j], power=power)  # Wrong mapping!
```

✅ **Correct**: Match power to qubit position for QFT
```python
# CORRECT: count_reg[n-1] controls U^1, count_reg[n-2] controls U^2, etc.
for j in range(n_count):
    power = 2**j
    apply_controlled_u(control=count_reg[n_count - 1 - j], power=power)
```

**Why it matters**: The QFT expects specific phase patterns on each qubit. Reversing gives wrong answer.

---

### ⚠️ Trap 2: Forgetting the Inverse QFT

❌ **Wrong**: Using forward QFT
```python
qc.compose(QFT(n_count), count_reg)  # Wrong direction!
```

✅ **Correct**: Use inverse QFT to decode
```python
qc.compose(QFT(n_count, inverse=True), count_reg)  # Correct!
```

**Why it matters**: After controlled-U, the state IS in QFT basis. We need inverse to get back to computational basis.

---

### ⚠️ Trap 3: Non-Eigenstate Input

❌ **Wrong**: Assuming any state works
```python
# If |ψ⟩ is NOT an eigenstate of U, results are unpredictable
qc.h(target_reg)  # Creates superposition, not eigenstate!
```

✅ **Correct**: Prepare an actual eigenstate
```python
# For T-gate, eigenstate is |1⟩
qc.x(target_reg)  # Prepares |1⟩, which IS an eigenstate of T
```

**What happens with non-eigenstates**: QPE returns a superposition of phases from all eigenstates in the decomposition of $|\psi\rangle$.

---

### ⚠️ Trap 4: Insufficient Precision

❌ **Wrong**: Using too few counting qubits
```python
# θ = 1/3 = 0.0101010101..._2 (infinite binary expansion)
qpe_circuit(U, num_counting_qubits=2)  # Only 2 bits of precision!
```

✅ **Correct**: Use enough qubits for desired precision
```python
# Need n qubits for precision 1/2^n
# For θ = 1/3, use at least 8 qubits for ~1% error
qpe_circuit(U, num_counting_qubits=8)
```

**Key insight**: If $2^n\theta$ is not an integer, you get a probability distribution peaked near the correct value.

---

### ⚠️ Trap 5: Interpreting Measurement Results

❌ **Wrong**: Reading measurement bits in wrong order
```python
result = "101"  # Qiskit returns LSB first!
theta_wrong = int(result, 2) / 8  # Would give 5/8, but it's actually 5/8 in reversed order!
```

✅ **Correct**: Account for bit ordering convention
```python
result = "101"  # In Qiskit: q[0]=1, q[1]=0, q[2]=1
# Reverse for integer interpretation
theta = int(result[::-1], 2) / 8  # Gives correct θ
```

---

## 🧠 Mnemonic

> **"CHIC"** - **C**ontrolled-U, **H**adamard, **I**nverse QFT, **C**ount the bits

Or: **"Phase Kicks Back, QFT Decodes"**

---

## ⚡ Quick Check

**Q1**: Why do we need the eigenstate $|\psi\rangle$ as input?

<details>
<summary>Answer</summary>

**A**: Phase kickback only works predictably for eigenstates. When $U|\psi\rangle = e^{2\pi i\theta}|\psi\rangle$, the controlled-U operation kicks exactly $\theta$ onto the control qubit. For non-eigenstates, you'd get a superposition of different phases.
</details>

**Q2**: If $\theta = 1/3$, what happens with 3 counting qubits?

<details>
<summary>Answer</summary>

**A**: Since $2^3 \times (1/3) = 8/3 \approx 2.67$ is not an integer, the measurement outcome is probabilistic. You'd most likely measure $|010\rangle$ (m=2, giving $\theta \approx 0.25$) or $|011\rangle$ (m=3, giving $\theta \approx 0.375$). More counting qubits improve precision.
</details>

**Q3**: What's the connection between QPE and Shor's algorithm?

<details>
<summary>Answer</summary>

**A**: Shor's algorithm uses QPE to find the period $r$ of the function $f(x) = a^x \mod N$. The unitary $U$ is modular multiplication by $a$, and QPE extracts eigenvalues that encode the period. Specifically, QPE finds $\theta = s/r$ for various $s$, from which $r$ can be computed.
</details>

---

## 📚 Summary

### Key Takeaways

1. **QPE estimates eigenvalue phases** of unitary operators with controllable precision
2. **Phase kickback** converts undetectable global phases to relative phases
3. **Inverse QFT decodes** the phase information into the computational basis
4. **$n$ counting qubits** give precision $1/2^n$ in the phase estimate
5. **Core subroutine** for Shor's algorithm, HHL, and quantum simulation

### What's Next

- **Shor's Algorithm** - Using QPE for factoring
- **HHL Algorithm** - Using QPE for linear systems
- **VQE** - Alternative to QPE for chemistry

---

## 📖 References

- Nielsen & Chuang, Section 5.2: "Phase Estimation"
- Kitaev, A. (1995): "Quantum measurements and the Abelian Stabilizer Problem"
- Cleve, R. et al. (1998): "Quantum algorithms revisited"

---
