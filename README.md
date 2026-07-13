# Op-Amp Modeling, Feedback Networks & Real Op-Amp Configurations

A hand-calculation + LTSpice simulation study of operational amplifiers, moving step by
step from the **ideal op-amp abstraction** to **real, datasheet-constrained devices**
(LM741, LT1055). The goal is to build intuition for *why* the ideal op-amp
approximation works so well in practice, *when* it breaks down, and *how* to
design a real inverting amplifier from a datasheet rather than from a textbook formula.

## Why this exists

Most op-amp courses teach the "virtual short / virtual open" shortcut and stop there.
This project instead treats the op-amp as a black box with **finite, non-ideal
parameters** — input resistance, output resistance, open-loop gain, offset voltage,
bias current, gain-bandwidth product, and slew rate — and asks what each one actually
costs you in a real circuit. Every result is derived twice: once by hand (KCL/KVL on
the non-ideal model) and once by simulation (LTSpice), so the two can be cross-checked
against each other.

## What's covered

| Problem | Topic | Key Question |
|---|---|---|
| **1** | Op-amp modeling with a VCVS (EOA) model | How much does finite `rd` and `r₀` actually change the closed-loop gain of an inverting amplifier? |
| **2** | Loading effect & buffer circuits | Why does a voltage divider sag under load, and how does an op-amp buffer fix it? |
| **3** | Real op-amp design (LM741 vs LT1055) | How do you go from a datasheet to resistor values and a bandwidth/slew-rate budget — and how much does input-stage technology (BJT vs JFET) matter? |

## Repository structure

```
.
├── README.md                  ← you are here
├── 741IC_Opamp_Assignment_Final.pdf   ← original assignment writeup (source document)
├── docs/
│   └── assignment_notes.md    ← full write-up in Markdown, images embedded inline
└── images/                    ← all figures/screenshots extracted from the PDF
```

## Problem 1 — Op-Amp Modeling & Feedback Network

**Circuit:** Inverting amplifier built around an EOA (Equivalent Op-Amp) model —
a VCVS with input resistance `rd`, output resistance `r₀`, and open-loop gain `Aol`.

| Parameter | Value |
|---|---|
| R1 | 1 kΩ |
| R2 (feedback) | 100 kΩ |
| rd | 2 MΩ |
| r₀ | 75 Ω |
| Aol | 200 V/mV |

![Inverting amplifier circuit with EOA model](images/fig01_inverting_amplifier_eoa_model.png)

**Case A — Ideal op-amp** (`rd = ∞`, `r₀ = 0`): virtual short/virtual open gives the
textbook result `Vo/Vi = -R2/R1 = -100`, so `Vout = -100 V` for `Vi = 1 V`.

**Case B — Practical op-amp** (`rd = 2 MΩ`, `r₀ = 75 Ω`): full KCL at the inverting
node and the output node gives `Vout = -99.949 V` — a gain of `-99.949`, i.e. **0.051%
deviation** from ideal.

![Hand calculation setup for Case A and B](images/fig02_hand_calc_caseA_caseB_setup.png)
![Full KCL derivation for Case B](images/fig03_hand_calc_caseB_derivation.png)

**Conclusion:** the ideal-op-amp approximation is extremely accurate here because
`rd >> R1` (negligible current diverted into the input) and `r₀ << R2` (negligible
output voltage drop). For R1/R2 in the kΩ–100 kΩ range this holds for essentially any
real op-amp.

**LTSpice verification** — the EOA was modeled with a `UniversalOpAmp` VCVS
(`Avol = 200000`, `Rin = 2 MΩ`), and the input was swept to confirm the gain stays
linear and constant.

![UniversalOpAmp VCVS parameters](images/fig04_ltspice_universalopamp_params.png)
![LTSpice inverting amplifier schematic](images/fig05_ltspice_inverting_amp_circuit.png)
![Transient result confirming Vo/Vi = -100](images/fig06_transient_result_vout_vinp.png)
![.step sweep of Rin from 1MΩ to 1GΩ](images/fig07_step_statement_editor_rinp_sweep.png)
![Vout vs Rin sweep — negligible change confirms rd >> R1](images/fig08_vout_vs_rinp_sweep_plot.png)
![Vo/Vi sweep from 1V to 5V — constant gain of -100](images/fig09_vo_vi_plot_vinp_sweep.png)

