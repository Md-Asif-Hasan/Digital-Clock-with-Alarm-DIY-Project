# Digital Alarm Clock — Detailed Workflow

> A complete, practical step-by-step workflow to collect components, design the PCB in PCBWeb, get the PCB manufactured (printed), solder/assemble the parts, and test the finished digital alarm clock. This workflow assumes a 6-digit (HH:MM:SS) arrangement using 7‑segment LED displays (7 LEDs per digit), a buzzer for the alarm, a single button for alarm setting, decoder/display driver ICs and comparator ICs, a 555 timer as the frequency generator, and other essential passive components.

---

## Table of contents

1. Project overview & scope
2. Required tools and software
3. Bill of Materials (BOM) — starter
4. Component procurement checklist & tips
5. Schematic design workflow (PCBWeb)
6. PCB layout workflow (PCBWeb)
7. Manufacturing / printing the PCB (fab steps)
8. Assembly (soldering) workflow
9. Testing, calibration & debugging
10. Final notes: accuracy, improvements, and version control
11. Appendix: BOM CSV + Design Rule suggestions

---

## 1) Project overview & scope

You will build a discrete-logic digital alarm clock on a PCB with:
- 6 × 7‑segment LED displays (each with 7 LED segments) for `HH:MM:SS` display.
- A buzzer driven by a transistor when the alarm condition is latched.
- One pushbutton (or tactile switch) for entering/setting the alarm.
- Logic ICs: decade/dual-decade counters, BCD-to-7seg decoders (drivers), equality comparators, a latch (D‑FF) for alarm, and a 555 timer used as the 1 Hz timebase.
- Supporting passives (resistors, caps), power regulator (5 V), and connectors.

This workflow covers procurement → PCB design in PCBWeb → fabrication → assembly → testing.

---

## 2) Required tools and software

**Software**
- PCBWeb (online CAD) — create account and start a new project.
- A text editor (VS Code, Notepad++) to view/export BOM/notes.
- Optional: KiCad or Eagle if you prefer those tools for schematic review.

**Hardware & shop tools**
- Soldering iron (temperature controlled, ~350 °C tip for lead-free; lower for leaded solder)
- Solder (lead-free SAC305 or 60/40 if you can use leaded)
- Flux (pen or paste) and solder wick / desoldering braid
- Helping hands / PCB holder
- Digital multimeter (DMM)
- Oscilloscope (helpful to check the 1 Hz timer signal & debug)
- Tweezers, flush cutters, small screwdriver set, needle files
- ESD mat & wrist strap recommended
- Fine-tip soldering iron tip (0.5–1.0 mm)
- Optional: hot-air rework station if you choose SMD parts

**Consumables**
- PCB standoffs, M3 screws, masking tape
- Isopropyl alcohol for cleaning flux

---

## 3) Bill of Materials (starter)

Below is a practical starter BOM. Quantities are for **one** alarm clock PCB.

| Ref | Qty | Part | Notes / Suggested footprint |
|-----|-----:|------|-----------------------------|
| U1 | 1 | NE555 (or LM555) | Timer configured as astable 1 Hz. DIP‑8 (through‑hole) |
| U2-U7 | 6 | 74HC390 (or 40190/4026 alternative) | Dual-decade counters, DIP‑16 |
| U8-U13 | 6 | 74HC4511 (or CD4511) | BCD → 7‑segment latch/driver, DIP‑16 |
| U14-U15 | 2 | 74HC688 (8‑bit equality comparator) | Each compares 8 bits — cascade 2 for 16 bits; DIP‑16 |
| U16 | 1 | CD4013 | Dual D flip‑flop for alarm latch, DIP‑14 |
| DSP1–DSP6 | 6 | 7‑segment LED displays (common‑cathode) | THT through‑hole displays suggested (0.36"–0.56" typical) |
| BZ1 | 1 | Passive buzzer | Driven by NPN transistor; check drive current |
| Q1 | 1 | 2N2222 / BC337 | NPN transistor for buzzer drive, TO‑92 |
| R_base | 1 | 10 kΩ | Base resistor for Q1 (adjust by transistor & buzzer) |
| R_seg | 7×6 | Segment resistors | One resistor per segment per digit; typical 220–470 Ω depending on LED Vf and desired brightness |
| R_pull | 1–4 | 10 kΩ | Pull‑ups/pull‑downs for switches as needed |
| C_timing | 1 | timing capacitor for 555 | Value chosen with R to make 1 Hz (see 555 calc) |
| C_decouple | per IC | 0.1 μF | Ceramic decoupling caps close to each IC VCC–GND |
| C_bulk | 1 | 10 μF | Bulk capacitor near power input |
| J1 | 1 | DC barrel jack / power header | For external 5 V supply, or 9 V + 5 V regulator |
| Ureg | 1 | 7805 (if using unregulated input) | TO‑220 or SOT‑223 as preferred |
| SW1 | 1 | Pushbutton (tactile) | Alarm set / mode button |
| PCB | 1 | 2‑layer FR4 PCB | 1.6 mm typical (specified at fab) |

