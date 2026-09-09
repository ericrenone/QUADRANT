# QUADRANT

### Quantum Unimodular Arithmetic · Direction-Recorded Rotation for Amplitude-Native Tails

**One shift-and-add ladder, run on a register that is in superposition. Every rotation is three shears. Every shear is a permutation. Every direction bit is kept, because it is the angle.**

> The reversible ladder does not compute a price. It computes a permutation of the register whose amplitude, read once per query, is the price. The instrument is a quadrant: it reads the shadow, not the sun.

---

## 0 · The Answer First

A quantum derivative-pricing engine is three machines bolted together, and only one of them is quantum in any interesting sense.

1. **A loader** that puts the path distribution of the underlying into the amplitudes of a register.
2. **A payoff oracle** that computes, in superposition, the contract's value along every path at once and writes it into the rotation angle of a single qubit.
3. **An estimator** — amplitude estimation — that reads that qubit's probability with error ε in about 1/ε queries instead of the 1/ε² samples that classical Monte Carlo needs.

Machines 1 and 2 are arithmetic. They are the entire cost. The estimator is a fixed multiplier — roughly 2/ε Grover iterations, each of which runs the oracle forwards and backwards — and it has been understood since 2002. The engine's economics therefore reduce to a single question: **what does one evaluation of `exp`, `ln`, `√` and `×` cost when it must be reversible, ancilla-disciplined and fault-tolerant?**

QUADRANT answers that question with a coordinate-rotation ladder — Volder's 1959 shift-and-add recurrence — recast for a register that cannot be overwritten, cannot be truncated, and cannot be read mid-computation. Three facts about that recast govern everything downstream:

- **A ladder step is not in-place unless it is unimodular.** The classical CORDIC pseudo-rotation has determinant 1 + 4⁻ⁱ. It cannot be a permutation of a fixed-width register. What *can* be is a shear, `x ← x − σ·(y ≫ i)`, and a rotation is three of them. The reversible ladder is a shear ladder, and the scale factor K that classical CORDIC carries as a constant reappears here as the cost of exactness in the first half of the ladder.
- **Half of the ladder needs no temporaries at all.** For iterations i ≥ n/2, the shear triple's departure from a true rotation is below 4⁻ⁿ/2 — under one unit in the last place — and the direction bits can be read straight from the residual angle register. Measured on a bit-exact reference model at n = 16 to 32 with four guard bits, the half-ancilla ladder lands within 0.9 ULP of the full-precision one. This halves the garbage and halves the direction-bit record.
- **The scaling is not improved. The constant is.** One transcendental costs Θ(n²) Toffoli gates with n = log₂(1/ε_arith) + guard, the same order as a piecewise polynomial evaluated by Horner. The ladder wins on the absence of any variable×variable multiplier, on the fact that `ln`, `exp`, `√`, `sinh` and `cosh` are the same circuit body with a different table, and on a direction-bit record that is itself the angle in signed-digit form. It does not change the exponent of anything.

What that buys, counted end-to-end for a one-asset Heston–Bates path oracle at 50 time steps and ε = 10⁻³: on the order of 10⁵–10⁶ T gates per oracle call, 10⁹–10¹⁰ T gates per price. That is the same neighbourhood as the published threshold estimate for an autocallable — 8k logical qubits, a T-depth of 54 million, and a logical clock of order 10 MHz to finish in about a second — arrived at by a different road. The road matters: the published route avoids path arithmetic by loading a pre-trained variational state. The shear ladder is what you use when the model is a *coupled* SDE — stochastic variance, correlated Brownians, Poisson jumps — where no closed-form loader exists and the coupled path must actually be computed in superposition.

Two further findings follow, and both are placed against the frameworks this document inherits (ASTROLABE, ORRERY and the JPMWM case) rather than beside them:

- **An estimate is not a measurement.** Amplitude estimation returns a value, a half-width and a confidence level, from a transcript of measurement outcomes that will not repeat. ASTROLABE's attested numeral — value, unit, sequence, residual, model, record — has no slot for that. QUADRANT adds one: `Estimated<U>` carries the interval, the query count and the outcome transcript, and the transcript is the record. Under ORRERY's Law 0 the quantum engine is not a second Greek source. It is a reader of D0 vectors and an aggregator of one expectation at a time, and its natural tier is D2 — the collateral book, one loss distribution, one number, high dimension, moderate ε.
- **The ε frontier is the whole argument.** Classical Monte Carlo costs C_c/ε². Amplitude estimation costs C_q/ε. Quantum wins only when ε < C_c/C_q. With a classical path at about a microsecond on one core and a quantum oracle at about a tenth of a second on a 10 MHz logical clock, C_q/C_c ≈ 10⁵, and the crossover sits near ε ≈ 10⁻⁶ in absolute price for a one-core baseline — below a basis point, below model error, and a further 64× lower against a modest CPU farm. The shear ladder moves that frontier by a constant factor. Nothing moves it by an exponent except a smaller oracle.

The recommendation: treat the quantum engine as an instrument for **coupled-path, high-dimensional, single-expectation problems at moderate ε** — a whole collateral book's tail loss, a basket autocallable, a CVA — and build its arithmetic as a shear ladder with a recorded direction sequence, an ancilla-free tail, and a substantiation object that carries the measurement transcript. Do not build it to replace the deterministic register. Two instruments, one sequence number.

---

## 1 · The Instrument

In 1594 the English navigator John Davis published *The Seaman's Secrets* and described an instrument that would, for two centuries, be the standard way to find latitude at sea. The backstaff — the Davis quadrant — is used with the observer's back to the sun. One does not sight the sun; one reads the position of its shadow on a graduated arc, and from the shadow the altitude follows. The sun is too bright to look at. The shadow is not.

An amplitude is a shadow. No quantum register will show you the payoff of a contract along any single path; the value is not there to be read. What can be read, once per query, is the probability that a marked qubit comes up `|1⟩` — the projection of the whole superposition onto one axis — and from a few thousand such readings the expectation follows, to a stated tolerance, at a stated confidence.

Two consequences of that are the framework's spine:

- **Everything before the reading must be a permutation.** The register is not overwritten, truncated, branched on or sampled. Every operation is a bijection of basis states, and the arithmetic must be built from operations that are bijections on a fixed-width word. Section 2 states which linear maps qualify. Fewer do than the classical ladder assumes.
- **The reading is a statistic.** It carries an interval and a confidence, and it will not reproduce. That is a different kind of number from the one ASTROLABE's register emits every 2.5 ns, and Section 8 gives it its own type.

The name also does a second job. Every coordinate-rotation implementation has a quadrant-correction stage — the ±π/2 pre-rotation that brings an arbitrary angle into the ladder's convergence range. In the reversible engine that stage is two direction bits, and they are kept.

---

## 2 · Three Laws of the Reversible Ladder

### Law Q0 · Unimodularity

*An in-place operation on an n-bit register is a permutation of its 2ⁿ states. A linear map with dyadic-rational entries is realisable in place, without temporaries, if and only if its determinant is ±1.*

