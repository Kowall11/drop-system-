# 🌊 Aqua SOS - Wireless Buoy Drop System via LoRa

> **Bezprzewodowy system zdalnego zrzutu boi ratowniczej sterowany przez LoRa**
> Wireless remote-controlled rescue buoy release system using LoRa radio communication.

---

## 📌 Project Overview

**Aqua SOS** is an embedded system for remotely deploying a rescue buoy in hard-to-reach or aquatic environments. When an operator presses a button, a LoRa radio signal is transmitted to a waterproof drop box, which activates two servo motors to open a hatch and release the buoy.

```
[Operator]
    │
    ▼
[Arduino Nano + LoRa TX] ──────── 📡 Radio ────────► [Arduino Uno + LoRa RX]
  Press button SW1                                        Activates M1 + M2
  LED D1 blinks                                          Opens hatch → buoy released
    ▲                                                     LED D2 lights up
    │                         ◄──── Confirmation ─────────────────┘
    └──── LED D1 off (done)        "SILNIKI_OK"
```

---

## 🏗️ Hardware Architecture

### Drop Box — Physical System

The system consists of two main physical parts:

| Part | Description |
|---|---|
| **Box (Boks)** | Watertight rectangular enclosure housing the folded rescue buoy. Reinforced corner mounting brackets. Released via a servo-controlled hatch on the bottom/side. |
| **External Platform** | Flat mounting plate attached to the underside of the box. Contains 6 dedicated servo motor mounting slots (3 groups of 2) for symmetric force distribution when opening the hatch. |

**Drop Sequence:**
1. LoRa signal received → Arduino Uno activates M1 + M2
2. Servos rotate to 90° → hatch opens
3. Buoy deploys to water surface
4. Arduino Uno sends `SILNIKI_OK` confirmation back to transmitter

---

## 🔌 Circuit Components

| Ref | Component | Model / Value | Role |
|---|---|---|---|
| **A2** | Arduino Nano v3.x | ATmega328P | Transmitter — button & LED |
| **A1** | Arduino Uno R3 | ATmega328P | Receiver — servo control |
| **U1** | LoRa Module | Reyax RYL993 | Radio TX (Nano side) |
| **U2** | LoRa Module | Reyax RYL993 | Radio RX (Uno side) |
| **U4** | OLED Display | ER_OLEDM0.91 (I2C, 128×32) | Status display on Nano |
| **M1** | Servo Motor | 5V PWM | Hatch motor 1 |
| **M2** | Servo Motor | 5V PWM | Hatch motor 2 |
| **SW1** | Tactile Switch | SW_MEC_5G | Trigger button (Nano) |
| **SW2** | Tactile Switch | SW_MEC_5G | Aux button (Uno) |
| **D1** | LED | Red/Green | Status indicator (Nano) |
| **D2** | LED | Red/Green | Status indicator (Uno) |
| **R1, R6** | Resistor | 220 Ω | LED current limiter |
| **R2, R5** | Resistor | 4.7 kΩ | Voltage divider (LoRa TX line) |
| **R3, R4** | Resistor | 10 kΩ | Voltage divider (LoRa TX line) |

---

## 🔧 Wiring

### Transmitter — Arduino Nano (A2)

| Nano Pin | Connected To | Via | Notes |
|---|---|---|---|
| D2 | SW1 | — | `INPUT_PULLUP` |
| D3 | D1 LED (anode) | R1 220Ω | Status signal |
| D10 (RX) | U1 TX | Direct | SoftwareSerial RX |
| D11 (TX) | U1 RX | R2/R3 divider | 5V → 3.3V level shift |
| A4 (SDA) | U4 SDA (OLED) | — | I2C data |
| A5 (SCL) | U4 SCL (OLED) | — | I2C clock |
| 5V | U1 VCC, U4 VCC | — | Module power |
| GND | U1 GND, U4 GND, D1 cathode, SW1 | — | Common ground |

### Receiver — Arduino Uno (A1)

| Uno Pin | Connected To | Via | Notes |
|---|---|---|---|
| D9 (PWM) | M1 signal | — | Servo motor 1 |
| D6 (PWM) | M2 signal | — | Servo motor 2 |
| D3 | D2 LED (anode) | R6 220Ω | Status signal |
| D2 | SW2 | — | `INPUT_PULLUP` |
| D10 (RX) | U2 TX | Direct | SoftwareSerial RX |
| D11 (TX) | U2 RX | R4/R5 divider | 5V → 3.3V level shift |
| 5V (**external**) | M1 VCC, M2 VCC | — | ⚠️ External supply only! |
| GND | U2 GND, D2 cathode, M1 GND, M2 GND | — | Common ground |

---

## ⚙️ LoRa Configuration

Both **Reyax RYL993** modules must be configured identically via UART before use:

```
Frequency  : 868 MHz (Europe) / 915 MHz (US)
Network ID : same on both modules
Device Addr: same address or broadcast mode
Baud Rate  : 9600
```

> The voltage divider on the TX line (R2/R3 and R4/R5) steps down 5V Arduino logic to ~3.4V, safe for the 3.3V LoRa module inputs.
>
> Formula: `Vout = 5V × (10kΩ / (4.7kΩ + 10kΩ)) ≈ 3.4V` ✅

---

## ⚠️ Important Notes

### Servo Power Supply
> **Do NOT power servos from Arduino's 5V pin.**

Each servo can draw up to **1A under load**. Use a dedicated external **5V / 2A** (or larger) supply:
- External 5V → M1 VCC, M2 VCC
- External GND **must** be tied to Arduino GND (common ground)
- Arduino only provides the PWM control signal

### Button Debouncing
SW1 and SW2 require software debouncing. Recommended delay: **50ms** after state change detection.

---

## 📁 Repository Structure

```
aqua-sos/
├── hardware/
│   ├── drop_system-_circuit.kicad_sch   # KiCad schematic (EDA 10.0)
│   └── drop_system_box.step             # 3D model of the drop box enclosure
├── docs/
│   └── drop_system_documentation.pdf   # Full project documentation (PL)
└── README.md
```

---

## 🛠️ Tools Used

| Tool | Purpose |
|---|---|
| [KiCad EDA 10.0](https://www.kicad.org/) | Schematic design |
| Autodesk Fusion / FreeCAD | 3D enclosure design (`.step`) |
| Arduino IDE | Firmware for Nano & Uno |

---

## 📄 License

This project is open hardware / open source. See individual files for details.

---

*Aqua SOS — designed for remote rescue operations in aquatic environments.*