## Problem 2 — Loading Effect & Buffer Circuits

**Question:** what is the loading effect in a voltage divider, and how does an
op-amp buffer solve it?

A load `RL` connected directly across a voltage divider's output pulls current from
the divider and drops the output voltage away from its unloaded (Thévenin) value. The
severity is set by the ratio `Rth/RL`: negligible when `RL >> Rth`, severe as `RL`
approaches `Rth`.

![Voltage divider with and without a buffer](images/fig10_problem2_voltage_divider_buffer_schematics.png)

**(a) Without buffer, RL swept 1 kΩ → 100 kΩ:** output sags noticeably at low RL.
**With buffer:** output stays pinned near 5 V regardless of RL — the op-amp isolates
the divider from the load entirely.

![LTSpice schematics for direct connection vs buffered](images/fig11_ltspice_schematic_loading_buffer.png)
![RL sweep results: unbuffered sags, buffered stays flat](images/fig12_rl_sweep_results_loading_effect.png)

**(b) With buffer, open-loop gain (Aol) swept 100 → 1000 (DC):** as `Aol` increases,
`Vout` converges toward the ideal unity-gain value — from ~994.5 mV at `Aol = 100` to
~999 mV at `Aol = 1000`. Higher open-loop gain makes the feedback loop track the input
more tightly.

![Buffer circuit with Aol swept](images/fig12b_buffer_circuit_aol_sweep_schematic.png)
![Vout for Aol sweep — converges toward 1V as Aol increases](images/fig13_vout_aol_sweep_dc_plot.png)

**(c) Same sweep, AC 1 kHz input:** for very low `Aol` (=1) the loop can't maintain
unity gain and the output is attenuated to roughly half amplitude. For `Aol ≥ 401` the
output is virtually indistinguishable from the input — confirming that a sufficiently
high open-loop gain is required for accurate buffering even at non-DC frequencies.

![AC transient output with Aol swept](images/fig14_vout_aol_sweep_ac_plot.png)

## Problem 3 — Real Op-Amp Design: LM741 vs LT1055

**Goal:** design an inverting amplifier with gain = 100 using a real op-amp, working
directly from the datasheet rather than assuming ideal behavior — then compare a
BJT-input part (LM741) against a JFET-input part (LT1055).

![LM741 pin connection diagram](images/fig15_lm741_pin_diagram.png)
![LM741 electrical characteristics from the datasheet](images/fig16_lm741_electrical_characteristics.png)

### Design methodology (4 layers)

The hand-calculation approach builds the design up in layers, each one bounded by a
datasheet limit rather than a rule of thumb:

1. **Layer 0 — Design specification.** Pin down what's fixed (supply voltage from the
   datasheet) vs. what's a free design choice (load, input amplitude/frequency).
2. **Layer 1 — Static (DC) analysis.** Input offset voltage, input bias current, and
   input offset current, and how each shows up as a DC error at the output. Derives
   the `Rb = R1‖Rf` compensation technique for canceling bias-current-induced offset.
3. **Layer 2 — Resistor values.** Choosing R1/Rf to hit the target gain while staying
   in a "safe" range — large enough that bias current doesn't matter, small enough
   that the BJT input stage stays in its active region.
4. **Layer 3 — Dynamic analysis.** Gain-bandwidth product and slew rate together set
   the maximum usable frequency for a given output swing; the point where the two
   constraints cross over determines which one governs at a given amplitude.
5. **Layer 4 — Power/thermal check.** Confirms the design stays within safe power
   dissipation and output current limits.

