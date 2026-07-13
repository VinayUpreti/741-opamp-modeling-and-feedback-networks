# Assignment: Op-Amp Modeling, Feedback Networks and Real Op-Amp Configurations

Full write-up with all hand calculations and simulation results. See the top-level
[README](../README.md) for a summarized overview.

---

## Problem 1: Op-amp Modeling and Feedback Network

**Tasks:**
1. Identify the configuration shown.
2. Calculate `Vo/Vi` for:
   - (a) `rd = ∞`, `r₀ = 0`, `Vi = 1V`
   - (b) `rd = 2MΩ`, `r₀ = 75Ω`, `Vi = 1V`
   - (c) Compare (a) vs (b) and draw a conclusion
   - (d) Model the EOA using a VCVS in LTSpice; plot `Vo/Vi` while sweeping the input
     from 1V to 5V, then vary `rd` to observe the effect.

![Fig 1: Inverting amplifier circuit with EOA model (rd = 2MΩ, R1 = 1kΩ, R2 = 100kΩ, ro = 75Ω, Aol = 200 V/mV)](../images/fig01_inverting_amplifier_eoa_model.png)

### Solution

The configuration is an **Inverting Amplifier**. `Vi` is applied through `R1` (1 kΩ)
to the inverting input; `R2` (100 kΩ) feeds back from the output to the inverting
input; the non-inverting input is grounded through `rd`.

**Case A: rd = ∞, R0 = 0, Vi = 1V (Ideal Op-Amp)**

Under ideal conditions (virtual short, virtual open): `V- = V+ = 0`.

```
Vo/Vi = -(R2/R1) = -(100k/1k) = -100
Vout  = -100 × Vi = -100 × 1V = -100V
```

![Fig 2: Hand calculation — Case A (ideal op-amp, Vout = -100V) and Case B circuit setup with KCL equations at node V-](../images/fig02_hand_calc_caseA_caseB_setup.png)

**Case B: rd = 2MΩ, r0 = 75Ω, Vi = 1V (Practical EOA Model)**

KCL is applied simultaneously at the inverting node (V-) and the output node:

```
V-[(1/10^3) + (1/2x10^6) + (1/10^5)] = Vout/10^5 + 1/10^3
```

Solving the two simultaneous equations (full derivation below):

![Fig 3: Hand calculation — Case B full derivation yielding Vout = -99.949V](../images/fig03_hand_calc_caseB_derivation.png)

```
Vout = -99.949V
```

**Case C: Comparison — Ideal vs Practical EOA**

| Parameter | Case A: Ideal (rd = ∞, r₀ = 0) | Case B: Practical (rd = 2MΩ, r₀ = 75Ω) |
|---|---|---|
| Input Resistance (rd) | ∞ | 2 MΩ |
| Output Resistance (r₀) | 0 | 75 Ω |
| Current into Input Stage | 0 (no loading on R1) | Negligible — rd >> R1 |
| Voltage Drop at Output | 0 | Tiny — r₀ << R2 |
| Output Voltage (Vout) | −99.949 V | −99.949 V |
| Closed-Loop Gain | −100.000 (exact) | −99.949 |
| Deviation from Ideal | 0 V / 0% | 0.051 V / 0.051% |

**Conclusion:** Case A (ideal) gives exactly `Vout = -100V` (gain `-R2/R1`). Case B
(practical) gives `Vout = -99.949V` — a 0.051% deviation. The ideal-op-amp
approximation is extremely accurate here because `rd >> R1` (negligible current
diverted into the input resistance) and `r₀ << R2` (negligible output voltage drop).
In practice, finite `rd` and `r₀` have almost no impact on the inverting-amplifier
gain when R1/R2 are in the kΩ–100 kΩ range, as long as `rd >> R1` and `r₀ << R2` hold.

**Case D: LTSpice VCVS Model — Simulation**

The EOA was modeled with LTSpice's `UniversalOpAmp` component: `Avol = 200000`,
`GBW = 10Meg`, `Rin = 2Meg`.

