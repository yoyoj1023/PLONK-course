# Final Module: Hands-on Implementation - Theory and Practice (Hands-on Lab)

## Module Objective
Transform theoretical knowledge into practical skills.

## Mental Model
From "armchair strategizing" to "battlefield practice," truly master the essence of PLONK.

---

## Lesson 1: Circuit Design Exercises

### 1.1 Simple Circuit: Square Root Check

**Problem**: Prove you know x such that x² = y, without revealing x's value

**Circuit Design**:
```
Gate 1: x × x = y
```

**PLONK Configuration**:
- q_L = 0
- q_R = 0  
- q_O = -1
- q_M = 1
- q_C = 0

**Wire Values** (assume x = 5, y = 25):
- w_a = 5
- w_b = 5
- w_c = 25

**Exercise 1.1**: Fill in wire values for x = 7, y = 49.

### 1.2 Medium Circuit: MiMC Hash Function

**MiMC Algorithm**:
```
for i in range(rounds):
    x = (x + key + constants[i])^3
return x
```

**Circuit Decomposition** (single round):
```
Gate 1: temp1 = x + key + c_i
Gate 2: temp2 = temp1 × temp1  
Gate 3: result = temp2 × temp1
```

**Copy Constraints**:
- Gate 1's output = Gate 2's left input
- Gate 1's output = Gate 3's right input
- Gate 2's output = Gate 3's left input

**Exercise 1.2**: Design complete circuit for 3-round MiMC.

### 1.3 Complex Circuit: Fibonacci Sequence

**Problem**: Prove the n-th Fibonacci number is F_n

**Circuit Design**:
```
F_0 = 0
F_1 = 1
for i from 2 to n:
    F_i = F_{i-1} + F_{i-2}
```

**Gate Design**:
```
Gate i: w_a[i] + w_b[i] = w_c[i]
```

Where:
- w_a[i] = F_{i-2}
- w_b[i] = F_{i-1}  
- w_c[i] = F_i

**Copy Constraints**:
- w_c[i] = w_b[i+1]
- w_b[i] = w_a[i+1]

**Exercise 1.3**: Design complete circuit for computing F_10 and list all copy constraints.

---

## Lesson 2: Polynomial Construction Practice

### 2.1 Hand-calculating Wire Polynomials

**Circuit**:
```
Gate 1: 2 + 3 = 5
Gate 2: 5 × 4 = 20
```

**Field Setup**: Use F_17, ω = 3 (since 3² = 9 ≠ 1, 3⁴ = 81 ≡ 13 ≠ 1, but 3^8 ≡ 1)

Wait, let's use simpler setup: F_5, ω = 2 (since 2⁴ = 16 ≡ 1 (mod 5))

**Wire Values**:
- w_a = [2, 5]  
- w_b = [3, 4]
- w_c = [5, 20] ≡ [5, 0] (mod 5)

**Polynomial Interpolation**:
Goal: Find w_a(X) such that w_a(ω⁰) = w_a(1) = 2, w_a(ω¹) = w_a(2) = 5

Using Lagrange interpolation:
```
w_a(X) = 2 × (X-2)/(1-2) + 5 × (X-1)/(2-1)
       = 2 × (X-2)/(-1) + 5 × (X-1)/1
       = -2(X-2) + 5(X-1)
       = -2X + 4 + 5X - 5
       = 3X - 1
```

**Verification**:
- w_a(1) = 3×1 - 1 = 2 ✓
- w_a(2) = 3×2 - 1 = 5 ✓

**Exercise 2.1**: Compute w_b(X) and w_c(X).

### 2.2 Constraint Polynomial Construction

**Selector Setup**:
- Gate 1 (addition): q_L=1, q_R=1, q_O=-1, q_M=0, q_C=0
- Gate 2 (multiplication): q_L=0, q_R=0, q_O=-1, q_M=1, q_C=0

**Selector Polynomials**:
```
q_L(X) = 1 × (X-2)/(1-2) + 0 × (X-1)/(2-1) = -(X-2) = -X+2
q_M(X) = 0 × (X-2)/(1-2) + 1 × (X-1)/(2-1) = X-1
```

**Constraint Polynomial**:
```
P(X) = q_L(X)w_a(X) + q_R(X)w_b(X) + q_O(X)w_c(X) + q_M(X)w_a(X)w_b(X) + q_C(X)
```

**Exercise 2.2**: Compute P(1) and P(2), verify they both equal 0.

### 2.3 Quotient Polynomial Computation

**Vanishing Polynomial**:
```
Z_H(X) = (X-1)(X-2) = X² - 3X + 2
```

**Quotient Polynomial**:
```
t(X) = P(X) / Z_H(X)
```

Since P(1) = P(2) = 0, P(X) must be divisible by Z_H(X).