**Notes:** If you want better timing accuracy consider a 32.768 kHz crystal + CD4060 instead of the 555 (or use DS3231 RTC). The 555 is simple but will drift with temperature and supply.

---

## 4) Component procurement checklist & tips

1. **Decide package types**: For a beginner-friendly assembly, choose primarily **through‑hole (THT)** parts (DIP ICs, axial/RADIAL passives). Through‑hole is easier to solder by hand.
2. **Select display size**: 0.36" or 0.56" common‑cathode 7‑segment displays are popular. Confirm pinout and orientation in vendor datasheet, and choose footprint in PCBWeb accordingly.
3. **Buzzer type**: Passive buzzer (requires driven tone frequency) vs active buzzer (built‑in oscillator). For simplicity use **active** buzzer so the alarm latch simply applies Vcc to it. If using passive, drive with oscillation transistor or square wave from 555.
4. **Buy sockets for DIP ICs**: 14/16‑pin IC sockets make rework safer and prevent heat damage.
5. **Order a few extras**: at least 2–3 of each IC and 7‑seg display in case of DOA or assembly mistakes.
6. **Sourcing**: use reputable suppliers (local electronics stores, or online suppliers like DigiKey, Mouser, or regional equivalents). For lower cost and larger lead times consider AliExpress or local distributors.

---

## 5) Schematic design workflow (PCBWeb)

> **Goal:** create a complete schematic with parts labelled, nets named (e.g., `CLK_1HZ`, `S_U_B0`), and annotation ready for PCB layout and BOM export.

### Steps

1. **Create project** in PCBWeb and name it e.g., `digital_clock_v1`.
2. **Add library parts**:
   - Search PCBWeb libraries for NE555, 74HC390, 74HC4511, 74HC688, CD4013, DIP sockets, 7‑seg part (or generic LED segment footprints), buzzer, 2N2222.
   - If a part is missing, add a generic part or create a custom symbol referencing exact pinouts from the datasheet.
3. **Place the power block**: DC jack, optional 7805 regulator, decoupling caps (0.1 μF per IC, 10 μF bulk), and connectors.
4. **Place the 555 timer**: configure in astable mode to target 1 Hz. Add trimmer or potentiometer in series with resistors to enable fine tuning of the frequency.
   - Add a note with the 555 charge/discharge equation or the R/C values you chose.
5. **Place counters and drivers**: place the 74HC390 chain for seconds, minutes and hours. Label all BCD outputs as nets (S_U_B0..B3, S_T_B0..B3, M_U_B0..B3, etc.).
6. **Place 4511 drivers and displays**: connect BCD nets into 4511 input pins; wire segment outputs of the 4511 to the 7‑segment display segments; place segment resistors close to either the 4511 outputs or the display pins (recommended near driver).
7. **Alarm set & comparator**: place DIP switches or pushbutton-based alarm-set inputs (AS_HU, AS_HT, AS_MU, AS_MT). Route these BCD bits into the 74HC688 comparators (U14/U15) and tie the comparator `A=B` output to the D input of CD4013 (alarm latch). Add debouncing (RC or small Schmitt buffer) for mechanical switches.
8. **Alarm output driver**: CD4013 Q → resistor → base of 2N2222 → buzzer (transistor collector to buzzer, emitter→GND). Add flyback diode if buzzer is inductive (usually not necessary for active buzzers but safe).
9. **Add labels & net names**: use consistent, descriptive net names. Add test points for `CLK_1HZ`, Vcc, GND, comparator outputs, and alarm latch output.
10. **Run ERC**: fix warnings about floating pins, missing power flags, no connection to ground, etc.
11. **Annotate and generate BOM**: export BOM as CSV for procurement and pick-and-place guidance.

