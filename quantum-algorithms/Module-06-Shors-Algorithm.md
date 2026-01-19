# Shor's Algorithm - Quantum Factoring

## Educational Framework: WHY-WHAT-WHEN

---

## 📍 Where This Fits

```
QUANTUM ALGORITHMS
├── Oracle-Based (Part 1)
│   ├── Deutsch-Jozsa
│   └── Bernstein-Vazirani
├── Period Finding (Part 2)
│   ├── Simon's Algorithm
│   └── QFT
├── Phase Estimation (Part 3) ← YOU ARE HERE
│   ├── QPE [Prerequisite]
│   ├── Shor's Algorithm [Current]
│   └── Grover's Search [Next]
└── Variational (Part 4)
```

**Prerequisites**: Quantum Phase Estimation, QFT, Modular Arithmetic Basics

---

## 1️⃣ WHY: The Cryptographic Earthquake

### The Problem That Keeps Security Experts Awake

> **RSA Security Assumption**: Given $N = p \times q$ (product of two large primes), finding $p$ and $q$ is computationally infeasible.

This is the foundation of:
- 🔐 Online banking
- 📧 Encrypted email
- 🌐 HTTPS websites
- 🔑 Digital signatures

**Classical complexity**: Best known algorithm (General Number Field Sieve) runs in sub-exponential time:
$$T_{classical} = e^{O(n^{1/3} \log^{2/3} n)}$$

For a 2048-bit number: ~$10^{30}$ operations → **billions of years**

**Shor's quantum algorithm**:
$$T_{quantum} = O(n^3)$$

For a 2048-bit number: ~$10^{10}$ operations → **minutes to hours**

### Why Factoring Is Hard Classically

```
Easy direction:   7 × 11 = 77  ✓ (multiplication)
Hard direction:   77 = ? × ?   ✗ (factoring)
```

The problem is **one-way**: easy to compute forward, hard to reverse.

### Shor's Insight: Factoring → Period Finding

Peter Shor's genius was reducing factoring to period finding—a problem where quantum computers have exponential advantage!

$$\boxed{\text{Factoring } N \xrightarrow{\text{number theory}} \text{Period of } a^x \mod N \xrightarrow{\text{QPE}} \text{Quantum speedup}}$$

---

## 2️⃣ WHAT: The Algorithm Architecture

### High-Level Structure

```
┌─────────────────────────────────────────────────────────────┐
│                    SHOR'S ALGORITHM                         │
├─────────────────────────────────────────────────────────────┤
│  CLASSICAL PREPROCESSING                                    │
│  1. Check if N is even, prime power, etc.                  │
│  2. Choose random a where 1 < a < N                        │
│  3. Compute gcd(a, N) - if > 1, we found a factor!         │
├─────────────────────────────────────────────────────────────┤
│  QUANTUM SUBROUTINE (Order Finding)                        │
│  4. Use QPE to find period r of f(x) = a^x mod N           │
├─────────────────────────────────────────────────────────────┤
│  CLASSICAL POSTPROCESSING                                   │
│  5. Use continued fractions to extract r from QPE result   │
│  6. Compute gcd(a^(r/2) ± 1, N) to get factors             │
│  7. Repeat if necessary                                     │
└─────────────────────────────────────────────────────────────┘
```

### The Key Mathematical Insight

**Theorem (Factoring via Period)**: If $r$ is the order of $a$ modulo $N$ (smallest positive $r$ where $a^r \equiv 1 \mod N$), and $r$ is even, then:

$$\gcd(a^{r/2} - 1, N) \quad \text{and} \quad \gcd(a^{r/2} + 1, N)$$

are likely non-trivial factors of $N$.

**Why this works**:
$$a^r \equiv 1 \mod N$$
$$a^r - 1 \equiv 0 \mod N$$
$$(a^{r/2} - 1)(a^{r/2} + 1) \equiv 0 \mod N$$

So $N$ divides $(a^{r/2} - 1)(a^{r/2} + 1)$, meaning factors of $N$ divide one or both terms!

### Example: Factoring N = 15

