# Module 0: Foundation & Toolbox (Prerequisites & Toolbox)

## Module Objective
Before officially starting, ensure we have the same vocabulary and mathematical tools.

## Mental Model
Before learning to build a house, first get familiar with bricks, cement, and measuring tools.

---

## Lesson 1: Intuition of Finite Fields

### 1.1 Why Do We Need Finite Fields?

In daily computation, we're accustomed to an infinite world of numbers. But in cryptography and zero-knowledge proofs, we need:

**Prevent Infinite Number Growth**
- Computer storage and processing capabilities are limited
- Need deterministic computation results
- Avoid overflow problems

**Bring Algebraic Properties**
- Ensure division operations are always defined
- Guarantee stability of polynomial operations
- Provide rich mathematical structures

### 1.2 Clock Arithmetic: Intuitive Understanding of Modular Operations

Imagine a clock with only 12 hours:
- 10 o'clock + 5 o'clock = 3 o'clock (not 15 o'clock)
- This is modular arithmetic: (10 + 5) mod 12 = 3

In finite field F_p:
- All operations are performed modulo p
- p is usually a large prime
- Example: F_17 = {0, 1, 2, ..., 16}

**Exercise 1.1**
Compute in F_7:
- 5 + 4 = ?
- 6 × 3 = ?
- 5 - 6 = ?

<details>
<summary>Answer</summary>

- 5 + 4 = 9 mod 7 = 2
- 6 × 3 = 18 mod 7 = 4  
- 5 - 6 = -1 mod 7 = 6

</details>

---

## Lesson 2: The Power of Polynomials

### 2.1 Core Idea: Encoding Information with Polynomials

A polynomial can "encode" massive amounts of information:
- Coefficients: f(x) = 3x² + 2x + 1
- Value at a point: f(5) = 3×25 + 2×5 + 1 = 86
- Roots: x values that make f(x) = 0

### 2.2 Lagrange Interpolation: From Points to Polynomials

**Core Theorem**: Given n+1 distinct points, there exists a unique polynomial of degree n that passes through these points.

**Example**:
Given points (1, 3), (2, 8), (3, 15)
Goal: Find the unique degree-2 polynomial f(x) = ax² + bx + c

Steps:
1. Set up system of equations:
   - f(1) = a + b + c = 3
   - f(2) = 4a + 2b + c = 8  
   - f(3) = 9a + 3b + c = 15

2. Solve: a = 1, b = 2, c = 0
3. Therefore f(x) = x² + 2x

**Exercise 2.1**
Given points (0, 1), (1, 4), (2, 9), find the polynomial that passes through these points.

<details>
<summary>Hint</summary>

Observe: 1, 4, 9 = 1², 2², 3²
So the polynomial might be f(x) = (x+1)²

</details>

### 2.3 Lagrange Interpolation Formula

For points (x₀, y₀), (x₁, y₁), ..., (xₙ, yₙ):

f(x) = Σᵢ yᵢ × Lᵢ(x)

where Lᵢ(x) = Π_{j≠i} (x - xⱼ)/(xᵢ - xⱼ)

**Intuitive Understanding**:
- Lᵢ(x) equals 1 when x = xᵢ
- Equals 0 at all other xⱼ
- Thus f(xᵢ) = yᵢ

---

## Lesson 3: Polynomial Commitment Scheme (PCS) Concepts

### 3.1 What is a Polynomial Commitment?

**Analogy**: You have a sealed envelope (commitment) with a secret polynomial written inside. You can:
1. Show others the envelope's "fingerprint" (commitment value)
2. Without opening the envelope, prove that the polynomial inside has a specific value at a certain point

### 3.2 Core Functions of PCS

**Commit**
- Input: Secret polynomial f(x)
- Output: Short commitment value C
- Property: Cannot infer any information about f(x) from C

**Open**
- Input: Commitment C, point z, claimed value y, proof π
- Claim: The committed polynomial's value at z is y
- Output: Accept/Reject

### 3.3 Security Properties of PCS

**Binding**
- Cannot generate two different valid proofs for the same polynomial

**Hiding**
- The commitment reveals no information about the polynomial

**Knowledge Extraction**
- If one can generate a valid proof, they must "know" the corresponding polynomial

### 3.4 Black Box Stage Understanding

At this stage, you only need to know:
- PCS is a tool that allows us to "commit" to a polynomial
- Can prove the polynomial's value at a point without revealing the polynomial
- Specific implementations (like KZG or FRI) will be learned later

**Exercise 3.1**
Reflection: Why is PCS so important for zero-knowledge proofs?

<details>
<summary>Thinking Direction</summary>

1. Polynomials can encode complex computations
2. PCS allows us to prove correctness while keeping computations private
3. This is key technology for achieving "zero-knowledge"

</details>

---

## Module Summary

Through this module, we established the foundation for PLONK learning:

1. **Finite Fields**: Provide a bounded mathematical environment
2. **Polynomials**: Powerful tools for encoding complex information
3. **PCS**: Key technology for proving computational correctness while maintaining privacy

These tools will be repeatedly used in subsequent modules. Please ensure you have a solid understanding of these concepts.

## Self-Assessment

Before proceeding to the next module, confirm you can:
- [ ] Perform basic operations in finite fields
- [ ] Understand the principles of Lagrange interpolation
- [ ] Explain the core functions and security properties of PCS

## Next Steps

Ready? Let's proceed to [Module 1: Minimal Components - The Mystery of a Single Gate](../module_1_gate_constraints/)!
