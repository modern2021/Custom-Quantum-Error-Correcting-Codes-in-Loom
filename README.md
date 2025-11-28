# 🧬 Custom Quantum Error-Correcting Codes in Loom
### Implementing the **[[8,3,2]] smallest interesting colour code** and the **[[16,6,4]] tesseract colour code**

---

## 🚀 Overview

This project implements **two non-factory quantum error-correcting codes** from scratch using **Loom**:

1. **[[8,3,2]] smallest interesting colour code**
2. **[[16,6,4]] tesseract colour code**

Instead of relying on existing code factories, we build stabilizers and logical operators manually, embed their geometry, validate commutation, visualize on the lattice, and run simple logical memory experiments.

This demonstrates how **research-grade QEC development** can be done in Loom for **custom stabilizer codes**.

---

# 📘 Background: CSS Stabilizer Codes

A stabilizer code is defined by an abelian subgroup of the Pauli group:

> All valid codewords are simultaneous +1 eigenstates of the stabilizers.

For **CSS codes**:
- Stabilizers split into **X-type** and **Z-type** checks.
- This makes error diagnosis and circuit construction simpler.

The number of encoded qubits is:

\[
k = n - \text{rank}(\mathcal{S})
\]

Where:
- \( n \) = physical qubits
- \( \text{rank}(\mathcal{S}) \) = number of independent stabilizers

---

# 🔷 1. [[8,3,2]] — Smallest Interesting Colour Code

**Code Parameters**

| Symbol | Meaning |
|------|--------|
| n = 8 | physical qubits |
| k = 3 | logical qubits |
| d = 2 | distance |

This is the smallest non-trivial colour code: very dense encoding (3 logical qubits in 8 physical qubits).

### 📐 Geometry
0 1 2 3
4 5 6 7
Each point is `(x, y, basis=0)` in Loom.

---

## Stabilizers

### X stabilizer (weight 8)
Acts on all data qubits:
XXXXXXXX

### Z stabilizers (4 of them)
Horizontal:
- Z0 Z1 Z2 Z3
- Z4 Z5 Z6 Z7

Vertical:
- Z0 Z1 Z4 Z5
- Z1 Z3 Z5 Z7

These generate a rank-5 stabilizer group → **k = 8 − 5 = 3 logical qubits**

---

## Logical Operators

We computed a valid logical basis via **symplectic solving**:

### Logical X

X̄1 = X4 X5 X6 X7
X̄2 = X2 X3 X6 X7
X̄3 = X1 X3 X5 X7

### Logical Z
Z̄1 = Z3 Z7
Z̄2 = Z5 Z7
Z̄3 = Z6 Z7

✔ X̄i commutes with all stabilizers  
✔ Z̄i commutes with all stabilizers  
✔ {X̄i, Z̄i} = 0 (anticommute)  
✔ All others commute

---

# 🟪 2. [[16,6,4]] — Tesseract Colour Code

A 4D colour code defined on a hypercube.  
Stabilizers live on **3D facets of the 4D cube**.

We use a flattened **4×4 embedding**:

0 1 2 3
4 5 6 7
8 9 10 11
12 13 14 15

---

## Stabilizers

This code is **self-dual CSS**:

- **5 X stabilizers**
- **5 Z stabilizers**
- Each of **weight 8**, corresponding to row/column pairs

Example X stabilizers:
R0⊕R1 → qubits 0..7
R1⊕R2 → qubits 4..11
R2⊕R3 → qubits 8..15
C0⊕C1
C1⊕C2


Same supports for Z stabilizers.

Since:
→ **k = 6 logical qubits**

---

## Logical Operators

Computed via exact symplectic linear algebra.

We generate **6 pairs (X̄i, Z̄i)** such that:

- X̄i commutes with all stabilizers
- Z̄i commutes with all stabilizers
- {X̄i, Z̄i} = 0
- X̄i commutes with Z̄j (i ≠ j)

These were implemented directly as `PauliOperator` objects.

---

# 🧩 Loom Implementation Workflow

### 1️⃣ Lattice Construction

```python
lattice = Lattice.square_2d((dx, dy))
2️⃣ Define coordinates

data_qubit(i) → (x, y, 0)

ancilla_qubit(j) → (x, y, 1)
Stabilizer(
    pauli="XXXX",
    data_qubits=[...],
    ancilla_qubits=[...],
)

PauliOperator(
    pauli="XZ...",
    data_qubits=[...],
)
block = Block(
    unique_label="q832",
    stabilizers=stabilizers,
    logical_x_operators=logical_X_ops,
    logical_z_operators=logical_Z_ops
)
stab_plot = StabilizerPlot(lattice)
stab_plot.add_dqubit_traces(mask)
stab_plot.plot_blocks([block])
stab_plot.show()
operations = [
   [ResetAllDataQubits("q832", state="0")],
   [MeasureLogicalZ("q832")],
]
eka = Eka(lattice=lattice, blocks=[color_832_block], operations=operations)
final_circuit = interpret_eka(eka).final_circuit


We use a **4×2 grid**:

