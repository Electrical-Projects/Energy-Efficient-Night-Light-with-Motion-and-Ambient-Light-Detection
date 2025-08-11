# Design Overview & Specifications

## Overview
This board implements an **energy-efficient night light** with:
- **Motion detection** via a PIR sensor.
- **Ambient light gating** so it only triggers under desired lighting conditions.
- **Configurable on-time** using a **low-power CMOS 555** timer.
- **Two LED outputs** for flexible lighting arrangements.
- **No microcontroller** — entirely discrete and inspectable.

**Power:** 4×1.5 V - AAA Batteries (≈6 V)  
**Rails:** 6 V (LEDs only) and 3.3 V (analog front-end + logic + 555).

---

## System Block Diagram

4×AAA (≈6 V)
│
├── 6 V rail ──────> LED driver (2× headers)
│ (series resistors / optional transistor)
│
└── 3.3 V LDO ─────> TLV8544 nanopower quad op-amp:

A: PIR AC preamp (HP+LP)
B: PIR gain + band-limit
C/D: Window comparator → logic → trigger

↑
LDR + pot (ambient-light threshold gating)
│
└──> ICM7555 (CMOS 555) monostable at 3.3 V → LED driver enable


---

## Key Specifications

| Parameter                 | Value / Range                         |
|---------------------------|----------------------------------------|
| Supply Voltage            |≈6.4 V fresh → 4.4 V depleted
| Rails                     | 6 V (LEDs only), 3.3 V (AFE/logic/555) |
| Quiescent Current         | µA range (TLV8544 + high-value bias)   |
| Motion Passband           | ≈ 0.6–5 Hz                              |
| Ambient Light Threshold   | Full daylight → full darkness           |
| On-Time Range             | 0–72 s (adjustable)                     |
| Outputs                   | 2× LED headers                          |
| PIR Lens                  | Round or inline Fresnel                 |

---

## Sensors

### PIR Sensor & Optics
- **Sensor:** Dual-element PIR (e.g., Zilog **ZRE200GE** or Murata **IRA-S210ST01**).
- **Lens:** Fresnel (inline or round) creating alternating hot/cold zones to produce AC motion signal.

### Ambient Light Sensor
- **LDR:** CdS photoresistor (e.g., **PDV-P8103**).  
  Dark ≥0.5 MΩ; ~16–33 kΩ @ 10 lux.  
- Divider with 1.5 MΩ fixed + 2 MΩ pot sets adjustable ambient-light threshold.

---

## Power Architecture

- **6 V rail (battery):** Feeds **only** the LED path and the **input** of the LDO.
- **3.3 V rail (LDO):** Feeds **TLV8544**, **ICM7555**, and logic (e.g., SN74LVC1G00).
- **Decoupling:**  
  - 6 V: bulk electrolytic near LED return + 100 nF local.  
  - 3.3 V: 10 µF + 100 nF at the LDO output, 100 nF at each IC.
- **Grounding:** Star from LDO ground; keep LED return currents off the analog ground where possible.

**Why 555 @ 3.3 V?** Lower supply spikes, guaranteed 3.3 V logic levels, and the CMOS 555 (ICM7555) maintains ultra-low supply current. Timing (`t ≈ 1.1RC`) is **independent of supply voltage**.

---

## Analog Front-End (TLV8544, Nanopower)

We use **TLV8544** — a **nanopower**, rail-to-rail quad op-amp — for the PIR chain.

### Stage A — AC Preamp & High-Pass
- Rejects the PIR’s DC output and slow thermal drifts.
- **Passband:** ~0.6–5 Hz.
- **Gain:** ×50–60 (set by feedback and input caps/resistors).

### Stage B — Second Band-Pass & Gain
- Same passband; main amplification.
- **Gain:** ~×50 (total chain ≈ ×2500–×3000).
- Mid-rail bias at **1.65 V** (½·3.3 V) from a high-value divider with local decoupling.

### Stages C & D — Window Comparator (3.3 V)
- Converts the amplified AC into a clean digital motion event.  
- Typical thresholds (vs 3.3 V mid-rail signal):  
  - **Upper:** ≈ **0.84·Vcc** ≈ **2.77 V**  
  - **Lower:** ≈ **0.16·Vcc** ≈ **0.53 V**  
- Outputs feed a small logic gate (NAND/NOR) at **3.3 V** logic level to trigger the 555.

**Why nanopower?** TLV8544’s nA–µA bias keeps standby current tiny, extending AAA life while providing enough bandwidth for human motion.

---

## Ambient Light Gating (3.3 V)
- LDR + 1.5 MΩ + 2 MΩ pot create an adjustable threshold into a 3.3 V comparator/op-amp gate.  
- Motion events are **blocked** unless the ambient light matches the user-set condition (daylight to darkness).

---

## Delay Stage — ICM7555 (CMOS 555 @ 3.3 V)
- **Mode:** Monostable; triggered by motion logic from the window comparator.
- **Timing:**  
  `t_on ≈ 1.1 × R × C`  
  Example: **C = 33 µF**, **Rpot ≈ 0–2 MΩ** → **0–72 s**.
- **Notes:**  
  - Timing is largely independent of supply voltage; running at **3.3 V** improves noise behavior.  
  - Use CMOS 555 (ICM7555) for **low supply current** and **no crowbar spikes**.

The 555 output (3.3 V logic) enables the **LED driver** path that resides on the 6 V rail.

---

## LED Outputs (6 V domain)
- 2× headers with series resistors for direct LED drive from 6 V.  
- Optional low-side NPN/MOSFET if you want higher LED current or multiple LEDs per channel.
- Tie driver **logic input** to the 3.3 V 555 output through a level-safe interface if required (many FET gates are fine; check your specific transistor logic-level specs).

---

## PCB & Layout Notes
- Mostly **through-hole** for easy hand assembly.  
- **SMD** in the PIR front-end to keep high-impedance nodes short and low-noise.  
- Keep PIR input traces short, guard high-impedance nodes, and isolate LED return currents.  
- Place the LDO and 3.3 V decoupling close to the TLV8544 and 555.

---

## Adjustment & Testing
1. Power with fresh AAA cells; verify **6 V** and **3.3 V** rails.  
2. Allow **10–20 s** PIR warm-up.  
3. Set **ambient-light pot** to the desired day/night threshold.  
4. Set **delay pot** for on-time (**0–72 s**).  
5. Walk test across lens zones; confirm clean triggering and LED on-time.

---

**Related files:**  
- [Electrical Design](../../Electrical-Design/) — Schematic, BOM.  
- [Mechanical Design](../../Mechanical-Design/) — Enclosure, diffuser.