A classical CORDIC iteration in circular mode is

```
x[i+1] = x[i] − σ_i · y[i] · 2⁻ⁱ
y[i+1] = y[i] + σ_i · x[i] · 2⁻ⁱ
z[i+1] = z[i] − σ_i · arctan(2⁻ⁱ)
```

The (x, y) part is the matrix `[[1, −σt],[σt, 1]]` with t = 2⁻ⁱ. Its determinant is 1 + t² = 1 + 4⁻ⁱ. It scales area, and a map that scales area is not a bijection of a finite lattice. That is exactly why classical CORDIC carries the scale factor K = Π√(1+4⁻ⁱ) ≈ 1.6468 as a constant to divide out at the end — and it is also why the step **cannot** be done in place on a quantum register. Writing the two updates sequentially does not help: `x −= σ(y≫i)` followed by `y += σ(x≫i)` uses the *new* x, and the result differs from the pseudo-rotation by a term `−(y ≫ 2i)` that needs the old y to remove.

What is unimodular is a **shear**:

```
x ← x − σ · (y ≫ i)          det = 1, exact, in place, no temporary
```

and a rotation is a triple of them — the lifting factorisation that Daubechies and Sweldens used in 1998 to make wavelet transforms integer-to-integer:

```
R(θ) = [[1, −tan(θ/2)],[0,1]] · [[1,0],[sinθ,1]] · [[1, −tan(θ/2)],[0,1]]
```

The catch is that tan(α_i/2) for tan α_i = 2⁻ⁱ is not a power of two. A shift-only triple must use `2⁻⁽ⁱ⁺¹⁾, 2⁻ⁱ, 2⁻⁽ⁱ⁺¹⁾`, and the exact matrix that produces is

```
S_i = [[1 − t²/2,   −σt(1 − t²/4)],
       [σt,          1 − t²/2     ]]        t = 2⁻ⁱ,   det S_i = 1
```

which is a rotation by φ_i = 2·arcsin(t/2) **conjugated by a diagonal scaling** `diag(d, 1/d)`, `d = (1 − t²/4)^¼ ≈ 1 − 4⁻ⁱ/16`. The step is exact and reversible; it is a rotation in a frame that differs from Cartesian by 4⁻ⁱ/16 per axis. For i = 0 that is a 6% distortion. For i = n/2 it is 4⁻ⁿ/²/16 — below a unit in the last place of an n-bit word.

One more fact closes the door on any cleverer arrangement: **the only orthogonal matrices with dyadic-rational entries are signed permutations** (c² + s² = 1 with c, s ∈ ℤ[½] forces c, s ∈ {0, ±1}). There is no shift-only step that is both in place and an exact rotation. Every reversible ladder is either garbage-bearing (exact pseudo-rotations with temporaries) or skewed (shear triples). QUADRANT uses both, and Section 3 says where the line falls.

### Law Q1 · A phase is not a number

*A controlled phase rotation on a computational-basis register does not add to it. It adds to its Fourier transform.*

`CRZ(θ)` applied to a qubit of a register in the computational basis multiplies one amplitude by e^{iθ/2} and another by e^{−iθ/2}. The register's *value* is unchanged. Draper's 2000 adder does add constants by phase kicks — but only after a quantum Fourier transform puts the register into the basis where phases *are* the number, and before an inverse transform brings it back. Between those transforms a phase rotation is a value; outside them it is a phase. A ladder step that updates the angle accumulator with `crz` on each bit of `z` has updated nothing that a later sign test can see.

The angle accumulator update `z −= σ·α_i` is a controlled constant addition in the computational basis: order n Toffoli gates with a ripple adder (Cuccaro 2004) or n Toffoli with n measured-out ancillas (Gidney 2018). There is no cheaper way to move a register's value.

### Law Q2 · A control drawn from its own target is not a control

*The direction bit must be copied out before it is used, and the copy is kept. The kept copies are the angle.*

In rotation mode σ_i = sgn(z). The natural circuit reads the sign bit of `z` and uses it to control the shears and the `z` update — but the `z` update changes `z`, sign bit included. A controlled operation whose control lies inside its target is not a unitary with the intended meaning. The sign must be copied to a fresh qubit `d_i` first (one CNOT), and every controlled operation of the step keyed to `d_i`.

`d_i` cannot be uncomputed without reversing the step, so the direction bits accumulate — one per head iteration. That is not waste. The sequence σ₁σ₂…σ_n **is the input angle in signed-digit form**: `z_in = Σ σ_i α_i + r`, where r is the residual left in the register. Classical CORDIC discards the σ sequence because it does not need it. The reversible ladder keeps it because it must, and having kept it, has an exact, invertible record of what was rotated by how much.

### Law Q3 · A fixed shift is a wire; a variable shift is a circuit

*Shifting by a constant i is a relabelling of qubits and costs nothing. Shifting by a data-dependent k is a Fredkin network and costs order n·log n controlled swaps.*

The ladder's `y ≫ i` shifts are fixed per stage: wiring. The **range reduction** in front of a logarithm — `ln w = k·ln 2 + ln w′`, with w′ pre-scaled into the convergence window by a leading-one detector and a barrel shift — is not. The shift amount k is data. On a classical FPGA it is a multiplexer tree; on a quantum register it is a log-depth network of controlled swaps, roughly n·log₂n Fredkin gates, and k itself must be written to a register and kept. Any resource count that books range reduction as free has booked a classical convenience as a quantum one.

---

## 3 · The Ladder

### 3.1 Registers

```
|x⟩   N = n + g bits    coordinate         Q(int).N fixed point, two's complement
|y⟩   N bits            coordinate
|z⟩   N bits            angle accumulator  (rotation mode) / angle output (vectoring mode)
|d⟩   h bits            direction record   h = ⌈N/2⌉ head iterations
|T_i⟩ N − i bits each   head temporaries   i = 0 … h−1, released by whole-function uncompute
|c⟩   N bits            adder carries      measured out (Gidney) or 1 bit (Cuccaro)
```

Guard bits g: the ladder's rounding error grows like log₂N ULP over N stages (Hu 1992 gives the bound for the classical case; the reversible case is identical because the arithmetic is). Four guard bits hold the output within one ULP through N = 36; six through the range measured below.

### 3.2 The head — exact, temporary-bearing, i < h

```
step i (rotation mode):
  d_i     ← MSB(z)                              1 CNOT           the record
  T_i     ← y                                   N−i CNOT         copy, kept dirty
  y       += σ(d_i) · (x ≫ i)                   ~N Toffoli       controlled shear, x still old
  x       −= σ(d_i) · (T_i ≫ i)                 ~N Toffoli       controlled shear, T_i = old y
  z       −= σ(d_i) · α_i                       ~N Toffoli       controlled constant add
```

The sign σ ∈ {−1, +1} is applied by conditionally complementing the operand — XOR every operand bit with d_i, add, and feed d_i in as the carry-in — so a signed controlled shear costs one uncontrolled adder plus 2(N−i) CNOTs, not a controlled adder. The pseudo-rotation is exact because both updates see old values. The price is T_i: it holds the old y, and it cannot be uncomputed from (x_new, y_new) without dividing by 1 + 4⁻ⁱ — the scale factor, again, appearing as the obstruction it always was.

