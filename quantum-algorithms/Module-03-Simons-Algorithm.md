# Simon's Algorithm

## 🎯 Learning Objectives

By the end of this section, you will be able to:

1. ✅ Explain Simon's Problem and its significance as inspiration for Shor's algorithm
2. ✅ Distinguish between one-to-one and two-to-one functions with hidden period
3. ✅ Construct the Simon oracle for a given hidden string
4. ✅ Understand why classical algorithms require exponential queries
5. ✅ Execute Simon's algorithm and solve the linear system to find the hidden string

---

## 📚 Prerequisites

| Concept | Why Needed | Quick Review |
|---------|-----------|--------------|
| Hadamard gates | Creates superposition for parallel evaluation | $H\|0\rangle = \|+\rangle$ |
| XOR operation | Understanding the hidden period relation | $a \oplus b$ is bitwise XOR |
| Linear algebra | Solving system of equations modulo 2 | Gaussian elimination |
| Deutsch-Jozsa | Foundation for oracle-based algorithms | See previous section |
| Bernstein-Vazirani | Phase kickback and Hadamard decoding | See previous section |

---

## 1️⃣ WHY: The Motivation

### The Historical Significance

Simon's algorithm (1994) was **the inspiration for Shor's algorithm**. Daniel Simon showed the first example of an exponential quantum speedup over any classical algorithm (not just deterministic ones). This directly led Peter Shor to develop his factoring algorithm.

### The Problem: Finding Hidden Periodicity

Imagine you're a cryptanalyst trying to break a code. The encryption function has a hidden structure—a **secret key** $s$ that creates a two-to-one mapping pattern. Given only black-box access to the function, can you find $s$?

**Why This Matters:**
- First proof of exponential quantum advantage over ALL classical algorithms
- Direct inspiration for Shor's factoring algorithm
- Introduces the concept of **hidden subgroup problems**
- Fundamental technique for cryptographic applications

### The Exponential Gap

