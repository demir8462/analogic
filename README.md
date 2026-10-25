<div align="center">

<img src="img/banner.svg" alt="ANALOGIC: event-driven 10-bit SAR ADC in sky130" width="100%">

<br>

![PDK](https://img.shields.io/badge/PDK-sky130A-0f172a?style=flat-square&labelColor=0f172a&color=22d3ee)
![ADC](https://img.shields.io/badge/SAR%20ADC-10%20bit%20%C2%B7%201%20MS%2Fs-0f172a?style=flat-square&labelColor=0f172a&color=22d3ee)
![Layout](https://img.shields.io/badge/layout-DRC%20%2F%20LVS%20clean-0f172a?style=flat-square&labelColor=0f172a&color=22d3ee)
![Status](https://img.shields.io/badge/status-simulation%20%2B%20layout-0f172a?style=flat-square&labelColor=0f172a&color=64748b)

</div>

---

## The idea

A conventional ADC samples and burns power whether or not the signal does anything. ANALOGIC does not. It **sleeps until something happens**, wakes itself, runs the conversion on a **clock it generates itself**, then goes back to sleep. No clock tree, almost no idle power.

This fits any application where the signal is mostly quiet: neural recording, vibration monitoring, event-based imaging, sensor nodes.

<div align="center">
<img src="img/architecture.svg" alt="ANALOGIC architecture" width="100%">
</div>

## Highlights

| | |
|---|---|
| **Energy per conversion** | **36 pJ**, about 13x lower than an open-source sky130 SAR from the literature |
| **Idle power** | **24 nW** at 25 °C, 65 nW at 75 °C |
| **Comparator** | Asynchronous (self-timed, READY output), rail-to-rail input, **σ 1.6 – 2.8 mV** offset with no calibration, layout DRC/LVS clean |
| **Accuracy** | 30 … 1770 mV input, across tt/ss/ff and 75 °C: **−1 … 0 LSB** |
| **CDAC linearity** | Shielded 10-bit layout, RC-extracted: **DNL −0.20 LSB, INL 0.12 LSB**, DRC clean, LVS matched |
| **Self-timed** | One START pulse, 10 bits resolved asynchronously in 123 ns |
| **Smart modes** | Early stop (6 bit, 26 pJ) and single-decision window check (9.7 pJ) |
| **Down to layout** | Comparator, capacitor DAC, switches and reference all drawn; DRC clean, LVS matched, post-layout RC verified |

<div align="center">
<img src="img/guc_olay_hizi.png" width="760" alt="Power vs event rate">
<br><sub><b>Average power scales with the event rate:</b> at a few events per second the ADC stays near 25 nW</sub>
</div>

<div align="center">
<img src="img/sar_donusum.png" width="760" alt="SAR conversion">
<br><sub><b>Asynchronous 10-bit conversion:</b> every READY pulse resolves one bit</sub>
</div>

---

## ★ The star: a rail-to-rail StrongARM comparator

<div align="center">
<img src="img/comparator.svg" alt="Comparator v6b key numbers" width="100%">
</div>

The comparator is the heart of an event-driven SAR, and it is **asynchronous**: there is no external clock. Once it has decided, it raises its own **READY** pulse, which re-arms it and clocks the next bit, so the conversion runs on a loop the comparator itself closes. It therefore had to work at any common-mode level and finish fast enough to drive that loop.

Most StrongARM comparators only work near one common-mode voltage. Ours works across the **entire supply range**, and in fact extends **beyond both rails**. We characterized it from 30 mV to 1.79 V in 1.8 V sky130 and did not sweep further, so every number below is for that window.

| Metric | Result |
|---|---|
| Input common mode | **Rail-to-rail and beyond.** Characterized: 30 … 1790 mV (range was not swept past this window) |
| Offset (extracted layout, Monte Carlo) | **σ 2.4 / 1.6 / 2.8 mV** at VCM 30 / 900 / 1790 mV, **no calibration** |
| Decision time (CLK → READY) | **4.1 / 3.0 / 2.4 ns** at VCM 30 / 900 / 1790 mV |
| Self-clocked loop (layout netlist, tt, 25 °C) | CLK → READY 3.7 / 2.9 / 2.3 ns, loop period 15.5 – 16.6 ns |
| Verification | **720 / 720** correct decisions on extracted layout (±1 mV, all corners) |
| Physical | **DRC clean, LVS matched**, 25.7 x 58.4 µm |
| Topology | StrongARM latch with a double-tail-style PMOS pre-stage and a 4/1 PMOS tail |

**Why it matters:** the rail-to-rail range lets the ADC converge around mid-supply (0.9 V) while the input swings the full range, and the built-in READY output turns the comparator into its own clock generator.

---

## System blocks

Each block was verified on its own, then added to the next level. Design files are not published; this page shows what each block does and how it performs.

### Capacitor DAC (CDAC)
Holds the sampled input and tries one bit per step: the SAR's balance.

| Metric | Result |
|---|---|
| Structure | 10-bit binary-weighted array plus dummy ring (first version was 9-bit) |
| Accuracy | Correct code in a manually driven SAR test; worst error **0.34 mV (< 0.1 LSB)** |
| Worst-case bit settling (9-bit) | 24 ns to 0.25 LSB |
| Parasitic fix | A 1.4 % gain error from parasitics was found and cut by 10x |
| Layout (9-bit) | 16 x 32 2D common-centroid array, 512 active + 100 dummy unit cells, **DRC 0** |
| Layout, RC-extracted (9-bit) | Correct code, gain 0.984, **worst INL 0.07 LSB**, worst settling 12 ns |
| **Layout (10-bit, shielded)** | 128.3 x 176.6 µm, **DRC 0, LVS matched**. Grounded metal shield between the bit lines and the top plate: **worst DNL −0.20 LSB, worst INL 0.12 LSB** (static weights, RC-extracted). RC-extracted: correct code 17 at all corners, worst 0.5 LSB settling **6.2 / 8.1 / 5.3 ns** (tt / ss 75 °C / ff) |
| First 10-bit layout (unshielded) | 130.5 x 151 µm. Worst DNL −0.82 LSB (511 → 512), worst INL 0.56 LSB, caused by bit-line coupling to the top plate; the shield cut both by about 4x |

### Sampling and analog switches
Takes the input onto the capacitors and ties the plates to the references.

| Metric | Result |
|---|---|
| Sampling switch | Bootstrapped: on-resistance independent of input voltage, rail-to-rail input |
| Plate switch v1 | Settles to half an LSB at 10 bit into 2 pF in 43 ns; layout DRC/LVS clean, layout = schematic |
| Plate switch v2 (low-threshold) | Settles in **13.6 ns** (3x faster), off-state leakage worst case **0.31 nA** |

| Switch | Settling (2 pF) | Peak Ron tt / ss / ff | Layout |
|---|---|---|---|
| v1 | 42.6 ns | 5.2k / 15.8k / 2.2k Ω | DRC 0, LVS matched |
| **v2 (LVT)** | **13.6 ns** | **1.6k / 2.4k / 1.2k Ω** | DRC 0, LVS matched |

### Common-mode reference (VDD/2)
Sets the comparison point; passive and very low current.

| Metric | Result |
|---|---|
| Output | **0.900 V** (VDD/2, ratiometric to supply) |
| Current / power | ~20 nA / ~36 nW (weak branch) |
| 10-bit sampling load | Strengthened v3: weak branch plus a strong branch that engages at the start of sampling. **No accumulation over 100 consecutive samples**; end-of-sample VCM 0.84 / 0.96 V (tt) |
| Layout | 24.7 x 49.9 µm, **DRC 0, LVS matched**, RC-extracted result matches the schematic |

### SAR logic and sequencer
Writes the bits, triggers and stops the comparator, generates its own timing.

| Metric | Result |
|---|---|
| Conversion | 10 bit from one START pulse, **123 ns** |
| Modes | Early stop (6 bit, 26 pJ), single-decision window check (9.7 pJ) |
| Safety | Timeout protection, returns to sleep by itself when done |

---

## How we compare

Our numbers are **simulation** (sky130, 1.8 V, tt, 25 °C; comparator numbers from extracted layout). Competitor numbers come from published work; names are withheld.

**SAR ADC**

| | **ANALOGIC (SAR v3, sim)** | A: open-source sky130 SAR | B: 180 nm SAR (silicon) |
|---|---|---|---|
| Technology / supply | sky130, 1.8 V | sky130, 1.8 V | 180 nm, 1.8 V |
| Resolution | 10 bit | 12 bit (non-binary) | 12 bit |
| Sample rate | 1 MS/s (123 ns conversion) | 1.44 MS/s | 2 MS/s |
| Power at full rate | **36 µW** | 703 µW | 183 µW |
| Energy per conversion | **36 pJ** | ~488 pJ | ~92 pJ |
| Walden FoM | ~43 fJ/step (assuming ENOB 9.7) | not published | 48.6 fJ/step (measured) |
| Idle power (waiting for events) | **24 nW** (65 nW at 75 °C) | runs continuously | runs continuously |
| Area | not yet | 0.175 mm² | 0.36 mm² |

**Comparator**

| | **ANALOGIC (v6b, layout + MC)** | C: StrongARM (45 nm, sim) |
|---|---|---|
| Input range | **Rail-to-rail and beyond** (characterized VCM 30 … 1790 mV) | around a fixed VCM |
| Offset | **σ 1.6 – 2.8 mV** (Monte Carlo) | 28 mV |
| Decision time | 2.4 – 4.1 ns (CLK → READY) | 63 – 136 ps |
| Supply | 1.8 V | 1.2 V |

**In short:** thanks to event-driven operation, idle power is about a thousand times lower than the others and energy per conversion is much lower. Competitor B is measured silicon; ours is simulation so far, with ENOB verification and full-system layout still ahead.

---

## Roadmap

- [x] Rail-to-rail StrongARM comparator: layout, DRC/LVS, Monte Carlo
- [x] Event-driven 10-bit SAR reference design: 36 pJ, 24 nW idle
- [x] CDAC, sampling switches and common-mode reference verified block by block
- [x] CDAC (10-bit) and analog switch layouts: DRC clean, LVS matched, RC-extracted verification
- [x] Common-mode reference v3: layout, multi-sample verification under 10-bit load
- [x] CDAC DNL/INL improvement by shielding: DNL −0.82 → −0.20 LSB, INL 0.56 → 0.12 LSB
- [ ] Desktop demo: 8-bit R-2R SAR on a Nexys A7 FPGA
- [ ] Assemble the whole system in a hierarchical schematic
- [ ] ENOB verification by FFT
- [ ] Full ADC layout
- [ ] Pixel/channel-level event detection integration

## Tools

SkyWater **sky130A** · xschem · ngspice · magic · netgen · KLayout · IIC-OSIC-TOOLS

---

<div align="center"><sub>Design files are not open source for now. Get in touch for collaboration.</sub></div>
