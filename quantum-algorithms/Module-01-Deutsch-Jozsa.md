# Deutsch-Jozsa Algorithm 🎯

> **Difficulty**: 🟡 Intermediate
> **Time**: 4-5 hours
> **Prerequisites**: Qubits, Superposition, Hadamard gates, Measurement
> **Framework**: 7-Element + WHY-WHAT-WHEN

---

## 🔍 WHY This Matters

The Deutsch-Jozsa algorithm is historically the **first quantum algorithm** to demonstrate an exponential speedup over classical deterministic computation. While not practically useful, it serves as a fundamental proof-of-concept that:

1. **Quantum parallelism** via superposition can query all inputs simultaneously
2. **Quantum interference** can be engineered to extract global properties
3. **Exponential speedup is possible**: O(1) quantum vs O(2^n) classical

> **Historical Note**: David Deutsch (1985) and Deutsch-Jozsa (1992) kickstarted the field of quantum algorithms, directly inspiring Shor's and Grover's breakthroughs.

---

## 📌 1. DEFINITION

**The Deutsch-Jozsa algorithm determines whether a Boolean function f: {0,1}ⁿ → {0,1} is constant or balanced using exactly ONE quantum query.**

### Problem Statement

Given an oracle for function f(x) where:
- **Constant**: f(x) = 0 for ALL inputs, OR f(x) = 1 for ALL inputs
- **Balanced**: f(x) = 0 for EXACTLY half the inputs, and f(x) = 1 for the other half

**Task**: Determine whether f is constant or balanced.

**Promise**: The function is guaranteed to be either constant or balanced (no other possibility).

### Classical Complexity

**Worst-case deterministic**: 2^(n-1) + 1 queries
- Must check slightly more than half the inputs to be certain
- If first 2^(n-1) outputs are all the same, could still be balanced

**Quantum complexity**: 1 query
- Single quantum query determines the answer with 100% certainty

### Key Insight

Classical approach: Check inputs one at a time, collect partial information
Quantum approach: Create superposition of ALL inputs, use interference to reveal global property

---

## 💡 2. ANALOGY & INTUITION

### The "Magic Library" Analogy

Imagine a library with 2ⁿ books, each with a single-digit number (0 or 1) on its cover. You're told either:
- **All books have the same number** (constant)
- **Exactly half have 0, half have 1** (balanced)

**Classical approach**: Read books one by one. In the worst case, you might read 2^(n-1) books (all showing "0") before the next one reveals "1" (balanced) or confirms all are "0" (constant).

**Quantum approach**: Create a "magical superposition" where you check ALL books simultaneously. The magic is that constant vs balanced produces completely different interference patterns—one measurement reveals the answer!

### Why Does Interference Work Here?

When f is **constant**:
- All computational paths contribute with the SAME sign
- Constructive interference at the |00...0⟩ state
- Probability of measuring |00...0⟩ = 100%

When f is **balanced**:
- Half the paths have + sign, half have - sign
- Destructive interference at |00...0⟩
- Probability of measuring |00...0⟩ = 0%

### Where Analogy Breaks Down

The "checking all books simultaneously" is misleading if you think you get ALL answers. You don't. You get ONE answer, but the quantum interference ensures that one answer reveals the global property you seek.

---

## 📐 3. MATH & VISUAL

### Circuit Diagram

```
     ┌───┐          ┌───┐ ░ ┌─┐
q_0: ┤ H ├──────────┤ H ├─░─┤M├──────
     ├───┤          ├───┤ ░ └╥┘┌─┐
q_1: ┤ H ├──────────┤ H ├─░──╫─┤M├───
     ├───┤   ┌────┐ ├───┤ ░  ║ └╥┘┌─┐
q_2: ┤ H ├───┤    ├─┤ H ├─░──╫──╫─┤M├
     ├───┤   │    │ └───┘ ░  ║  ║ └╥┘
 ⋮    ⋮      │ Uf │       ░  ⋮  ⋮  ⋮
     ├───┤   │    │ ┌───┐ ░  ║  ║  ║
q_n: ┤ X ├───┤    ├─┤ H ├─░──╫──╫──╫─
     └───┘   └────┘ └───┘ ░  ║  ║  ║
              oracle          measurements
```

### Algorithm Steps

**Step 1: Initialize**
$$|0\rangle^{\otimes n}|1\rangle$$

Apply X to ancilla qubit to prepare |1⟩.

