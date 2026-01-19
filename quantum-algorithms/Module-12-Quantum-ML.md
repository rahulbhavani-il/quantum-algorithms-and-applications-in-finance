# Quantum Neural Networks - Learning in Hilbert Space 🧠

> **When Variational Circuits Meet Machine Learning**
> **Difficulty**: 🔴 Advanced | **Time**: 4-5 hours | **Prerequisites**: VQE, QAOA
> **Codelab**: [12_qnn_codelab.ipynb](codelabs/12_qml_codelab.ipynb)

---

## 🎯 Learning Objectives

By the end of this section, you will be able to:

1. ✅ Understand the analogy between classical neural networks and parameterized quantum circuits
2. ✅ Design quantum feature maps for data encoding
3. ✅ Build variational classifier circuits
4. ✅ Implement gradient-based training for QNNs
5. ✅ Recognize the barren plateau problem and mitigation strategies

---

## 📋 Table of Contents

1. [WHY: The Quantum ML Promise](#why-the-quantum-ml-promise)
2. [WHAT: QNN Core Concepts](#what-qnn-core-concepts)
3. [Classical Neural Networks Refresher](#classical-neural-networks-refresher)
4. [Quantum Neural Network Architecture](#quantum-neural-network-architecture)
5. [Training QNNs](#training-qnns)
6. [Challenges and Limitations](#challenges-and-limitations)
7. [Trap Alerts](#trap-alerts)
8. [Quick Checks](#quick-checks)
9. [Summary](#summary)

---

## WHY: The Quantum ML Promise

### The Machine Learning Landscape

Machine learning has transformed industries:

| Domain | Application | Challenge |
|--------|-------------|-----------|
| **Vision** | Image classification | Billions of parameters |
| **NLP** | Language models | Massive compute costs |
| **Science** | Drug discovery | Exponential search space |
| **Finance** | Fraud detection | Real-time inference |

### The Classical Wall

Deep learning faces fundamental challenges:

1. **Training costs**: Large language model training can exceed $100M+
2. **Energy consumption**: Growing exponentially
3. **Expressibility limits**: Some functions require exponential parameters
4. **Data requirements**: Need massive labeled datasets

### The Quantum Opportunity

Quantum Neural Networks might offer:

| Potential Advantage | Mechanism |
|--------------------|-----------|
| **Exponential feature space** | 2^n amplitudes with n qubits |
| **Entanglement-based correlations** | Capture complex patterns |
| **Quantum kernel methods** | Efficient inner products |
| **Natural quantum data** | Direct processing of quantum states |

> **⚠️ Reality check**: Proven quantum advantages in ML are limited. This is an active research area with more questions than answers.

### WHEN to Consider QNNs

| ✅ Promising Applications | ❌ Probably Not Helpful |
|--------------------------|------------------------|
| Quantum chemistry data | Large image datasets |
| Quantum simulation outputs | Tabular classical data |
| Small, high-dimensional data | Tasks solved well classically |
| Kernel methods on quantum features | Require millions of parameters |

---

## WHAT: QNN Core Concepts

### 🔷 Definition (7-Element Framework)

A **Quantum Neural Network (QNN)** is a parameterized quantum circuit that:
1. **Encodes** classical data into quantum states (feature map)
2. **Processes** data through variational layers with trainable parameters
3. **Measures** outputs to produce predictions
4. **Optimizes** parameters using classical gradient descent

### 🎭 Analogy: The Quantum Perceptron

Classical neural network:
```
Input → [Weights × Input + Bias] → Activation → Output
        ↑                           ↑
    Parameters               Non-linearity
```

Quantum neural network:
```
|0⟩ → [Feature Map U(x)] → [Variational V(θ)] → Measurement → Output
       ↑                     ↑                    ↑
    Data encoding        Parameters         Non-linearity
```

**Key insight**: Measurement is the non-linearity in QNNs!

### 📐 Mathematical Foundation

#### The QNN Model

A QNN computes:
$$f_\theta(x) = \langle 0 | U^\dagger(x) V^\dagger(\theta) M V(\theta) U(x) | 0 \rangle$$

Where:
- $U(x)$: Feature map (encodes classical data $x$)
- $V(\theta)$: Variational ansatz (trainable parameters $\theta$)
- $M$: Measurement observable (e.g., $Z$ on one qubit)

#### Loss Function

For classification with labels $y \in \{-1, +1\}$:
$$\mathcal{L}(\theta) = \sum_i \ell(y_i, f_\theta(x_i))$$

Common choices:
- Mean squared error: $\ell = (y - f_\theta(x))^2$
- Hinge loss: $\ell = \max(0, 1 - y \cdot f_\theta(x))$

### 🖥️ QNN Circuit Structure

```
        ┌───────────────────┐ ┌───────────────────┐
|0⟩ ────┤                   ├─┤                   ├─── M
        │                   │ │                   │
|0⟩ ────┤   Feature Map     ├─┤   Variational     ├─── (discard)
        │     U(x)          │ │   Ansatz V(θ)     │
|0⟩ ────┤                   ├─┤                   ├─── (discard)
        │                   │ │                   │
|0⟩ ────┤                   ├─┤                   ├─── (discard)
        └───────────────────┘ └───────────────────┘
              ↑                      ↑
         Data x               Parameters θ
```

---

## Classical Neural Networks Refresher

### The Perceptron Model

A single neuron computes:
$$y = \sigma\left(\sum_j w_j x_j + b\right)$$

Where:
- $x_j$: Input features
- $w_j$: Learnable weights
- $b$: Bias term
- $\sigma$: Activation function (ReLU, sigmoid, etc.)

### Multi-Layer Networks

```
Input Layer     Hidden Layer(s)     Output Layer
    x₁  ─────────┐
                 ├─────→ h₁ ─────┐
    x₂  ─────────┤               ├─────→ y₁
                 ├─────→ h₂ ─────┤
    x₃  ─────────┤               ├─────→ y₂
                 └─────→ h₃ ─────┘
    
    W₁, b₁           W₂, b₂
```

Forward pass: $h = \sigma(W_1 x + b_1)$, $y = \sigma(W_2 h + b_2)$

### Training via Backpropagation

1. **Forward pass**: Compute predictions
2. **Loss calculation**: Compare to labels
3. **Backward pass**: Compute gradients $\partial \mathcal{L}/\partial \theta$
4. **Parameter update**: $\theta \leftarrow \theta - \eta \cdot \nabla_\theta \mathcal{L}$

**Key insight**: Gradients flow backward through differentiable operations.

---

## Quantum Neural Network Architecture

### Component 1: Feature Map (Data Encoding)

The feature map $U(x)$ embeds classical data into quantum states.

#### Angle Encoding

Each feature → rotation angle:
$$U(x) = \bigotimes_i R_Y(x_i)$$

```
|0⟩ ──Ry(x₁)──
|0⟩ ──Ry(x₂)──
|0⟩ ──Ry(x₃)──
```

**Pros**: Simple, uses n qubits for n features
**Cons**: Limited expressibility

#### Amplitude Encoding

Encode $2^n$ features in $n$ qubits:
$$|x\rangle = \frac{1}{||x||}\sum_i x_i |i\rangle$$

**Pros**: Exponentially efficient
**Cons**: Expensive state preparation (O(2^n) gates generally)

#### ZZ Feature Map (Hardware-Efficient)

Combination of rotations and entanglement:
$$U(x) = \left[\prod_{j<k} e^{i x_j x_k Z_j Z_k}\right] \cdot \left[\bigotimes_i R_Z(x_i) R_Y(x_i)\right]$$

```python
def zz_feature_map(qc, x, reps=2):
    """
    ZZ Feature Map for data encoding.
    
    Parameters:
        qc: QuantumCircuit
        x: Input features (array)
        reps: Number of repetitions
    """
    n = len(x)
    
    for _ in range(reps):
        # Single-qubit rotations
        for i in range(n):
            qc.h(i)
            qc.rz(2 * x[i], i)
        
        # ZZ entanglement
        for i in range(n):
            for j in range(i + 1, n):
                qc.cx(i, j)
                qc.rz(2 * x[i] * x[j], j)
                qc.cx(i, j)
```

### Component 2: Variational Ansatz

The trainable part of the circuit:

#### Hardware-Efficient Ansatz

```
Layer 1                 Layer 2
┌───┐┌─────────┐       ┌───┐┌─────────┐
┤Ry ├┤ Rz(θ₁)  ├──●────┤Ry ├┤ Rz(θ₅)  ├──●────
├───┤├─────────┤  │    ├───┤├─────────┤  │
┤Ry ├┤ Rz(θ₂)  ├──X──●─┤Ry ├┤ Rz(θ₆)  ├──X──●─
├───┤├─────────┤     │ ├───┤├─────────┤     │
┤Ry ├┤ Rz(θ₃)  ├─────X─┤Ry ├┤ Rz(θ₇)  ├─────X─
└───┘└─────────┘       └───┘└─────────┘
```

```python
def variational_layer(qc, params, n_qubits):
    """
    Single variational layer.
    
    Parameters:
        qc: QuantumCircuit
        params: Array of 2*n_qubits parameters
        n_qubits: Number of qubits
    """
    # Rotation layer
    for i in range(n_qubits):
        qc.ry(params[2*i], i)
        qc.rz(params[2*i + 1], i)
    
    # Entanglement layer (linear connectivity)
    for i in range(n_qubits - 1):
        qc.cx(i, i + 1)
```

### Component 3: Measurement

Extract classical output:

$$\langle M \rangle = \langle \psi | M | \psi \rangle$$

For binary classification, measure $Z$ on first qubit:
- $\langle Z \rangle > 0$: Class +1
- $\langle Z \rangle < 0$: Class -1

---

## Training QNNs

### The Parameter Shift Rule

Quantum circuits are differentiable! For gates $R_\alpha(\theta) = e^{-i\theta G/2}$:

$$\frac{\partial f}{\partial \theta} = \frac{f(\theta + \pi/2) - f(\theta - \pi/2)}{2}$$

This is **exact** (not finite difference approximation)!

### Training Loop

```python
def train_qnn(X_train, y_train, n_qubits, n_layers, 
              learning_rate=0.1, epochs=100):
    """
    Train a QNN classifier.
    """
    # Initialize parameters
    n_params = 2 * n_qubits * n_layers
    params = np.random.uniform(0, 2*np.pi, n_params)
    
    for epoch in range(epochs):
        total_loss = 0
        gradients = np.zeros_like(params)
        
        for x, y in zip(X_train, y_train):
            # Forward pass
            prediction = qnn_forward(x, params, n_qubits, n_layers)
            
            # Loss
            loss = (y - prediction)**2
            total_loss += loss
            
            # Compute gradients via parameter shift
            for i in range(len(params)):
                params_plus = params.copy()
                params_plus[i] += np.pi/2
                params_minus = params.copy()
                params_minus[i] -= np.pi/2
                
                f_plus = qnn_forward(x, params_plus, n_qubits, n_layers)
                f_minus = qnn_forward(x, params_minus, n_qubits, n_layers)
                
                df_dtheta = (f_plus - f_minus) / 2
                gradients[i] += -2 * (y - prediction) * df_dtheta
        
        # Update parameters
        params -= learning_rate * gradients / len(X_train)
        
        if epoch % 10 == 0:
            print(f"Epoch {epoch}: Loss = {total_loss/len(X_train):.4f}")
    
    return params
```

### Gradient Computation Cost

| Method | Circuit Evaluations per Gradient |
|--------|----------------------------------|
| Parameter Shift | $2 \times$ (number of parameters) |
| Finite Difference | $2 \times$ (number of parameters) |
| Stochastic | $O(1)$ but higher variance |

---

## Challenges and Limitations

### Challenge 1: Barren Plateaus 🏜️

**Problem**: Random initialization leads to vanishing gradients.

For deep random circuits:
$$\text{Var}\left[\frac{\partial f}{\partial \theta}\right] \sim \frac{1}{2^n}$$

Gradients exponentially small in number of qubits!

**Mitigations**:
1. **Shallow circuits**: Fewer layers
2. **Local cost functions**: Measure few qubits
3. **Smart initialization**: Pre-training, layer-wise training
4. **Architecture design**: Symmetry-respecting ansätze

### Challenge 2: Noise

NISQ devices have errors:

| Error Type | Impact on QNN |
|------------|---------------|
| **Gate errors** | Reduce fidelity, distort gradients |
| **Readout errors** | Bias predictions |
| **Decoherence** | Limit circuit depth |

**Mitigations**:
1. Error mitigation techniques (ZNE, PEC)
2. Hardware-efficient ansätze
3. Noise-aware training

### Challenge 3: Data Loading

Classical data → quantum state is expensive:

| Encoding | Qubits | Gates | Advantage Loss? |
|----------|--------|-------|-----------------|
| Angle | n | O(n) | Minimal |
| Amplitude | log(n) | O(n) | Yes (for general data) |

**Rule of thumb**: If your data is purely classical, the encoding cost may eliminate any quantum advantage.

### Challenge 4: Expressibility vs Trainability

```
                    Expressibility
                         ↑
    Deep random      ●───────────── High
    circuits              |
                          |  ← Barren plateau zone
                          |
    Shallow          ●───────────── Low
    circuits              |
                     trainable    hard to train
                         
                    ←───────────────→
                       Trainability
```

**Tradeoff**: More expressive circuits are harder to train!

---

## Trap Alerts 🚨

### Trap 1: Claiming Quantum Advantage Prematurely
```python
# ❌ WRONG THINKING:
# "My QNN achieves 95% accuracy, therefore quantum advantage!"

# ✅ CORRECT: Compare fairly
# 1. Same data, same train/test split
# 2. Classical model with similar parameter count
# 3. Account for encoding cost
# 4. Check if simple kernel methods work
```

### Trap 2: Ignoring Barren Plateaus
```python
# ❌ WRONG: Deep random circuit, many qubits
def bad_qnn(n_qubits=20, n_layers=50):
    # Gradients will be ~0 everywhere!
    pass

# ✅ CORRECT: Start shallow, monitor gradients
def good_qnn(n_qubits=4, n_layers=3):
    # Verify gradients are non-vanishing
    pass
```

### Trap 3: Over-parameterization
```python
# ❌ WRONG: More parameters than training data
n_params = 100
n_training_samples = 50  # Overfitting guaranteed!

# ✅ CORRECT: Balance parameters with data
# Rule of thumb: n_params < n_samples / 10
```

### Trap 4: Forgetting Shot Noise
```python
# ❌ WRONG: Expecting perfect expectation values
prediction = measure_expectation(circuit)  # Actually has variance!

# ✅ CORRECT: Account for statistical noise
# Variance ~ 1/shots
# Use enough shots for reliable gradients
shots = max(1024, 100 * len(params))  # Scale with parameters
```

### Trap 5: Ignoring Classical Benchmarks
```python
# ❌ WRONG: "QNN works on my data!"
# Did you try a simple classical model first?

# ✅ CORRECT: Always compare
from sklearn.svm import SVC
classical_accuracy = SVC(kernel='rbf').fit(X_train, y_train).score(X_test, y_test)
# If classical_accuracy > qnn_accuracy, no quantum benefit!
```

---

## Mnemonic: QUILT 🧵

Remember QNN design with **QUILT**:

| Letter | Component | Key Question |
|--------|-----------|--------------|
| **Q** | **Qubit count** | Enough for data, not too many (barren plateaus) |
| **U** | **U(x) feature map** | How is data encoded? Efficient? |
| **I** | **Initialization** | Random? Structured? Pre-trained? |
| **L** | **Layers** | How deep? Expressibility vs trainability |
| **T** | **Training** | Parameter shift? Enough shots? Converging? |

---

## Quick Checks ✅

### 🟢 Basic Understanding

1. **What is the "non-linearity" in a quantum neural network?**
   <details>
   <summary>Answer</summary>
   Measurement! While unitary operations are linear, the act of measurement (and the Born rule squaring amplitudes) introduces non-linearity. Also, the expectation value ⟨ψ|M|ψ⟩ depends quadratically on the state amplitudes.
   </details>

2. **What is the parameter shift rule?**
   <details>
   <summary>Answer</summary>
   A method to compute exact gradients of quantum circuits. For rotation gates, ∂f/∂θ = (f(θ+π/2) - f(θ-π/2))/2. This requires only 2 circuit evaluations per parameter, matching finite difference but being exact rather than approximate.
   </details>

### 🟡 Intermediate Application

3. **Why might a QNN with 20 qubits and 50 layers fail to train?**
   <details>
   <summary>Answer</summary>
   Barren plateaus! For deep random circuits, gradient variance decays as ~1/2^n. With 20 qubits, gradients would be ~10^-6, effectively zero for any practical training. The optimizer sees a flat landscape and can't find the minimum.
   </details>

4. **A QNN achieves 90% accuracy but a classical SVM achieves 92% on the same data. Is there quantum advantage?**
   <details>
   <summary>Answer</summary>
   No! Quantum advantage requires the quantum method to outperform the best classical method (or match it with fewer resources). If a simple SVM beats the QNN, there's no advantage. Also consider the encoding cost and training time.
   </details>

### 🔴 Advanced Analysis

5. **How does entanglement help in QNNs?**
   <details>
   <summary>Answer</summary>
   Entanglement allows QNNs to represent correlations that would require exponentially many parameters classically. For example, a product state ansatz can only capture classical correlations, while entangled ansätze can represent quantum correlations. This is related to "quantum feature maps" accessing Hilbert spaces inaccessible to classical kernels.
   </details>

6. **What's the relationship between QNNs and quantum kernel methods?**
   <details>
   <summary>Answer</summary>
   They're closely related! A QNN computes f(x) = ⟨0|U†(x)V†(θ)M V(θ)U(x)|0⟩. If we fix θ, this becomes a quantum kernel method: K(x,x') = |⟨φ(x)|φ(x')⟩|². The feature map U(x) creates the kernel implicitly. QNNs add trainable layers V(θ), making them more flexible but harder to analyze.
   </details>

---

## Practical Considerations

### When to Use QNNs

| Criterion | Favorable | Unfavorable |
|-----------|-----------|-------------|
| Data size | Small (< 1000 samples) | Large (millions) |
| Data type | Quantum states, high-D | Images, text |
| Classical baseline | Hard to beat | Easy 95%+ accuracy |
| Hardware access | Good NISQ device | Simulation only |
| Research goal | Explore quantum ML | Production system |

### Recommended Starting Point

1. **Small dataset** (n < 100 samples, d < 10 features)
2. **Few qubits** (4-8 qubits)
3. **Shallow circuit** (2-4 layers)
4. **Simple feature map** (angle encoding)
5. **Compare to classical SVM/kernel methods**

---

## Summary

### Key Takeaways

1. **QNN = Feature Map + Variational Ansatz + Measurement**
   - U(x) encodes data, V(θ) is trainable, measurement extracts predictions

2. **Parameter shift gives exact gradients**
   - Evaluate circuit at θ ± π/2, take difference

3. **Barren plateaus are the main enemy**
   - Deep circuits + many qubits = vanishing gradients
   - Stay shallow, start small

4. **Quantum advantage is NOT guaranteed**
   - Always compare to classical baselines
   - Encoding cost may eliminate speedup

5. **This is active research**
   - Few proven advantages exist
   - Exciting area but proceed with scientific rigor

### What's Next?

- **Codelab**: Build and train a QNN in [12_qml_codelab.ipynb](codelabs/12_qml_codelab.ipynb)
- **Extensions**: Quantum kernel methods, QNN for quantum chemistry
- **Research**: Explore provable quantum advantages

---

## References

1. Schuld, M., & Petruccione, F. (2021). "Machine Learning with Quantum Computers"
2. Cerezo, M., et al. (2021). "Variational Quantum Algorithms" Nature Reviews Physics
3. McClean, J. R., et al. (2018). "Barren plateaus in quantum neural network training landscapes"
4. Havlíček, V., et al. (2019). "Supervised learning with quantum-enhanced feature spaces"
5. Huang, H. Y., et al. (2021). "Power of data in quantum machine learning"

---

*Part of the Interactive Quantum Computing Learning Series*