### 3.3 The tail — shear triples, ancilla-free, i ≥ h

```
step i (rotation mode):
  control ← bit of |r| at weight 2⁻ⁱ            0 gates          read, not copied
  x       −= σ_r · (y ≫ (i+1))                  ~(N−i) Toffoli   half-shear
  y       += σ_r · (x ≫ i)                      ~(N−i) Toffoli   shear
  x       −= σ_r · (y ≫ (i+1))                  ~(N−i) Toffoli   half-shear
```

After the head, `z` holds a residual r with |r| < α_{h−1} ≈ 2⁻⁽ʰ⁻¹⁾, and for i ≥ h the micro-angle α_i = arctan(2⁻ⁱ) equals 2⁻ⁱ to within 2⁻³ⁱ — below the word. The bits of r **are** the remaining direction sequence. No sign test, no copy, no z update: the tail rotates by r directly, one shear triple per set bit, controlled by the bit itself. The register z is untouched; it leaves the ladder holding r, and (d, r) together reconstruct the input exactly. This is the reversible form of the hybrid CORDIC that Wang, Piuri and Swartzlander proposed in 1997 — first half iterative, second half by the residual — with the residual applied by shears instead of a multiplier.

### 3.4 Measured accuracy of the split

Reference model, integer arithmetic, floor shifts, 3,000 uniformly random angles in [−1.5, 1.5], output compared to double-precision cos and sin, error in units of the n-bit ULP:

| n | guard | head h | tail | max err / ULP (half head) | max err / ULP (full head) |
|---|---|---|---|---|---|
| 16 | 0 | 8 | 9 | 8.3 | 4.8 |
| 16 | 4 | 8 | 13 | 0.7 | 0.5 |
| 20 | 4 | 10 | 15 | 0.8 | 0.5 |
| 24 | 4 | 12 | 17 | 0.8 | 0.5 |
| 32 | 4 | 16 | 21 | 0.9 | 0.6 |
| 32 | 6 | 16 | 23 | 0.3 | 0.2 |

The half-head ladder costs at most 0.4 ULP over the full-head one at four guard bits and is indistinguishable at six. The skew of the tail's shear triples, at ≤ 4⁻ʰ/16 per axis, is invisible; the guard bits are what matter, exactly as in the classical case.

### 3.5 Cost per function

Toffoli count, N-bit word, h = N/2, before uncomputation:

```
head:   h · (2N + N)                       ≈ 1.5 N²        two shears + z update per step
tail:   3 · Σ_{i=h}^{N} (N − i)            ≈ 0.375 N²      three shears of shrinking width
range reduction (ln, √ only):  N log₂ N     leading-one + Fredkin barrel shift
scale correction:              N²           constant multiply by 1/K, shift-and-add
                                           ─────────
                                           ≈ 1.9 N²  (+ N² if the scale must be applied)
```

| N | Toffoli (ladder) | T (×4, Jones 2013) | T with whole-function uncompute | qubits (3N + 3N²/8 + N/2 + N) |
|---|---|---|---|---|
| 16 | ~490 | ~2.0k | ~3.9k | ~176 |
| 20 | ~760 | ~3.0k | ~6.1k | ~240 |
| 24 | ~1,100 | ~4.4k | ~8.8k | ~324 |
| 32 | ~1,950 | ~7.8k | ~15.6k | ~528 |

The N² term in qubits is the head's temporaries. Bennett's 1989 pebble game trades it for time: O(N log N) space at a constant-factor time penalty, which is the right trade whenever logical qubits are dearer than T gates — which is every fault-tolerant architecture on the roadmap.

For comparison, a degree-4 piecewise polynomial in Horner form costs four N×N multiplications — about 4N² Toffoli with schoolbook, so roughly 2× the ladder at the same N — plus a table lookup for the interval coefficients. The two are the same order. The ladder is preferred for a different reason than asymptotics: **it has no variable×variable multiplier anywhere in it**, its stages are identical up to a table constant, and `exp`, `ln`, `√`, `sinh`, `cosh` and `atanh` are all the same circuit body. One module, one verification, five functions.

### 3.6 Hyperbolic mode

Setting the curvature parameter m = −1 (Walther 1971) turns the ladder into `sinh`, `cosh`, `exp`, `ln`, `√` and `atanh` with the table `artanh(2⁻ⁱ)`, the same shears, and two well-known extra rules:

- Iterations 4, 13, 40, 121, … (i → 3i + 1) are executed twice for convergence. In the reversible ladder each repeat is an extra stage — an extra temporary and direction bit in the head, an extra shear triple in the tail. A pipeline sized on the nominal count is three stages short at N = 40, and its scale factor is wrong, because K_h is a product over the *executed* sequence.
- Vectoring-mode `ln w` needs w′ ∈ [0.125, 8.0] after range reduction; the convergence window is [0.106848, 9.359071] and the pre-scale must land well inside it. Under Law Q3 that pre-scale is a Fredkin barrel shift and a kept exponent register.

`√w` seeds vectoring with x₀ = w + ¼, y₀ = w − ¼ and reads K_h·√w from x. `ln w` seeds x₀ = w + 1, y₀ = w − 1 and reads ½·ln w from z — from the direction record, under Law Q2. `exp z` seeds x₀ = y₀ = 1/K_h in rotation mode and reads x + y.

### 3.7 Reference model

```python
def ladder(z0, n, g=4):
    """Bit-exact model of the head/tail reversible circular ladder.
    Returns (cos, sin, direction_bits, residual). Integer arithmetic only."""
    import math
    N = n + g; F = 1 << N; h = N // 2
    atan = [int(round(math.atan(2.0 ** -i) * F)) for i in range(N + 2)]
    x, y, z, d = F, 0, int(round(z0 * F)), []
    for i in range(h):                       # head: exact pseudo-rotations, temporaries kept
        s = 1 if z >= 0 else -1; d.append(s)
        T = y                                #   T_i is the kept temporary
        y = y + s * (x >> i)                 #   sees old x
        x = x - s * (T >> i)                 #   sees old y
        z = z - s * atan[i]
    K = math.prod(math.sqrt(1 + 4.0 ** -i) for i in range(h))
    s, r = (1 if z >= 0 else -1), abs(z)     # residual angle; its bits are the tail's controls
    for i in range(h - 1, N + 1):            # tail: shear triples, no temporaries, no z update
        if r & (1 << (N - i)):
            x = x - s * (y >> (i + 1))
            y = y + s * (x >> i)
            x = x - s * (y >> (i + 1))
    return x / F / K, y / F / K, d, z
```

Every line is a permutation of the register or a kept copy. Nothing is truncated that was not already below the word, and nothing is read that is not in a register.

---

## 4 · What the Ladder Does Inside a Pricing Oracle

### 4.1 The model

The point of doing path arithmetic at all is a model that has no closed-form loader. The 1993 Heston model with the 1996 Bates jump extension:

