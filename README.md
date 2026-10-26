<div align="center">

<img src="img/banner.svg" alt="ANALOGIC: event-driven 10-bit SAR ADC in sky130" width="100%">

<br>

![PDK](https://img.shields.io/badge/PDK-sky130A-0f172a?style=flat-square&labelColor=0f172a&color=22d3ee)
![ADC](https://img.shields.io/badge/SAR%20ADC-10%20bit%20%C2%B7%201%20MS%2Fs-0f172a?style=flat-square&labelColor=0f172a&color=22d3ee)
![Layout](https://img.shields.io/badge/layout-DRC%20%2F%20LVS%20clean-0f172a?style=flat-square&labelColor=0f172a&color=22d3ee)
![Verified](https://img.shields.io/badge/verified-post--layout%2C%20all%20corners-0f172a?style=flat-square&labelColor=0f172a&color=22d3ee)

</div>

---

**ANALOGIC** is a clockless, event-driven SAR ADC: it sleeps until something happens, wakes on a single pulse, converts on a clock its own comparator generates, then goes back to sleep. It fits signals that are mostly quiet: neural recording, vibration monitoring, event-based imaging, sensor nodes.

The project has two parts:

<table>
<tr>
<td width="50%" valign="top" align="center">

### [🔬 The ASIC →](ASIC.md)

<a href="ASIC.md"><img src="img/adc_top_layout.png" alt="The ASIC" width="100%"></a>

**10-bit, 1 MS/s SAR ADC in sky130**

Complete layout, DRC 0 / LVS matched<br>
Post-layout: within 1 LSB at tt, ss 75 °C, ff<br>
Asynchronous rail-to-rail StrongARM comparator<br>
0.084 mm²

**[See the ASIC →](ASIC.md)**

</td>
<td width="50%" valign="top" align="center">

### [🛠️ Hardware proof →](HARDWARE.md)

<a href="HARDWARE.md"><img src="img/hw_pcb_3d.png" alt="Hardware proof" width="100%"></a>

**8-bit R-2R SAR board + Nexys A7 FPGA**

KiCad board, schematic to 3D<br>
FPGA SAR controller, timing met at 100 MHz<br>
Full transfer sweep: 1101 points, 0 code errors<br>
Bench measurements coming

**[See the hardware →](HARDWARE.md)**

</td>
</tr>
</table>

---

## At a glance

| | |
|---|---|
| ⚡ **No clock** | The comparator times every bit with its own READY pulse |
| 💤 **Event-driven** | One START pulse samples, converts and returns to sleep |
| ↔️ **Rail-to-rail comparator** | σ 1.6–2.8 mV offset with no calibration, 30 mV … 1.79 V common mode |
| 🔬 **Verified post-layout** | Complete ADC simulated with extracted parasitics at every corner: within 1 LSB, 1 MS/s |
| 📐 **Small** | 316 x 267 µm complete ADC |
| 🛠️ **Proven in hardware** | Same SAR algorithm on a discrete board driven by an FPGA |

---

## Roadmap

- [x] Rail-to-rail asynchronous StrongARM comparator: layout, DRC/LVS, Monte Carlo
- [x] Shielded 10-bit CDAC, switches and common-mode reference: layout, DRC/LVS, RC-extracted verification
- [x] Comparator with STOP input
- [x] SAR logic: RTL, synthesis, place-and-route, gate-level verification
- [x] Complete ADC layout: DRC 0, LVS matched, 0.084 mm²
- [x] Transistor-level conversions on the extracted ADC: within 1 LSB, 1 MS/s at every corner
- [x] Post-layout conversion energy per supply
- [x] Hardware proof: 8-bit R-2R SAR board (KiCad) and Nexys A7 controller, 14 / 14 testbench tests, timing met
- [ ] Full per-sample and idle power over back-to-back conversions
- [ ] ENOB verification by FFT
- [ ] Hardware proof: bench bring-up and oscilloscope captures
- [ ] Pixel/channel-level event detection integration

## Tools

**ASIC:** SkyWater **sky130A** · xschem · ngspice · Magic · netgen · KLayout · Yosys · OpenROAD · IIC-OSIC-TOOLS<br>
**Hardware:** KiCad · Vivado · Digilent Nexys A7-100T

---

<div align="center"><sub>Design files are not open source. Get in touch for collaboration.</sub></div>
