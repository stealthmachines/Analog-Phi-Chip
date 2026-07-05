# φ Substrate — Hardware Designs (Three Paths)

Status gate: no further behavioral simulation is required. The primitive set
(P1–P11), the three architecture rules, and the two-mode operating principle
are measured. The remaining open items — √(Fₙ·Pₙ·2ⁿ) origin (a Bessel/basin
geometry question) and trained-readout capacity (post-build
characterization) — do not gate hardware. What follows are buildable
designs, with component values computed from the validated SPICE decks.

Every design implements the same three rules learned from real failures:
**(1) scale C, not the coupling/rotation element**, for frequency ladders;
**(2) energy injection must saturate** (tanh-class); **(3) clamp the state
nodes and power up near the operating point.** And the same two-mode
principle: write mode = self-oscillating (memory in phase); read mode =
sub-threshold + pilot comb (coherent forced response).

---

## Path 1 — Silicon (bench build now; IC later)

### 1a. Bench strand array — OTA + BJT-array, audio band

The discovery that de-risks this build: the mandatory tanh injector **is an
OTA**. The LM13700's diff-pair transfer I = Iabc·tanh(Vin/2VT) is exactly the
saturating element the array requires; its Iabc pin is the write/read mode
switch.

**Frequency plan (C₀ = 10 nF → audio band, f₀ = 477 Hz):**

| n | Cₙ = 10n·φ^(−n/2) | E24 combo | error | fₙ (Hz) |
|---|------|------------|-------|--------|
| 0 | 10.000 nF | 10n | 0.00% | 477.5 |
| 1 | 7.862 nF | 7.5n + 360p | 0.02% | 607.3 |
| 2 | 6.180 nF | 5.1n + 1.1n | 0.32% | 772.6 |
| 3 | 4.859 nF | 4.3n + 560p | 0.03% | 982.7 |
| 4 | 3.820 nF | 3n + 820p | 0.01% | 1250.0 |
| 5 | 3.003 nF | 3n | 0.09% | 1590.0 |
| 6 | 2.361 nF | 2.2n + 160p | 0.03% | 2022.6 |
| 7 | 1.856 nF | 1.3n + 560p | 0.22% | 2572.8 |

Use C0G/NP0 or polypropylene. The audio band means the **pilot comb, probe
tones, and readout are all a computer sound card** (or any 96–192 kS/s ADC/
DAC): the entire instrumentation rack is a laptop.

**Per-strand circuit (two state nodes x1, x2):**

- **Gyrator (rotation, k = 30 µ℧):** LM13700 A→B and B→A cross-connection.
  30:1 resistive input dividers (linear to ±0.9 V), **Iabc = 47 µA**
  (set by R from V+ to pin: R ≈ (V+ − 1.2V)/47µ; at ±12 V rails, 220 kΩ).
  One LM13700 (dual) = one gyrator.
- **Saturating injector (write mode):** second LM13700 section per node,
  output tied back to its own node, referenced to the array's xr rail.
  **10.3:1 input divider** (10 kΩ / 1.0 kΩ), **Iabc = 3 µA** → small-signal
  5.6 µ℧, saturation ±3 µA — the SPICE deck's exact numbers.
  **Mode switch: Iabc(write) = 3 µA, Iabc(read) = 1 µA** — one resistor
  switched per strand (or one shared DAC line).
- **φ-cell (operating-point reference + restoring):** the translinear core
  needs matched NPNs: THAT300P (quad, current production) or CA3046-class
  arrays; hand-matched BC547B (ΔVbe < 1 mV) is acceptable per the √N result —
  the lattice averages mismatch. One shared reference cell for the array
  (the P10 architecture), buffered by a TL072 follower to drive all
  divider references.
- **Clamps:** BAT54S dual Schottky per state node to the rails.
- **Coupling:** 2 MΩ metal-film between adjacent x1 nodes (chain), exactly
  as simulated.
- **Supply:** ±12 V linear; currents are µA-scale — battery operation is
  trivial and kills mains hum at these frequencies.

**BOM per strand:** 2× LM13700 (~$2 ea), 1× THAT300P or 5× BC547B, 1× TL072,
4× BAT54S, C ladder pair, ~20 resistors. **8-strand array ≈ $60–90** plus a
sound card. Perfboard is fine at 0.5–2.6 kHz; star-ground the xr rail.

**Bring-up = the validated sequence:** free-run check (8 lines, √φ adjacent
ratios, φ spiral ratios) → pilot comb lock check → read-mode ladder (the
1–3% product test, now against real-device theory) → genome write/read
(phase demod against the comb).

