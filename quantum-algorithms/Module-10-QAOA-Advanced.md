# QAOA - Quantum Approximate Optimization Algorithm 🔧

> **From NP-Hard Problems to Quantum Circuits**
> **Difficulty**: 🔴 Advanced | **Time**: 4-5 hours | **Prerequisites**: Adiabatic quantum computing, QPE
> **Codelab**: [10_qaoa_advanced_codelab.ipynb](codelabs/10_qaoa_advanced_codelab.ipynb)

---

## 🎯 Learning Objectives

By the end of this section, you will be able to:

1. ✅ Convert combinatorial optimization problems to Ising Hamiltonians
2. ✅ Understand the QAOA ansatz structure (problem + mixer Hamiltonians)
3. ✅ Implement ZZ gates for cost function encoding
4. ✅ Run QAOA circuits and optimize variational parameters
5. ✅ Analyze solution quality vs circuit depth tradeoffs

---

## 📋 Table of Contents

1. [WHY: The Optimization Revolution](#why-the-optimization-revolution)
2. [WHAT: QAOA Core Concepts](#what-qaoa-core-concepts)
3. [Deep Dive: MaxCut as QAOA](#deep-dive-maxcut-as-qaoa)
4. [Circuit Implementation](#circuit-implementation)
5. [Parameter Optimization](#parameter-optimization)
6. [Trap Alerts](#trap-alerts)
7. [Quick Checks](#quick-checks)
8. [Summary](#summary)

---

## WHY: The Optimization Revolution

### The Problem Landscape

Many of the world's most important problems are **combinatorial optimization**:

| Domain | Problem | Impact |
|--------|---------|--------|
| **Logistics** | Vehicle routing | $B savings annually |
| **Finance** | Portfolio optimization | Better risk-adjusted returns |
| **Drug Discovery** | Molecular docking | Faster drug development |
| **Networks** | Graph partitioning | Efficient data centers |
| **Machine Learning** | Hyperparameter tuning | Better AI models |

### The Classical Wall

These problems share a troubling feature: **NP-hardness**.

```
Problem Size n     Classical Time          Quantum Hope (QAOA)
─────────────────────────────────────────────────────────────
n = 10             1,024 combinations      Polynomial advantage?
n = 50             10^15 combinations      Potential speedup
n = 100            10^30 combinations      ❓ Active research
```

> **QAOA Promise**: Not proven exponential speedup, but **practical advantages** on near-term hardware with approximate solutions.

### WHEN to Use QAOA

| ✅ Use QAOA | ❌ Don't Use QAOA |
|-------------|-------------------|
| Combinatorial optimization | Continuous optimization |
| Approximate solutions OK | Exact solutions required |
| Problem maps to Ising model | Arbitrary constraints |
| NISQ hardware available | Only classical resources |
| Exploring quantum advantage | Proven classical solution exists |

---

## WHAT: QAOA Core Concepts

### 🔷 Definition (7-Element Framework)

**QAOA** (Quantum Approximate Optimization Algorithm) is a **variational hybrid algorithm** that:
1. Encodes an optimization problem as a **cost Hamiltonian** $H_C$
2. Uses a **mixer Hamiltonian** $H_M$ to explore the solution space
3. Alternates applications of $e^{-i\gamma H_C}$ and $e^{-i\beta H_M}$
4. Measures and **classically optimizes** parameters $(\gamma, \beta)$

### 🎭 Analogy: The Mountain Climber

Imagine searching for the lowest valley in a mountain range blindfolded:

```
CLASSICAL APPROACH:
    🚶 Take small random steps
    📉 Check if lower → accept
    🔄 Repeat many times
    ⚠️ Often stuck in local minima

QAOA APPROACH:
    👻 Become quantum "ghost"
    🌊 Exist in ALL valleys simultaneously  
    🎚️ Turn "gravity dial" (γ) to feel where low points are
    🔀 Turn "mixing dial" (β) to tunnel between valleys
    📏 Measure to collapse to ONE valley
    🔄 Adjust dials and repeat
```

**The magic**: Quantum superposition explores all solutions at once!

### 📐 Mathematical Foundation

#### The Cost Function

For an optimization problem, we want to minimize:
$$C(x) = \sum_{\text{clauses}} c_\alpha(x)$$

where $x = (x_1, x_2, ..., x_n)$ are binary variables.

#### The Cost Hamiltonian

Convert to quantum by the substitution:
$$x_i \in \{0, 1\} \rightarrow Z_i \in \{+1, -1\}$$

Using: $x_i = \frac{1 - Z_i}{2}$

This gives us the **cost Hamiltonian** $H_C$ where:
$$H_C|x\rangle = C(x)|x\rangle$$

The ground state of $H_C$ corresponds to the optimal solution!

#### The Mixer Hamiltonian

The standard mixer:
$$H_M = \sum_{i=1}^n X_i$$

Generates transitions between computational basis states.

#### The QAOA Ansatz

Starting from uniform superposition:
$$|\psi(\gamma, \beta)\rangle = e^{-i\beta_p H_M} e^{-i\gamma_p H_C} \cdots e^{-i\beta_1 H_M} e^{-i\gamma_1 H_C} |+\rangle^{\otimes n}$$

**Key insight**: As depth $p \to \infty$, QAOA can find the exact ground state (via quantum adiabatic theorem connection).

### 🖥️ Circuit Structure

```
          ┌───┐ ┌──────────┐ ┌──────────┐   ┌──────────┐ ┌──────────┐
|0⟩ ──────┤ H ├─┤          ├─┤          ├───┤          ├─┤          ├── M
          └───┘ │          │ │          │   │          │ │          │
          ┌───┐ │ e^{-iγHc}│ │ e^{-iβHm}│   │ e^{-iγHc}│ │ e^{-iβHm}│
|0⟩ ──────┤ H ├─┤          ├─┤          ├───┤          ├─┤          ├── M
          └───┘ │          │ │          │   │          │ │          │
          ┌───┐ │          │ │          │   │          │ │          │
|0⟩ ──────┤ H ├─┤          ├─┤          ├───┤          ├─┤          ├── M
          └───┘ └──────────┘ └──────────┘   └──────────┘ └──────────┘
               
               ←────── Layer 1 ───────→   ←────── Layer 2 ───────→
```

---

## Deep Dive: MaxCut as QAOA

### The MaxCut Problem

**Given**: A graph $G = (V, E)$ with vertices and edges

**Goal**: Partition vertices into two sets to **maximize** the number of edges cut

```
    Before Cut           After Optimal Cut
                         
    A ─── B              A │ B
    │ ╲   │                │
    │  ╲  │      →       C │ D
    │   ╲ │                │
    C ─── D              Edges cut: 4 ✓
```

### Step 1: Cost Function

The cost function counts edges between partitions:
$$C(x) = \sum_{(i,j) \in E} x_i(1-x_j) + x_j(1-x_i)$$

Simplifies to:
$$C(x) = \sum_{(i,j) \in E} (x_i + x_j - 2x_i x_j)$$

### Step 2: Convert to Hamiltonian

Using $x_i = \frac{1-Z_i}{2}$:

$$H_C = \sum_{(i,j) \in E} \frac{1}{2}(1 - Z_i Z_j)$$

> **Key insight**: Each edge contributes a **ZZ interaction term**!

### Step 3: The ZZ Gate

To implement $e^{-i\gamma H_C}$, we need:
$$e^{-i\gamma \cdot \frac{1}{2}(1 - Z_i Z_j)} = e^{-i\gamma/2} \cdot e^{i\gamma Z_i Z_j / 2}$$

The **ZZ gate** implements $e^{i\gamma Z_i Z_j / 2}$:

```
Circuit for e^{iγ Z_i Z_j / 2}:

q_i: ───●───────────────●───
        │               │
q_j: ───X───Rz(γ)───────X───

Proof:
• CNOT flips target if control is |1⟩
• In |00⟩,|11⟩ basis: Rz acts on |0⟩ → phase e^{-iγ/2}
• In |01⟩,|10⟩ basis: Rz acts on |1⟩ → phase e^{+iγ/2}
• Result: Z⊗Z eigenvalue determines phase sign ✓
```

### Step 4: The Mixer

The mixer $e^{-i\beta H_M} = e^{-i\beta \sum_j X_j}$ factorizes:
$$e^{-i\beta \sum_j X_j} = \prod_j e^{-i\beta X_j} = \prod_j R_X(2\beta)$$

Just single-qubit X rotations!

### Complete QAOA Circuit for MaxCut

```
        ┌───┐ ┌─────────────────────────┐ ┌──────────┐
q_0: ───┤ H ├─┤                         ├─┤ Rx(2β₁)  ├─── ... ─── M
        └───┘ │   ZZ(γ₁) on each edge   │ ├──────────┤
q_1: ───┤ H ├─┤                         ├─┤ Rx(2β₁)  ├─── ... ─── M
        ├───┤ │                         │ ├──────────┤
q_2: ───┤ H ├─┤                         ├─┤ Rx(2β₁)  ├─── ... ─── M
        ├───┤ │                         │ ├──────────┤
q_3: ───┤ H ├─┤                         ├─┤ Rx(2β₁)  ├─── ... ─── M
        └───┘ └─────────────────────────┘ └──────────┘
         │              │                      │
      Initial    Cost Hamiltonian          Mixer
     |+⟩^⊗n        exp(-iγHc)            exp(-iβHm)
```

---

## Circuit Implementation

### ZZ Gate Implementation

```python
def zz_gate(qc, q1, q2, gamma):
    """
    Apply exp(i * gamma * Z_i Z_j / 2) gate.
    
    This is the building block for cost Hamiltonian evolution.
    """
    qc.cx(q1, q2)          # CNOT: maps ZZ eigenvalue to Z on q2
    qc.rz(gamma, q2)       # Rz(γ) rotation based on ZZ eigenvalue
    qc.cx(q1, q2)          # Undo CNOT
```

### Full QAOA Layer

```python
def qaoa_layer(qc, graph, gamma, beta):
    """
    Single QAOA layer: cost evolution + mixer.
    
    Parameters:
    - graph: NetworkX graph with edges
    - gamma: cost Hamiltonian parameter
    - beta: mixer parameter
    """
    # Cost Hamiltonian: ZZ on each edge
    for (i, j) in graph.edges():
        zz_gate(qc, i, j, gamma)
    
    # Mixer Hamiltonian: Rx on each qubit
    for i in range(graph.number_of_nodes()):
        qc.rx(2 * beta, i)
```

### Complete QAOA Circuit

```python
from qiskit import QuantumCircuit
import networkx as nx

def create_qaoa_circuit(graph, gammas, betas):
    """
    Create full QAOA circuit with p layers.
    
    Parameters:
    - graph: NetworkX graph
    - gammas: list of p gamma values
    - betas: list of p beta values
    
    Returns:
    - QuantumCircuit with QAOA ansatz
    """
    n_qubits = graph.number_of_nodes()
    p = len(gammas)  # Number of layers
    
    qc = QuantumCircuit(n_qubits, n_qubits)
    
    # Initial state: uniform superposition
    qc.h(range(n_qubits))
    
    # Apply p QAOA layers
    for layer in range(p):
        qaoa_layer(qc, graph, gammas[layer], betas[layer])
    
    # Measurement
    qc.measure(range(n_qubits), range(n_qubits))
    
    return qc
```

---

## Parameter Optimization

### The Optimization Loop

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Classical  │     │   Quantum   │     │  Classical  │
│  Optimizer  │────▶│   Circuit   │────▶│  Evaluate   │
│ (γ, β)      │     │   Execute   │     │  ⟨H_C⟩      │
└─────────────┘     └─────────────┘     └─────────────┘
       ▲                                       │
       │                                       │
       └───────────── Update ──────────────────┘
```

### Cost Function Evaluation

The expectation value of $H_C$ from measurement results:

$$\langle H_C \rangle = \sum_{(i,j) \in E} \frac{1 - \langle Z_i Z_j \rangle}{2}$$

From measurement counts:
$$\langle Z_i Z_j \rangle = \frac{N_{same} - N_{different}}{N_{total}}$$

where $N_{same}$ counts bitstrings where qubits $i$ and $j$ have the same value.

### Optimization Implementation

```python
from scipy.optimize import minimize
from qiskit_aer import AerSimulator

def compute_expectation(graph, counts):
    """
    Compute MaxCut cost from measurement counts.
    """
    total_cost = 0
    total_shots = sum(counts.values())
    
    for bitstring, count in counts.items():
        # Count edges cut
        edges_cut = 0
        for (i, j) in graph.edges():
            # Bitstring is reversed in Qiskit
            bit_i = int(bitstring[-(i+1)])
            bit_j = int(bitstring[-(j+1)])
            if bit_i != bit_j:
                edges_cut += 1
        total_cost += edges_cut * count
    
    return total_cost / total_shots

def qaoa_objective(params, graph, p, shots=1024):
    """
    Objective function for classical optimizer.
    """
    gammas = params[:p]
    betas = params[p:]
    
    qc = create_qaoa_circuit(graph, gammas, betas)
    
    simulator = AerSimulator()
    result = simulator.run(qc, shots=shots).result()
    counts = result.get_counts()
    
    # Return negative because we maximize cuts
    return -compute_expectation(graph, counts)

def run_qaoa(graph, p=1, initial_params=None):
    """
    Run QAOA optimization.
    """
    if initial_params is None:
        # Random initialization
        import numpy as np
        initial_params = np.random.uniform(0, np.pi, 2*p)
    
    result = minimize(
        qaoa_objective,
        initial_params,
        args=(graph, p),
        method='COBYLA'
    )
    
    return result
```

---

## QAOA Performance Analysis

### Depth vs Quality Tradeoff

| Depth p | Parameters | Expected Quality | Circuit Depth | NISQ Feasibility |
|---------|------------|-----------------|---------------|------------------|
| 1 | 2 | ~70% optimal | O(|E|) | ✅ Excellent |
| 2 | 4 | ~80% optimal | O(2|E|) | ✅ Good |
| 3 | 6 | ~85% optimal | O(3|E|) | ⚠️ Moderate |
| 5+ | 10+ | ~90%+ optimal | O(p|E|) | ❌ Challenging |

### Approximation Ratio

For MaxCut on 3-regular graphs, QAOA with $p=1$ achieves:
$$\alpha_{QAOA} \geq 0.6924$$

Compared to classical Goemans-Williamson: $\alpha_{GW} \approx 0.878$

> **Research frontier**: What depth $p$ beats classical algorithms?

---

## Trap Alerts 🚨

### Trap 1: Wrong ZZ Phase Sign
```python
# ❌ WRONG: Opposite sign gives wrong Hamiltonian
qc.rz(-gamma, q2)  # Negative sign

# ✅ CORRECT: Match your Hamiltonian convention
qc.rz(gamma, q2)   # For H = (1-ZZ)/2
```

### Trap 2: Forgetting Global Phase
```python
# The cost Hamiltonian has a constant term!
# H_C = Σ (1 - Z_i Z_j) / 2
#     = |E|/2 - Σ Z_i Z_j / 2
#       ↑ constant  ↑ operator

# This affects the energy scale but NOT the optimization!
# Just remember when computing ⟨H_C⟩
```

### Trap 3: Qiskit Bitstring Ordering
```python
# ❌ WRONG: Assuming left-to-right qubit ordering
for i, j in graph.edges():
    bit_i = int(bitstring[i])  # Wrong index!

# ✅ CORRECT: Qiskit uses reversed ordering (qubit 0 rightmost)
for i, j in graph.edges():
    bit_i = int(bitstring[-(i+1)])  # Correct!
```

### Trap 4: Parameter Initialization
```python
# ❌ WRONG: All zeros (trivial evolution)
params = np.zeros(2*p)

# ❌ WRONG: Large random values (bad local minima)
params = np.random.uniform(0, 10*np.pi, 2*p)

# ✅ CORRECT: Small random values or heuristic
params = np.random.uniform(0, np.pi, 2*p)
# Or use INTERP initialization for p > 1
```

### Trap 5: Insufficient Shots
```python
# ❌ WRONG: Too few shots for reliable statistics
shots = 100  # High variance in ⟨H_C⟩

# ✅ CORRECT: Scale shots with problem size
shots = max(1024, 100 * graph.number_of_nodes())
```

---

## Mnemonic: MIXER 🎛️

Remember the QAOA workflow with **MIXER**:

| Letter | Step | Details |
|--------|------|---------|
| **M** | **Map** problem to Ising | $C(x) \to H_C$ via $x_i \to (1-Z_i)/2$ |
| **I** | **Initialize** superposition | $\|+\rangle^{\otimes n}$ uniform over all solutions |
| **X** | **eXponentiate** alternately | $e^{-i\gamma H_C}$ then $e^{-i\beta H_M}$ |
| **E** | **Evaluate** cost function | Measure and compute $\langle H_C \rangle$ |
| **R** | **Repeat** optimization | Classical optimizer updates $(\gamma, \beta)$ |

---

## Quick Checks ✅

### 🟢 Basic Understanding

1. **What is the mixer Hamiltonian's role in QAOA?**
   <details>
   <summary>Answer</summary>
   The mixer Hamiltonian $H_M = \sum_i X_i$ generates transitions between computational basis states, allowing exploration of the solution space. Without it, states would only accumulate phases but never mix amplitudes.
   </details>

2. **Why do we use ZZ gates for MaxCut?**
   <details>
   <summary>Answer</summary>
   Each edge $(i,j)$ in MaxCut contributes $(1-Z_iZ_j)/2$ to the Hamiltonian. The ZZ gate implements $e^{i\gamma Z_iZ_j}$, which is exactly what we need for time evolution under this Hamiltonian term.
   </details>

### 🟡 Intermediate Application

3. **A 4-node complete graph (K4) has how many ZZ gates per layer?**
   <details>
   <summary>Answer</summary>
   K4 has $\binom{4}{2} = 6$ edges, so there are 6 ZZ gates per QAOA layer (one for each edge in the cost Hamiltonian).
   </details>

4. **If QAOA returns bitstring "0110", how many edges are cut in K4?**
   <details>
   <summary>Answer</summary>
   "0110" partitions vertices {0,3} from {1,2}. In K4:
   - Edges cut: (0,1), (0,2), (1,3), (2,3) = 4 edges
   - Edges not cut: (0,3), (1,2) = 2 edges
   Maximum possible is 4, so this is optimal!
   </details>

### 🔴 Advanced Analysis

5. **Why doesn't QAOA guarantee finding the global optimum?**
   <details>
   <summary>Answer</summary>
   QAOA with finite depth $p$ is a variational algorithm. The ansatz may not span the full Hilbert space needed to reach the ground state. Additionally, classical optimization can get stuck in local minima. Only as $p \to \infty$ does QAOA recover adiabatic quantum computing guarantees.
   </details>

6. **How does QAOA relate to the quantum adiabatic algorithm?**
   <details>
   <summary>Answer</summary>
   QAOA is a "Trotterized" or discretized version of quantum adiabatic evolution. In adiabatic QC, you slowly evolve from the ground state of $H_M$ to that of $H_C$. QAOA uses discrete steps with optimizable parameters, trading continuous evolution for variational flexibility.
   </details>

---

## Beyond MaxCut: Other QAOA Applications

| Problem | Hamiltonian Form | Industry Application |
|---------|------------------|---------------------|
| **Max-SAT** | Clause penalties | Circuit verification |
| **Traveling Salesman** | Distance + constraints | Logistics, routing |
| **Graph Coloring** | Edge conflict penalties | Scheduling |
| **Portfolio Optimization** | Return - risk | Finance |
| **Minimum Vertex Cover** | Coverage incentive | Network design |

---

## Summary

### Key Takeaways

1. **QAOA converts optimization to quantum**: Map $C(x) \to H_C$ using $x_i \to (1-Z_i)/2$

2. **Two-Hamiltonian dance**: 
   - $e^{-i\gamma H_C}$: Encodes problem structure (ZZ gates for MaxCut)
   - $e^{-i\beta H_M}$: Enables exploration (Rx gates)

3. **ZZ gate = CNOT sandwich**: `CNOT - Rz(γ) - CNOT` implements ZZ evolution

4. **Hybrid optimization**: Quantum evaluates $\langle H_C \rangle$, classical updates $(\gamma, \beta)$

5. **Depth-quality tradeoff**: Higher $p$ → better solutions but deeper circuits

### What's Next?

- **Codelab**: Implement QAOA for MaxCut in [10_qaoa_advanced_codelab.ipynb](codelabs/10_qaoa_advanced_codelab.ipynb)
- **Next**: [HHL Algorithm](Module-11-HHL-Algorithm.md) for linear systems
- **Extensions**: Warm-start QAOA, constraint handling, ADAPT-QAOA

---

## References

1. Farhi, E., Goldstone, J., & Gutmann, S. (2014). "A Quantum Approximate Optimization Algorithm"
2. Hadfield, S., et al. (2019). "From the Quantum Approximate Optimization Algorithm to a Quantum Alternating Operator Ansatz"
3. Zhou, L., et al. (2020). "Quantum Approximate Optimization Algorithm: Performance, Mechanism, and Implementation on Near-Term Devices"

---

*Part of the Interactive Quantum Computing Learning Series*
