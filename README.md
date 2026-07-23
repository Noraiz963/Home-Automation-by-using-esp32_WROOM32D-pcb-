# ESP32 Home Automation — 2-Channel Mains Relay Controller

A mains-powered, WiFi/Bluetooth-enabled 2-channel relay controller designed in KiCad, built around the **ESP32-WROOM-32**. Designed for switching mains-connected loads (lights, appliances, etc.) with manual switch override, status LEDs, and a dedicated UART header for programming and debugging.

## Why This Exists

Home automation projects need a safe, reliable way to let a microcontroller switch mains-voltage loads without exposing the low-voltage logic circuitry to mains hazards or noise. This board handles that end-to-end: an isolated AC-DC module supplies clean low-voltage power, optocouplers electrically isolate the ESP32 from the relay/mains side, and flyback-protected relays do the actual switching — all while exposing manual switch inputs and a UART header for flashing and debugging without a dedicated USB-serial chip onboard.

## Circuit Overview

- **Power supply**: Mains AC input (J2) → HLK-PM01 isolated AC-DC module (5V) → AMS1117-3.3 linear regulator (3.3V) for ESP32 and logic
- **Core MCU**: ESP32-WROOM-32 (U1) — WiFi/Bluetooth microcontroller driving all logic, relay control, and connectivity
- **Relay channels (x2)**: GPIO → NPN transistor (Q1/Q2) → PC817 optocoupler (U2/U3) → relay coil (K1/K2), with 1N4148 flyback diodes (D2/D4) for coil protection
- **Status LEDs**: LED1/LED2 driven from ESP32 GPIOs through current-limiting resistors
- **UART header (J1)**: 3.3V/RXD/TXD/GND — for external USB-to-UART programmer/debugger
- **Switch input (J3)**: External manual switch inputs (IN1/IN2) for local override control
- **Reset & Boot (SW1/SW2)**: Manual boot-mode and reset buttons with pull-up resistors, standard for ESP32 flashing workflow

## Signal Flow (Concept)

**Power path:**
1. Mains AC enters via J2 (L/N).
2. HLK-PM01 isolates and converts mains AC directly to 5V DC — a safety-isolated, self-contained module (no custom transformer/rectifier design needed).
3. AMS1117-3.3 regulates the 5V down to a clean 3.3V rail, filtered by C1/C2, to power the ESP32 and logic-side components.
4. L1/L2 provide EMI filtering on the incoming mains lines.

**Relay switching path (per channel):**
1. ESP32 GPIO (IN1 or IN2) drives a current-limiting resistor into the base of an NPN transistor (Q1/Q2).
2. The transistor switches current through a PC817 optocoupler's internal LED.
3. The optocoupler's output drives the relay coil circuit — this is the critical **isolation boundary** between the low-voltage ESP32 logic and the mains-connected relay/load side.
4. The relay coil energizes, mechanically switching its contacts to connect mains L1/L2 to the output screw terminal (the load).
5. A flyback diode (1N4148) across the coil absorbs the voltage spike generated when the coil de-energizes, protecting the driving transistor from damage.

**Programming/debug path:**
- J1 exposes 3.3V, RXD, TXD, GND for an external USB-to-UART adapter.
- SW1 ("boot") pulls ESP32's IO0 low to enter flashing/bootloader mode.
- SW2 ("ENB") pulls EN low to reset the chip.
- R7/R8 pull-ups hold both lines high during normal operation.

**Manual override path:**
- J3 provides external switch inputs (IN1/IN2), allowing physical switches to control relay state independent of WiFi/app-based control — useful as a local override or fallback if connectivity is lost.

**Status indication:**
- LED1/LED2, driven from dedicated ESP32 GPIOs, can be used in firmware to indicate relay state, WiFi connection status, or general system status.

**Ground referencing:**
- Care should be taken that the low-voltage logic-side GND and mains-side relay contacts remain properly isolated per the PC817 optocouplers — the optical isolation is what keeps the ESP32 and any connected programmer safe from mains-side faults.