**Exercise 2.3**: Perform polynomial long division to compute t(X).

---

## Lesson 3: Code Base Exploration

### 3.1 Plonky2 Source Code Structure

**Core Modules**:
```
plonky2/
├── field/           # Finite field implementation
├── fri/             # FRI commitment scheme
├── gates/           # Various gate implementations
├── hash/            # Hash functions
├── iop/             # IOP (Interactive Oracle Proof)
├── plonk/           # PLONK protocol core
├── poly/            # Polynomial operations
└── util/            # Utility functions
```

### 3.2 Gate Implementation Analysis

**Find Addition Gate**:
```rust
// plonky2/src/gates/arithmetic_gate.rs
pub struct ArithmeticGate {
    pub num_ops: usize,
}

impl Gate for ArithmeticGate {
    fn eval_unfiltered(&self, vars: EvaluationVars) -> Vec<F> {
        // Implement q_L * w_a + q_R * w_b + q_O * w_c + q_M * (w_a * w_b) + q_C
    }
}
```

**Exercise 3.1**: Read `arithmetic_gate.rs`, find code corresponding to the PLONK formula we learned.

### 3.3 Polynomial Commitment Implementation

**FRI Implementation**:
```rust
// plonky2/src/fri/prover.rs
pub fn prove(&self, polynomial: &Polynomial) -> FriProof {
    // FRI proof generation
}

// plonky2/src/fri/verifier.rs  
pub fn verify(&self, proof: &FriProof) -> bool {
    // FRI proof verification
}
```

**Exercise 3.2**: Trace complete flow of a polynomial from commitment to opening.

### 3.4 Permutation Argument Implementation

**Permutation Polynomial**:
```rust
// plonky2/src/plonk/permutation_argument.rs
pub fn compute_permutation_z_polys(
    gate_constraints: &[GateConstraint],
    permutation: &Permutation,
) -> Vec<Polynomial> {
    // Compute permutation polynomial Z(X)
}
```

**Exercise 3.3**: Understand the specific computation process of permutation polynomials.

---

## Lesson 4: Using High-level Languages

### 4.1 Circom Implementation

**Install Circom**:
```bash
npm install -g circom
npm install -g snarkjs
```

**Simple Circuit**:
```javascript
// square.circom
pragma circom 2.0.0;

template Square() {
    signal input x;
    signal output y;
    
    y <== x * x;
}

component main = Square();
```

**Compile and Test**:
```bash
circom square.circom --r1cs --wasm --sym
echo '{"x": "5"}' > input.json
node square_js/generate_witness.js square_js/square.wasm input.json witness.wtns
```

**Exercise 4.1**: Implement MiMC hash Circom version.

### 4.2 Noir Implementation

**Install Noir**:
```bash
curl -L noirup.dev | bash
noirup
```

**Simple Circuit**:
```rust
// main.nr
fn main(x: Field, y: pub Field) {
    assert(x * x == y);
}
```

**Testing**:
```bash
nargo check
nargo test
nargo prove
nargo verify
```

**Exercise 4.2**: Implement Fibonacci sequence verification using Noir.

### 4.3 Direct Plonky2 Usage

**Rust Project Setup**:
```toml
[dependencies]
plonky2 = "0.1"
```

**Simple Circuit**:
```rust
use plonky2::plonk::circuit_builder::CircuitBuilder;
use plonky2::field::goldilocks_field::GoldilocksField;

fn main() {
    let mut builder = CircuitBuilder::<GoldilocksField, 2>::new();
    
    // Add inputs
    let x = builder.add_virtual_target();
    let y = builder.add_virtual_target();
    
    // Add constraint: x * x = y
    let x_squared = builder.mul(x, x);
    builder.connect(x_squared, y);
    
    // Set as public input
    builder.register_public_input(y);
    
    // Build circuit
    let data = builder.build::<C>();
}
```

**Exercise 4.3**: Implement a simple range check circuit.

---

## Lesson 5: Performance Analysis and Optimization

### 5.1 Benchmarking

**Test Circuits**:
- Small: 100 gates
- Medium: 10,000 gates  
- Large: 1,000,000 gates

**Metric Measurement**:
```rust
use std::time::Instant;

let start = Instant::now();
let proof = prover.prove(witness);
let prove_time = start.elapsed();

let start = Instant::now();
let is_valid = verifier.verify(proof);
let verify_time = start.elapsed();

println!("Prove time: {:?}", prove_time);
println!("Verify time: {:?}", verify_time);
println!("Proof size: {} bytes", proof.len());
```

**Exercise 5.1**: Test performance for different circuit sizes.

### 5.2 Bottleneck Identification

**Common Bottlenecks**:
1. **FFT Computation**: Main cost of polynomial operations
2. **Hash Computation**: Merkle tree construction
3. **Memory Access**: Large polynomial access

