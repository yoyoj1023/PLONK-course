# Module 4: System Integration - Complete Proving and Verification Flow (The Full Protocol)

## Module Objective
Assemble all components and walk through the complete PLONK protocol flow.

## Mental Model
Watch a complete factory assembly line, from Prover inputting secrets to Verifier outputting "accept/reject."

---

## Lesson 1: Protocol Overview

### 1.1 Participants and Inputs

**Prover**:
- Has secret inputs (private witness)
- Wants to prove correctness of some computation
- Doesn't want to reveal secret information

**Verifier**:
- Only knows public inputs and computation results
- Wants to be convinced that Prover indeed performed correct computation
- Doesn't want to do too much computational work

### 1.2 Protocol Phases

PLONK protocol consists of **4 rounds of interaction**:

1. **Witness Phase**: Prover fills circuit, commits to wire polynomials
2. **Permutation Phase**: Using random challenges, construct permutation polynomial
3. **Quotient Polynomial Phase**: Linearly combine all constraints, compute quotient polynomial
4. **Evaluation Phase**: Evaluate all polynomials at random points

### 1.3 Source of Security

- **Random Challenges**: Prevent Prover from preparing fake proofs in advance
- **Polynomial Commitments**: Ensure Prover cannot change polynomials after commitment
- **Fiat-Shamir Transform**: Make the protocol non-interactive

---

## Lesson 2: Round 1 - Witness Phase

### 2.1 Prover's Work

**Step 1**: Execute computation
- Fill secret inputs into each gate of the circuit
- Compute all intermediate values and final outputs
- Obtain complete witness: w_a[0..n], w_b[0..n], w_c[0..n]

**Step 2**: Construct wire polynomials
- Using Lagrange interpolation, construct:
  - w_a(X): through points (ω^i, w_a[i])
  - w_b(X): through points (ω^i, w_b[i])
  - w_c(X): through points (ω^i, w_c[i])

**Step 3**: Polynomial commitments
- Compute commitments: [w_a], [w_b], [w_c]
- Send to Verifier

### 2.2 Verifier's Work

- Receive three commitments
- Generate random challenges β, γ (or use Fiat-Shamir)
- Send to Prover

### 2.3 Example Demonstration

**Circuit**: Compute x² + 3x + 2

**Witness** (assume x = 5):
- Gate 1: temp1 = x × x = 25
- Gate 2: temp2 = 3 × x = 15  
- Gate 3: temp3 = temp1 + temp2 = 40
- Gate 4: result = temp3 + 2 = 42

**Wire values**:
- w_a = [5, 3, 25, 40]
- w_b = [5, 5, 15, 2]
- w_c = [25, 15, 40, 42]

---

## Lesson 3: Round 2 - Permutation Phase

### 3.1 Using Random Challenges

After Prover receives β, γ:
- Construct permutation polynomial Z_perm(X)
- Use β, γ to ensure cannot precompute

### 3.2 Construction of Permutation Polynomial

**Initialization**: Z_perm(ω^0) = 1

**Recurrence relation** (for i = 0, 1, ..., n-2):
```
Z_perm(ω^(i+1)) = Z_perm(ω^i) × numerator(ω^i) / denominator(ω^i)
```

Where:
```
numerator(ω^i) = (w_a(ω^i) + β·ω^i + γ) × (w_b(ω^i) + β·k₁·ω^i + γ) × (w_c(ω^i) + β·k₂·ω^i + γ)

denominator(ω^i) = (w_a(ω^i) + β·σ_a(ω^i) + γ) × (w_b(ω^i) + β·σ_b(ω^i) + γ) × (w_c(ω^i) + β·σ_c(ω^i) + γ)
```

### 3.3 Committing Permutation Polynomial

- Compute commitment [Z_perm]
- Send to Verifier

### 3.4 Verifier's Response

- Generate random challenge α
- Send to Prover

---

## Lesson 4: Round 3 - Quotient Polynomial Phase

### 4.1 Linear Combination of Constraints

Prover uses α to combine all constraints:

**Gate constraints**:
```
P_gate(X) = q_L(X)w_a(X) + q_R(X)w_b(X) + q_O(X)w_c(X) + q_M(X)w_a(X)w_b(X) + q_C(X)
```

**Permutation constraints**:
```
P_perm(X) = α·[(Z_perm(X)·numerator(X) - Z_perm(ωX)·denominator(X))] + α²·L_1(X)(Z_perm(X) - 1)
```

