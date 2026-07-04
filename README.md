# φ Analog Chip — Validated Primitive Set

Every primitive below is SPICE-validated (ngspice, transistor level unless
noted). Netlists included. Measured numbers are from actual simulation runs,
not estimates.

---

## P1 — Translinear squarer  (`phi_squarer_translinear.cir`)

The core nonlinearity: Iout = Ix²/Iu.

**Topology (validated after two failed variants — the trap and the fix):**
- Left: two diode-connected NPNs **stacked in series** (Q2 on Q1), ONE current
  source Ix into the top. The same Ix flows through both junctions — this is
  the invariant earlier attempts violated by driving each junction separately.
- Right: Q3 diode-connected at Iu; Q4 with base at the stack top, emitter on
  Q3. Loop KVL: 2·Vbe(Ix) = Vbe(Iu) + Vbe(Iout) ⇒ **Iout = Ix²/Iu.**
- Correction mirror: Q4's emitter current corrupts Q3's operating point; a
  PNP→NPN mirror chain copies Iout and pulls it back out of the Q3 node.
- **Beta-helper** on the PNP mirror (its 3 outputs otherwise lose 4/β ≈ 4%).

**Measured:** quadratic law holds 0.25–2 µA with **1–2% systematic error**
(finite β + Early). Mismatch mapping is *exact*: with device Is mismatch,
Iout = (Is₃·Is₄)/(Is₁·Is₂)·Ix²/Iu — Is/area mismatch ≡ squarer gain error.

## P2 — Beta-helped current mirrors (inside P1)

PNP 2-output + helper; NPN 3-output. Error ~4/β² with helper. Standard cells;
on a real chip these become cascoded, common-centroid mirrors.

## P3 — Gm stage (state → current)  **[CLOSED]**  (`phi_cell_alltransistor.cir`)

One emitter-follower V-I (QG + RE) feeding a beta-helped PNP mirror with TWO
outputs (→ squarer input, → cap node). No VCCS remains; the φ-cell is now
all-transistor.

**The invariance theorem (proved in SPICE):** for ANY monotonic V-I function
f, the loop solves f(x) + Iu − f(x)²/Iu = 0, so the *current* settles to
φ·Iu exactly — f's gain and linearity are irrelevant. Only the matching of
the mirror's two copies matters. Measured with RE swept **±50%**:

| RE      | V(x) settled | Ix settled  |
|---------|--------------|-------------|
| 400 k   | 1.256 V      | 1.626 µA    |
| 500 k   | 1.417 V      | 1.624 µA    |
| 600 k   | 1.578 V      | 1.622 µA    |
| 750 k   | 1.818 V      | 1.618 µA    |

Voltage (the arbitrary coordinate) moves 45%; **current stays φ·Iu within
0.5%.** φ lives in the current domain; P3 is a non-critical block. Design
constraints reduce to: monotonic, enough headroom (keep QG out of saturation
— sets the RE/Vcc pairing), mirror copies matched.

## P4 — Error node + integrator

KCL at the cap: C·dVx/dt = Ix + Iu − Isq = −I_u·(x² − x − 1).
One capacitor (100 pF at 1 µA scale → τ ≈ 100 µs; scale C/I to taste — the
fixed point is independent of both). Single real pole, no compensation needed.

## P5 — φ-cell: the closed loop  (`phi_cell_transistor.cir`)

