# Module 3: Language Translation - From Circuits to Polynomials (From Circuits to Polynomials)

## Module Objective
Understand how PLONK "packages" all circuit constraints into several polynomial identities.

## Mental Model
Transform discrete, individual gate constraints into continuous, smooth polynomial curves.

---

## Lesson 1: Perspective Elevation

### 1.1 From Discrete to Continuous

In the previous two modules, we thought about "the i-th gate's" constraints. Now we elevate our perspective:

**Old Perspective**:
- Gate 1: q_L[1] * w_a[1] + ... = 0
- Gate 2: q_L[2] * w_a[2] + ... = 0
- ...

**New Perspective**:
- w_a is no longer an array, but a **polynomial** w_a(X)
- q_L is also a polynomial q_L(X)
- At specific point ω^i, w_a(ω^i) = w_a[i]

### 1.2 Roots of Unity

**Key Tool**: Use n-th roots of unity ω

Properties:
- ω^n = 1
- ω^0, ω^1, ω^2, ..., ω^(n-1) are n distinct values
- These values form a cyclic group

**Intuitive Understanding**:
Imagine a circular dial with n markings, ω is the rotation angle from the 0 mark to the first marking.

### 1.3 Application of Polynomial Interpolation

Given gate data:
- w_a[0], w_a[1], ..., w_a[n-1]

We can construct a unique polynomial w_a(X) such that:
- w_a(ω^0) = w_a[0]
- w_a(ω^1) = w_a[1]
- ...
- w_a(ω^(n-1)) = w_a[n-1]

**Exercise 3.1**
If there are 3 gates with w_a values [2, 5, 8], and ω³ = 1, what is the value of w_a(X) at ω²?

<details>
<summary>Answer</summary>

w_a(ω²) = 8, because this corresponds to the 3rd gate (index 2) value.

</details>

---

## Lesson 2: Integrating Gate Constraints

### 2.1 Definition of Constraint Polynomial

Define **constraint polynomial** P(X):

```
P(X) = q_L(X) * w_a(X) + q_R(X) * w_b(X) + q_O(X) * w_c(X) + q_M(X) * w_a(X) * w_b(X) + q_C(X)
```

### 2.2 Key Property