![Fig 4: UniversalOpAmp parameters — Avol = 200000, Rin = 2MΩ](../images/fig04_ltspice_universalopamp_params.png)

![Fig 5: LTSpice circuit — Inverting amplifier with UniversalOpAmp (R1 = 1kΩ, Rf = 100kΩ, ±10V supply), V1 = 1V DC](../images/fig05_ltspice_inverting_amp_circuit.png)

![Fig 6: Transient result — V(vout) = -100V (red), V(vinp) = 1V (blue), confirming Vo/Vi = -100 for Vi = 1V](../images/fig06_transient_result_vout_vinp.png)

A `.step param Rinp 1Meg 1Gega 1000Meg` sweep was used to observe the effect of
varying input resistance on `Vout`:

![Fig 7: Sweep of Rinp from 1MΩ to 1GΩ to observe effect of varying input resistance on Vout](../images/fig07_step_statement_editor_rinp_sweep.png)

![Fig 8: V(vout) vs Rinp sweep — Rinp = 1M gives -99.9495V, Rinp = 1G gives -99.9495V; negligible difference confirms rd >> R1](../images/fig08_vout_vs_rinp_sweep_plot.png)

![Fig 9: Vo/Vi plot sweeping Vinp from 1V to 5V — Vout scales linearly (-100V to -500V), confirming constant gain of -100 across the input range](../images/fig09_vo_vi_plot_vinp_sweep.png)

---

## Problem 2: Loading Effect and Use of Buffer Circuit

**Tasks:**
1. Explain the loading effect in a voltage divider network.
2. Build both schematics in LTSpice and compare outputs:
   - (a) Sweep RL from 1Ω to 100MΩ, Rs = 100Ω fixed.
   - (b) With the buffer, sweep the buffer's gain from 100 to 10k; measure VL. Rs =
     100Ω, Vs = 1V DC.
   - (c) Same as (b) but with an AC source (1 kHz).
   - (d) Model the EOA using a VCVS in LTSpice and plot `Vo/Vi` sweeping the input
     from 1V to 5V, then vary `rd`.

![Fig 10: Voltage divider without buffer (loading effect) vs. voltage divider with op-amp buffer](../images/fig10_problem2_voltage_divider_buffer_schematics.png)

### Solution

**1. Loading Effect in a Voltage Divider Network**

When a load `RL` is connected directly to the output of a voltage divider, it alters
the effective resistance of the lower arm — `RL` appears in parallel with the lower
divider resistor, reducing the effective resistance and therefore the output voltage.
This is the loading effect. Severity is captured by `Rth/RL`: when `Rth/RL << 1`
(`RL >> Rth`) loading is negligible; as `RL` approaches `Rth`, the output voltage drops
significantly.

**2(a): Without Buffer — RL Swept from 1kΩ to 100kΩ**

![Fig 11: LTSpice schematic — direct connection (a) with Rs = 100Ω, RL swept; buffer circuit (b) with Rs = 100Ω, Aol swept, RL = {R}](../images/fig11_ltspice_schematic_loading_buffer.png)

![Fig 12: RL parametric sweep results (R = 1k, 21k, 41k, 61k, 81k, 100k). Top: without buffer — voltage drops noticeably at low RL. Bottom: with buffer — output remains nearly constant (~5V) regardless of RL](../images/fig12_rl_sweep_results_loading_effect.png)

**Observation:** Without a buffer, output voltage decreases as RL decreases since RL
loads the divider — at R = 1k the output is noticeably lower than 5V. With the op-amp
buffer, output stays close to 5V for all RL values, demonstrating that the buffer
isolates the divider from the load and eliminates the loading effect.

**2(b): With Buffer — Aol Swept (DC, Rs = 100Ω, Vs = 1V)**

![Fig 12b: Buffer circuit with .step param Aol from 100 to 1k in steps of 100, V2 = 1V DC](../images/fig12b_buffer_circuit_aol_sweep_schematic.png)

![Fig 13: V(vout) for Aol sweep (100 to 1K) — as Aol increases, Vout converges toward 1V; lower Aol gives slight deviation](../images/fig13_vout_aol_sweep_dc_plot.png)

**Observation:** As Aol increases from 100 to 1000, `V(vout)` approaches 1V more
closely — from about 994.5 mV at `Aol = 100` to about 999 mV at `Aol = 1000`. A higher
open-loop gain makes the buffer track the input more accurately, reinforcing why
practical op-amps (`Aol = 10^5` to `10^6`) are effectively ideal for this application.

**2(c): With Buffer — Aol Swept (AC 1kHz, Rs = 100Ω, Vs = 5V)**

![Fig 14: AC transient output V(vout) with Aol swept (1, 201, 401, 601, 801, 1k) — for Aol = 1 the output amplitude is reduced (~2.5V peak) while higher Aol values produce clean unity-gain sine waves at 5V peak](../images/fig14_vout_aol_sweep_ac_plot.png)

**Observation:** For very low Aol (=1), the feedback loop is insufficient to maintain
unity gain and the output sine wave is attenuated to roughly half the input amplitude.
As Aol increases, the output amplitude approaches the input amplitude. For `Aol ≥ 401`
the output is virtually indistinguishable from the input, confirming that a
sufficiently high open-loop gain is required for accurate buffering even at AC
frequencies.

---

## Problem 3: Op-config Using Real Op-Amp in LTSpice

**Tasks:**
1. Build an inverting amplifier with a gain of 100 using an LM741 op-amp. Do the AC
   analysis and show the input/output voltage plot.
2. Build the same amplifier using an LT1055 op-amp. Do the AC analysis and show the
   input/output voltage plot.
3. Describe the difference between the two cases (hint: check the input pairs).

![Fig 15: LM741 pin connection diagram (8-pin DIP)](../images/fig15_lm741_pin_diagram.png)

### Hand Calculations — Design specification derivation using the LM741 datasheet

![Fig 16: LM741 Electrical Characteristics (Texas Instruments) — Aol = 50-200 V/mV, Rin = 0.3-2MΩ, Vo swing = ±10V at RL ≥ 2kΩ, SR = 0.5 V/μs, GBW = 1MHz](../images/fig16_lm741_electrical_characteristics.png)

**Layer 0: Design Specification**

- Vs = ±15V — selected as design parameter from the datasheet.
- Load = not stated.
- Input amplitude = not stated → the IC's output-swing limit will tell us the max
  amplitude.
- Input frequency = not stated → the IC's GBW and slew rate will tell us.
- DC accuracy = not stated.

Every calculation that follows is a ratio of *design value* over *limit*, where the
limit comes from the datasheet and the design value comes from the requirement.

![Fig 17: Hand calculation Layer 0 — Design specification: Vs = ±15V from datasheet, gain A = 100, Vos = 6mV max causes 600mV DC error at output](../images/fig17_hand_calc_layer0_design_spec.png)

**Layer 1: Datasheet (Static Analysis)**

1. **Input offset voltage:** max 6 mV at 25°C → `Vout,offset = 6mV × 100 = 600mV` DC
   error at the output. This can't be eliminated by resistor choice — it's a property
   of the IC (manufacturing defect in the input transistors).
2. **Input bias current:** max 500 nA (25°C). If `Rf` is large (MΩ), `Vout` error
   grows — this constrains the feedback resistor.
3. **Input offset current:** max 200 nA (25°C). An op-amp's input terminals aren't
   ideal — the differential-pair transistors (BJTs in the LM741) need a small base
   current to stay in their active region. Due to manufacturing mismatch (`β1 ≠ β2`
   between the pair), `Ib1 ≠ Ib2`, so `Ios = Ib1 - Ib2`.

![Fig 18: Hand calculation Layer 1 — Input bias current (max 500nA), input offset current effects, and Rb = Rf‖R1 technique to minimize offset error](../images/fig18_hand_calc_layer1_bias_offset_current.png)

Setting `Rb = Rf‖R1` at the non-inverting input compensates for matched bias current,
leaving only the *offset* current (`Ios`) as an irreducible error term:

```
ΔV = Rb × Ios         (always present at the input pins)
Vout,error = (Ios × Rb) × (1 + Rf/R1) = Ios × Rf
```

![Fig 19: Hand calculation Layer 1 cont. — Derivation of output offset due to Ios: op error = Ios × Rf, with Ios = 200nA (datasheet). Shows Rb = R1‖Rf for compensation](../images/fig19_hand_calc_layer1_offset_derivation.png)

4. **Output voltage swing:** ±10V at RL ≥ 2kΩ (Vs = ±15V) → output is bounded between
   ±10V.
5. **Input voltage range:** ±12V at 25°C.
6. **Slew rate:** 0.5 V/µs.
7. **GBW:** 1 MHz (from simulation/typical spec, not explicitly tabulated).

**Layer 2: Resistance Values**

`|Rf/R1| = 100` for the target gain. Choosing `R1 = 1kΩ`, `Rf = 100kΩ` keeps bias
current error small while avoiding excessively large or small resistor values (too
small a R1 risks pulling too much input current and can push the BJTs out of their
active region).

![Fig 20: Hand calculation Layer 2 — Resistance values: |Rf/R1| = 100, so Rf = 100kΩ, R1 = 1kΩ (safe range). If Rf → MΩ, Ib error becomes large. If R1 → 1Ω, BJT may leave active region](../images/fig20_hand_calc_layer2_resistance_values.png)

With `R1 = 1kΩ`, `Rf = 100kΩ`: offset-current-driven error = `200nA × 100kΩ = 20mV`.
Offset-voltage-driven error = `6mV × 100 = 600mV` (fixed, cannot be eliminated by
resistor choice). `Rb = R1‖Rf ≈ 990Ω ≈ 1kΩ`.

![Fig 21: Hand calculation Layer 2 cont. — Safe range R1 = 1kΩ, Rf = 100kΩ. Due to input offset voltage (6mV fixed), voltage error = 6mV × 100 = 600mV (fixed, cannot be eliminated by resistors). Rb = R1‖Rf = 990Ω ≈ 1kΩ](../images/fig21_hand_calc_layer2_safe_range.png)

*Why resistors are analyzed before Vin/frequency:* static analysis always comes
first — frequency is a dynamic quantity, so the first analysis is done when nothing
is moving (DC operating point). In static analysis, `Rf`, `Rb`, `R1`, and
`Vout(max)`/`Vin(max)` are all known quantities.

**Layer 3: Dynamic Analysis**

```
f3dB = GBW / |Av|,  |Av| = Rf/R1 = 100
GBW = 1MHz → BW = 1MHz/100 = 10kHz
```

Beyond `f3dB = 10kHz`, `Vout` starts attenuating.

```
fmax(slew rate) = SR / (2π × Vout_peak) = 0.5×10^6 / (2π × 5) ≈ 16kHz
```