Let's factor 15 step by step:

**Step 1**: Choose $a = 7$ (coprime to 15)

**Step 2**: Find period of $f(x) = 7^x \mod 15$

| x | 7^x mod 15 |
|---|------------|
| 0 | 1          |
| 1 | 7          |
| 2 | 4          |
| 3 | 13         |
| 4 | 1  ← back to start! |

Period $r = 4$

**Step 3**: Compute factors
- $a^{r/2} = 7^2 = 49$
- $\gcd(49 - 1, 15) = \gcd(48, 15) = 3$ ✓
- $\gcd(49 + 1, 15) = \gcd(50, 15) = 5$ ✓

Found: $15 = 3 \times 5$ 🎉

---

## 3️⃣ The Quantum Part: Order Finding with QPE

### Why Period Finding Is Perfect for Quantum

The function $f(x) = a^x \mod N$ has a **hidden period** $r$:
$$f(x + r) = f(x) \quad \forall x$$

This periodicity creates interference patterns that QPE can extract!

### The Order-Finding Unitary

Define unitary operator $U_a$ acting on $|y\rangle$ (for $y < N$):
$$U_a |y\rangle = |ay \mod N\rangle$$

**Key property**: The states
$$|u_s\rangle = \frac{1}{\sqrt{r}} \sum_{k=0}^{r-1} e^{-2\pi i sk/r} |a^k \mod N\rangle$$

are eigenstates of $U_a$ with eigenvalues $e^{2\pi i s/r}$:
$$U_a |u_s\rangle = e^{2\pi i s/r} |u_s\rangle$$

### QPE for Order Finding

```
Counting Register (2n qubits)        Target Register (n qubits)
         |0⟩⊗2n                              |1⟩
           │                                  │
           H⊗2n                               │
           │                                  │
     ┌─────┴─────┐                           │
     │           │                           │
     ├── CU^1 ──────────────────────────────►│
     │           │                           │
     ├── CU^2 ──────────────────────────────►│
     │           │                           │
     ├── CU^4 ──────────────────────────────►│
     │    ...    │                           │
     └───────────┘                           │
           │                                  │
        QFT†                                  │
           │                                  │
      ┌────┴────┐                            │
      │ Measure │                            │
      └─────────┘                            │
           ↓
       m ≈ s·2^(2n)/r
```

### The Superposition Magic

When we input $|1\rangle$ to the target register (not an eigenstate), we implicitly create a superposition over all eigenstates $|u_s\rangle$:

$$|1\rangle = \frac{1}{\sqrt{r}} \sum_{s=0}^{r-1} |u_s\rangle$$

QPE then produces:
$$\frac{1}{\sqrt{r}} \sum_{s=0}^{r-1} |s/r\rangle |u_s\rangle$$

Measurement gives $s/r$ for some random $s$!

---

## 4️⃣ Extracting the Period: Continued Fractions

### The Problem

QPE gives us $m/2^{2n} \approx s/r$ for unknown $s$ and $r$.

How do we find $r$ from this noisy fraction?

### Continued Fractions Algorithm

Any real number $x$ has a unique continued fraction representation:
$$x = a_0 + \cfrac{1}{a_1 + \cfrac{1}{a_2 + \cfrac{1}{a_3 + \cdots}}}$$

**Key insight**: Truncating the continued fraction gives the **best rational approximation** with small denominator.

### Example: Extracting Period

Suppose QPE measures $m = 7$ with $2n = 4$ qubits (so $2^{2n} = 16$).

We get the fraction $\frac{7}{16}$.

Continued fraction expansion of $7/16$:
$$\frac{7}{16} = 0 + \cfrac{1}{2 + \cfrac{1}{3 + \cfrac{1}{2}}} = [0; 2, 3, 2]$$

Convergents:
- $[0] = 0$
- $[0; 2] = 1/2$
- $[0; 2, 3] = 3/7$  ← denominator 7 might be our period!
- $[0; 2, 3, 2] = 7/16$

If actual period $r = 7$, we'd verify: $a^7 \equiv 1 \mod N$? ✓