If all gate constraints are satisfied, then:
- P(ω^0) = 0 (1st gate's constraint)
- P(ω^1) = 0 (2nd gate's constraint)
- ...
- P(ω^(n-1)) = 0 (n-th gate's constraint)

**Core Insight**: A single polynomial P(X) encodes all gate constraints!

### 2.3 Example Demonstration

**Circuit**:
- Gate 1: a + b = c (addition gate)
- Gate 2: c × d = e (multiplication gate)

**Polynomials**:
- w_a(X): interpolates [a, c]
- w_b(X): interpolates [b, d]  
- w_c(X): interpolates [c, e]
- q_L(X): interpolates [1, 0] (Gate 1 enables left input, Gate 2 doesn't)
- q_R(X): interpolates [1, 0] (Gate 1 enables right input, Gate 2 doesn't)
- ...

**Verification**:
- P(ω^0) = 1·a + 1·b + (-1)·c + 0·(a×b) + 0 = a + b - c = 0 ✓
- P(ω^1) = 0·c + 0·d + (-1)·e + 1·(c×d) + 0 = cd - e = 0 ✓

---

## Lesson 3: Vanishing Polynomial

### 3.1 Definition of Vanishing Polynomial

**Vanishing polynomial** Z_H(X) is a polynomial that equals 0 at all gate coordinates:

```
Z_H(X) = (X - ω^0)(X - ω^1)...(X - ω^(n-1)) = X^n - 1
```

### 3.2 Why "Vanishing"?

- At each gate position ω^i, Z_H(ω^i) = 0
- "Vanishes" at all points we care about

### 3.3 Key Insight about Divisibility

**Core Theorem**: If polynomial f(X) has value 0 at point a, then f(X) is divisible by (X-a).

**Extension**: If P(X) equals 0 at ω^0, ω^1, ..., ω^(n-1), then P(X) is divisible by Z_H(X).

### 3.4 Introduction of Quotient Polynomial

If all constraints are satisfied, there exists polynomial t(X) such that:

```
P(X) = t(X) * Z_H(X)
```

This t(X) is the **quotient polynomial**.

---

## Lesson 4: Quotient Polynomial

### 4.1 Meaning of Quotient Polynomial

**Mathematical Meaning**: t(X) = P(X) / Z_H(X)

**Practical Meaning**:
- If we can compute t(X), it shows P(X) is indeed divisible by Z_H(X)
- This proves all gate constraints are satisfied

### 4.2 Degree Analysis

Assume the circuit has n gates:
- P(X) has degree at most 2n-1 (due to w_a(X) × w_b(X) terms)
- Z_H(X) has degree n
- So t(X) has degree at most n-1

### 4.3 Prover's Task

The Prover needs to:
1. Compute all wire polynomials w_a(X), w_b(X), w_c(X)
2. Construct constraint polynomial P(X)
3. Compute quotient polynomial t(X) = P(X) / Z_H(X)
4. Prove P(X) = t(X) × Z_H(X)

### 4.4 Efficient Computation Techniques

**Polynomial Long Division**:
- Direct computation would be slow
- Using FFT can accelerate to O(n log n)

**Segmented Computation**:
- Split t(X) into several low-degree polynomials
- Commit to each part separately

---

## Lesson 5: Integrating Permutation Constraints

### 5.1 Polynomial Representation of Permutation Constraints

Similarly, the permutation arguments from Module 2 also need to be transformed into polynomial form.

**Permutation polynomial** Z_perm(X) satisfies:

```
L_1(X) * (Z_perm(X) - 1) = 0
```

Where L_1(X) is a Lagrange basis polynomial that equals 1 at ω^0 and 0 elsewhere.

### 5.2 Recurrence Relation for Permutation

For i = 0, 1, ..., n-2:

```
Z_perm(ω^(i+1)) * denominator(ω^i) = Z_perm(ω^i) * numerator(ω^i)
```

This relation also needs to be expressed at the polynomial level.

### 5.3 Unified Constraint System

Finally, the PLONK system has two major types of constraints:
1. **Gate constraints**: P_gate(X) = t_gate(X) × Z_H(X)
2. **Permutation constraints**: P_perm(X) = t_perm(X) × Z_H(X)

---

## Lesson 6: Complete Polynomial Identity

### 6.1 Linear Combination of Constraints

Using random challenge α, combine all constraints:

```
P_total(X) = P_gate(X) + α * P_perm(X)
```

Correspondingly:

```
t_total(X) = t_gate(X) + α * t_perm(X)
```

### 6.2 Final Identity

PLONK's core identity:

```
P_total(X) = t_total(X) * Z_H(X)
```

### 6.3 Verification Strategy

The Verifier only needs to:
1. Evaluate both sides at random point z
2. Check P_total(z) = t_total(z) × Z_H(z)
3. Use polynomial commitments to ensure evaluation correctness

---

## Lesson 7: Managing Polynomial Degrees

### 7.1 Degree Explosion Problem

When circuits are complex, polynomial degrees can be very high:
- Storage and computation costs increase dramatically
- Requires larger trusted setup

### 7.2 Splitting Technique

Split high-degree t(X) into multiple low-degree polynomials:

```
t(X) = t_lo(X) + X^n * t_mid(X) + X^(2n) * t_hi(X)
```

Each part has degree less than n.

### 7.3 Separate Commitments

Commit to t_lo, t_mid, t_hi separately, reducing overall complexity.

---

## Module Summary

Through this module, we achieved the important "language translation":

1. **Perspective Elevation**: From discrete arrays to continuous polynomials
2. **Constraint Integration**: All gate constraints encoded into one polynomial
3. **Vanishing Polynomial**: Mathematical tool providing divisibility checks
4. **Quotient Polynomial**: Core object proving constraint satisfaction

### Core Insights

- Polynomials are powerful tools for encoding complex information
- Divisibility checks transform constraint verification into algebraic problems
- Random linear combinations allow simultaneous handling of multiple constraint types

## Self-Assessment

Before proceeding to the next module, confirm you can:
- [ ] Explain the transformation from discrete constraints to polynomial constraints
- [ ] Understand the role and properties of vanishing polynomials
- [ ] Describe the computation and meaning of quotient polynomials
- [ ] Analyze the impact of polynomial degrees on system performance

## Next Steps

Now we have the complete mathematical framework, it's time to see how the PLONK protocol actually runs. Let's proceed to [Module 4: System Integration - Complete Proving and Verification Flow](../module_4_protocol/)!
