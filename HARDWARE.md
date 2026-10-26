<div align="center"><sub><a href="README.md">← Home</a> · <a href="ASIC.md">The ASIC</a> · <a href="HARDWARE.md">Hardware proof</a></sub></div>

# Hardware proof: discrete 8-bit SAR on an FPGA

The SAR algorithm, the sample-and-hold sequence and the board-level control are proven on real hardware: a **discrete 8-bit R-2R SAR ADC board**, designed in KiCad, driven by a **Digilent Nexys A7-100T** (Artix-7) FPGA.

## The setup

```
  VIN ──► MCP6001 buffer ──► hold capacitor (10 nF) ──► MCP6561 comparator ◄── R-2R DAC (8 bit)
                ▲  SH                                           │ CMP              ▲ SEL1…SEL8
                └──────────── Nexys A7-100T (SAR controller, 7-segment, LEDs) ◄────┘
```

| Block | Implementation |
|---|---|
| DAC | 8-bit R-2R ladder, 100 kΩ ±0.5 % 25 ppm thin film (R = two 100 k in parallel) |
| Bit switches | 74HC4053 analog multiplexers, each bit switched between VREF and GND |
| Sample and hold | MCP6001 rail-to-rail buffer, 10 nF film hold capacitor, mux channel as switch |
| Comparator | MCP6561 rail-to-rail comparator |
| Reference / range | Nexys 3.3 V rail, 0 … 3.3 V, 1 LSB = 12.9 mV |
| Controller | Verilog SAR state machine on the Nexys A7, majority vote on every bit decision |
| User interface | 7-segment shows code and voltage, LEDs show code, comparator and state; single-shot, continuous 10 Hz and manual-DAC modes |

## The board

<table>
<tr>
<td align="center" width="50%"><img src="img/hw_pcb_3d.png" alt="Demo board 3D view"><br><sub><b>KiCad 3D view</b></sub></td>
<td align="center" width="50%"><img src="img/hw_pcb_layout.png" alt="Demo board PCB layout"><br><sub><b>PCB layout</b>, top and bottom copper, two layers with ground pour</sub></td>
</tr>
</table>

<div align="center">
<img src="img/hw_schematic.svg" width="900" alt="Demo board schematic">
<br><sub><b>Schematic:</b> buffer and hold capacitor, comparator, three 4053 switch banks, R-2R ladder, FPGA headers</sub>
</div>

## FPGA controller verification

The controller is simulated against a behavioural model of the board, including the board's real bit order and switch polarity taken from the KiCad netlist.

| Test | Result |
|---|---|
| Testbench, board model in the loop | **14 / 14 tests passed, 0 errors** |
| Full transfer sweep, 0 … 3.3 V | **1101 points, 0 code errors** |
| Codes across the range | 0 mV → 0, 13 mV → 1, 1.000 V → 77, 1.650 V → 128, 2.222 V → 172, 3.300 V → 255: **all exact** |
| Continuous mode and manual-DAC mode | **pass** |
| Timing at 100 MHz (Vivado) | **met, WNS +3.73 ns, WHS +0.12 ns** |
| FPGA resources | **188 LUTs, 243 flip-flops, 1 DSP** (0.3 % of the Artix-7 100T) |
| Conversion time | 100 µs sampling + 8 × 20 µs bits ≈ 262 µs |

<div align="center">
<img src="img/hw_sim.png" width="900" alt="FPGA controller simulation">
<br><sub><b>Controller + board model:</b> full transfer curve with no code errors, and one conversion stepping onto 2.222 V</sub>
</div>

## Bring-up

Board assembly and bench bring-up: **coming**.

| Measurement | Status |
|---|---|
| Oscilloscope: sample-and-hold, bit-by-bit DAC steps, comparator output | coming |
| Measured transfer curve and DNL / INL from a code histogram | coming |

---

<div align="center"><sub><a href="README.md">← Home</a> · <a href="ASIC.md">The ASIC</a> · <a href="HARDWARE.md">Hardware proof</a></sub></div>