---

## 5️⃣ WHEN: Implementing Modular Exponentiation

### The Circuit Challenge

We need controlled versions of $U_a^{2^j}$ where:
$$U_a^{2^j}|y\rangle = |a^{2^j} \cdot y \mod N\rangle$$

This is **modular multiplication**, and it's the most expensive part of Shor's algorithm.

### Modular Multiplication Circuit

For $N = 15$, $a = 7$, we need:
- $U_7: |y\rangle \mapsto |7y \mod 15\rangle$
- $U_7^2: |y\rangle \mapsto |49y \mod 15\rangle = |4y \mod 15\rangle$
- $U_7^4: |y\rangle \mapsto |16y \mod 15\rangle = |y \mod 15\rangle$ (identity!)

```python
def controlled_modular_mult(a: int, power: int, N: int, n_qubits: int) -> QuantumCircuit:
    """
    Create controlled-U^(2^j) for modular multiplication.
    
    U|y⟩ = |a*y mod N⟩
    """
    # Compute a^(2^power) mod N
    a_power = pow(a, 2**power, N)
    
    # Build permutation circuit
    qc = QuantumCircuit(n_qubits, name=f'U_{a}^{{{2**power}}}')
    
    # Implementation depends on specific (a, N) values
    # For small N, can use lookup table approach
    # For large N, use reversible modular arithmetic circuits
    
    return qc
```

### Resource Requirements

For factoring $n$-bit number $N$:

| Resource | Count |
|----------|-------|
| Counting qubits | $2n$ |
| Target qubits | $n$ |
| Controlled-U operations | $2n$ |
| Total gates | $O(n^3)$ |
| Circuit depth | $O(n^3)$ |

For RSA-2048: ~4,000 logical qubits, millions of T-gates

---

## 6️⃣ Complete Algorithm Pseudocode

```
SHOR(N):
    # Classical preprocessing
    if N is even: return 2
    if N = a^b for some a, b > 1: return a
    
    repeat:
        # Step 1: Random base
        a = random(2, N-1)
        
        # Step 2: Easy case check
        d = gcd(a, N)
        if d > 1: return d  # Lucky!
        
        # Step 3: QUANTUM - Find period
        r = quantum_order_finding(a, N)
        
        # Step 4: Classical postprocessing
        if r is even:
            x = a^(r/2) mod N
            if x ≠ -1 mod N:
                p = gcd(x - 1, N)
                q = gcd(x + 1, N)
                if p * q = N:
                    return (p, q)
    
    # Repeat with new random a (prob > 1/2 per iteration)
```

---

## 7️⃣ 🚨 Trap Alerts

### Trap 1: Choosing Bad Values of $a$

❌ **Wrong**:
```python
a = N - 1  # a^2 ≡ 1 mod N always!
# Period r = 2, gives trivial factors
```

✅ **Correct**:
```python
# Choose random a, verify gcd(a, N) = 1
import random
from math import gcd

a = random.randint(2, N - 2)
while gcd(a, N) != 1:
    a = random.randint(2, N - 2)
```

### Trap 2: Insufficient Precision in QPE

❌ **Wrong**:
```python
n_counting = n  # Same as number of bits in N
# May not have enough precision to distinguish s/r
```

✅ **Correct**:
```python
n_counting = 2 * n + 1  # Extra precision for continued fractions
# Guarantees high success probability
```

### Trap 3: Forgetting Failure Cases

❌ **Wrong**:
```python
r = quantum_order_finding(a, N)
x = pow(a, r // 2, N)
p = gcd(x - 1, N)
q = gcd(x + 1, N)  # Assumes this works!
```

✅ **Correct**:
```python
r = quantum_order_finding(a, N)

# Check failure conditions
if r % 2 != 0:
    continue  # Odd period, try again
    
x = pow(a, r // 2, N)
if x == N - 1:  # x ≡ -1 mod N
    continue  # Trivial square root, try again
    
p = gcd(x - 1, N)
q = gcd(x + 1, N)

if p == 1 or p == N or q == 1 or q == N:
    continue  # Trivial factors, try again
```