```
dS_t = μ S_t dt + √V_t S_t dW¹_t + (J − 1) S_t dN_t
dV_t = κ(θ − V_t) dt + ξ √V_t dW²_t          E[dW¹ dW²] = ρ dt
```

The variance path V_t is not a sum of independent increments; it feeds back into itself and into S. There is no product-form distribution to load with Grover–Rudolph, and even where there is, Herbert showed in 2021 that Grover–Rudolph loading of a log-concave distribution costs enough to cancel the quadratic speedup on its own. The coupled path has to be *computed*, step by step, in superposition, from Gaussian and Poisson increment registers that are cheap to prepare precisely because they are independent.

### 4.2 One time step, in ladder operations

Euler–Maruyama on log-price and full-truncation on variance, per asset, per step:

```
√V_k              1 hyperbolic vectoring ladder                   ~1.9 N² Toffoli
ξ √V_k √Δt ΔW²_k  1 variable×variable multiply (linear-mode ladder) ~N² Toffoli
√V_k √Δt ΔW¹_k    1 variable×variable multiply                     ~N² Toffoli
κ(θ − V_k)Δt      constant multiply, shift-and-add                 ~N log N
V_k/2 · Δt        wire (shift) + add                                ~N
jump indicator    QROM lookup on the Poisson register              ~2^b Toffoli, b bits
ln J              QROM lookup on the jump-size register             ~2^b Toffoli
────────────────────────────────────────────────────────────────────────────────
per step          ≈ 4 N² Toffoli                                    N = 20 → ~1,600 Toffoli ≈ 6.4k T
```

The transcendental has become the *cheaper* half. The two variable×variable products — `√V · ΔW` — are the floor, and they stay N² whether they are done by a linear-mode ladder or a schoolbook multiplier. That is the correct place for a designer's attention.

### 4.3 The oracle

```
N_steps = 50, one asset, N = 20:
  path arithmetic                    50 × 6.4k          ≈  3.2 × 10⁵ T
  payoff: exp(ln S_T − ln K), max, scale                 ≈  1.0 × 10⁴ T
  angle encoding into the marked qubit (Φ or arcsin)     ≈  1.0 × 10⁴ T
  whole-function uncompute (Bennett)                     ×2
  ───────────────────────────────────────────────────────────────
  one oracle call A                                      ≈  7 × 10⁵ T
  one Grover iterate Q = A S₀ A† S_χ                     ≈  1.4 × 10⁶ T
```

Amplitude estimation to absolute error ε = 10⁻³ at confidence 1 − α needs of order (π/2ε)·log(1/α) applications of Q in the iterative scheme — call it 5 × 10³ — giving **≈ 7 × 10⁹ T gates per price**. At a logical T rate of 1 MHz that is two hours; at 10 MHz, twelve minutes; at 100 MHz, seventy seconds. The published autocallable estimate — 8k logical qubits, T-depth 5.4 × 10⁷, and the stated goal of running in about a second — is two orders of magnitude lighter because its re-parameterisation method replaces this entire path oracle with a pre-trained loader. Where that loader exists, use it. Where the SDE is coupled, it does not, and the ladder is the floor.

### 4.4 Two ways to spend the direction record

The kept direction bits are not merely garbage awaiting uncompute.

- **Vectoring output is the record.** In vectoring mode the angle is *only* ever in the σ sequence plus the residual. `ln`, `atanh` and `arctan` read their answer from `d`, not from `z`; the accumulator exists to be zeroed. Burge, Barbeau and Garcia-Alfaro's reversible arcsine (arXiv 2411.14434, to appear at IEEE QCE 2026 in Toronto next week) is this mode, and reports order-n qubits, order n·log n layers and order n² CNOTs — the same shape as Section 3.5.
- **The record is a checksum.** Σ σ_i α_i + r must equal the input angle exactly. A stuck direction qubit, a dropped repeat stage or a pre-scale outside the window breaks that identity, and it can be checked by an adder against the untouched input copy at the end of the head. This is ASTROLABE's PDE residual moved into the arithmetic: a conservation law checked beside the output path, not in front of it.

---

## 5 · The ε Frontier

Every claim of quantum advantage in pricing reduces to one inequality, and it is worth writing down with numbers in it.

```
T_classical(ε) = C_c / ε²           C_c = cost of one classical path
T_quantum(ε)   = C_q / ε            C_q = cost of one oracle call × (π/2)·log(1/α)

advantage  ⇔  ε  <  C_c / C_q
```

| Baseline | C_c | C_q at 10 MHz T-rate | Crossover ε | In basis points of a $100 price |
|---|---|---|---|---|
| 1 CPU core, 50-step Heston–Bates path ≈ 1 µs | 1 µs | 7 × 10⁵ T ≈ 0.07 s, × ~1,600 ≈ 110 s | ≈ 10⁻⁵ … 10⁻⁶ | 0.1 … 0.01 bp |
| 64 cores | 16 ns | 110 s | ≈ 1.5 × 10⁻⁷ | 0.0015 bp |
| One deterministic FPGA lane (HELICON), 372.5 ns per full Greek vector, pipelined at 2.5 ns | 2.5 ns per evaluation | 110 s | ≈ 2 × 10⁻⁸ | 0.0002 bp |
| Same, with the re-parameterised loader (C_q ÷ 100) | — | 1.1 s | ≈ 10⁻³ … 10⁻⁵ | 10 … 0.1 bp |

A basis point is 10⁻⁴. Against a single core the quantum engine wins below a tenth of a basis point; against a rack it wins below a hundredth; against a pipelined lane it wins below a thousandth. Model error on a Heston–Bates calibration is larger than all three. The ladder shifts C_q by perhaps 2×. The loader shifts it by 100×. **Only the loader moves the frontier into a region a desk cares about**, and only for contracts whose distribution the loader can carry.

What survives the arithmetic is a narrower and better claim: the quantum engine is worth building for problems where (a) the dimension is high enough that classical variance does not fall with smarter sampling, (b) one expectation is wanted, not a million, and (c) ε in the 10⁻³ to 10⁻⁴ range is the target because the number is a capital figure, not a quote. Section 8 names that problem.

---

## 6 · The Simulator Cannot Testify

A statevector of the three N-bit ladder registers at N = 12 is 2³⁶ complex amplitudes — 68.7 billion, about 1.1 TB in double precision. Every gate touches every amplitude. A simulated amplitude-estimation run therefore costs 2^q operations per gate times the gate count, and its wall-clock time measures the simulator's memory bandwidth, not the algorithm's query complexity. A run that reports 0.18 seconds against 42 seconds of classical Monte Carlo has measured a circuit that fits in memory, which at three 12-bit registers this one does not, or a circuit that does nothing: a ladder whose every controlled gate is keyed to a register initialised to `|0…0⟩` applies the identity to it, at any depth the transpiler chooses to report.

The quantities that can testify, and the only ones the published threshold work reports, are:

```
logical qubits · T-count · T-depth · code distance · logical clock rate
```