![Hand calc: Layer 0 — design specification](images/fig17_hand_calc_layer0_design_spec.png)
![Hand calc: Layer 1 — bias & offset current](images/fig18_hand_calc_layer1_bias_offset_current.png)
![Hand calc: Layer 1 — offset derivation](images/fig19_hand_calc_layer1_offset_derivation.png)
![Hand calc: Layer 2 — resistance values](images/fig20_hand_calc_layer2_resistance_values.png)
![Hand calc: Layer 2 — safe resistor range](images/fig21_hand_calc_layer2_safe_range.png)
![Hand calc: Layer 3 — dynamic (GBW/slew rate) analysis](images/fig22_hand_calc_layer3_dynamic_analysis.png)
![Hand calc: Layer 3 — GBW vs slew-rate crossover](images/fig23_hand_calc_layer3_crossover_analysis.png)

### Final LM741 design

`R1 = 1 kΩ`, `Rf = 100 kΩ`, `Rb = 1 kΩ`, `Vs = ±15 V`, driven at 1 kHz (chosen because
it satisfies both the GBW-limited bandwidth of ~10 kHz and the slew-rate limit of
~16 kHz for the target output swing).

![Final LM741 inverting amplifier design](images/fig24_final_lm741_circuit_design.png)
![LM741 LTSpice schematic](images/fig25_lm741_ltspice_schematic.png)
![LM741 transient — clean gain-of-100 inversion](images/fig26_lm741_transient_plot.png)
![LM741 AC Bode plot](images/fig27_lm741_bode_plot.png)
![LM741 Bode cursor measurement at -3dB](images/fig28_lm741_bode_cursor_measurement.png)

### LT1055 design (same procedure, JFET input)

![LT1055 electrical characteristics from the datasheet](images/fig29_lt1055_electrical_characteristics.png)
![LT1055 LTSpice schematic](images/fig30_lt1055_ltspice_schematic.png)
![LT1055 transient — cleaner waveform, lower distortion](images/fig31_lt1055_transient_plot.png)
![LT1055 AC Bode plot — wider bandwidth than LM741](images/fig32_lt1055_bode_plot.png)
![LT1055 Bode cursor measurement at -3dB](images/fig33_lt1055_bode_cursor_measurement.png)

### LM741 vs LT1055 — comparison

| Parameter | LM741 (BJT input) | LT1055 (JFET input) |
|---|---|---|
| Input bias current (Ib) | 80 nA (typ) | ±30 pA (~3000× smaller) |
| Input offset voltage (Vos) | 6 mV (max) | 500 µV (max, 12× better) |
| DC error from Ios | Ios × Rf = 20 mV | Negligible (pA-level) |
| DC error from Vos | Vos × (1+Rf/R1) ≈ 606 mV | ~50 mV |
| Gain-bandwidth product | 1 MHz | 4.5 MHz |
| -3dB frequency at gain=100 | ~10 kHz | ~45 kHz |
| Slew rate | 0.5 V/µs | 7.5 V/µs (15× faster) |
| Best suited for | General purpose, low cost | Precision & high-frequency work |

**Bottom line:** the LM741's BJT input stage needs base current to keep its
differential-pair transistors in their active region, which is the root cause of its
larger bias current, offset current, and offset voltage. The LT1055's JFET input
draws essentially zero gate current, so all three DC error sources shrink by orders
of magnitude — at the cost of slightly higher price and power draw.

## Tools used

- **Hand analysis:** KCL/KVL on the non-ideal op-amp model (VCVS with finite `rd`,
  `r₀`, `Aol`); datasheet-driven design methodology for LM741/LT1055.
- **LTSpice:** `UniversalOpAmp` (VCVS) model for Problem 1 & 2; vendor `.lib` models
  for LM741 and LT1055 in Problem 3. `.step param` sweeps used throughout to observe
  sensitivity to `rd`, `Aol`, and `RL`.

## Full write-up

See [`docs/assignment_notes.md`](docs/assignment_notes.md) for the complete solution
set (all three problems, all hand-calculation steps, all simulation results) as a
single Markdown document. The original source PDF is included at the repo root for
reference.