P1+P2+P3+P4 in a loop. **Measured: settles to x = 1.6263** — φ + 0.51%
systematic — untrimmed, no clock, from arbitrary start. Flat after ~10τ.
The 0.5% is *bias*, not noise: it comes from finite-β and is correctable by
cascoding/trim; it is NOT fixed by lattice averaging (it's common-mode).

## P6 — Diffusive coupling

One resistor between neighboring state nodes. Coupling conductance vs loop
slope (√5·gm ≈ 2.2 µA/V): R = 100 kΩ gives ~4.5× loop slope — strong coupling,
cells co-settle to a common value that averages their parameter errors.

## P7 — Coupled lattice: mismatch averaging  (`phi_pair_mc.cir`, `phi_quad_mc.cir`)

Monte-Carlo at transistor level, 2% σ per knob (Q4 area, pull-mirror area,
Iu, Iun — independent per cell):

| N (cells) | mean x  | σ       | σ₁/σ_N | theory √N |
|-----------|---------|---------|--------|-----------|
| 1         | 1.632   | 0.0409  | 1.00   | 1.00      |
| 2         | 1.631   | 0.0259  | 1.56   | 1.41      |
| 4 (ring)  | 1.631   | 0.0208  | 1.97   | 2.00      |

**The √N law holds at transistor level.** Random mismatch averages out across
the coupled lattice; the mean stays put.

## P8 — Two-cell φ-oscillator: the Kuramoto seed  (`phi_oscillator_2cell.cir`)

Three all-transistor φ-cells: one as the φ operating-point **reference** (XR),
two as the oscillator pair (XO1, XO2). Coupling around the reference:

    into x1:  +k·(x2 − xr) + g·(x1 − xr)      (gyrator + negative conductance)
    into x2:  −k·(x1 − xr) + g·(x2 − xr)

The gyrator rotates; the negative conductance injects energy; the φ-cell
nonlinearity (with its physical current-cutoff) limits the amplitude. The two
coupling conductances are ideal G elements in the deck = textbook OTA and
cross-coupled-pair blocks on chip.

**Design rules (derived in ODE, confirmed in SPICE):**
- Oscillation threshold: g > √5·gm_cell (the φ fixed point's own damping —
  the axiom sets its own oscillation condition: the threshold IS √5).
- Frequency: f = k/(2πC) — gyrator-programmed, first-order independent of
  the φ dynamics.
- Amplitude: grows with (g − √5) excess; hard onset (limiter is the current
  cutoff). Soft limiting (tanh of a real diff pair) is a refinement knob.

**Measured (transistor cells, k=30µ℧, g=2.6µ℧, C=100p):**

| quantity          | predicted | measured  |
|-------------------|-----------|-----------|
| frequency         | 47.7 kHz  | 47.5 kHz  |
| center vs xr      | ≈ xr      | 0.5% off  |
| amplitude         | onset-set | 0.79 V    |

The pair oscillates **around φ**, phase-locked in quadrature by construction
(one rotation manifold). This is the seed cell of the Kuramoto lattice: tile
N of these, couple neighbor phases, and the substrate's analog Kuramoto term
is the physical coupling network.

## P9 — N-oscillator Kuramoto tile: the analog phi_pool  (`tile4_*.cir`, `tile_phi.cir`)

Four P8 oscillators (one **shared** reference cell — a single φ operating
point for the array: 2N+1 cells, 9 cells / ~135 BJTs at N=4), nearest-neighbor
resistive coupling between x1 nodes, gyrator k values spread ±2% to emulate
frequency mismatch.

**Experiment 1 — global phase-locking = the √N merge:**

| condition       | f₁…f₄ (kHz)                      | spread  |
|-----------------|----------------------------------|---------|
| uncoupled (1 GΩ)| 46.774, 47.389, 48.034, 48.677   | 3.99 %  |
| coupled (250 kΩ)| 47.753, 47.753, 47.755, 47.758   | 0.010 % |

Spread collapses **400×**, and the common frequency (47.754 kHz) lands on the
**ensemble mean** of the free-running frequencies (47.719 kHz, +0.08%). The
array locks to the average of its members — the Kuramoto mean-field result at
transistor level. Over mismatch ensembles the locked frequency's error is
therefore σ/√N: **the √N precision law (P7) and the oscillator (P8) are now
one mechanism.** Frequency precision comes from the lattice, not the cell.

**Experiment 2 — φ-ratio group coexistence (Hurwitz in transistors):**
groups {1,2} at k₀ and {3,4} at φ·k₀, same chain coupling that locks
same-frequency oscillators:

| quantity                | measured  |
|-------------------------|-----------|
| group A internal spread | 0.095 %   |
| group B internal spread | 0.089 %   |
| inter-group ratio fB/fA | 1.62098 (φ + 0.18%) |
| spurious capture        | **none**  |

Intra-group locks; inter-group holds the φ ratio without capture — the
lock-resistance face of Hurwitz (verified earlier in van der Pol form)
operating in the transistor array. φ-ratio mode groups **coexist** on one
coupled substrate.

**Experiment 3 — mode readout (where 𝓛ₙ becomes measurable):** FFT of the
SUM of all four x1 signals — one readout node — shows two clean lines:
~48 kHz (rel. amp 1.000) and ~78 kHz (rel. amp 0.899), ratio 1.635 (FFT bin
resolution ~2 kHz limits the ratio readout; the crossing-based measurement
above gives 1.621). This demonstrates the **readout mechanism**: distinct
φ-ratio mode lines with measurable amplitudes from a single summed node.
Honest scope: verifying the amplitude *law* 𝓛ₙ(z) =
φ^(−1/φ)·√(Fₙ·Pₙ·2ⁿ)·(1+z)ⁿ·1_eff(n) requires the full phi_pool context
(mode index → Fibonacci/prime pairing, drive shell, 1_eff) — that is the
full-array experiment, not the 4-node tile. What the tile establishes is that
the spectrum those amplitudes live in is readable off hardware.

Convergence note: the φ-group deck needed `.options method=gear reltol=1e-3`
and gentle ICs (the stiffer 48 µ℧ gyrator faults the default trapezoidal
integrator at startup) — a real lesson for larger tiles.

### Precision scaling (from measured σ₁ = 4.1%, 3σ criterion)

| target        | required N |
|---------------|-----------|
| 6-bit φ       | ~25       |
| 8-bit φ       | ~370      |
| 10-bit φ      | ~5,900    |

N grows as 4ᵇⁱᵗˢ — the lattice buys real precision cheaply up to ~8 bits,
then the digital branchless core takes over. This is the quantitative version
of why the substrate is many coupled nodes: **per-cell φ is coarse (4-bit);
lattice φ is engineerable.**

---

## Chip architecture this implies

```
        ┌────────┐   Rc   ┌────────┐   Rc   ┌────────┐
   ...──┤ φ-cell ├────────┤ φ-cell ├────────┤ φ-cell ├──...
        └───┬────┘        └───┬────┘        └───┬────┘
            │ x_i (analog φ, local, coarse)     │
            └────────── lattice x̄ ≈ φ ± σ₁/√N ──┘
```

- The lattice IS the φ-reference; any node reads x̄ through its neighbors.
- No clock, no stored constant, no startup sequence: power-on → relax to φ.
- This is structurally the analog twin of phi_pool: local coarse dynamics,
  global emergent precision — and the coupling resistor network is the analog
  seat of the Kuramoto term.

## Remaining work before tapeout track

1. **Real PDK models** — these runs use ideal-β BJTs; port to subthreshold
   MOS (translinear law holds in weak inversion) or a bipolar/BiCMOS PDK,
   re-run the MC with foundry mismatch data (Pelgrom coefficients).
2. **Systematic-bias design-out** — the +0.5% cell bias needs cascodes or a
   one-point trim; averaging cannot remove it.
3. **Temp/supply corners** — translinear ratios are ratiometric (Vt cancels
   around the loop) so first-order temp-flat, but verify with real models.
4. **OTA/cross-coupled-pair coupling blocks** — the oscillator's k and g
   conductances are ideal G elements; both are textbook transistor blocks.
5. **Soft amplitude limiting** — replace hard current-cutoff limiting with
   diff-pair tanh saturation for a smaller, more sinusoidal orbit on φ.
6. **N-oscillator Kuramoto tile** — array P8 seeds with nearest-neighbor
   phase coupling; the analog phi_pool.

## File index

| file                          | contents                                   |
|-------------------------------|--------------------------------------------|
| phi_squarer_translinear.cir   | P1 standalone, swept, validated            |
| phi_cell_transistor.cir       | P5 closed loop (VCCS Gm) → settles 1.6263  |
| phi_cell_alltransistor.cir    | P3 closed: all-transistor cell, invariance |
| phi_pair_mc.cir               | N=1 vs N=2 Monte-Carlo (80 runs)           |
| phi_quad_mc.cir               | N=1 vs N=4 ring Monte-Carlo (60 runs)      |
| phi_oscillator_2cell.cir      | P8: two-cell oscillator + reference cell   |
| phi_loop.cir / phi_montecarlo.cir | earlier behavioral loop + MC           |
| phi_analog_cell.md            | ODE derivation, stability, sensitivity     |