**Schematic tips**
- Place decoupling capacitors physically adjacent to IC power pins (in layout this matters most).
- Use DIP sockets symbols and footprints so you can solder sockets and insert ICs later.
- If using a 555 for the timebase, include a small trimmer (10 kΩ) and specify the tolerance of the timing capacitor.

---

## 6) PCB layout workflow (PCBWeb)

> **Goal:** convert the validated schematic into a neat, manufacturable 2‑layer PCB.

### Pre-layout checks
- Confirm final BOM and footprints for every symbol.
- Choose board size based on display and front‑panel mounting. Keep display area on front edge with correct mounting holes and mechanical clearances.

### Layout steps in PCBWeb
1. **Start PCB layout from the netlist** (Generate Board). Choose 2‑layer, FR‑4, 1.6 mm thickness (default), 1 oz copper.
2. **Set design rules**: clearance 0.2 mm (8 mil) or larger for hand routing; minimum trace width 0.25 mm (10 mil); via size 0.6 mm drill 0.4 mm (for thru‑hole friendly). If unsure use default constraints from the fab.
3. **Place mechanical / display area**: place 7‑segment displays along the top edge (front-facing); ensure you leave room for their terminals and any required bezel.
4. **Place connectors and switches**: DC jack, pushbutton, and buzzer location should be placed for easy access on the board edge.
5. **Place ICs and sockets**: place 4511s close to their respective displays (minimize segment routing), counters near 4511s if possible, and the 555/clock near the power entry to simplify routing.
6. **Group related parts**: keep decoupling capacitors within 3–5 mm of VCC and GND pins of each IC; place the buzzer transistor near the buzzer to shorten high-current traces.
7. **Route power and ground**: create a solid ground pour on one layer (preferably bottom) and route VCC traces on top. Give ground a low-impedance path and wide traces for VCC.
8. **Route critical nets first**: CLK_1HZ (from 555), comparator outputs, and segment nets. Keep clock trace short and away from noisy switching lines; avoid running clock under crystal area or along long digital trace bunches.
9. **Via & thermal relief**: use through‑vias for board-to-plane connections. For prototyping, use standard vias; for finalization choose via sizes per fab capabilities.
10. **Silkscreen labeling**: label the displays (D1..D6), switches, and IC reference designators for easy assembly.
11. **Run DRC**: perform design rule check and fix violations. Verify clearance around display pads & mounting holes.

**Layout tips**
- Place segment resistors close to the driver (74HC4511) or to the displays; this makes brightness tuning easier.
- Keep the 555 timing components physically near each other to reduce stray capacitance and noise.
- Keep analog-ish elements (timer caps) away from digital buses.
- Add small test pads or header pins for signals like `CLK_1HZ`, comparator outputs, and VCC/GND.

---

## 7) Manufacturing / printing the PCB (fab steps)

1. **Prepare fabrication outputs**:
   - Export Gerber files (top copper, bottom copper, top paste, top mask, bottom mask, top silkscreen, board outline) and Excellon drill file.
   - Export BOM and Pick‑and‑Place file (if you plan to use an SMT assembler), though for through‑hole hand assembly pick‑and-place is optional.
2. **Choose fabrication parameters** (typical values to specify):
   - Material: FR‑4, 1.6 mm thickness
   - Copper: 1 oz (35 μm)
   - Surface finish: HASL (lead‑free) or ENIG (more expensive but better for fine pitch)
   - Soldermask: green or your preference
   - Silkscreen: white or black depending on mask color
   - Min trace/space: 6–8 mil (0.15–0.2 mm) recommended for hobbyist fabs
3. **Upload to a PCB fab**: choose a board house (JLCPCB, PCBWay, OSH Park, local fab). Fill order quantity, PCB thickness, finish, and shipping.
4. **Panelization**: for 1 board, most fabs will accept single boards. If you want multiple in a panel for ease of manufacture, panelize or ask the fab to panelize.
5. **Order review**: confirm DRC/DFM warnings from the fab and accept any automated checks.
6. **Lead times**: small boards often ship in 3–10 business days; express shipping may add cost.

**What to expect**: you’ll receive a set of PCBs ready for soldering (unpopulated). Carefully inspect for fabrication defects before assembly.

---

## 8) Assembly (soldering) workflow

**Preparation**
- Clean PCB if needed. Gather all components and the BOM. Lay out components in order of soldering.
- Use IC sockets for DIP parts to avoid heat damage.