| Approach | Query Complexity | For n=50 |
|----------|------------------|----------|
| Classical (deterministic) | $O(2^{n/2})$ (birthday bound) | ~33 million queries |
| Classical (randomized) | $O(2^{n/2})$ | ~33 million queries |
| **Quantum (Simon's)** | **$O(n)$** | **~50 queries** |

---

## 2️⃣ WHAT: The Core Definitions

### Simon's Problem Statement

Given a black-box function $f: \{0,1\}^n \rightarrow \{0,1\}^n$ with the **promise** that there exists a hidden string $s \in \{0,1\}^n$ such that:

$$f(x) = f(y) \iff y = x \oplus s$$

**Find the hidden string $s$.**

### Two Cases

**Case 1: $s = 0^n$ (one-to-one function)**
- $f(x) = f(y)$ only when $x = y$
- Function is a permutation (bijection)

**Case 2: $s \neq 0^n$ (two-to-one function)**
- Exactly two inputs map to each output
- Those inputs differ by XOR with $s$
- Example: $f(010) = f(010 \oplus s) = f(111)$ when $s = 101$

### The Key Insight

The XOR relation creates **pairs** of inputs that collide:
- $(x, x \oplus s)$ always map to the same output
- Finding any collision reveals $s = x \oplus y$

---

## 3️⃣ ANALOGY: The Library with Paired Books

Imagine a vast library with $2^n$ books, each with a unique call number (n-bit string). The library has a strange rule:

**The Pairing Rule:** Every book has exactly one "twin" book with the same content but different call numbers. The twins' call numbers differ by a secret pattern $s$.

| Call Number | Content | Twin's Call Number |
|-------------|---------|-------------------|
| 000 | "Quantum Tales" | 000 ⊕ 101 = 101 |
| 001 | "Classical Dreams" | 001 ⊕ 101 = 100 |
| 010 | "Superposition Stories" | 010 ⊕ 101 = 111 |
| 011 | "Entangled Fables" | 011 ⊕ 101 = 110 |

**Classical Approach:** Check books one by one until you find two with identical content. Expected time: $\sqrt{2^n}$ books (birthday paradox).

**Quantum Approach:** Borrow all books "simultaneously" (superposition), use interference to extract constraints on $s$, solve with $n$ library visits.

---

## 4️⃣ FORMAL: Mathematical Foundation

### The Algorithm Structure

```
1. Initialize: |0⟩^n |0⟩^n
2. Apply H^⊗n to first register: Create uniform superposition
3. Apply oracle U_f: Entangle with function values
4. Measure second register: Collapse to f(x) superposition
5. Apply H^⊗n to first register: Decode to constraints
6. Measure first register: Obtain vector z where z·s = 0
7. Repeat O(n) times and solve linear system
```

### Step-by-Step Analysis

**Step 1-2: Create Superposition**
$$|0^n\rangle|0^n\rangle \xrightarrow{H^{\otimes n} \otimes I} \frac{1}{\sqrt{2^n}} \sum_{x=0}^{2^n-1} |x\rangle|0^n\rangle$$

**Step 3: Oracle Application**
$$\xrightarrow{U_f} \frac{1}{\sqrt{2^n}} \sum_{x=0}^{2^n-1} |x\rangle|f(x)\rangle$$

**Step 4: Measure Second Register**

When we measure the second register and obtain some value $f(x_0)$, the first register collapses to:
$$\frac{1}{\sqrt{2}}(|x_0\rangle + |x_0 \oplus s\rangle)$$

(The two inputs that produce this output)

**Step 5: Apply Hadamard to First Register**

Using the identity $H^{\otimes n}|a\rangle = \frac{1}{\sqrt{2^n}}\sum_z (-1)^{a \cdot z}|z\rangle$:

$$H^{\otimes n}\left[\frac{1}{\sqrt{2}}(|x_0\rangle + |x_0 \oplus s\rangle)\right]$$

$$= \frac{1}{\sqrt{2^{n+1}}} \sum_z \left[(-1)^{x_0 \cdot z} + (-1)^{(x_0 \oplus s) \cdot z}\right]|z\rangle$$

$$= \frac{1}{\sqrt{2^{n+1}}} \sum_z (-1)^{x_0 \cdot z}\left[1 + (-1)^{s \cdot z}\right]|z\rangle$$

**The Interference:**
- When $s \cdot z = 0$ (mod 2): Terms add → amplitude $\neq 0$
- When $s \cdot z = 1$ (mod 2): Terms cancel → amplitude $= 0$

**Step 6: Measurement Outcome**

We only observe $z$ values where $s \cdot z = 0$ (mod 2)!

### Solving for s

After $n$ measurements, we collect vectors $z_1, z_2, \ldots, z_n$ satisfying:
$$z_i \cdot s \equiv 0 \pmod{2}$$

This is a system of linear equations over $\mathbb{F}_2$ (binary field). Solve using Gaussian elimination to find $s$.

### Why n Measurements Suffice

Each measurement gives one linear constraint. With high probability, $n$ measurements give $n-1$ linearly independent equations (the trivial $s = 0^n$ always satisfies all equations). We solve for the non-trivial solution.

---

## 5️⃣ VISUAL: Circuit and State Evolution

### Simon's Algorithm Circuit

```
           ┌───┐                     ┌───┐┌─┐
q_0: ──────┤ H ├─────────■──────────┤ H ├┤M├
           ├───┤         │          ├───┤├─┤
q_1: ──────┤ H ├─────────┼──■───────┤ H ├┤M├
           ├───┤         │  │       ├───┤├─┤
q_2: ──────┤ H ├─────────┼──┼──■────┤ H ├┤M├
           └───┘    ┌────┴──┴──┴────┐└───┘└─┘
                    │               │
out_0: ─────────────┤               ├──────────
                    │   Simon's     │
out_1: ─────────────┤   Oracle      ├──────────
                    │   (s=101)     │
out_2: ─────────────┤               ├──────────
                    └───────────────┘
```

### Oracle Construction for Hidden String s

**Step 1: Copy Register** (CNOT from each input to corresponding output)
```
For each i: CNOT(input[i], output[i])
```

**Step 2: XOR with s** (Controlled on first '1' in s)
```
Find first index j where s[j] = 1
For each i where s[i] = 1:
    CNOT(input[j], output[i])
```

### Example: Oracle for s = "11" (2 qubits)

| Input x | x ⊕ s | f(x) |
|---------|-------|------|
| 00 | 11 | → same output, e.g., 00 |
| 01 | 10 | → same output, e.g., 01 |
| 10 | 01 | → same as 01 |
| 11 | 00 | → same as 00 |

---

## 6️⃣ CODE: Qiskit 2.x Implementation

```python
from qiskit import QuantumCircuit, QuantumRegister, ClassicalRegister
from qiskit_aer import AerSimulator
import numpy as np

def create_simon_oracle(n: int, s: str) -> QuantumCircuit:
    """
    Create Simon's oracle for hidden string s.
    
    The oracle implements f such that f(x) = f(x ⊕ s).
    
    Args:
        n: Number of input/output qubits
        s: Hidden string (binary, length n)
    
    Returns:
        Oracle circuit with 2n qubits
    """
    if len(s) != n:
        raise ValueError(f"Hidden string length {len(s)} != n ({n})")
    
    oracle = QuantumCircuit(2 * n, name=f"Oracle(s={s})")
    
    # Step 1: Copy input to output (CNOT from input[i] to output[i])
    for i in range(n):
        oracle.cx(i, n + i)
    
    # Step 2: XOR with s based on first non-zero bit
    # Find first index j where s[j] = '1'
    j = -1
    for idx, bit in enumerate(reversed(s)):  # LSB to MSB
        if bit == '1':
            j = idx
            break
    
    if j != -1:  # s is not all zeros
        # For each position where s has '1', XOR output with input[j]
        for i, bit in enumerate(reversed(s)):
            if bit == '1':
                oracle.cx(j, n + i)
    
    return oracle


def simon_circuit(oracle: QuantumCircuit, n: int) -> QuantumCircuit:
    """
    Construct complete Simon's algorithm circuit.
    
    Args:
        oracle: Simon's oracle circuit
        n: Number of input qubits
    
    Returns:
        Complete Simon's algorithm circuit
    """
    qr_input = QuantumRegister(n, 'input')
    qr_output = QuantumRegister(n, 'output')
    cr = ClassicalRegister(n, 'result')
    
    qc = QuantumCircuit(qr_input, qr_output, cr)
    
    # Step 1: Apply Hadamard to input register
    qc.h(qr_input)
    
    qc.barrier(label="H⊗n")
    
    # Step 2: Apply oracle
    qc.compose(oracle, inplace=True)
    
    qc.barrier(label="oracle")
    
    # Step 3: Apply Hadamard to input register again
    qc.h(qr_input)
    
    qc.barrier(label="H⊗n")
    
    # Step 4: Measure input register only
    qc.measure(qr_input, cr)
    
    return qc


def solve_linear_system(equations: list, n: int) -> str:
    """
    Solve system of linear equations mod 2 using Gaussian elimination.
    
    Args:
        equations: List of measurement outcomes (binary strings)
        n: Number of bits
    
    Returns:
        Hidden string s (or "0"*n if trivial)
    """
    # Convert to numpy array
    matrix = np.array([[int(b) for b in eq] for eq in equations])
    
    # Gaussian elimination mod 2
    rows, cols = matrix.shape
    pivot_row = 0
    
    for col in range(cols):
        # Find pivot
        found = False
        for row in range(pivot_row, rows):
            if matrix[row, col] == 1:
                # Swap rows
                matrix[[pivot_row, row]] = matrix[[row, pivot_row]]
                found = True
                break
        
        if not found:
            continue
        
        # Eliminate
        for row in range(rows):
            if row != pivot_row and matrix[row, col] == 1:
                matrix[row] = (matrix[row] + matrix[pivot_row]) % 2
        
        pivot_row += 1
    
    # Find null space (solutions to Ax = 0)
    # For Simon's problem, we look for the non-trivial solution
    # Try all possible s values and check which satisfies all equations
    for s_int in range(1, 2**n):
        s = format(s_int, f'0{n}b')
        s_vec = np.array([int(b) for b in s])
        
        valid = True
        for eq in equations:
            eq_vec = np.array([int(b) for b in eq])
            if np.dot(eq_vec, s_vec) % 2 != 0:
                valid = False
                break
        
        if valid:
            return s
    
    return "0" * n  # Trivial solution


# Example usage
n = 2
s = "11"

# Create oracle and circuit
oracle = create_simon_oracle(n, s)
circuit = simon_circuit(oracle, n)

print("Simon's Algorithm Circuit (s='11'):")
print(circuit.draw())

# Run and collect measurements
simulator = AerSimulator()
shots = 100
result = simulator.run(circuit, shots=shots).result()
counts = result.get_counts()

print(f"\nMeasurement outcomes: {counts}")

# Collect unique non-trivial equations
equations = [outcome for outcome in counts.keys() if '1' in outcome]
print(f"Non-trivial equations (z where z·s = 0): {equations}")

# Solve for s
if equations:
    found_s = solve_linear_system(equations, n)
    print(f"\nRecovered hidden string: s = {found_s}")
    print(f"Actual hidden string:    s = {s}")
    print(f"Match: {found_s == s}")
```

---

## 7️⃣ TRAPS: Common Mistakes

### ⚠️ Trap 1: Forgetting to Measure Only Input Register

**Wrong:**
```python
# Measuring both registers
qc.measure(range(2*n), range(2*n))
```

**Right:**
```python
# Measure only input register (first n qubits)
qc.measure(range(n), range(n))
```

**Why:** Measuring the output register is done implicitly during the algorithm to collapse the superposition. We only need the input register measurement result $z$.

### ⚠️ Trap 2: Not Collecting Enough Independent Equations

**Problem:** Getting the trivial solution $s = 0^n$

**Solution:** 
- Run the circuit $O(n)$ times
- Filter out trivial outcomes (all zeros)
- Check that equations are linearly independent

```python
# Collect until we have n-1 linearly independent equations
equations = []
while len(equations) < n:
    result = simulator.run(circuit, shots=1).result()
    z = list(result.get_counts().keys())[0]
    if z != "0" * n:  # Non-trivial
        equations.append(z)
```

### ⚠️ Trap 3: Incorrect Oracle Construction

**Problem:** Oracle doesn't satisfy $f(x) = f(x \oplus s)$

**Why It Happens:**
- Forgetting to copy input to output first
- Incorrect control qubit selection for XOR step

**Verification:**
```python
def verify_simon_oracle(oracle, n, s):
    """Verify oracle satisfies f(x) = f(x⊕s) for all x."""
    for x in range(2**n):
        x_xor_s = x ^ int(s, 2)
        # Check f(x) == f(x⊕s)
        fx = evaluate_oracle(oracle, x, n)
        fx_xor_s = evaluate_oracle(oracle, x_xor_s, n)
        assert fx == fx_xor_s, f"Oracle failed: f({x}) ≠ f({x_xor_s})"
```

### ⚠️ Trap 4: Qiskit Bit Ordering Confusion

**Problem:** Hidden string appears reversed

**Remember:**
- Qiskit uses LSB ordering (q[0] is rightmost)
- When reading measurement results, bit order is reversed from circuit order

```python
# If s = "101" in MSB notation
# In Qiskit circuit, this means:
#   q[0] = 1 (LSB)
#   q[1] = 0
#   q[2] = 1 (MSB)
# Measurement result "101" in Qiskit = "101" in our notation (LSB first)
```

### ⚠️ Trap 5: Confusing Simon's with Bernstein-Vazirani

| Aspect | Simon's | Bernstein-Vazirani |
|--------|---------|-------------------|
| Output | n bits | 1 bit |
| Relation | $f(x) = f(x \oplus s)$ | $f(x) = s \cdot x$ |
| Queries | O(n) | 1 |
| Post-processing | Solve linear system | Direct readout |

---

## 🔗 Connections

### Backward Links
- **Deutsch-Jozsa:** Oracle-based algorithms, phase kickback
- **Bernstein-Vazirani:** Same circuit structure, different oracle
- **Qubits:** Superposition enables parallel evaluation

### Forward Links
- **Shor's Algorithm:** Simon's algorithm directly inspired Shor
- **QFT:** Used in Shor's period-finding subroutine
- **Hidden Subgroup Problems:** Simon's is a special case

### Cross-Cutting Themes
- **Quantum Parallelism:** Query oracle on all inputs simultaneously
- **Interference:** Hadamard transform creates destructive interference
- **Hybrid Algorithms:** Quantum measurement + classical post-processing

---

## ✅ Quick Knowledge Check

**Q1:** Why do we need O(n) runs of Simon's algorithm instead of just one?

<details>
<summary>Click for answer</summary>

Each run gives us one equation $z \cdot s = 0$ (mod 2). This single equation has many solutions (half of all n-bit strings satisfy it). We need n-1 linearly independent equations to uniquely determine s (up to the trivial solution $s = 0^n$). The linear system over $\mathbb{F}_2$ requires this many constraints to find the non-trivial solution.
</details>

**Q2:** What happens if we run Simon's algorithm on a one-to-one function ($s = 0^n$)?

<details>
<summary>Click for answer</summary>

When $s = 0^n$, every measurement outcome z is equally likely (no interference cancellation). All z values satisfy $z \cdot 0^n = 0$ trivially. The linear system has only the trivial solution $s = 0^n$, correctly indicating the function is one-to-one.
</details>

**Q3:** Why is Simon's algorithm historically significant?

<details>
<summary>Click for answer</summary>

Simon's algorithm (1994) was the **first** to show an exponential speedup over any classical algorithm, including randomized ones. Previous quantum algorithms (Deutsch-Jozsa, Bernstein-Vazirani) only showed speedup over deterministic algorithms. This inspired Peter Shor to develop his factoring algorithm by adapting Simon's technique to the period-finding problem.
</details>

---

## 📝 Exercises

### Exercise 1: Build and Verify (Beginner)
Create Simon's oracle for $s = "101"$ (3 qubits). Verify that $f(000) = f(101)$ and $f(010) = f(111)$ by simulating the oracle on computational basis states.

### Exercise 2: Full Algorithm Run (Intermediate)
Implement the complete Simon's algorithm for $n = 3$ with $s = "110"$. Run until you collect enough linearly independent equations and solve for $s$ using Gaussian elimination.

### Exercise 3: Oracle Truth Table (Intermediate)
Write a function that takes a Simon oracle and produces its complete truth table. Verify it's two-to-one with the correct pairing structure.

### Exercise 4: Statistical Analysis (Advanced)
Run Simon's algorithm 1000 times for $n = 4$ and $s = "1011"$. Plot the distribution of measurement outcomes. Verify that outcomes not satisfying $z \cdot s = 0$ never appear.

### Exercise 5: Noise Effects (Advanced)
Add depolarizing noise to the Simon's algorithm circuit. How many runs are needed to reliably recover $s$ as noise increases? Plot recovery success rate vs. error rate.

---

## 📖 Summary

| Concept | Key Point |
|---------|-----------|
| **Problem** | Find hidden string s where f(x) = f(x⊕s) |
| **Classical** | O(2^{n/2}) queries (birthday bound) |
| **Quantum** | O(n) queries + classical post-processing |
| **Key Technique** | Hadamard interference + linear algebra |
| **Output** | Vectors z satisfying z·s = 0 (mod 2) |
| **Historical** | First exponential speedup over ALL classical |

---

## 🎯 What's Next?

**Quantum Fourier Transform** - The key subroutine that powers Shor's algorithm. We'll learn how to efficiently implement the discrete Fourier transform on a quantum computer, enabling period-finding and many other algorithms.