**Step 2: Apply Hadamard to all qubits**
$$\frac{1}{\sqrt{2^{n+1}}}\sum_{x=0}^{2^n-1}|x\rangle(|0\rangle - |1\rangle)$$

The n input qubits become uniform superposition over all 2ⁿ basis states.
The ancilla becomes the |-⟩ state.

**Step 3: Apply Oracle Uf**

The oracle performs: |x⟩|y⟩ → |x⟩|y ⊕ f(x)⟩

With ancilla in |-⟩ state, this becomes:
$$|x\rangle \rightarrow (-1)^{f(x)}|x\rangle$$

This is called **phase kickback** - the function value becomes a phase!

**Step 4: Apply Hadamard to input qubits**

Using the identity:
$$H^{\otimes n}|x\rangle = \frac{1}{\sqrt{2^n}}\sum_{z=0}^{2^n-1}(-1)^{x \cdot z}|z\rangle$$

The final state becomes:
$$\frac{1}{2^n}\sum_{x=0}^{2^n-1}\sum_{z=0}^{2^n-1}(-1)^{f(x)+x \cdot z}|z\rangle$$

**Step 5: Measure**

The amplitude of |0⟩^⊗n is:
$$\frac{1}{2^n}\sum_{x=0}^{2^n-1}(-1)^{f(x)}$$

- **Constant f**: All terms have same sign → amplitude = ±1 → P(00...0) = 1
- **Balanced f**: Half +1, half -1 → amplitude = 0 → P(00...0) = 0

### Mathematical Proof of Correctness

**For constant f(x) = c:**
$$\text{amplitude}(|0\rangle^{\otimes n}) = \frac{1}{2^n}\sum_{x}(-1)^c = (-1)^c$$

Probability = |(-1)^c|² = 1

**For balanced f:**
$$\text{amplitude}(|0\rangle^{\otimes n}) = \frac{1}{2^n}\left(\sum_{f(x)=0}(+1) + \sum_{f(x)=1}(-1)\right) = \frac{1}{2^n}(2^{n-1} - 2^{n-1}) = 0$$

---

## 💻 4. IMPLEMENTATION

### Quantum Oracle Construction

For quantum implementation, we need to construct oracles as quantum circuits:

**Constant Oracle (f(x) = 0)**: No gates needed (identity)

**Constant Oracle (f(x) = 1)**: Apply X gate to ancilla

**Balanced Oracle**: Apply CNOT gates from specific input qubits to ancilla

Example balanced oracle for n=3:
```
       ┌───┐
q_0: ──■──┼───
       │  │
q_1: ──┼──■───
       │  │
q_2: ──┼──┼───
     ┌─┴─┐│
q_3: ┤ X ├┴───
     └───┘
```

This implements f(x₀, x₁, x₂) = x₀ ⊕ x₁

### Qiskit 2.x Implementation

```python
from qiskit import QuantumCircuit
from qiskit.quantum_info import Statevector
from qiskit_aer import AerSimulator
import numpy as np

def create_constant_oracle(n: int, output: int) -> QuantumCircuit:
    """
    Create constant oracle: f(x) = output for all x.
    
    Args:
        n: Number of input qubits
        output: Constant output value (0 or 1)
    
    Returns:
        Oracle quantum circuit
    """
    oracle = QuantumCircuit(n + 1, name=f"Uf(const={output})")
    if output == 1:
        oracle.x(n)  # Flip ancilla
    return oracle

def create_balanced_oracle(n: int, hidden_bits: str = None) -> QuantumCircuit:
    """
    Create balanced oracle: f(x) = s·x mod 2 for some hidden bitstring s.
    
    Args:
        n: Number of input qubits
        hidden_bits: Bitstring determining which qubits control the CNOT
                     (default: "1" * n, i.e., all qubits)
    
    Returns:
        Oracle quantum circuit
    """
    if hidden_bits is None:
        hidden_bits = "1" * n
    
    oracle = QuantumCircuit(n + 1, name=f"Uf(balanced)")
    
    # Apply CNOT from each qubit position where hidden_bits has a "1"
    for i, bit in enumerate(reversed(hidden_bits)):
        if bit == "1":
            oracle.cx(i, n)
    
    return oracle

def deutsch_jozsa(oracle: QuantumCircuit, n: int) -> QuantumCircuit:
    """
    Construct Deutsch-Jozsa algorithm circuit.
    
    Args:
        oracle: The oracle circuit (n+1 qubits)
        n: Number of input qubits
    
    Returns:
        Complete DJ circuit with measurements
    """
    qc = QuantumCircuit(n + 1, n)
    
    # Step 1: Initialize ancilla to |1⟩
    qc.x(n)
    
    # Step 2: Apply Hadamard to all qubits
    qc.h(range(n + 1))
    
    qc.barrier()
    
    # Step 3: Apply oracle
    qc.compose(oracle, inplace=True)
    
    qc.barrier()
    
    # Step 4: Apply Hadamard to input qubits
    qc.h(range(n))
    
    # Step 5: Measure input qubits
    qc.measure(range(n), range(n))
    
    return qc

# Example usage
n = 3

# Test with constant oracle
print("Testing CONSTANT oracle (f(x) = 0):")
constant_oracle = create_constant_oracle(n, 0)
dj_circuit = deutsch_jozsa(constant_oracle, n)
print(dj_circuit.draw())

# Run on simulator
simulator = AerSimulator()
job = simulator.run(dj_circuit, shots=1024)
result = job.result()
counts = result.get_counts()
print(f"Results: {counts}")
print("→ All zeros = CONSTANT function ✓" if "000" in counts else "→ Non-zero = BALANCED")
```