A benchmark in seconds is meaningful for exactly one thing: the classical baseline. Report it there, and report the quantum side in T gates and the clock rate needed to run them in the same time.

---

## 7 · Estimation

### 7.1 Three estimators, one interval

| Scheme | Mechanism | Queries for ε, α | Extra qubits | Output |
|---|---|---|---|---|
| Canonical (Brassard et al. 2002) | Phase estimation on Q | O(1/ε) | m = log₂(1/ε) + 2 | point + Heisenberg-limited error |
| MLQAE (Suzuki et al. 2020) | Grover powers 2^k, maximum likelihood | O(1/ε) up to log factors | none | point + Fisher interval |
| IQAE (Grinko et al. 2021) | Adaptive Grover powers, Chernoff–Hoeffding / Clopper–Pearson bounds | ≤ (π/2ε)·log(2/α)·const | none | **rigorous interval [a_lo, a_hi] at confidence 1 − α** |

QUADRANT specifies IQAE, for one reason that overrides its constant factors: it returns an *interval with a proof*. The value, its half-width and the confidence level are outputs of a deterministic post-processing of a transcript of measurement outcomes. Two runs will give different transcripts and different intervals; every transcript is auditable, and the interval it implies is recomputable by anyone holding it.

### 7.2 Greeks

Two routes, and they are not equivalent under attestation:

- **Finite differences of estimates.** Price at S and S + h, subtract. The interval on the difference is the sum of the intervals; for Γ it is four intervals over h². This is how a simulator does it and it is how an attested numeral must *not* be produced, because the residual explodes.
- **Quantum gradient estimation** (Stamatopoulos, Mazzola, Woerner and Zeng 2022) computes all first-order sensitivities in one run at O(1/ε) total, and the same authors put the logical clock for advantage on a basket's Greeks near 7 MHz — lower than the pricing threshold, because the Greeks share the oracle.

The second route produces Δ, ν, Θ and ρ with intervals, and those intervals let the PDE identity be checked *statistically*:

```
|Θ + (r − q)SΔ + ½σ²S²Γ − rC|  ≤  Σ half-widths (propagated)    ⇒  PASS
                                >  Σ half-widths                  ⇒  FAULT
```

An identity that holds only to within the confidence interval is still an identity. Section 8 calls this the interval residual and makes it a field.

---

## 8 · Placement — Where a Statistical Instrument Sits in a Deterministic Stack

### 8.1 An estimate is not a measurement

ASTROLABE's register emits one attested vector per 2.5 ns, deterministically, with a residual at one ULP. Its type:

```rust
pub struct Attested<U: Unit> { value: Q24_40, unit: U, seq: Seq, residual: Q24_40, model: ModelId, record: RecordId }
```

Nothing an amplitude-estimation engine produces fits that type. The value is a point inside an interval; the residual is a bound, not a floor; the run does not reproduce. Rather than force it, QUADRANT adds a second type, and a rule about mixing them:

```rust
/// A figure produced by amplitude estimation. Reaches a client only as an interval.
pub struct Estimated<U: Unit> {
    value:      Q24_40,        // IQAE point estimate (midpoint)
    halfwidth:  Q24_40,        // rigorous half-width at `confidence`
    confidence: Q0_16,         // 1 − α
    unit:       U,
    seq:        Seq,           // the D0 snapshot the oracle's inputs were read at
    queries:    u32,           // total Grover applications
    transcript: TranscriptId,  // the measurement outcomes; the interval is a pure function of these
    residual:   Q24_40,        // interval residual — PDE identity slack after propagation
    model:      ModelId,       // lognormal | heston | bates | bachelier
    record:     RecordId,
}

impl<U: Unit> Add<Estimated<U>> for Attested<U> {
    type Output = Result<Estimated<U>, Incomparable>;   // attested + estimated = estimated, never attested
    // seq must match; model must match; the sum's half-width is the estimate's; nothing is silently narrowed.
}
```

Three consequences:

- **An interval never collapses to a point by arithmetic.** Adding a deterministic figure to an estimate yields an estimate. A statement, a margin call or an assistant response that contains an `Estimated<U>` prints the interval, or it does not print the figure.
- **The transcript is the record.** ASTROLABE's `replay/` reconstructs any figure from (record, seq). For an estimate, replay reconstructs the *interval* from (transcript, seq) — bit-for-bit, because the post-processing is deterministic — and cannot reconstruct the point any other way. The March 2024 fact pattern — a figure with nothing behind it — is answered by a transcript, not a rerun.
- **Coherence is preserved.** The oracle reads D0 vectors and position records at one seq; the estimate inherits that seq; the coherence ratio of an artefact containing it is unaffected.

### 8.2 Under ORRERY's laws

| Law | What it says | Where QUADRANT lands |
|---|---|---|
| **Law 0 — One Greek source** | No consumer computes its own sensitivities | The quantum engine is **not** a second source. It reads the register's vectors and position records as oracle inputs. Its outputs are expectations over scenarios, typed `Estimated<U>`, never fed back as sensitivities. A tier that links the estimator as a pricer fails the build. |
| **Law I — Constraints down, measurements up** | Slow tiers emit bounds; fast tiers act | An interval is a bound. `Estimated<U>` flows *downward* as a constraint (a stressed-loss envelope, a capital figure) and never as an order. The estimator is structurally a slow tier. |
| **Law II — Clock-domain crossing** | A slow reader never samples a fast writer mid-update | The oracle's inputs are a copy-on-write snapshot at a named seq. A run that takes twelve minutes reads a book frozen twelve minutes ago and says so. |

### 8.3 The natural tier is D2

Run the four tiers against the ε frontier:

| Tier | What it wants | ε it needs | Count of expectations | Verdict |
|---|---|---|---|---|
| D1 hedging | Δ, Γ, ν per contract, microseconds | quoting-grade, 10⁻⁴ | 10⁴ per 20 ms | Deterministic register. The estimator is 10⁸× too slow and wants the wrong output. |
| D4 advisory | Household stress paths, nightly | 10⁻³ per household | 2 × 10⁶ households | One expectation per household is 2 × 10⁶ runs. Not this instrument. |
| D3 rebalancing | After-tax objective, lot-level | 10⁻⁴ | thousands of candidates | Optimiser, not integrator. |
| **D2 collateral** | **Tail of the loss distribution of the whole pledged book** | **10⁻³ … 10⁻⁴ of book** | **one** | **One expectation, dimension = positions × risk factors, intraday deadline. This is the problem.** |

On the JPMWM numbers — $284 billion of average loans in the wealth segment, growing 18%, substantially securities-backed — the collateral cycle is one loss distribution over half a million accounts and forty positions each. Woerner and Egger's 2019 quantum risk analysis is exactly VaR and CVaR of a portfolio by amplitude estimation. Classically that cycle is 50 seconds on one deterministic lane at a thousand scenarios; the estimator's opening is not throughput but *dimension* — a coupled-factor model with stochastic volatility and jumps across the whole book, where classical variance reduction gives up and one number, to a basis point of the book, before the next cycle, is the deliverable.

