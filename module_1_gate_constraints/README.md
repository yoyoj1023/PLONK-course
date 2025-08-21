# Module 1: Minimal Components - The Mystery of a Single Gate (The Gate Constraint)

## Module Objective
In-depth analysis of PLONK's core constraint formula, understanding how it becomes a configurable "arithmetic component."

## Mental Model
Understand the formula as a programmable Lego block.

---

## Lesson 1: Dissecting the PLONK Arithmetization Formula

### 1.1 PLONK's Core Formula

```
q_L * w_a + q_R * w_b + q_O * w_c + q_M * (w_a * w_b) + q_C = 0
```

This seemingly simple formula is actually the heart of the PLONK protocol. Let's analyze each component:

### 1.2 Wires - Data Carriers

**w_a, w_b, w_c** represent three "wires":
- **w_a**: Left input wire
- **w_b**: Right input wire  
- **w_c**: Output wire

**Intuitive Understanding**:
Imagine an electronic component with two input terminals and one output terminal. w_a and w_b are the input values, w_c is the output value.

### 1.3 Selectors - Circuit Configuration Switches

**q_L, q_R, q_O, q_M, q_C** are five "selectors":
- **q_L**: Left selector
- **q_R**: Right selector
- **q_O**: Output selector
- **q_M**: Multiplication selector
- **q_C**: Constant selector

**Key Insight**: Selectors are "configuration parameters" set by the circuit designer beforehand, while wire values are runtime "variables."

---

## Lesson 2: Becoming a Circuit Designer

### 2.1 Designing an Addition Gate

**Goal**: Implement a + b = c

**Configure Selectors**:
- q_L = 1 (enable left input)
- q_R = 1 (enable right input)  
- q_O = -1 (negate output)
- q_M = 0 (disable multiplication)
- q_C = 0 (no constant term)

**Verification**:
```
1 * w_a + 1 * w_b + (-1) * w_c + 0 * (w_a * w_b) + 0 = 0
=> w_a + w_b - w_c = 0
=> w_a + w_b = w_c ✓
```

**Exercise 2.1**
If w_a = 5, w_b = 3, what should w_c equal to satisfy the constraint?

<details>
<summary>Answer</summary>

w_c = 8, because 5 + 3 = 8

</details>

### 2.2 Designing a Multiplication Gate

**Goal**: Implement a × b = c

**Configure Selectors**:
- q_L = 0
- q_R = 0
- q_O = -1
- q_M = 1 (enable multiplication term)
- q_C = 0

**Verification**:
```
0 * w_a + 0 * w_b + (-1) * w_c + 1 * (w_a * w_b) + 0 = 0
=> -w_c + w_a * w_b = 0
=> w_a * w_b = w_c ✓
```

### 2.3 Designing a Constant Gate

**Goal**: Implement a = 5

**Configure Selectors**:
- q_L = 1
- q_R = 0
- q_O = 0
- q_M = 0
- q_C = -5 (negate constant value)

**Verification**:
```
1 * w_a + 0 * w_b + 0 * w_c + 0 * (w_a * w_b) + (-5) = 0
=> w_a - 5 = 0
=> w_a = 5 ✓
```

### 2.4 Designing Custom Gates

**Goal**: Implement a × b + a = c

**Configure Selectors**:
- q_L = 1 (include linear term a)
- q_R = 0
- q_O = -1
- q_M = 1 (include a × b term)
- q_C = 0

**Verification**:
```
1 * w_a + 0 * w_b + (-1) * w_c + 1 * (w_a * w_b) + 0 = 0
=> w_a - w_c + w_a * w_b = 0
=> w_a * w_b + w_a = w_c ✓
```

**Exercise 2.2**
Design a gate to implement: 2a + 3b = c

<details>
<summary>Hint</summary>

Need to use q_L = 2, q_R = 3, q_O = -1

</details>

---

## Lesson 3: R1CS vs PLONK Gate

### 3.1 Limitations of R1CS

In R1CS (Rank-1 Constraint System), each constraint is:
```
(A₁w₁ + A₂w₂ + ... + Aₙwₙ) * (B₁w₁ + B₂w₂ + ... + Bₙwₙ) = C₁w₁ + C₂w₂ + ... + Cₙwₙ
```

Simplified form: **A · B = C**

### 3.2 Problems with R1CS

**Limited Expressiveness**:
- Can only express bilinear relationships
- Requires multiple constraints for simple operations

**Example**: Computing a + b + c
- R1CS needs intermediate variables:
  - temp = a + b
  - result = temp + c
- Requires 2 constraints and 1 additional variable

### 3.3 Advantages of PLONK Gates

**Richer Expressiveness**:
- One gate can include both addition and multiplication
- Fewer constraint count
- More intuitive circuit design

**Comparison Table**:

| Operation | R1CS Constraints | PLONK Gates |
|-----------|------------------|-------------|
| a + b | 1 | 1 |
| a × b | 1 | 1 |
| a + b + c | 2 | 1 |
| a × b + c | 2 | 1 |
| a × b + a | 2 | 1 |

### 3.4 Flexibility Demonstration

PLONK gates can express in one constraint:
- Linear combinations: q_L·a + q_R·b + q_O·c
- Bilinear terms: q_M·(a×b)
- Constant terms: q_C

This makes circuit design more flexible and efficient.

---

## Lesson 4: Practical Exercises

### 4.1 Designing a Square Gate

**Goal**: Implement a² = c

**Approach**: Let w_a = w_b = a, use multiplication gate

**Configuration**:
- q_L = 0
- q_R = 0  
- q_O = -1
- q_M = 1
- q_C = 0

**Constraint**: w_a × w_a = w_c

### 4.2 Designing Polynomial Evaluation

**Goal**: Implement f(x) = x² + 2x + 1 evaluation at some point

This requires combining multiple gates:
1. Gate 1: Compute x² 
2. Gate 2: Compute 2x
3. Gate 3: Compute x² + 2x + 1

**Exercise 4.1**
Design specific gate configurations for the above polynomial evaluation.

### 4.3 Boolean Constraints

**Goal**: Constrain a to be 0 or 1

**Approach**: Use a × (a - 1) = 0

This means a = 0 or a = 1

**Configuration**:
- q_L = -1
- q_R = 0
- q_O = 0  
- q_M = 1
- q_C = 0

Constraint: w_a × w_a - w_a = 0

---

## Module Summary

Through this module, we deeply understood:

1. **PLONK Core Formula** components and their roles
2. **Selector Configuration** determining gate behavior
3. **Circuit Design** basic techniques and patterns
4. **PLONK vs R1CS** advantage comparison

### Core Insights

- PLONK gates are **programmable constraint templates**
- By adjusting selectors, various arithmetic operations can be implemented
- Single PLONK gates are more expressive than R1CS constraints

## Self-Assessment

Before proceeding to the next module, confirm you can:
- [ ] Explain the meaning of each term in PLONK's core formula
- [ ] Design basic arithmetic gates (addition, multiplication, constant)
- [ ] Understand how selectors control gate behavior
- [ ] Compare differences between PLONK and R1CS

## Next Steps

Having mastered single gate design, we need to learn how to "connect" multiple gates to form complete circuits. Let's proceed to [Module 2: Connecting Circuits - The Magic of Permutation Arguments](../module_2_permutation/)!