---

## ⚠️ 5. COMMON TRAPS

### Trap 1: Confusing Superposition with Parallel Computation

❌ **Wrong thinking**: "I'm computing f(x) for all x in parallel and getting all answers"

✅ **Correct**: You're computing all values in superposition, but measurement collapses to ONE outcome. The trick is using interference to make that one outcome reveal the global property.

### Trap 2: Forgetting the Promise

❌ **Error**: "I'll use Deutsch-Jozsa to check if my function is constant or balanced"

✅ **Correct**: The algorithm only works with the PROMISE that f is either constant OR balanced. If f is neither, the algorithm gives meaningless results.

### Trap 3: Misunderstanding the Speedup

❌ **Claim**: "Deutsch-Jozsa gives exponential speedup for real problems"

✅ **Reality**: The speedup is only vs DETERMINISTIC classical. A randomized classical algorithm can solve this problem with O(1) queries and high probability (just check a few random inputs).

### Trap 4: Incorrect Oracle Phase Sign

❌ **Bug**: Implementing |x⟩|y⟩ → |x⟩|f(x)⟩ instead of |x⟩|y⟩ → |x⟩|y ⊕ f(x)⟩

✅ **Correct**: The oracle XORs f(x) with the ancilla. This enables phase kickback when ancilla is in |-⟩ state.

### Trap 5: Measuring the Ancilla

❌ **Bug**: Measuring all n+1 qubits

✅ **Correct**: Only measure the n input qubits. The ancilla is just a "catalyst" for phase kickback.

---

## 🔗 6. CONNECTIONS

### Prerequisites (Upstream)

- **Qubits & Superposition**: Understand how qubits exist in superposition states
- **Measurement & Born Rule**: Understand measurement collapse and probability
- **Single-Qubit Gates**: Hadamard, X, and Z gates
- **Multi-Qubit Gates**: CNOT gate operation

### Related Concepts (Lateral)

- **Bernstein-Vazirani**: Uses identical circuit structure, different oracle interpretation
- **Simon's Algorithm**: Another oracle-based algorithm with exponential quantum speedup
- **Phase Kickback**: Fundamental technique used throughout quantum algorithms

### Leads To (Downstream)

- **Grover's Algorithm**: Extends interference idea to unstructured search
- **Shor's Algorithm**: Uses similar interference patterns for period finding
- **Quantum Oracles**: Understanding how to construct and use oracle circuits

### NISQ Era Relevance

While Deutsch-Jozsa itself isn't practically useful, the techniques are foundational:
- Phase kickback appears in VQE and QAOA
- Oracle construction techniques used in quantum machine learning
- Interference design patterns used throughout quantum algorithms

---

## 📝 7. QUICK CHECKS

### Conceptual Questions

**Q1**: Why do we initialize the ancilla qubit to |1⟩ before applying Hadamard?
<details>
<summary>Answer</summary>
To create the |-⟩ = (|0⟩ - |1⟩)/√2 state. This enables phase kickback: when the oracle applies |y⊕f(x)⟩, the |-⟩ state acquires a phase (-1)^f(x), which "kicks back" to the input register as a global phase factor.
</details>