```
D2 QUANTUM AGGREGATOR — one cycle
  inputs   position snapshot @ seq n, D0 Greek vectors @ seq n, factor model (bates, ρ-matrix), stress set
  oracle   coupled-factor path ladder → portfolio loss → marked qubit
  estimator IQAE, ε = 10⁻³ of book, 1 − α = 0.99
  output   Estimated<Usd>{ value, halfwidth, confidence, seq: n, queries, transcript }
  emits    ↓ margin envelope (a bound)          ↑ IM consumed, headroom (observations)
  never    a haircut per account, a Greek, an order
```

### 8.4 What the JPMWM stack does with it

The five surfaces still read one register at one seq. The assistant still binds slots to attested numerals or blocks. What changes is one line in the manifest:

```
  {D}  $1.42B  ±$0.14B  @99%   loss_tail_book   rec:COL-Q3   seq:0x7A22E1   transcript:T-88a1  D2 estimate
  MODEL bates   RESIDUAL interval 2.1e−3   WINDOW ok   COHERENCE 1.000
```

A figure with a half-width beside it, a confidence beside that, and a transcript that any examiner can replay to the same interval. The substantiation object the firm already prints on its marketing footers, extended to a number it could not previously produce at all.

---

## 9 · Failure Atlas

| # | Failure | Where it bites | Guard |
|---|---|---|---|
| Q-F1 | Pseudo-rotation applied in place | det 1 + 4⁻ⁱ; the register state is not a permutation; results are wrong by the scale factor, silently | Law Q0 — shears only; exact head with temporaries; skewed tail below ULP |
| Q-F2 | Phase rotation used as a value update | `z` never changes; every subsequent sign test reads the input | Law Q1 — computational-basis adders; Fourier-basis only between QFT pairs |
| Q-F3 | Control taken from the MSB of the register being updated | Non-unitary intent; direction flips mid-step | Law Q2 — copy the sign to `d_i`, key everything to the copy, keep the copy |
| Q-F4 | Variable shift booked as wiring | Range reduction for `ln` and `√` costs N log N Fredkins and a kept exponent | Law Q3 — count the barrel shifter; keep k |
| Q-F5 | All controls keyed to an all-zero register | The compiled circuit is the identity; depth is reported anyway | Load inputs; test on a nonzero basis state; check the direction-record identity |
| Q-F6 | Wall-clock benchmark of a simulated estimator | Measures memory bandwidth, not query complexity; 2³⁶ amplitudes do not fit | Report T-count, T-depth, logical qubits, clock rate; seconds only for the classical side |
| Q-F7 | Grover–Rudolph loading of the path distribution | Loading cost cancels the quadratic speedup on its own | Independent-increment registers plus computed coupled path, or a pre-trained loader where the model admits one |
| Q-F8 | Repeat stages omitted in hyperbolic mode | Three stages short at N = 40; K_h wrong | Repeat schedule derived from i → 3i + 1, never hand-listed; direction-record checksum |
| Q-F9 | Pre-scale outside [0.125, 8.0] | Deterministic garbage at every query | Window guard qubit set at ingress; oracle marks the state, estimator sees a flagged fraction |
| Q-F10 | Guard bits omitted | 5–15 ULP error at N = 16–32 | 4 guard bits to N ≈ 36; 6 beyond |
| Q-F11 | Finite-difference Greeks from estimates | Interval residual explodes as 1/h² | Quantum gradient; interval residual as a typed field |
| Q-F12 | Estimate typed as `Attested<U>` | A point with no interval reaches a client | `Estimated<U>`; attested + estimated = estimated; interval printed or figure blocked |
| Q-F13 | Estimator deployed at D1 or D4 | 10⁸× too slow, or 2 × 10⁶ runs | Placement rule — one expectation, high dimension, moderate ε: D2 |
| Q-F14 | Negative underlying in log-space oracle | `ln` undefined; ladder does not converge; flagged fraction → 1 | `model` field; Bachelier bypass in the oracle; sign guard at ingress |
| Q-F15 | Advantage asserted without the ε frontier | "Quadratic speedup" with C_q/C_c ≈ 10⁵ unmentioned | State C_c, C_q, the crossover ε, in basis points of the price |
| Q-F16 | Speedup asserted on scaling of the arithmetic | Θ(N²) ladder vs Θ(N²) polynomial; no exponent moved | Claim the constant, the regularity and the absent multiplier; nothing else |

---

## 10 · Reference Layout

```
quadrant/
├── ladder/                      ← reversible shift-and-add core
│   ├── shear.qsl                signed controlled shear: XOR-complement, add, carry-in
│   ├── head.qsl                 exact pseudo-rotation; T_i kept; d_i kept
│   ├── tail.qsl                 shear triples keyed to residual bits; no z update
│   ├── circular.qsl             m = +1, arctan table
│   ├── hyperbolic.qsl           m = −1, artanh table, repeat schedule i → 3i+1
│   ├── linear.qsl               m = 0, multiply / divide by shears
│   ├── prescale.qsl             leading-one detector + Fredkin barrel shift; k kept
│   ├── scale.qsl                1/K, 1/K_h as constant shift-and-add
│   └── checksum.qsl             Σ σ_i α_i + r == z_in   → FAULT qubit
├── pebble/
│   └── bennett89.py             head-temporary schedule: space O(N log N) vs time
├── oracle/
│   ├── increments/              independent Gaussian (Grover–Rudolph, cheap), Poisson, jump-size QROM
│   ├── bates_step.qsl           √V ladder, two products, drift, jump; one step per call
│   ├── payoff/                  vanilla, autocallable, basket, portfolio-loss
│   ├── angle_encode.qsl         value → marked-qubit rotation (arcsin ladder or Φ)
│   └── bachelier_step.qsl       price-space path for the model field
├── estimate/
│   ├── iqae.rs                  adaptive Grover powers; Chernoff–Hoeffding / Clopper–Pearson
│   ├── mlqae.rs                 fallback
│   ├── gradient.qsl             all first-order sensitivities in one run
│   └── transcript.rs            outcomes → interval, deterministic, replayable
├── ledger/
│   ├── estimated.rs             Estimated<U>: value, halfwidth, confidence, seq, queries, transcript, residual
│   ├── mixing.rs                Attested + Estimated → Estimated; never the reverse
│   └── interval_residual.rs     PDE identity with propagated half-widths
├── d2_aggregator/
│   ├── snapshot.rs              copy-on-write book @ seq, frozen for the run
│   ├── loss_oracle.qsl          coupled-factor path over the pledged book
│   ├── envelope.rs              emits the margin bound downward
│   └── cycle_budget.rs          wall-clock per cycle against the logical clock rate
└── cost/
    ├── tcount.py                per-function and per-oracle T-count, N, g, steps, assets
    ├── frontier.py              C_c, C_q, crossover ε, in basis points
    └── clock.py                 T gates ÷ wall-clock target → required logical T-rate
```

### 10.1 Build