**Total constraints**:
```
P_total(X) = P_gate(X) + P_perm(X)
```

### 4.2 Quotient Polynomial Computation

Compute quotient polynomial:
```
t(X) = P_total(X) / Z_H(X)
```

### 4.3 Splitting and Commitment

Split t(X) into low-degree parts:
```
t(X) = t_lo(X) + X^n·t_mid(X) + X^(2n)·t_hi(X)
```

Compute commitments: [t_lo], [t_mid], [t_hi]

### 4.4 Verifier's Final Challenge

- Generate random evaluation point z
- Send to Prover

---

## Lesson 5: Round 4 - Evaluation Phase

### 5.1 Polynomial Evaluations

Prover computes all polynomial values at z and zω:

**Basic evaluations**:
- a = w_a(z)
- b = w_b(z)  
- c = w_c(z)
- z_perm = Z_perm(z)

**Shifted evaluations**:
- z_perm_shifted = Z_perm(zω)

**Selector evaluations**:
- q_L_eval = q_L(z)
- q_R_eval = q_R(z)
- etc...

**Quotient polynomial evaluations**:
- t_eval = t(z)

### 5.2 Sending Evaluation Values

Prover sends all evaluation values to Verifier

### 5.3 Generating Evaluation Proofs

For each claimed evaluation, Prover generates polynomial commitment "opening proofs"

---

## Lesson 6: Verifier's Final Checks

### 6.1 Gate Constraint Verification

Verifier checks:
```
q_L_eval × a + q_R_eval × b + q_O_eval × c + q_M_eval × a × b + q_C_eval = gate_constraint_eval
```

### 6.2 Permutation Constraint Verification

Check permutation polynomial's recurrence relation:
```
z_perm_shifted × denominator_eval = z_perm × numerator_eval
```

### 6.3 Quotient Polynomial Verification

Check core equation:
```
gate_constraint_eval + α × permutation_constraint_eval = t_eval × Z_H_eval
```

where Z_H_eval = z^n - 1

### 6.4 Polynomial Commitment Verification

Verify opening proofs for all polynomial evaluations

---

## Lesson 7: Security Analysis

### 7.1 Completeness

If Prover is honest and knows correct witness:
- All constraints will be satisfied
- Quotient polynomial computation is correct
- Verifier always accepts

### 7.2 Soundness

If Prover tries to cheat:
- Probability of forging constraint satisfaction is extremely small (about 1/|𝔽|)
- Random challenges prevent precomputation attacks
- Binding property of polynomial commitments prevents post-hoc modifications

### 7.3 Zero-Knowledge

- All sent values are "masked" by random numbers
- Verifier learns nothing about the witness
- Only can confirm correctness of computation

### 7.4 Succinctness

- Proof size: O(1) group elements
- Verification time: O(1) pairing operations
- Independent of circuit size

---

## Lesson 8: Practical Considerations

### 8.1 Trusted Setup

**Structured Reference String (SRS)**:
- Contains commitments [1], [x], [x²], ..., [x^d]
- Requires one-time trusted setup ceremony
- Can be reused across multiple circuits

### 8.2 Preprocessing

Circuit selector polynomials can be precomputed:
- q_L(X), q_R(X), q_O(X), q_M(X), q_C(X)
- σ_a(X), σ_b(X), σ_c(X)

### 8.3 Batch Verification

Multiple proofs can be verified simultaneously:
- Use random linear combination
- Amortize verification costs

---

## Module Summary

Through this module, we completely walked through the PLONK protocol:

1. **4-round interaction flow**: witness → permutation → quotient polynomial → evaluation
2. **Prover's tasks**: commit, compute, prove
3. **Verifier's checks**: constraint verification, commitment verification
4. **Security guarantees**: completeness, soundness, zero-knowledge, succinctness

### Core Insights

- Random challenges are key to protocol security
- Polynomial commitments achieve "binding but hiding" properties
- Phased design makes complexity manageable

## Self-Assessment

Before proceeding to the next module, confirm you can:
- [ ] Describe the 4 phases of the PLONK protocol
- [ ] Explain the role of random challenges
- [ ] Understand Verifier's verification logic
- [ ] Analyze the protocol's security properties

## Next Steps

The basic PLONK protocol is already very powerful, but there's room for further optimization. Let's learn how to use lookup tables to handle complex operations: [Module 5: Performance Turbo - Using Lookup Tables](../module_5_lookup/)!