**Q2**: What happens if you run Deutsch-Jozsa on a function that is neither constant nor balanced?
<details>
<summary>Answer</summary>
The algorithm may give any output. The result is undefined because the algorithm was designed with the promise that f is either constant or balanced. Without this promise, no guarantees can be made about the output.
</details>

**Q3**: Why can a randomized classical algorithm also solve this problem efficiently?
<details>
<summary>Answer</summary>
Check ~log(1/ε) random inputs. If all give the same output, the function is almost certainly constant (probability of balanced function giving same output decreases exponentially). This achieves O(1) queries with high probability, matching quantum's O(1) queries.
</details>

**Q4**: In the circuit, what is the final state of the ancilla qubit?
<details>
<summary>Answer</summary>
The ancilla remains in the |-⟩ state (up to global phase). The oracle doesn't change the ancilla's reduced state because XORing with f(x)=0 gives |-⟩ and XORing with f(x)=1 gives -|-⟩ = |-⟩ (same state with global phase).
</details>

### Calculation Practice

**P1**: For a 3-qubit input Deutsch-Jozsa, what is the state immediately after applying Hadamards to the input qubits (before oracle)?

<details>
<summary>Answer</summary>

$$|\psi\rangle = \frac{1}{\sqrt{8}}\sum_{x=0}^{7}|x\rangle \otimes |-\rangle$$

$$= \frac{1}{\sqrt{8}}(|000\rangle + |001\rangle + |010\rangle + |011\rangle + |100\rangle + |101\rangle + |110\rangle + |111\rangle) \otimes |-\rangle$$

This is a uniform superposition over all 8 input states, tensored with |-⟩ on the ancilla.
</details>

**P2**: If the oracle implements f(x) = x₀ ⊕ x₁ (XOR of first two bits), show this is a balanced function.

<details>
<summary>Answer</summary>

For 3-qubit inputs x = x₀x₁x₂:
- f(000) = 0⊕0 = 0
- f(001) = 0⊕0 = 0
- f(010) = 0⊕1 = 1
- f(011) = 0⊕1 = 1
- f(100) = 1⊕0 = 1
- f(101) = 1⊕0 = 1
- f(110) = 1⊕1 = 0
- f(111) = 1⊕1 = 0

Outputs: {0, 0, 1, 1, 1, 1, 0, 0} → 4 zeros, 4 ones → Balanced ✓
</details>

---

## 🧪 EXERCISES

### Exercise 1: Verify Algorithm by Hand (Beginner)
For n=2 with constant oracle f(x)=0, trace through the algorithm step by step. Write out the state after each operation and verify that measuring |00⟩ has probability 1.

### Exercise 2: Design a Balanced Oracle (Intermediate)
Construct a balanced oracle for n=4 that implements f(x) = x₀ ⊕ x₂ (XOR of bits 0 and 2). Draw the circuit and verify it produces exactly 8 outputs of 0 and 8 outputs of 1.

### Exercise 3: General Oracle Construction (Advanced)
Prove that for any balanced Boolean function f, there exists a quantum circuit using at most 2ⁿ Toffoli gates and n CNOT gates that implements the oracle Uf.

### Exercise 4: Error Analysis (NISQ)
Run Deutsch-Jozsa on IBM Quantum hardware with n=3. Compare results for constant vs balanced oracles. Analyze how circuit depth affects the error rate.

---

## 📚 Further Reading

1. **Original Paper**: Deutsch, D. & Jozsa, R. (1992). "Rapid solution of problems by quantum computation"
2. **Nielsen & Chuang**: Chapter 1.4 - Quantum algorithms
3. **Qiskit Textbook**: [Deutsch-Jozsa Algorithm](https://qiskit.org/textbook/ch-algorithms/deutsch-jozsa.html)

---

## 🎯 Learning Outcomes

After completing this section, you should be able to:

1. ✅ Explain the Deutsch-Jozsa problem and the promise it requires
2. ✅ Construct oracle circuits for constant and balanced functions
3. ✅ Trace through the algorithm mathematically, showing state evolution
4. ✅ Explain why phase kickback enables the algorithm to work
5. ✅ Implement Deutsch-Jozsa in Qiskit 2.x and interpret results
6. ✅ Discuss the algorithm's significance and limitations in the context of quantum speedup

---

**Next**: [Bernstein-Vazirani Algorithm](Module-02-Bernstein-Vazirani.md) →
