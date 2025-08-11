# Project Review – Design Rationale & Theory

## 1. Project Overview
This project is a discrete motion-activated LED circuit with **user-adjustable light sensitivity** and **adjustable output delay**, powered by 4×AAA batteries.  
It requires **no microcontroller**, instead using **nanopower analog circuitry** to minimise standby drain for long battery life.  

Most of the design is through-hole for ease of assembly, except the PIR sensor board, which uses SMD parts to accommodate pre-processing circuitry and ensure low power draw.

## 2. Functional Requirements
- Adjustable delay: **0–72 seconds**.
- Adjustable light sensitivity: **from full daylight (~215 kΩ LDR) to complete darkness (>10 MΩ)**.
- Continuous PIR monitoring while maximising battery life.
- Dual LED outputs.
- Operates from ~6 V battery pack with onboard 3.3 V regulation.

## 3. Circuit Block Diagram
*(Insert diagram here)*

**Stages**:
1. **Power Regulation** – 3.3 V LDO supply for comparators and logic.
2. **PIR Stage** – Nanopower amplification and filtering.
3. **Light Sensitivity Stage** – Nanopower window comparator.
4. **Logic Gate** – NAND combination of PIR and light conditions.
5. **Delay Stage** – 555 timer monostable.
6. **LED Driver** – NPN transistor output stage.

## 4. Key Component Selection
- **PIR Module** – Compact SMD design for low quiescent draw.
- **TLV8544 Quad Nanopower Op-Amp** – ~1 µA/channel quiescent current, rail-to-rail I/O, 1.7–5.5 V operation.
- **SN74LVC1G00 NAND Gate** – Single NAND in SOT-23-5, 3.3 V logic.
- **CMOS NE555 Timer** – Low power monostable.
- **2N3904 Transistor** – NPN LED driver.

## 5. Detailed Stage Analysis

### 5.1 Power Regulation
- **Source:** 4×AAA cells (~6.0 V fresh, ~4.8 V near depletion).
- **Regulator:** 3.3 V LDO with low quiescent current.
- **Why 3.3 V?**
  - Matches TLV8544 optimal low-power operation.
  - Keeps SN74LVC1G00 within voltage spec.
  - Reduces LED drive power consumption when desired.
- **Nanopower Impact:**  
  At 3.3 V, the TLV8544 draws **~4 µA total** for all four op-amp channels. This means the always-on PIR/light detection stages can operate for months on AAA batteries without significant standby drain.

### 5.2 PIR Signal Conditioning (Stages A & B)
- Two TLV8544 channels form a high-gain, low-noise amplifier and filter for the PIR signal.
- Nanopower architecture allows the PIR circuitry to remain continuously active with minimal drain.
- Because PIR signals are low frequency (<10 Hz) and low amplitude, the TLV8544’s rail-to-rail input and low bias current are critical.

### 5.3 Light Sensitivity Control (Stages C & D)
- Implemented as a **window comparator** using two TLV8544 channels.
- **Nanopower advantage:**  
  The light sensing stage remains powered 24/7 without impacting overall current budget — vital for battery longevity.
- **Divider configuration:**  
  - LDR ("photo boi") + 2 MΩ pot + 1.5 MΩ series resistor (final chosen values).
  - Gives adjustable switching from ~215 kΩ (daylight) to 1 MΩ (low light) with fine control.
- Threshold points verified through modelling and PCB testing.

### 5.4 Logic Gate Stage
- SN74LVC1G00 NAND ensures motion AND correct light conditions are required before triggering delay.
- Minimal quiescent current due to CMOS design.

### 5.5 Delay Stage (555 Timer)
- CMOS NE555 for monostable timing, drawing minimal standby current.
- Adjustable via pot + timing capacitor (33 µF).
- Delay range calculated as:
  \[
  T = 1.1 \times R \times C
  \]
  Achieves ~0–72 seconds range.

### 5.6 LED Driver Stage
- 2N3904 NPN in low-side switching configuration.
- PCB supports **two LED outputs** — used in this project to drive the eyes of a decorative cat ornament for night lighting.
- Current-limiting resistors set for brightness vs battery life trade-off.

---
**Next:** [02_build.md](02_build.md) – Assembly & Fabrication
