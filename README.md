# Quantum Algorithms & Applications in Finance 🔬💰

> **A comprehensive learning resource for quantum computing algorithms and their real-world applications in finance**

[![Qiskit](https://img.shields.io/badge/Qiskit-2.x-blueviolet)](https://qiskit.org/)
[![Python](https://img.shields.io/badge/Python-3.10+-blue)](https://python.org/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

---

## 📖 Overview

This repository bridges the gap between **theoretical quantum algorithms** and **practical financial applications**. It provides:

1. **Quantum Algorithms** — 12 comprehensive modules covering foundational to advanced quantum algorithms with theory (Markdown) and hands-on codelabs (Jupyter notebooks)
2. **Quantum Finance Applications** — Real-world financial use cases demonstrating how quantum algorithms solve complex problems in portfolio optimization, option pricing, and credit risk classification

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    QUANTUM ALGORITHMS & FINANCE                             │
├─────────────────────────────────┬───────────────────────────────────────────┤
│       QUANTUM ALGORITHMS        │         FINANCE APPLICATIONS              │
│         (12 Modules)            │           (3 Projects)                    │
├─────────────────────────────────┼───────────────────────────────────────────┤
│  🔹 Deutsch-Jozsa               │                                           │
│  🔹 Bernstein-Vazirani          │                                           │
│  🔹 Simon's Algorithm           │                                           │
│  🔹 Quantum Fourier Transform ──┼──▶ QAE Option Pricing                     │
│  🔹 Quantum Phase Estimation ───┼──▶ QAE Option Pricing                     │
│  🔹 Shor's Algorithm            │                                           │
│  🔹 Grover's Search ────────────┼──▶ VQC Credit Classifier                  │
│  🔹 VQE Quantum Chemistry       │                                           │
│  🔹 Grover's Applications ──────┼──▶ All Finance Applications               │
│  🔹 QAOA Advanced ──────────────┼──▶ Portfolio Optimization                 │
│  🔹 HHL Algorithm               │                                           │
│  🔹 Quantum ML ─────────────────┼──▶ VQC Credit Classifier                  │
└─────────────────────────────────┴───────────────────────────────────────────┘
```

---

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone <repository-url>
cd quantum-algorithms-and-applications-in-finance
```

### 2. Set Up Environment
```bash
# Create virtual environment
python -m venv quantum-env

# Activate (macOS/Linux)
source quantum-env/bin/activate

# Activate (Windows)
quantum-env\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### 3. Verify Installation
```bash
python -c "import qiskit; print(f'Qiskit version: {qiskit.__version__}')"
```

### 4. Start Learning!
```bash
# Option A: Start with algorithm fundamentals
jupyter lab quantum-algorithms/codelabs/

# Option B: Jump into finance applications
jupyter lab quantum_finance_applications/
```

---

## 📂 Repository Structure

```
quantum-algorithms-and-applications-in-finance/
│
├── README.md                          # This file
├── requirements.txt                   # Python dependencies
│
├── quantum-algorithms/                # 📚 Algorithm Theory & Codelabs
│   ├── Module-01-Deutsch-Jozsa.md     # Theory modules (12 total)
│   ├── Module-02-Bernstein-Vazirani.md
│   ├── ...
│   ├── Module-12-Quantum-ML.md
│   └── codelabs/                      # Hands-on Jupyter notebooks
│       ├── 01_deutsch_jozsa_codelab.ipynb
│       ├── 02_bernstein_vazirani_codelab.ipynb
│       └── ... (12 codelabs total)
│
└── quantum_finance_applications/      # 💰 Finance Applications
    ├── 01_qaoa_portfolio_optimization.ipynb
    ├── 02_qae_option_pricing.ipynb
    └── 03_vqc_credit_classifier.ipynb
```

---

## 🎓 Learning Path

### Recommended Learning Flow

```
                        ┌─────────────────────────┐
                        │   START HERE            │
                        │   Quantum Algorithms    │
                        │   Fundamentals          │
                        └───────────┬─────────────┘
                                    │
        ┌───────────────────────────┼───────────────────────────┐
        │                           │                           │
        ▼                           ▼                           ▼
┌───────────────┐         ┌───────────────┐         ┌───────────────┐
│   TRACK 1     │         │   TRACK 2     │         │   TRACK 3     │
│   Foundations │         │   Cryptography│         │   NISQ/VQA    │
│   (8-10 wks)  │         │   (4 weeks)   │         │   (4 weeks)   │
└───────┬───────┘         └───────┬───────┘         └───────┬───────┘
        │                         │                         │
        └─────────────────────────┼─────────────────────────┘
                                  │
                                  ▼
                    ┌─────────────────────────┐
                    │   FINANCE APPLICATIONS  │
                    │   Apply Your Knowledge  │
                    └─────────────────────────┘
```

---

## 📚 Part 1: Quantum Algorithms

### Algorithm Modules

| Module | Algorithm | Difficulty | Time | Finance Connection |
|--------|-----------|------------|------|-------------------|
| **01** | [Deutsch-Jozsa](quantum-algorithms/Module-01-Deutsch-Jozsa.md) | 🟢 Beginner | 3-4 hrs | Oracle foundations |
| **02** | [Bernstein-Vazirani](quantum-algorithms/Module-02-Bernstein-Vazirani.md) | 🟢 Beginner | 2-3 hrs | Hidden pattern finding |
| **03** | [Simon's Algorithm](quantum-algorithms/Module-03-Simons-Algorithm.md) | 🟡 Intermediate | 3-4 hrs | Period finding concepts |
| **04** | [Quantum Fourier Transform](quantum-algorithms/Module-04-Quantum-Fourier-Transform.md) | 🟡 Intermediate | 4-5 hrs | **→ Option Pricing (QAE)** |
| **05** | [Quantum Phase Estimation](quantum-algorithms/Module-05-Quantum-Phase-Estimation.md) | 🟡 Intermediate | 4-5 hrs | **→ Option Pricing (QAE)** |
| **06** | [Shor's Algorithm](quantum-algorithms/Module-06-Shors-Algorithm.md) | 🔴 Advanced | 5-6 hrs | Cryptographic implications |
| **07** | [Grover's Search](quantum-algorithms/Module-07-Grovers-Search.md) | 🟡 Intermediate | 4-5 hrs | **→ Credit Classification** |
| **08** | [VQE Quantum Chemistry](quantum-algorithms/Module-08-VQE-Quantum-Chemistry.md) | 🔴 Advanced | 5-6 hrs | Variational methods |
| **09** | [Grover's Applications](quantum-algorithms/Module-09-Grovers-Applications.md) | 🟡 Intermediate | 3-4 hrs | **→ All Finance Apps** |
| **10** | [QAOA Advanced](quantum-algorithms/Module-10-QAOA-Advanced.md) | 🔴 Advanced | 4-5 hrs | **→ Portfolio Optimization** |
| **11** | [HHL Algorithm](quantum-algorithms/Module-11-HHL-Algorithm.md) | 🔴 Advanced | 4-5 hrs | Linear systems in finance |
| **12** | [Quantum ML](quantum-algorithms/Module-12-Quantum-ML.md) | 🔴 Advanced | 4-5 hrs | **→ Credit Classification** |

**Total Learning Time**: ~45-55 hours for complete mastery

### Codelab Structure

Each algorithm module includes a corresponding Jupyter notebook codelab with:

1. **Theory Recap** — Key equations and circuit diagrams
2. **Basic Implementation** — Step-by-step construction
3. **Intermediate Implementation** — Parameterized versions
4. **Advanced Implementation** — Hardware-aware optimizations
5. **Visualization Lab** — Plots and diagrams
6. **Trap Demonstrations** — Common mistakes with fixes
7. **Exercises & Challenges** — Graded difficulty levels

---

## 💰 Part 2: Quantum Finance Applications

### Finance Projects Overview

| Project | Algorithm | Problem | Quantum Advantage |
|---------|-----------|---------|-------------------|
| **01** | QAOA | Portfolio Optimization | Explores 2^N portfolios simultaneously |
| **02** | QAE | Option Pricing | Quadratic speedup over Monte Carlo |
| **03** | VQC | Credit Classification | Exponential feature space access |

---

### Project 1: QAOA Portfolio Optimization

**Notebook**: [01_qaoa_portfolio_optimization.ipynb](quantum_finance_applications/01_qaoa_portfolio_optimization.ipynb)

| Aspect | Details |
|--------|---------|
| **Problem** | Select optimal k assets from n candidates |
| **Classical Complexity** | O(2^N) — Exponential |
| **Quantum Complexity** | O(poly(N)) — Polynomial |
| **Algorithm** | QAOA (Quantum Approximate Optimization Algorithm) |
| **Quantum Advantage** | Superposition explores all portfolios in parallel |
| **Expected Result** | Approximation ratio > 0.90 |

**Prerequisites**: Module 10 (QAOA Advanced)

---

### Project 2: QAE Option Pricing

**Notebook**: [02_qae_option_pricing.ipynb](quantum_finance_applications/02_qae_option_pricing.ipynb)

| Aspect | Details |
|--------|---------|
| **Problem** | Price derivatives faster than Monte Carlo |
| **Classical Complexity** | O(1/ε²) samples needed |
| **Quantum Complexity** | O(1/ε) samples needed |
| **Algorithm** | QAE (Quantum Amplitude Estimation) |
| **Quantum Advantage** | Quadratic reduction in sample complexity |
| **Expected Result** | √N speedup factor |

**Prerequisites**: Module 04 (QFT) + Module 05 (QPE)

---

### Project 3: VQC Credit Classifier

**Notebook**: [03_vqc_credit_classifier.ipynb](quantum_finance_applications/03_vqc_credit_classifier.ipynb)

| Aspect | Details |
|--------|---------|
| **Problem** | Classify loan applicants by risk level |
| **Classical Complexity** | O(N_features) |
| **Quantum Complexity** | O(2^N_features) feature space |
| **Algorithm** | VQC (Variational Quantum Classifier) |
| **Quantum Advantage** | Access to 2^N dimensional Hilbert space |
| **Expected Result** | Test accuracy 85-92% |

**Prerequisites**: Module 07 (Grover) + Module 12 (Quantum ML)

---

## 🎯 Study Tracks

### Track A: Complete Foundations → Finance (12 weeks)

For learners who want comprehensive understanding before applications.

| Weeks | Focus | Modules/Projects |
|-------|-------|-----------------|
| 1-2 | Query Algorithms | Modules 01-03 |
| 3-4 | QFT & Phase Estimation | Modules 04-05 |
| 5-6 | Grover's Algorithm | Modules 07, 09 |
| 7-8 | Variational Algorithms | Modules 08, 10 |
| 9-10 | Advanced Topics | Modules 11-12 |
| 11-12 | **Finance Applications** | All 3 Projects |

### Track B: Fast-Track to Finance (6 weeks)

For practitioners who want to quickly apply quantum algorithms to finance.

| Weeks | Focus | Modules/Projects |
|-------|-------|-----------------|
| 1 | QAOA Foundations | Module 10 |
| 2 | **Portfolio Optimization** | Project 01 |
| 3 | QFT + QPE | Modules 04-05 |
| 4 | **Option Pricing** | Project 02 |
| 5 | Grover + QML | Modules 07, 12 |
| 6 | **Credit Classification** | Project 03 |

### Track C: Algorithm Deep Dive (10 weeks)

For researchers focusing on algorithmic understanding.

| Weeks | Focus | Modules |
|-------|-------|---------|
| 1-2 | Query Complexity | Modules 01-03 |
| 3-4 | Fourier Analysis | Modules 04-06 |
| 5-6 | Search & Optimization | Modules 07, 09 |
| 7-8 | Variational Methods | Modules 08, 10 |
| 9-10 | Advanced Algorithms | Modules 11-12 |

---

## 📦 Dependencies

### Core Requirements

```
qiskit>=2.0.0,<3.0.0      # Core quantum computing
qiskit-aer>=0.14.0         # Quantum simulator
qiskit-ibm-runtime>=0.30.0 # IBM Quantum hardware access
```

### Scientific & Finance

```
numpy>=1.24.0              # Numerical computing
scipy>=1.10.0              # Scientific computing
matplotlib>=3.7.0          # Visualization
pandas>=2.0                # Data manipulation
yfinance                   # Financial data
scikit-learn               # Machine learning
seaborn>=0.12.0            # Statistical visualization
```

### Development Tools

```
jupyter>=1.0.0             # Notebook interface
notebook>=7.0.0            # Jupyter notebook
pytest>=7.4.0              # Testing framework
```

Install all dependencies:
```bash
pip install -r requirements.txt
```

---

## 💻 Running the Materials

### Option 1: VS Code with Jupyter Extension
```bash
# Open any notebook
code quantum-algorithms/codelabs/01_deutsch_jozsa_codelab.ipynb
code quantum_finance_applications/01_qaoa_portfolio_optimization.ipynb
```

### Option 2: Jupyter Lab
```bash
# Launch Jupyter Lab
jupyter lab

# Navigate to desired notebook in the browser
```

### Option 3: Google Colab
Upload notebooks to Google Colab and install dependencies:
```python
!pip install qiskit qiskit-aer qiskit-algorithms qiskit-optimization qiskit-machine-learning qiskit-finance
```

---

## 🔗 Algorithm → Finance Mapping

Understanding which algorithms power which financial applications:

```yaml
QAOA (Module 10):
  → Portfolio Optimization
  → Mechanism: Superposition explores all asset combinations
  → Advantage: Polynomial vs exponential classical

QAE (Module 05):
  → Option Pricing
  → Mechanism: Amplitude estimation of expected payoff
  → Advantage: Quadratic speedup over Monte Carlo

VQC (Module 12):
  → Credit Classification
  → Mechanism: Quantum kernel in high-dimensional space
  → Advantage: Exponential feature space access

Grover (Modules 07, 09):
  → Supports all finance applications
  → Mechanism: Quadratic search speedup
  → Advantage: Faster constraint satisfaction
```

---

## 📊 Learning Objectives

By completing this repository, you will:

### Algorithm Understanding
- [ ] Implement quantum algorithms from scratch
- [ ] Analyze complexity (query, time, space)
- [ ] Run algorithms on simulators and IBM Quantum
- [ ] Debug algorithm failures systematically
- [ ] Explain WHY-WHAT-WHEN for each algorithm

### Finance Applications
- [ ] Formulate QUBO problems for portfolio optimization
- [ ] Implement amplitude encoding for probability distributions
- [ ] Design variational ansätze for classification
- [ ] Compare classical vs quantum approaches
- [ ] Interpret quantum advantage in financial contexts

---

## 📚 References

### Textbooks
- Nielsen & Chuang: "Quantum Computation and Quantum Information"
- Kaye, Laflamme, Mosca: "An Introduction to Quantum Computing"

### Qiskit Resources
- [Qiskit Textbook](https://qiskit.org/textbook/)
- [Qiskit Finance](https://qiskit-community.github.io/qiskit-finance/)
- [Qiskit Machine Learning](https://qiskit-community.github.io/qiskit-machine-learning/)

### Original Algorithm Papers
- Referenced in individual module markdown files

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit issues or pull requests for:
- Bug fixes in notebooks
- Additional finance applications
- Improved explanations
- New algorithm implementations

---

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🚀 Ready to Start?

**New to quantum computing?**
→ Begin with [Module 01: Deutsch-Jozsa](quantum-algorithms/Module-01-Deutsch-Jozsa.md)

**Want to dive into finance?**
→ Start with [QAOA Portfolio Optimization](quantum_finance_applications/01_qaoa_portfolio_optimization.ipynb)

**Already familiar with algorithms?**
→ Review the [Finance Applications README](quantum_finance_applications/README.md)

---

*Happy quantum learning! 🎉*
