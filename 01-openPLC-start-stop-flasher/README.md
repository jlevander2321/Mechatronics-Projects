# OpenPLC ESP32 Start/Stop Flasher

**PLC Programming · OpenPLC · ESP32 · Ladder Logic**

A PLC-style start/stop control system built on an ESP32 running OpenPLC. A seal-in circuit with Stop priority runs two TON timers that flash an LED 500 ms on and 500 ms off.

---

## Contents

- [Demo](#demo)
- [Hardware](#hardware)
- [Wiring](#wiring)
- [I/O Table](#io-table)
- [Ladder Program](#ladder-program)
- [How It Works](#how-it-works)
- [Testing](#testing)
- [Problems and Fixes](#problems-and-fixes)
- [Design Notes](#design-notes)
- [Limitations](#limitations)
- [What I Learned](#what-i-learned)
- [Files](#files)

---

## Demo

https://github.com/user-attachments/assets/9763dca3-9098-405a-b3b5-6d1c204e3448

---

## Hardware

| Part | Qty | Purpose |
|------|-----|---------|
| ESP32 dev board (ELEGOO) | 1 | Runs the OpenPLC runtime |
| Pushbutton, normally open | 2 | Start and Stop inputs |
| 10 kΩ resistor | 2 | Pull-downs for the button inputs |
| LED (red) | 1 | Output indicator |
| 220 Ω resistor | 1 | LED current limiting |
| Breadboard and jumper wires | | Connections |

---

## Wiring

![Wiring diagram](wiring-diagram.svg)

*Schematic made in KiCad. Source files: [`start-stop-flasher.kicad_sch`](start-stop-flasher.kicad_sch) and [`start-stop-flasher.kicad_pro`](start-stop-flasher.kicad_pro).*

- **Start button:** one side to 3.3 V, the other side to GPIO 18. A 10 kΩ resistor goes from GPIO 18 to GND.
- **Stop button:** one side to 3.3 V, the other side to GPIO 19. A 10 kΩ resistor goes from GPIO 19 to GND.
- **LED:** GPIO 5 to the 220 Ω resistor, resistor to the LED anode, LED cathode to GND.

<img width="400" alt="Breadboard wiring with pull-down resistors" src="https://github.com/user-attachments/assets/128b830d-5d33-46b5-b612-bcb634254d3e" />

---

## I/O Table

| Device | Type | ESP32 Pin | PLC Address | Variable |
|--------|------|-----------|-------------|----------|
| Start button | Digital input | GPIO 18 | `%IX0.0` | `start` |
| Stop button | Digital input | GPIO 19 | `%IX0.1` | `stop` |
| LED | Digital output | GPIO 5 | `%QX0.0` | `light` |
| Run bit | Internal BOOL | | | `RUN` |
| Timer 1 done | Internal BOOL | | | `T1_DONE` |
| Timer 2 done | Internal BOOL | | | `T2_DONE` |
| Timer 1 | TON | | | `T1` |
| Timer 2 | TON | | | `T2` |

<img alt="OpenPLC variable table" src="https://github.com/user-attachments/assets/54bf9e0a-ed38-4a12-8022-5c3d1bc08da7" />

---

## Ladder Program

<img alt="Ladder program in OpenPLC Editor" src="https://github.com/user-attachments/assets/efe8609b-00c6-40fc-9e5f-d73fc29e79a6" />

*Four rungs: seal-in, Timer 1, Timer 2, and LED output. Full program in `plc.xml`.*

---

## How It Works

**Rung 1: Seal-in.** Pressing Start turns on `RUN`. The `RUN` contact in parallel with Start keeps the rung on after Start is released. Stop sits in series with both branches, so pressing Stop breaks the rung and drops `RUN`. Because Stop is in series after the branch, Stop wins if both buttons are pressed.

**Rung 2: Timer 1.** While `RUN` is on and `T2_DONE` is off, T1 times for 500 ms. When it finishes, `T1_DONE` turns on.

**Rung 3: Timer 2.** When `T1_DONE` turns on, T2 starts timing for 500 ms. When it finishes, `T2_DONE` turns on.

**Rung 4: LED.** The LED follows `T1_DONE`.

### Flash sequence

| Time after Start | T1 | T2 | LED |
|------------------|----|----|-----|
| 0 to 500 ms | Timing | Off | Off |
| 500 to 1000 ms | Done | Timing | On |
| At 1000 ms | `T2_DONE` resets T1, which resets T2 | | Off |
| Cycle repeats | | | |

The LED stays off for the first 500 ms after Start because T1 has to finish before the LED turns on.

---

## Testing

| # | Test | Expected | Result |
|---|------|----------|--------|
| 1 | Press and release Start | LED flashes and keeps flashing after release | Pass. Measured 0.98 s per full cycle (1.0 s expected) |
| 2 | Press Stop while flashing | LED turns off and stays off | Pass |
| 3 | Hold Start and Stop together | Nothing happens (Stop priority) | Pass |

**How I measured timing:** I timed 10 full on/off cycles with a stopwatch in one run and divided by 10. Timing multiple cycles spreads my reaction-time error across all 10, so each cycle is accurate to about ±0.03 s. The measured 0.98 s is within that error of the expected 1.0 s.

---

## Problems and Fixes

### 1. Floating inputs

- **Problem:** The LED changed state when I touched the board, even with no buttons pressed.
- **Cause:** With the button open, the input pin wasn't connected to anything. It picked up electrical noise and read random HIGH and LOW values.
- **Fix:** I added a 10 kΩ pull-down resistor from each input to GND. The inputs now sit at a known LOW until a button is pressed.

### 2. Wrong data type on an I/O address

- **Problem:** I declared an I/O variable as `DINT` and gave it a `%IX` location. OpenPLC rejected it.
- **Cause:** `%IX` is a single input bit that can only be 0 or 1. A `DINT` is a 32-bit number, so it doesn't fit in one bit.
- **Fix:** I changed the I/O variables to `BOOL`. Physical digital I/O uses `BOOL` with `X` (bit) addresses like `%IX0.0` and `%QX0.0`. Internal bits like `RUN`, `T1_DONE`, and `T2_DONE` are `BOOL` with no location, because they don't connect to a pin.

### 3. TON block outlined in red

- **Problem:** The timer blocks showed a red outline and wouldn't compile.
- **Cause:** Each TON block needs its own instance variable to store its elapsed time.
- **Fix:** I created separate TON variables, `T1` and `T2`, and assigned one to each block.

---

## Design Notes

### Why the Stop contact is normally closed in the ladder

The physical Stop button is normally open, so the `stop` input reads 0 until it's pressed. In the ladder, Stop uses a normally closed contact. That contact passes power while `stop` is 0 and breaks the rung when `stop` goes to 1.

### Why 10 kΩ pull-downs

When a button is pressed, current flows from 3.3 V through the pull-down to GND. At 10 kΩ, that's 3.3 V / 10 kΩ = 0.33 mA, which is tiny. A much smaller resistor like 100 Ω would draw 33 mA every press and waste power. A much larger resistor like 1 MΩ would hold the input LOW so weakly that noise could still flip it. 10 kΩ sits between those extremes.

The ESP32 has internal 45 kΩ pull-down resistors, but I used external 10 kΩ resistors instead. A lower resistance holds the input LOW more firmly against noise, and an external resistor works no matter how the firmware is configured. Using the internal ones would have required changing the firmware setup instead of just wiring a resistor.

### LED current (measured)

I measured the voltages with the LED on:

| Measurement | Value |
|-------------|-------|
| LED forward voltage | 1.920 V |
| Resistor voltage | 1.243 V |
| Resistor | 220 Ω |
| **LED current** (1.243 V ÷ 220 Ω) | **5.65 mA** |

The textbook formula, (3.3 V − 1.92 V) ÷ 220 Ω, predicts 6.27 mA. The measured current came out lower because the pin only delivered about 3.16 V under load instead of a full 3.3 V. That drop is within spec. The datasheet only guarantees the pin stays above 0.8 × 3.3 V = 2.64 V while sourcing current.

5.65 mA is far below the 40 mA typical high-level source current that Espressif's ESP32 Series Datasheet lists for the VDD3P3_CPU power domain (measured at maximum drive strength).

---

## Limitations

**The Stop circuit isn't fail-safe.** Stop uses a normally open button. If the wire to it breaks, the input reads 0 forever, which looks exactly like "not pressed." The system could never be stopped. Industrial stop buttons are wired normally closed for this reason. The input reads 1 during normal operation, and the ladder uses a normally open contact. Pressing Stop *or* a broken wire both drop the input to 0, and the machine stops either way.

**This isn't an emergency stop.** A real E-stop uses a hardwired safety circuit with safety-rated relays that cut power directly. It doesn't rely on a PLC program.

**This isn't industrial hardware.** Industrial PLCs usually use 24 VDC I/O with isolated input cards. The ESP32 runs 3.3 V logic and isn't rated for a plant floor. This project demonstrates PLC programming concepts on low-cost hardware.

---

## What I Learned

- How a seal-in circuit latches a run bit and why series Stop gives Stop priority
- How two TON timers reset each other to make a flasher
- Why floating inputs happen and how pull-down resistors fix them
- How IEC 61131-3 data types match PLC addresses (`BOOL` to `%IX` and `%QX`)
- Why real stop circuits are wired normally closed
- How to measure real circuit values and compare them to the datasheet

---

## Files

| File | Description |
|------|-------------|
| `plc.xml` | OpenPLC Editor project file |
| `beremiz.xml` | OpenPLC Editor project settings |
| `wiring-diagram.svg` | Wiring diagram |
| `start-stop-flasher.kicad_sch` | KiCad schematic source |
| `start-stop-flasher.kicad_pro` | KiCad project file |