**Analysis Tools**:
```bash
# CPU profiling
perf record ./your_program
perf report

# Memory profiling  
valgrind --tool=massif ./your_program
```

**Exercise 5.2**: Analyze performance bottlenecks in your circuit.

### 5.3 Optimization Strategies

**Parallelization**:
```rust
use rayon::prelude::*;

// Parallel FFT
let coeffs: Vec<_> = (0..n).into_par_iter()
    .map(|i| compute_coeff(i))
    .collect();
```

**Memory Optimization**:
```rust
// Stream processing of large polynomials
for chunk in polynomial.chunks(CHUNK_SIZE) {
    process_chunk(chunk);
}
```

**Exercise 5.3**: Implement optimized version of circuit and compare performance.

---

## Lesson 6: Real Application Development

### 6.1 Zero-Knowledge Voting System

**Requirements**:
- Users can vote secretly
- Anyone can verify vote totals
- Cannot trace individual votes

**Circuit Design**:
```
// Each vote
Input: voter_id (private), vote (private), nullifier (public)
Constraints:
1. vote ∈ {0, 1}  // Range check
2. nullifier = hash(voter_id, election_id)  // Prevent double voting
3. Publicly commit to vote
```

**Exercise 6.1**: Implement complete voting circuit.

### 6.2 Privacy-Preserving Identity Verification

**Requirements**:
- Prove age ≥ 18
- Don't reveal specific age
- Prevent replay attacks

**Circuit Design**:
```
Input: age (private), threshold=18 (public), nonce (public)
Constraints:
1. age ≥ threshold
2. Output hash(age, nonce)
```

**Exercise 6.2**: Add additional constraints (like age ≤ 120).

### 6.3 Zero-Knowledge Machine Learning

**Requirements**:
- Prove ML model prediction results
- Don't reveal model parameters
- Protect input data privacy

**Circuit Design**:
```
Input: features (private), weights (private), prediction (public)
Constraints:
1. prediction = model(features, weights)
2. Correctness of model computation
```

**Exercise 6.3**: Implement simple linear regression proof.

---

## Lesson 7: Production Environment Considerations

### 7.1 Security Checklist

**Trusted Setup**:
- [ ] Use audited trusted setup
- [ ] Verify correctness of setup parameters
- [ ] Ensure transparency of setup process

**Implementation Security**:
- [ ] Use audited libraries
- [ ] Prevent side-channel attacks
- [ ] Secure random number generation

**Protocol Security**:
- [ ] Correct field parameter selection
- [ ] Appropriate security parameters
- [ ] Prevent common attack vectors

### 7.2 Scalability Design

**Batch Processing**:
```rust
// Batch verify multiple proofs
fn batch_verify(proofs: &[Proof]) -> bool {
    let combined = combine_proofs(proofs);
    verify_single(combined)
}
```

**Recursive Proofs**:
```rust
// Prove validity of other proofs
fn recursive_verify(inner_proof: Proof) -> Proof {
    // Build verification circuit
    // Generate wrapper proof
}
```

**Exercise 7.1**: Implement system supporting batch verification.

### 7.3 Deployment and Monitoring

**Deployment Checks**:
- Performance benchmarking
- Memory usage monitoring
- Error handling mechanisms

**Monitoring Metrics**:
```rust
struct Metrics {
    prove_time_avg: Duration,
    verify_time_avg: Duration,
    proof_size_avg: usize,
    success_rate: f64,
}
```

**Exercise 7.2**: Design complete monitoring system.

---

## Module Summary

Through this module's practice, you have:

1. **Hand-designed** PLONK circuits of various complexities
2. **Deep explored** actual ZK system implementations
3. **Used high-level languages** to quickly build ZK applications
4. **Analyzed and optimized** system performance
5. **Developed real applications** considering production environment requirements

### Core Skills

- Circuit design and constraint analysis
- Polynomial operations and performance optimization  
- Implementation and deployment of real systems
- Security considerations and best practices

## Final Assessment

Confirm you can now:
- [ ] Independently design medium-complexity ZK circuits
- [ ] Read and understand ZK system source code
- [ ] Use mainstream tools to develop ZK applications
- [ ] Analyze and optimize ZK system performance
- [ ] Consider various production environment requirements

## Congratulations on Graduation!

You have now completely mastered the essence of the PLONK protocol, possessing:
- Deep theoretical understanding
- Solid practical skills
- Systematic thinking approach
- Ability to solve real problems

Ready to explore the broader world of zero-knowledge proofs! Next, you can:
- Deep dive into Plonky3 and latest developments
- Explore other ZK systems (STARKs, Nova, etc.)
- Develop your own ZK applications
- Contribute to open-source ZK projects

**The future of zero-knowledge proofs is in your hands!** 🚀