Beyond `fmax(SR)`, the output gets distorted (slope can't keep up) regardless of gain.

![Fig 22: Hand calculation Layer 3 — GBW = 1MHz, |Av| = 100 → f3dB = 10kHz, fmax(SR) = SR/(2π × Vout_peak) = 0.5×10^6/(2π×5) ≈ 16kHz](../images/fig22_hand_calc_layer3_dynamic_analysis.png)

There is a tradeoff between GBW and slew rate; slew rate dominates in priority since
it changes the *shape* of the output signal. For a 5V peak input signal,
`fmax(SR) ≤ 16kHz` and `f3dB = 10kHz`, so 1 kHz is a safe, comfortable operating
frequency for both small and large signals.

![Fig 23: Hand calculation Layer 3 cont. — Crossover analysis: GBW vs SR limits. For 5V peak, f-SR = 16kHz > f-GBW = 10kHz → GBW governs. For 10V peak, f-SR = 8kHz < f-GBW = 10kHz → SR governs. Best choice: 1kHz signal](../images/fig23_hand_calc_layer3_crossover_analysis.png)

**Layer 4: Power & Thermal Check**

```
Power = 2.8mA × 30V = 84mW
Thermal rise = 84mW × 150°C/W = 12.6°C → 25°C + 12.6°C = 37.6°C < 150°C → PASS
Output current = 5V / 100Ω = 0.05mA < 25mA → PASS
```

### Final LM741 Circuit

`R1 = 1kΩ`, `Rf = 100kΩ`, `Rb = 1kΩ`, `Vs = ±15V` at 25°C.
`Vinp = (0.01 to 0.1)sin(2000πt)V` → `Vout = -(1 to 10)sin(2000πt)V`.

![Fig 24: Final LM741 inverting amplifier design](../images/fig24_final_lm741_circuit_design.png)

### LM741 Inverting Amplifier (Gain = 100) — Simulation

![Fig 25: LM741 LTSpice circuit — R1 = 1kΩ, R2 = 100kΩ, R3 = 1kΩ (Rb), V1 = SINE(0, 0.1, 1000), ±15V supply](../images/fig25_lm741_ltspice_schematic.png)

![Fig 26: LM741 transient — V(vout) peak = ±10V (red), V(vip) peak = ±100mV (blue). Gain = 10/0.1 = 100. Clean inversion confirmed](../images/fig26_lm741_transient_plot.png)

![Fig 27: LM741 AC analysis (Bode plot) — flat gain ≈ 40dB from 1Hz to ~10kHz, then rolls off at -20dB/dec. Phase starts near 180° and drops](../images/fig27_lm741_bode_plot.png)

![Fig 28: LM741 Bode cursor measurement at -3dB point: Freq = 10.05kHz, Magnitude = 36.91dB (≈ 40 - 3dB), confirming f3dB ≈ 10kHz as predicted by GBW/|Av| = 1MHz/100](../images/fig28_lm741_bode_cursor_measurement.png)

### LT1055 Inverting Amplifier (Gain = 100)

![Fig 29: LT1055 Electrical Characteristics — key differences vs LM741: Vos = 500μV (max) vs 6mV, Ib = ±30pA vs 80nA, GBW = 4.5MHz vs 1MHz, SR = 7.5 V/μs vs 0.5 V/μs](../images/fig29_lt1055_electrical_characteristics.png)

**Hand Calculations (LT1055)** follow the same 4-layer methodology as the LM741. Key
differences:

- **Input offset voltage:** `Vos = 500µV` (max 1500µV) → `Vout,offset = 500µV × 100 =
  50mV` DC error — 12× better than the LM741's 600mV.
- **Input bias current:** `Ib = ±30pA` (max ±100pA) — the LT1055 uses a **JFET**
  differential pair, so bias current is ~3000× smaller than the LM741's BJT input,
  even though the constraint on `Rf` is conceptually the same.
- **Input offset current:** `Ios = 5pA` (max 30pA). Because the JFET gate draws
  essentially no current, `Ios × Rf = 5pA × 100kΩ = 0.5µV` — negligible, versus 20mV
  for the LM741.
- **Output voltage swing:** ±12V at RL ≥ 2kΩ, Vs = ±15V.
- **Slew rate:** 7.5 V/µs (15× faster than LM741's 0.5 V/µs).
- **GBW:** 4.5 MHz (vs 1 MHz).

Resistor values: same safe range, `R1 = 1kΩ`, `Rf = 100kΩ`, `Rb = R1‖Rf ≈ 1kΩ`.

**Dynamic analysis:**

```
f3dB = GBW/|Av| = 4.5MHz/100 = 45kHz
fmax(SR) = SR/(2π × Vout_peak) = 7.5×10^6/(2π×5) ≈ 238kHz
Vout_crossover = SR × |A| / (2π × GBW) = (7.5×10^6 × 100)/(2π × 4.5×10^6) ≈ 26.5V (peak)
```

For both 5V and 10V peak signals, 1 kHz comfortably satisfies both the GBW-limited
bandwidth and the slew-rate limit — same operating frequency as chosen for the LM741,
enabling a like-for-like comparison.

**Power & thermal check:** `Power = 2.8mA × 30V = 84mW`; thermal rise `12.6°C`;
`25°C + 12.6°C = 37.6°C < 150°C` → PASS. Output current `0.05mA < 25mA` → PASS.

**Final LT1055 circuit:** identical topology to the LM741 design —
`R1 = 1kΩ`, `Rf = 100kΩ`, `Rb = 1kΩ`, `Vs = ±15V`.
`Vinp = (0.01 to 0.1)sin(2000πt)V` → `Vout = -(1 to 10)sin(2000πt)V`.

![Fig 30: LT1055 LTSpice circuit — same configuration as LM741: R1 = 1kΩ, R2 = 100kΩ, R3 = 1kΩ, V1 = SINE(0, 0.1, 1000), ±15V supply](../images/fig30_lt1055_ltspice_schematic.png)

![Fig 31: LT1055 transient — V(vout) peak = ±10V (red), V(vip) peak = ±100mV (green). Gain = 100. Waveform is cleaner than LM741 due to lower distortion](../images/fig31_lt1055_transient_plot.png)

![Fig 32: LT1055 AC analysis (Bode plot) — flat gain ≈ 40dB up to ~40kHz, then rolls off. Higher bandwidth than LM741 due to GBW = 4.5MHz](../images/fig32_lt1055_bode_plot.png)

![Fig 33: LT1055 Bode cursor at -3dB: Freq = 46.58kHz, Magnitude = 37.1dB ≈ 40-3dB. f3dB ≈ 45kHz, confirming GBW/|Av| = 4.5MHz/100 = 45kHz](../images/fig33_lt1055_bode_cursor_measurement.png)

### Comparison: LM741 vs LT1055

| Parameter | LM741 (BJT Input) | LT1055 (JFET Input) |
|---|---|---|
| Input Stage Technology | BJT Differential Pair | JFET Differential Pair |
| Input Bias Current (Ib) | 80 nA (typical) | ±30 pA (≈3000× smaller) |
| Input Offset Voltage (Vos) | 6 mV (max) | 500 µV max (12× better) |
| DC Error from Offset Current | Ios × Rf = 200nA × 100kΩ = 20 mV | Negligible (pA-level bias) |
| DC Error from Offset Voltage | Vos × (1 + Rf/R1) = 6mV × 101 ≈ 606 mV | ~50 mV (far lower) |
| Gain-Bandwidth Product (GBW) | 1 MHz | 4.5 MHz |
| −3dB Frequency at Gain = 100 | ~10 kHz | ~45 kHz |
| Slew Rate | 0.5 V/µs | 7.5 V/µs (15× faster) |
| Precision (DC Accuracy) | Low — large offsets cause significant error | High — minimal DC errors |
| High-Frequency Performance | Limited — slow slew rate, low GBW | Excellent — wide bandwidth, fast slew |
| Overall Suitability | General purpose, low-cost applications | Precision & high-frequency inverting amp |
| Cost / Power | Lower cost, lower power | Slightly higher cost and power |

**Conclusion:** the key distinction is input-stage technology. The LM741's BJT
differential pair needs base current to stay in its active region, causing a
relatively large input bias current (`Ib = 80nA` typ) and input offset voltage
(`Vos = 6mV` max). This causes a DC error at the output of `Ios × Rf = 200nA × 100kΩ =
20mV` from offset current, plus `Vos × (1 + Rf/R1) = 6mV × 101 ≈ 606mV` from offset
voltage. The LT1055's JFET input draws almost no gate current (`Ib = ±30pA`, ~3000×
smaller) and has a much lower offset voltage (`Vos = 500µV` max, 12× better), so it
produces far less DC error. On the AC side, the LT1055's `GBW = 4.5MHz` (vs 1MHz)
gives a `-3dB` frequency of ~45kHz at gain=100 (vs ~10kHz for LM741), and a slew rate
of `7.5 V/µs` (vs `0.5 V/µs`) allows undistorted output at much higher frequencies and
amplitudes. The LT1055 is the clearly superior choice for precision and
high-frequency inverting-amplifier applications, at the cost of slightly higher price
and power.