### Trap 4: Circuit Depth for Modular Exponentiation

❌ **Wrong**:
```python
# Naive implementation: O(2^n) gates for U^(2^j)
for _ in range(2**j):
    apply_modular_mult(a, target_register)
```

✅ **Correct**:
```python
# Efficient: Compute a^(2^j) mod N classically, then apply once
a_power = pow(a, 2**j, N)
apply_modular_mult(a_power, target_register)
```

---

## 8️⃣ Memory Hook: PRIMES

**P**eriod finding is the core  
**R**andom a with gcd check first  
**I**nverse QFT decodes the phase  
**M**odular exponentiation is the hard part  
**E**xtract with continued fractions  
**S**uccess probability > 1/2 per trial  

---

## 9️⃣ Visual Summary

```
┌────────────────────────────────────────────────────────────────┐
│                SHOR'S ALGORITHM FLOW                           │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  INPUT: N to factor                                            │
│         ↓                                                      │
│  ┌─────────────────┐                                           │
│  │ Pick random a   │ ─── gcd(a,N) > 1? ──► DONE (lucky!)      │
│  │ 1 < a < N       │                                           │
│  └────────┬────────┘                                           │
│           ↓                                                    │
│  ╔═══════════════════════════════════════════════════╗        │
│  ║           QUANTUM ORDER FINDING                    ║        │
│  ║  ┌────────┐  ┌──────────────┐  ┌─────┐  ┌───────┐ ║        │
│  ║  │   H    │→ │  CU^(2^j)    │→ │QFT† │→ │Measure│ ║        │
│  ║  │ ⊗ 2n   │  │ on target    │  │     │  │       │ ║        │
│  ║  └────────┘  └──────────────┘  └─────┘  └───┬───┘ ║        │
│  ╚═══════════════════════════════════════════╪═══════╝        │
│                                              ↓                 │
│  ┌─────────────────────────────────┐        m                  │
│  │  Continued Fractions            │←───────┘                  │
│  │  m/2^(2n) → s/r                 │                           │
│  └────────────┬────────────────────┘                           │
│               ↓                                                │
│  ┌─────────────────────────────────┐                           │
│  │  Compute factors                │                           │
│  │  gcd(a^(r/2) ± 1, N)           │                           │
│  └────────────┬────────────────────┘                           │
│               ↓                                                │
│  OUTPUT: factors p, q                                          │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

---

## 🔟 Quick Check

### Q1: Why can't we use Shor's algorithm to break AES encryption?

<details>
<summary>Answer</summary>

AES is a symmetric cipher, not based on factoring or discrete logarithms. Shor's algorithm specifically solves:
1. Integer factorization
2. Discrete logarithm problem

For AES, Grover's algorithm provides only a quadratic speedup (doubling key length restores security).
</details>

### Q2: What's the success probability of one run of Shor's algorithm?

<details>
<summary>Answer</summary>

Greater than 1/2 for each iteration! The probability that:
- $r$ is even, AND
- $a^{r/2} \not\equiv -1 \mod N$

is at least $1 - 1/2^{k-1}$ where $k$ is the number of distinct prime factors of $N$.

For $N = pq$ (RSA modulus), probability ≥ 1/2.
</details>

### Q3: Why do we need $2n$ counting qubits instead of just $n$?

<details>
<summary>Answer</summary>

We need to estimate $s/r$ with enough precision to recover $r$ via continued fractions. With $2n$ qubits, we can distinguish between fractions with denominators up to $N$, giving high probability of extracting the correct period.
</details>

---

## 📚 References

1. Shor, P. W. (1994). "Algorithms for quantum computation: discrete logarithms and factoring"
2. Nielsen & Chuang, Chapter 5.3: "Applications: Order-Finding and Factoring"
3. Beauregard (2003). "Circuit for Shor's algorithm using 2n+3 qubits"

---

## 🔗 Next

**[Grover's Search Algorithm →](Module-07-Grovers-Search.md)****

Learn how quantum amplitude amplification achieves quadratic speedup for unstructured search problems.
