<div align="center">

<img src="img/banner.svg" alt="ANALOGIC: event-driven 10-bit SAR ADC in sky130" width="100%">

<br>

![PDK](https://img.shields.io/badge/PDK-sky130A-0f172a?style=flat-square&labelColor=0f172a&color=22d3ee)
![ADC](https://img.shields.io/badge/SAR%20ADC-10%20bit%20%C2%B7%201%20MS%2Fs-0f172a?style=flat-square&labelColor=0f172a&color=22d3ee)
![Layout](https://img.shields.io/badge/layout-DRC%20%2F%20LVS%20clean-0f172a?style=flat-square&labelColor=0f172a&color=22d3ee)
![Status](https://img.shields.io/badge/full%20ADC-in%20layout%2C%20RC%20verified-0f172a?style=flat-square&labelColor=0f172a&color=64748b)

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
| **Down to layout** | Complete ADC (analog core + SAR logic) in one layout: **DRC 0, LVS matched**, extracted conversions within 1 LSB at slow and fast corners, **1 MS/s at ss 75 °C** |

<div align="center">
<img src="img/guc_olay_hizi.png" width="760" alt="Power vs event rate">
<br><sub><b>Average power scales with the event rate:</b> at a few events per second the ADC stays near 25 nW</sub>
</div>

<div align="center">
<img src="img/sar_donusum.png" width="760" alt="SAR conversion">
<br><sub><b>Asynchronous 10-bit conversion:</b> every READY pulse resolves one bit</sub>
</div>

---

## The complete ADC, end to end

The whole converter is in one layout: signal CDAC, **replica CDAC**, bit drivers and input switches, common-mode generators, the asynchronous comparator **and the SAR logic**. It is RC-extracted and simulated at transistor level, from the START pulse to the final code.

| | Result |
|---|---|
| Physical verification | **DRC 0, LVS matched** for the complete ADC |
| Full conversion, extracted parasitics | Bottom, mid and top of the range at **slow 75 °C** and **fast 25 °C**: every code **within 1 LSB** |
| Speed, worst corner (ss, 75 °C) | 500 ns sampling + **284–288 ns** conversion = **788 ns** → **1 MS/s** holds at the slow corner |
| Speed, fast corner (ff, 25 °C) | **147–149 ns** conversion |
| Sampling accuracy | Top plate settles within **0.15 mV** by the end of sampling |

<div align="center">
<img src="img/adc_full_cycle.png" width="900" alt="Full ADC cycle from START">
<br><sub><b>One full cycle from START (ss, 75 °C):</b> the input is sampled on the DAC bottom plates while START is high, then the comparator's own READY pulses walk through the 10 bits. Code 512, as expected</sub>
</div>

<div align="center">
<img src="img/adc_binary_search.png" width="900" alt="SAR binary search">
<br><sub><b>The binary search, bit by bit:</b> DAC trial level against the input, with the comparator input at every decision</sub>
</div>

<div align="center">
<img src="img/adc_corners.png" width="900" alt="Corner results">
<br><sub><b>Corner results</b> on the extracted layout</sub>
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

**STOP version:** the SAR logic can freeze the self-timed loop cleanly, with no fight current. Layout DRC 0, LVS matched. On the RC-extracted layout (tt): during STOP, CLK stays below **0.58 mV** and the cell draws **5.4 µA**, and decisions before and after STOP are correct.

**Why it matters:** the rail-to-rail range lets the ADC converge around mid-supply (0.9 V) while the input swings the full range, and the built-in READY output turns the comparator into its own clock generator.

---

## System blocks

Each block was verified on its own, then added to the next level. Design files are not published; this page shows what each block does and how it performs.

### Capacitor DAC (CDAC)
Holds the sampled input and tries one bit per step: the SAR's balance.

| Metric | Result |
|---|---|
| Structure | 10-bit binary-weighted MIM array, 2D common-centroid, dummy ring, grounded shield between bit lines and top plate |
| Linearity (RC-extracted) | **DNL −0.20 LSB, INL 0.12 LSB** worst case |
| Bit settling to 0.5 LSB (RC-extracted) | **6.2 / 8.1 / 5.3 ns** (tt / ss 75 °C / ff), correct code at all corners |
| Layout | 128.6 x 177.3 µm, **DRC 0** (Magic + KLayout), **LVS matched** |

### Sampling and analog switches
Takes the input onto the capacitors and ties the plates to the references.

Bottom-plate sampling through per-bit input switches, top plate held at VCM. Two transmission-gate cells, both with layout DRC 0, LVS matched and layout = schematic:

| Switch | Settling to 0.5 LSB (2 pF) | Peak Ron tt / ss / ff | Off leakage (worst) |
|---|---|---|---|
| Standard-Vt | 42.6 ns | 5.2k / 15.8k / 2.2k Ω | |
| **Low-Vt** | **13.6 ns** | **1.6k / 2.4k / 1.2k Ω** | **0.31 nA** (ff, 75 °C) |

### Common-mode reference (VDD/2)
Sets the comparison point; passive and very low current.

| Metric | Result |
|---|---|
| Output | **0.900 V** (VDD/2, ratiometric to supply) |
| Current / power | ~20 nA / ~36 nW (weak branch) |
| 10-bit sampling load | Always-on weak branch plus a strong branch that engages only during sampling. **No drift over 100 consecutive samples** |
| Layout | 24.7 x 49.9 µm, **DRC 0, LVS matched**, RC-extracted result matches the schematic |

### SAR logic and sequencer
Writes the bits, triggers and stops the comparator, generates its own timing.

| Metric | Result |
|---|---|
| Conversion | 10 bit from one START pulse, **123 ns** |
| Modes | Early stop (6 bit, 26 pJ), single-decision window check (9.7 pJ) |
| Safety | Timeout protection, returns to sleep by itself when done |

---

## How it was verified

Every block follows the same flow before it is used one level up:

1. **Schematic simulation** (xschem + ngspice) at tt / ss / ff, 25 °C and 75 °C
2. **Layout** in Magic, mostly script-generated so it can be rebuilt after a change
3. **DRC** (Magic, plus KLayout for the CDAC and STOP comparator) and **LVS** (netgen)
4. **Parasitic extraction** (R and C) and the same testbench re-run on the extracted netlist
5. **Monte Carlo** where matching matters (comparator offset), with a small sample set
6. **System test** on the complete extracted ADC at slow and fast corners, with a connectivity check script that runs before every simulation

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

**In short:** thanks to event-driven operation, idle power is about a thousand times lower than the others and energy per conversion is much lower. Competitor B is measured silicon; ours is simulation so far, with the complete ADC already in layout and ENOB verification next.

---

## Roadmap

- [x] Rail-to-rail StrongARM comparator: layout, DRC/LVS, Monte Carlo
- [x] Event-driven 10-bit SAR reference design: 36 pJ, 24 nW idle
- [x] CDAC, sampling switches and common-mode reference verified block by block
- [x] CDAC (10-bit) and analog switch layouts: DRC clean, LVS matched, RC-extracted verification
- [x] Common-mode reference v3: layout, multi-sample verification under 10-bit load
- [x] Shielded 10-bit CDAC: DNL −0.20 LSB, INL 0.12 LSB (RC-extracted)
- [x] Full analog core layout (CDAC + replica, drivers, VCM, comparator): DRC 0, LVS matched
- [x] Comparator with STOP input: layout DRC 0, LVS matched, extracted simulation
- [x] SAR logic placed with the analog core: complete ADC DRC 0, LVS matched
- [x] Transistor-level conversions on the extracted ADC: within 1 LSB, 1 MS/s at ss 75 °C
- [ ] Desktop demo: 8-bit R-2R SAR on a Nexys A7 FPGA
- [ ] ENOB verification by FFT
- [ ] Pixel/channel-level event detection integration

## Tools

SkyWater **sky130A** · xschem · ngspice · magic · netgen · KLayout · IIC-OSIC-TOOLS

---

<div align="center"><sub>Design files are not open source for now. Get in touch for collaboration.</sub></div>