## Schematic
![Schematic]([images/schematic.png](https://github.com/Noraiz963/Home-Automation-by-using-esp32_WROOM32D-pcb-/blob/e5d580b0eca3b133f65bf457ccc5ca4573bc8832/Screenshot%202026-07-23%20222736.png))

## PCB Layout
![PCB Layout](https://github.com/Noraiz963/Home-Automation-by-using-esp32_WROOM32D-pcb-/blob/e5d580b0eca3b133f65bf457ccc5ca4573bc8832/Screenshot%202026-07-23%20222756.png)

## 3D Render
![3D Render 1](https://github.com/Noraiz963/Home-Automation-by-using-esp32_WROOM32D-pcb-/blob/e5d580b0eca3b133f65bf457ccc5ca4573bc8832/Screenshot%202026-07-23%20222907.png)
![3D Render 2](https://github.com/Noraiz963/Home-Automation-by-using-esp32_WROOM32D-pcb-/blob/e5d580b0eca3b133f65bf457ccc5ca4573bc8832/Screenshot%202026-07-23%20222926.png)

## Bill of Materials

| Ref | Component | Value / Part |
|-----|-----------|--------------|
| U1 | Microcontroller Module | ESP32-WROOM-32 |
| U4 | Linear Regulator | AMS1117-3.3 |
| U6 | Isolated AC-DC Module | HLK-PM01 |
| U2, U3 | Optocoupler | PC817 |
| K2 | Relay | PR13-5V-450-1C |
| K1 | Relay | RAYEX-L90AS |
| Q1, Q2 | NPN Transistor | S8050 |
| D2, D4 | Diode (flyback) | 1N4148 |
| D1 | Status LED | — |
| LED1, LED2 | Status LEDs | — |
| R1, R2 | Resistor (transistor base) | 1kΩ |
| R3, R4 | Resistor (optocoupler output) | 1kΩ |
| R5, R6 | Resistor (LED current limit) | 1kΩ |
| R7, R8 | Resistor (boot/reset pull-up) | 10kΩ |
| C1 | Capacitor (regulator input) | 10µF |
| C2 | Capacitor (regulator output) | 22µF |
| L1, L2 | Inductor/Ferrite (EMI filter) | — |
| J1 | UART Header (4-pin) | 3.3V / RXD / TXD / GND |
| J2 | Mains Input (Screw Terminal, 4-pin) | AC L/N |
| J3 | Switch Input (Screw Terminal, 3-pin) | IN1 / IN2 / GND |
| SW1 | Push Button (Boot) | — |
| SW2 | Push Button (Reset/EN) | — |

## Pin Reference

### J1 — UART (Programming/Debug)
| Pin | Signal |
|---|---|
| 1 | 3.3V |
| 2 | RXD |
| 3 | TXD |
| 4 | GND |

### J3 — Switch Input
| Pin | Signal |
|---|---|
| 1 | IN1 |
| 2 | IN2 |
| 3 | GND |

## Features

- Two independently controlled relay channels for switching mains-connected loads
- Optically isolated relay drive circuitry for safety between logic and mains sides
- Onboard isolated AC-DC power supply — no external power adapter required
- Manual switch input for local override control
- Dedicated UART header for firmware flashing/debugging (no onboard USB, keep an external USB-UART adapter handy)
- Manual boot/reset buttons for standard ESP32 flashing workflow
- Status LEDs for visual system feedback
- Mounting holes for enclosure integration

## Getting Started

1. Wire mains AC to J2 (L/N) — **use extreme caution; this board has mains-voltage sections. Ensure proper enclosure and isolation before powering on.**
2. Connect an external USB-to-UART adapter to J1 (3.3V, RXD, TXD, GND).
3. Hold **SW1 (boot)**, tap **SW2 (reset)**, then release SW1 to enter flashing mode.
4. Flash your firmware (e.g., via Arduino IDE, ESP-IDF, or ESPHome).
5. Connect loads to the relay output screw terminals.
6. Optionally wire physical switches to J3 for manual override control.

## Tools Used
- KiCad 10.0.4 (schematic capture, PCB layout, 3D visualization)

## Status
Completed schematic and PCB layout design; 3D render verified.

## Safety Notice

⚠️ This board includes mains-voltage circuitry. Improper assembly, wiring, or enclosure design can result in electric shock or fire hazard. Ensure proper isolation clearances, use an appropriately rated enclosure, and follow local electrical safety codes. Do not operate this board without proper enclosure while connected to mains power.

## Notes
Built as a home automation project combining ESP32 WiFi connectivity with safe, isolated mains relay switching — designed for controlling lights or appliances with both remote (WiFi/app) and local (manual switch) control options.