```bash
# tables: derived, never typed; asserts the repeat schedule and the window
python3 tools/gen_tables.py --mode hyperbolic --bits 20 --guard 4 --repeats auto --out ladder/rom_atanh.qsl

# the split: head/tail boundary from the ULP, not from a constant
python3 tools/split.py --bits 20 --guard 4          # → h = 12, tail skew 4^-12/16 = 3.7e-9 < ulp 9.5e-7

# cost of one function, one step, one oracle, one price
python3 cost/tcount.py --bits 20 --guard 4 --steps 50 --assets 1 --eps 1e-3 --alpha 0.01

# where the frontier is, in basis points, against three classical baselines
python3 cost/frontier.py --classical-path-us 1.0 --cores 1,64 --lane-ns 2.5 --trate-mhz 1,10,100

# the type gate: an estimate cannot masquerade as a measurement
cargo build --features deny-unbound-numerals,deny-pointwise-estimates
```

---

## 11 · Nine Predictions

Stated so that each can be wrong.

**P1 · The first end-to-end fault-tolerant derivative price at parity with a CPU farm will be for a coupled-SDE contract, not a lognormal one.** Where the loader is closed-form, classical shortcuts are strongest and the ε frontier is furthest away; where the path must be computed, the two sides pay the same arithmetic and the estimator's 1/ε shows. Falsifier: the first claimed parity result, by 2031, is a vanilla or GBM-basket contract.

**P2 · Multiplier-free shear ladders become the default transcendental unit in fault-tolerant arithmetic libraries by 2028.** One circuit body, five functions, no variable×variable product, a direction record that doubles as a checksum. Falsifier: leading resource estimators in 2028 still costing `exp`, `ln` and `√` exclusively as polynomial-times-multiplier.

**P3 · Resource estimates split the ladder into a garbage-bearing head and a garbage-free tail, and quote the boundary.** h = ⌈N/2⌉ is a property of the word width, not a tuning. Falsifier: published ladder costings by 2028 that quote one temporary per iteration through the full depth.

**P4 · No demonstrated end-to-end advantage above ε = 10⁻⁵ in absolute price before 2030; the first demonstrated advantages are tail-risk aggregates, not prices.** One expectation, high dimension, capital-grade ε. Falsifier: a single-contract pricing advantage at basis-point precision against a modest CPU farm before 2030.

**P5 · The attested-numeral type grows an estimate variant, and examiners ask for the transcript.** A figure with a half-width and a confidence is a different object from a figure with a residual, and it will be typed as such wherever the register discipline already exists. Falsifier: quantum-estimated figures reaching client artefacts as points, with no interval and no replayable transcript, at any platform running an attested register in 2029.

**P6 · The first production quantum engine on a wealth platform lands in the collateral tier.** It is the only tier that wants one number, at moderate ε, with a legal deadline and a growing book. Falsifier: a first deployment at D1 (quoting) or D4 (per-household advisory) rather than D2.

**P7 · Logical T-rate replaces qubit count as the headline metric for finance-relevant hardware.** Seven billion T gates per price is a clock problem, not a qubit problem. Falsifier: hardware roadmaps in 2028 still leading with physical or logical qubit counts and no stated logical clock.

**P8 · The direction record is reused as a data format.** A signed-digit angle sequence is a compact, exactly invertible encoding of a rotation; it will be stored and transported as such, first inside the engine and then between engines. Falsifier: no published use of CORDIC direction sequences as a persisted representation by 2029.

**P9 · The deterministic register stays.** Through 2032 the D0 source on any platform running both instruments is the classical fixed-point lane; the quantum engine reads it and never replaces it, and both stamp the same seq. Falsifier: a platform in 2032 sourcing its per-contract Greeks from an amplitude estimator.

---

## 12 · Notation

| Symbol | Meaning |
|---|---|
| n, g, N | Output fractional bits; guard bits; working width N = n + g |
| h | Head length ⌈N/2⌉ — last iteration with kept temporaries |
| σ_i, d_i | Direction of iteration i; its kept copy (the record) |
| α_i, t | arctan(2⁻ⁱ) or artanh(2⁻ⁱ); t = 2⁻ⁱ |
| T_i | Head temporary holding y before step i, width N − i |
| r | Residual angle after the head; its bits control the tail |
| S_i | Shear triple (t/2, t, t/2): rotation by 2·arcsin(t/2) conjugated by diag(d, 1/d), d = (1 − t²/4)^¼ |
| K, K_h | Circular / hyperbolic scale over the *executed* head; 1.6468 / 0.8282 at full depth |
| m | Curvature: +1 circular, 0 linear, −1 hyperbolic |
| A, Q | Oracle unitary; Grover iterate Q = A S₀ A† S_χ |
| ε, α | Absolute estimation error; 1 − confidence |
| C_c, C_q | Cost of one classical path; cost of one quantum Grover iterate × log(1/α) |
| ε* | Crossover C_c / C_q; advantage iff ε < ε* |
| `Attested<U>` | ASTROLABE's deterministic numeral (value, unit, seq, residual, model, record) |
| `Estimated<U>` | QUADRANT's statistical numeral (+ halfwidth, confidence, queries, transcript) |
| Interval residual | PDE identity slack after propagating half-widths |
| D0–D4 | Register, hedging, collateral, rebalancing, advisory (ORRERY) |
| seq | Gray-coded monotone sequence number; the only ordering |

---

## 13 · Lineage

