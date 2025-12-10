# Executable Graph Networks (EGN)
**Technical Reference & Architecture Specification**

## 1. Abstract
This document outlines the architecture of **Executable Graph Networks (EGN)**, a neuro-symbolic model designed to replace dense matrix operations with sparse, conditional logic execution. Unlike traditional Deep Neural Networks (DNNs) that utilize fixed topology and floating-point arithmetic ($O(N)$ complexity), EGN represents knowledge as a Directed Acyclic Graph (DAG) of discrete, CPU-native instructions.

During inference, EGN utilizes **conditional computation**, activating only a specific branch of the graph relevant to the input instance. This results in $O(\log N)$ computational complexity, orders of magnitude lower latency, and native interpretability. Preliminary benchmarks suggest EGN is particularly effective in algorithmic reasoning tasks  due to its inherent capability to synthesize logical control flows, including loops and conditional branching.

---

## 2. Core Architecture: The Logic Node
The fundamental atomic unit of an EGN is not a neuron, but a **Dual-Input Logic Cell**.

Mathematically, a single cell $C_i$ computes a decision boundary based on a linear projection of two inputs, enabling relational reasoning (gradients, differences, or sums):

```
{Activation} = (w_A \cdot x_A) + (w_B \cdot x_B)
```

The control flow is determined by:

$$\text{Next\_Node} = \begin{cases} \text{Child}_{\text{True}} & \text{if } \text{Activation} > \text{Threshold} \\ \text{Child}_{\text{False}} & \text{otherwise} \end{cases}$$

### 2.1 Memory Layout (C-Struct Specification)
The graph is serialized in memory as a flat array of structures, optimizing for L1/L2 cache locality and branch prediction on standard x86/ARM architectures.

```c
todo: Add example
```

## 3. Learning Methodology

Since the architecture relies on discrete branching (non-differentiable `IF/ELSE` logic) and integer arithmetic, standard Backpropagation is not directly applicable. EGN utilizes a multi-stage optimization strategy:

* **Structural Evolution (Exploratory Overgrowth):**
    During the initial training phase, the graph is allowed to expand dynamically, creating redundant nodes and branches. This "overgrowth" maximizes the search space, preventing the model from getting stuck in local minima while searching for complex logic patterns.

* **Discrete Parameter Optimization:**
    Algorithms designed for non-convex spaces (e.g., Evolutionary Strategies) are used to fine-tune the integer weights and thresholds ($w_a, w_b, T$) within the active nodes.

* **Graph Pruning & Collapsing (Final Optimization):**
    Once the logical solution is found, a separate optimization pass is applied to "collapse" the graph. This step involves:
    1.  **Dead Code Elimination:** Removing nodes that are never reached by valid inputs.
    2.  **Branch Merging:** Identifying and combining duplicate sub-trees (common subexpression elimination).
    3.  **Logic Simplification:** Converting complex chains of weak classifiers into fewer, stronger nodes.