**Soldering order (recommended for THT-focused board)**
1. **Low-profile & passive SMD**: If any SMD passives are present, solder them first. (For a THT project you may not have SMDs.)
2. **Through‑hole small components**: resistors, small capacitors — solder on the bottom side. Trim leads.
3. **IC sockets**: solder the DIP sockets next, then decoupling caps, larger capacitors.
4. **Headers, connectors, switches**: solder mechanical components early to lock the board.
5. **Transistor, regulators**: solder transistor and any heatsink-mounted parts.
6. **Displays last**: mount the 7‑segment displays and solder them; they are mechanically bulky and benefit from being last (so you can hold them squarely while soldering).
7. **Insert ICs after testing**: place DIP ICs after initial power test to avoid frying chips if wiring mistakes exist.

**Soldering tips**
- Use a small amount of flux; clean with IPA to remove flux residues afterwards.
- For through‑hole, heat both pad and lead, apply solder to the joint, and remove heat. A good solder fillet is smooth and shiny (for leaded solder).
- If using lead‑free solder, set iron temperature slightly higher; be mindful of overheating components.
- Use sockets for all ICs until you confirm the board works.

---

## 9) Testing, calibration & debugging

**Pre‑power checks**
1. Visual inspection for solder bridges, missing parts, or reversed components.
2. Continuity check: VCC to GND short? If so, investigate before powering.
3. Verify polarity of electrolytic capacitors, diodes, and regulators.

**First power‑up**
1. Use a current‑limited bench supply (set to ~100–200 mA limit initially). If current draw is unexpectedly high, disconnect and debug.
2. Check voltage rails (VCC = 5.0 V ±5%).

**Functional checks**
1. Probe `CLK_1HZ` with an oscilloscope or LED to confirm ~1 Hz square/saw signal from the 555.
2. Force a few clock pulses (manually or with a function generator) and watch the seconds counters roll on the displays.
3. Test each 74HC4511 by driving its BCD inputs manually (or with a bench pattern generator) and verifying segment illumination.
4. Set the alarm BCD switches to a known time; fast‑advance the clock and confirm comparator activates the alarm latch and buzzer.
5. Check the `ALARM_OFF` or reset button clears the latch and buzzer stops.

**Calibration** (if using NE555)
- Use a trimmer resistor in the 555 timing network and adjust while measuring frequency with an oscilloscope until you get nominally 1 Hz. Note: this will still drift with temperature and supply.

**Debugging tips**
- If a digit shows incorrect segments, verify segment resistor & 4511 wiring first.
- If counters don’t advance, probe the carry outputs and clock nets. Are the counter chips receiving the clock? Use manual pulses to test.
- If comparator never trips, measure the comparator inputs when the alarm match should occur; ensure the alarm-set DIP outputs are connected and debounced.

---

## 10) Final notes: accuracy, improvements, and version control

**Accuracy considerations**
- A 555‑based 1 Hz clock will drift; if you need accurate timekeeping, upgrade to a crystal divider (CD4060 + 32.768 kHz crystal) or a DS3231/DS1307 RTC module.

**Expandability & features**
- Add a snooze button wired to temporarily hold the latch off for `N` seconds.
- Implement brightness control using PWM (if displays are multiplexed or with transistor drivers).
- Add battery backup for retain time using RTC or capacitor-backed circuitry.

**Version control**
- Keep all schematic and PCB project files in a folder with revision numbering (e.g., `digital_clock_v1`, `digital_clock_v2`).
- Consider a Git repo for text-based files and exported Gerbers.

---

## 11) Appendix: example BOM CSV

```csv
Ref,Qty,Part,Package,Notes
U1,1,NE555,DIP-8,Astable 1Hz (use trimmer to tune)
U2-U7,6,74HC390,DIP-16,Dual-decade counter
U8-U13,6,74HC4511,DIP-16,BCD to 7-seg driver
U14-U15,2,74HC688,DIP-16,8-bit comparator
U16,1,CD4013,DIP-14,Alarm latch
DSP1-DSP6,6,7seg_cc,0.56in,Common-cathode display
BZ1,1,Passive_Buzzer,,Alternatively use active buzzer for simpler drive
Q1,1,2N2222,TO-92,Buzzer driver
R_seg,42,220R,1/4W,Segment resistors (7 segments × 6 digits)
C_decouple,10+,0.1uF,0805,Per-IC decoupling
C_bulk,1,10uF,Elect,For power smoothing
```