### 1b. IC path (tapeout track)

Port the cells to subthreshold CMOS (translinear law holds in weak
inversion) or BiCMOS; MIM-cap φ ladder (capacitor matching is the process's
best asset — rule 1 was chosen for this); Monte-Carlo with foundry Pelgrom
data; cascode the mirrors to kill the +0.5% systematic. The bench array 1a
is the direct de-risk of every block.

---

## Path 2 — Vacuum tubes (the native phase medium)

Per-node: **Colpitts triode oscillator**, one half of a 12AU7 (or 6J1/6AK5
pentode) per strand. Van der Pol limiting is native (grid conduction);
Adler injection locking is native (this physics was discovered here).

**Tank ladder (L = 2.5 mH fixed on every strand — rule 1 in tube form:
Cₙ = C₀·φ^(−n), since f ∝ 1/√C gives the √φ frequency steps):**

| n | Cₙ | fₙ |
|---|-----|-----|
| 0 | 1013 pF | 100.0 kHz |
| 1 | 626 pF | 127.2 kHz |
| 2 | 387 pF | 161.8 kHz |
| 3 | 239 pF | 205.8 kHz |
| 4 | 148 pF | 261.8 kHz |
| 5 | 91 pF | 333.0 kHz |
| 6 | 57 pF | 423.6 kHz |
| 7 | 35 pF | 538.8 kHz |

Silver-mica caps; split each Cₙ 2:1 for the Colpitts divider. Grid leak
100 kΩ + 100 pF (the saturation mechanism — rule 2 is free). Cathode bias
1 kΩ bypassed. B+ 150–200 V, heaters 6.3 VAC. Coupling: 2–5 pF gimmick caps
between adjacent tanks (weak, per the quenching lesson — start weak).
Clamps/rule 3: grid conduction self-limits; power-up is inherently gentle
(heater warm-up ≈ soft start).

**Role:** the phase/Kuramoto medium. Verified prediction to demonstrate on
the bench: the φ-detuned pair is the **last to injection-lock** as coupling
increases (Hurwitz; measured in the van der Pol sims). Drift is irrelevant —
sync tolerates drift; that is what sync is for. The x²=x+1 amplitude
identity does not port (3/2-power law, no translinear loop) — tubes carry
the phase identity only.

**Scale:** 4 nodes = 2× 12AU7 + PSU ≈ shoebox. Readout: any oscilloscope or
an RTL-SDR at 100–540 kHz.

---

## Path 3 — Motors, lamps, gears (the exact-ℕ medium)

- **Nodes:** DC motor + flywheel units (swing equation = second-order
  Kuramoto; verified locking at K=0.15 with 20% drive mismatch).
- **Coupling/damping:** #47 incandescent pilot lamps (6.3 V, 150 mA) wired
  between generator windings — the lamp's thermal R(T) is the adaptive
  damper (HP200A mechanism), and **filament temperature is the sync
  readout**: the verified signature is that locked machines' lamps equalize
  brightness (380 K/380 K vs 363/402 K unlocked). Photodiodes on each lamp
  → the order parameter as a light level.
- **φ itself:** Fibonacci gear pairs, 3D-printed —
  34:21 (626 ppm), 89:55 (91 ppm), **233:144 (13 ppm)**. Integer teeth make
  mechanical φ *cheaper and more precise than the analog silicon lattice*
  (which needs ~5,900 cells for ~1,000 ppm). This is LAYER 1 — Fibonacci as
  the axiom iterated in ℕ — in brass (or PLA).
- **Rules:** rule 1 = inertia (flywheel mass) scales the frequency, not
  drive torque; rule 2 = the lamp IS the saturating damper; rule 3 =
  soft-start the drive current.

**Role:** room-scale exhibit and the highest-precision φ artifact of the
three paths. Speed: Hz.

---

## The split, restated as a build plan

The axiom has two physical identities and the media divide them:
**amplitude identity x²=x+1** → silicon translinear loop (path 1) and
Fibonacci teeth (path 3); **phase identity φ = most-irrational winding
number** → tube injection locking (path 2) and swing-equation machines
(path 3). Build order by information gained per dollar:

1. **Path 1a bench array (~$90, sound-card instrumented)** — every P1–P11
   result becomes physically checkable, and it is the direct de-risk of the
   IC.
2. **Path 2 four-node tube rack** — the Hurwitz lock-resistance demo in the
   medium that invented the physics.
3. **Path 3 gear-and-lamp machine** — 13 ppm mechanical φ and sync you can
   see, for the bench that visitors understand in ten seconds.