- **1594** — John Davis, *The Seaman's Secrets*. The backstaff: read the shadow, not the sun.
- **1959** — Volder, "The CORDIC Trigonometric Computing Technique," *IRE Trans. Electronic Computers* EC-8(3). Rotation by shift and add.
- **1971** — Walther, "A unified algorithm for elementary functions," AFIPS SJCC. Three geometries, one parameter m.
- **1973** — Bennett, "Logical reversibility of computation," *IBM J. Res. Dev.* 17(6). Compute, copy, uncompute.
- **1989** — Bennett, "Time/space trade-offs for reversible computation," *SIAM J. Comput.* 18(4). The pebble game the head plays.
- **1992** — Hu, "The quantization effects of the CORDIC algorithm," *IEEE Trans. Signal Processing* 40(4). Where the guard bits come from.
- **1993 / 1996** — Heston, *Rev. Financial Studies* 6(2); Bates, *Rev. Financial Studies* 9(1). Stochastic variance; jumps. The coupled path.
- **1997** — Wang, Piuri, Swartzlander, "Hybrid CORDIC algorithms," *IEEE Trans. Computers* 46(11). Half iterative, half residual.
- **1998** — Daubechies & Sweldens, "Factoring wavelet transforms into lifting steps," *J. Fourier Anal. Appl.* 4(3). A rotation is three shears.
- **2000** — Draper, "Addition on a quantum computer," quant-ph/0008033. Phases add in the Fourier basis.
- **2002** — Brassard, Høyer, Mosca, Tapp, "Quantum amplitude amplification and estimation," *Contemp. Math.* 305. The 1/ε estimator. Grover & Rudolph, quant-ph/0208112. The loader.
- **2004** — Cuccaro, Draper, Kutin, Moulton, quant-ph/0410184. The ripple-carry adder.
- **2013** — Jones, *Phys. Rev. A* 87, 022328. Four T gates per Toffoli.
- **2015** — Montanaro, *Proc. R. Soc. A* 471, 20150301. Quantum speedup of Monte Carlo, in general.
- **2018** — Rebentrost, Gupt, Bromley, *Phys. Rev. A* 98, 022321. Option pricing by amplitude estimation. Häner, Roetteler, Svore, arXiv 1805.12445. Piecewise polynomial arithmetic. Gidney, *Quantum* 2, 74. Halving the adder. Babbush et al., *Phys. Rev. X* 8, 041015. QROM.
- **2019** — Woerner & Egger, *npj Quantum Information* 5, 15. VaR and CVaR by amplitude estimation — the D2 problem.
- **2020** — Stamatopoulos et al., *Quantum* 4, 291. Option pricing on hardware. Suzuki et al., *Quantum Inf. Process.* 19, 75. Estimation without phase estimation.
- **2021** — Grinko, Gacon, Zoufal, Woerner, *npj Quantum Information* 7, 52. Iterative amplitude estimation with rigorous intervals. Herbert, *Phys. Rev. E* 103, 063302. No speedup with Grover–Rudolph loading. Chakrabarti, Krishnakumar, Mazzola, Stamatopoulos, Woerner, Zeng, *Quantum* 5, 463. The threshold: 8k logical qubits, T-depth 54 million, about a second.
- **2022** — Stamatopoulos, Mazzola, Woerner, Zeng, *Quantum* 6, 770. Greeks by quantum gradient; the clock threshold moves to ~7 MHz.
- **2023** — Herman et al., *Nature Reviews Physics* 5, 450–465. Quantum computing for finance, surveyed.
- **2024** — Burge, Barbeau, Garcia-Alfaro, arXiv 2411.14434. Reversible CORDIC arcsine: order-n qubits, n·log n layers, n² CNOTs.
- **13–18 September 2026** — IEEE QCE 2026, Toronto. Quantum CORDIC arcsine and a CORDIC ray-marching implementation on the accepted-papers list. The ladder has a conference track.
- **September 2026** — ORRERY, ASTROLABE and the JPMWM case. One register, one seq, one type. QUADRANT adds the second type and names the tier.

---

## 14 · Sources

- Chakrabarti, S., Krishnakumar, R., Mazzola, G., Stamatopoulos, N., Woerner, S., Zeng, W. J. — *A Threshold for Quantum Advantage in Derivative Pricing*, Quantum 5, 463 (2021); arXiv 2012.03819. Reported resources for an autocallable and a TARF: 8k logical qubits, T-depth 54 million, target runtime of order one second; an earlier version stated 7.5k logical qubits, T-depth 46 million, logical clock ~10 MHz.
- Herbert, S. — *No quantum speedup with Grover-Rudolph state preparation for quantum Monte Carlo integration*, Phys. Rev. E 103, 063302 (2021); arXiv 2101.02240.
- Burge, I., Barbeau, M., Garcia-Alfaro, J. — *Quantum CORDIC — Arcsine on a Budget*, arXiv 2411.14434; accepted, IEEE QCE 2026, Toronto, 13–18 September 2026. Companion repository github.com/iain-burge/QuantumCORDIC.
- IEEE QCE 2026 accepted technical papers, QALG track — including *Quantum Ray Marching: A CORDIC Implementation* (Gordon & Gordon).
- Stamatopoulos, N., Mazzola, G., Woerner, S., Zeng, W. J. — *Towards Quantum Advantage in Financial Market Risk using Quantum Gradient Algorithms*, Quantum 6, 770 (2022).
- Woerner, S., Egger, D. J. — *Quantum risk analysis*, npj Quantum Information 5, 15 (2019).
- Grinko, D., Gacon, J., Zoufal, C., Woerner, S. — *Iterative quantum amplitude estimation*, npj Quantum Information 7, 52 (2021).
- Suzuki, Y., Uno, S., Raymond, R., Tanaka, T., Onodera, T., Yamamoto, N. — *Amplitude estimation without phase estimation*, Quantum Inf. Process. 19, 75 (2020).
- Brassard, G., Høyer, P., Mosca, M., Tapp, A. — *Quantum amplitude amplification and estimation*, Contemp. Math. 305 (2002).
- Montanaro, A. — *Quantum speedup of Monte Carlo methods*, Proc. R. Soc. A 471, 20150301 (2015).
- Rebentrost, P., Gupt, B., Bromley, T. R. — *Quantum computational finance: Monte Carlo pricing of financial derivatives*, Phys. Rev. A 98, 022321 (2018).
- Stamatopoulos, N., Egger, D. J., Sun, Y., Zoufal, C., Iten, R., Shen, N., Woerner, S. — *Option Pricing using Quantum Computers*, Quantum 4, 291 (2020).
- Häner, T., Roetteler, M., Svore, K. M. — *Optimizing Quantum Circuits for Arithmetic*, arXiv 1805.12445 (2018).
- Gidney, C. — *Halving the cost of quantum addition*, Quantum 2, 74 (2018). Cuccaro, S., Draper, T., Kutin, S., Moulton, D. — arXiv quant-ph/0410184 (2004). Draper, T. — arXiv quant-ph/0008033 (2000). Jones, C. — Phys. Rev. A 87, 022328 (2013).
- Grover, L., Rudolph, T. — arXiv quant-ph/0208112 (2002). Babbush, R. et al. — Phys. Rev. X 8, 041015 (2018).
- Volder, J. E. — IRE Trans. Electronic Computers EC-8(3), 330–334 (1959). Walther, J. S. — AFIPS Spring Joint Computer Conf., 379–385 (1971). Hu, Y. H. — IEEE Trans. Signal Processing 40(4), 834–844 (1992). Wang, S., Piuri, V., Swartzlander, E. E. — IEEE Trans. Computers 46(11), 1202–1207 (1997).
- Daubechies, I., Sweldens, W. — J. Fourier Anal. Appl. 4(3), 247–269 (1998). Bennett, C. H. — IBM J. Res. Dev. 17(6), 525–532 (1973); SIAM J. Comput. 18(4), 766–776 (1989).
- Heston, S. L. — Rev. Financial Studies 6(2), 327–343 (1993). Bates, D. S. — Rev. Financial Studies 9(1), 69–107 (1996).
- Herman, D. et al. — *Quantum computing for finance*, Nature Reviews Physics 5, 450–465 (2023).
- JPMorganChase second-quarter 2026 results, 14 July 2026 — Asset & Wealth Management average loans $284.3B, +18%.
- ORRERY, ASTROLABE and *J.P. Morgan Wealth Management — A Case Study* (this repository's siblings, September 2026): the register, the sequence number, the attested numeral, the five surfaces, the D2 collateral cycle.
- Reference-model measurements in Section 3.4: integer ladder, 3,000 random angles per configuration, this repository, `ladder/reference.py`.

---

*One ladder. Every step a permutation. Every direction kept. Every estimate an interval with a transcript behind it. The register measures; the quadrant reads the shadow; both stamp the same sequence number.*
