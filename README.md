# Under Frequency Load Shedding-PSCAD-Project
UFLS Simulation using PSCAD
# Multi-Stage Under-Frequency Load Shedding (UFLS) & AVR System Simulation

A PSCAD-based dynamic power system simulation modeling a 3-stage Automatic Under-Frequency Load Shedding (AUFLS) protection scheme integrated with closed-loop Automatic Voltage Regulation (AVR).

The project evaluates transient frequency stability, rotor dynamics, and selective load tripping during major power deficit events on an $11\text{ kV}$ grid.

---

## 1. System Parameters & Configuration

| Parameter | Value | Description |
| --- | --- | --- |
| **Grid Base Voltage** | $11.0\text{ kV}$ | Line-to-Line nominal bus voltage |
| **Nominal Frequency ($f_0$)** | $50.0\text{ Hz}$ | Target operating frequency |
| **Base Generation** | Synchronous Machine | Salient-pole model with dynamic speed/torque tracking |
| **Connected Base Loads** | $3 \times 29.4\text{ MW}$ | $10\text{ MW/phase}$ constant impedance load branches |
| **Disturbance Load (`BRK_DIST`)** | $58.8\text{ MW}$ | Switched load step injected at $t = 2.0\text{ s}$ |
| **Voltage Regulator** | IEEE AC1C Model | Closed-loop field excitation maintaining $1.0\text{ pu}$ terminal voltage |

---

## 2. Protection Scheme & Control Logic

Bus frequency ($f_{\text{bus}}$) is monitored continuously using an $f, V_{\text{rms}}, \theta$ meter. Signal processing utilizes edge-triggered comparators feeding SR Flip-Flops to latch breaker trip signals upon crossing defined frequency setpoints:

```
                  ┌────────────────────────┐     ┌──────────────┐     ┌───────────┐
  f_bus Signal ──►│ Comparator (A < B)    │────►│ SR Flip-Flop │────►│ Breaker   │
                  └────────────────────────┘     └──────────────┘     └───────────┘
                              ▲
  Setpoints (Hz) ─────────────┘  (Stage 1: 49.6 Hz | Stage 2: 49.2 Hz | Stage 3: 49.0 Hz)

```

### Threshold Coordination

* **Stage 1 (`BRK_1`):** **`49.6 Hz`** — Primary load shedding step ($29.4\text{ MW}$ load relief).
* **Stage 2 (`BRK_2`):** **`49.2 Hz`** — Secondary defense step for continuing frequency decay.
* **Stage 3 (`BRK_3`):** **`49.0 Hz`** — Emergency backup step to prevent islanding or generator loss-of-mains.

---

## 3. Dynamic Simulation Results

```
  Frequency (Hz)
  50.05 ──┐   ┌─ Start-up Transient (AVR settling)
  50.00 ──┴───┘ \
  49.90          \____ Disturbance Event (t = 2.0s)
  49.80                \
  49.70                 \__ Transient Nadir @ 49.60 Hz
  49.60 ───────────────────X──► BRK_1 Trips (49.6 Hz threshold breached)
  49.50                     \____ Frequency Recovers & Settles @ 49.90 Hz
         0.0s    1.0s    2.0s    3.0s    4.0s    5.0s    6.0s    7.0s    8.0s

```

* **$t = 0.0\text{ s} - 1.5\text{ s}$ (Initialization Phase):** AVR dynamics adjust field voltage ($E_f$) to settle $V_{\text{bus}}$ to $1.0\text{ pu}$, causing minor startup frequency oscillation before holding steady at $50.0\text{ Hz}$.
* **$t = 2.0\text{ s}$ (Disturbance Event):** `BRK_DIST` switches on, injecting a $58.8\text{ MW}$ active power deficit ($\Delta P < 0$). Rotor inertia decelerates, initiating a rapid frequency drop.
* **$t \approx 2.1\text{ s}$ (Stage 1 Action):** Frequency reaches the dynamic nadir of **$49.60\text{ Hz}$**, crossing the $49.6\text{ Hz}$ threshold. Comparator output switches high (`1`), latching the SR flip-flop and opening `BRK_1`.
* **$t > 4.0\text{ s}$ (Post-Shedding Equilibrium):** Dropping $29.4\text{ MW}$ restores power balance. Frequency rebounds rapidly and stabilizes at **$49.90\text{ Hz}$**.
* **Selectivity Verification:** Frequency remains well above $49.2\text{ Hz}$ and $49.0\text{ Hz}$ post-trip. `BRK_2` and `BRK_3` remain low (`0`), verifying correct staged coordination without over-tripping.

---

## 4. How to Run in PSCAD

1. Open **PSCAD** and load the project workspace.
2. Link the **`master_library`** dependency for the **IEEE AC1C Exciter** block.
3. Configure simulation duration to **`10.0 s`** with a time step of **`50 µs`**.
4. Run the simulation and observe `f_bus`, `BRK_1`, `BRK_2`, and `BRK_3` channel plots.
