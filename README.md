# KiCad 9 ESP32 demo project, iteration 1 (archived)

> **This version has known PCB errors. Do not send these files to a fabricator.**
> The corrected design is in the iteration 2 repository:
> **[KiCad9-ESP32-demo-project-v2-KiCad10](https://github.com/futureshocked/KiCad9-ESP32-demo-project-v2-KiCad10)**

This repository holds the first iteration of the ESP32 sensor board that I design in the KiCad 9 video lectures. The files are exactly as they were at the end of iteration 1, including the errors described below. I keep them here so that you can follow the lectures with the same files you see on screen, and so that the history of the project is preserved.

## About the project

The board is a battery-powered environmental sensor built around the ESP32-C3. It is a four-layer PCB, about 80 × 54 mm, and includes:

1. An ESP32-C3-WROOM-02 module (U3) with Wi-Fi and Bluetooth LE.
2. A USB-C connector with a CP2102N USB-to-UART bridge (U4), USBLC6-2SC6 ESD protection (U5) and automatic boot/reset circuitry (Q2, Q3).
3. An MCP73871 Li-ion charger with power-path management (U1) and an LM1117 3.3 V regulator (VR1).
4. A BME280 temperature, humidity and pressure sensor (U2).
5. A TEMT6000 ambient light sensor (Q1).
6. An electret microphone with a MAX4466 amplifier (U7, MK1).
7. A microSD card slot (J3) and a W25Q32 SPI flash chip (U6).
8. A header for an I2C OLED display, status LEDs, and Boot and Reset buttons.

The project was made with **KiCad 9.0**.

## Two repositories, and why

There are two repositories for this project:

| Repository | Contents | Use it to |
|---|---|---|
| **KiCad9-ESP32-demo-project-v1** (this one) | Iteration 1, made in KiCad 9, with the errors listed below | Follow the original lectures and see how the project developed |
| **[KiCad9-ESP32-demo-project-v2-KiCad10](https://github.com/futureshocked/KiCad9-ESP32-demo-project-v2-KiCad10)** | Iteration 2, corrected and updated in KiCad 10 | Study, modify or manufacture the board |

I have kept both so that the history of the project stays intact, while anyone who wants to experiment with the design, or have it made, has a clean and correct version to start from.

## Known issues in this version

### 1. The PCB is out of sync with the schematic (U2, U4, U5, C15)

The schematic is correct, but the PCB was not updated from it. Nine pads that should be on GND, USB D- or the VBUS sense net are on +3.3V instead, and +3.3V tracks are routed to them:

| Part and pad | Pin function | Net in this PCB | Correct net |
|---|---|---|---|
| U4 (CP2102N) pad 3 | GND | +3.3V | GND |
| U4 pad 12 | GND | +3.3V | GND |
| U4 pad 21 (exposed pad) | GND | +3.3V | GND |
| U4 pad 5 | USB D- | +3.3V | /ESP32-C3-02/USB_D- |
| U4 pad 8 | VBUS sense | +3.3V | Net-(U4-VBUS) |
| U5 (USBLC6) pad 1 | I/O1, D- side | +3.3V | /ESP32-C3-02/USB_D- |
| U2 (BME280) pad 5 | GND | +3.3V | GND |
| U2 pad 7 | GND | +3.3V | GND |
| C15 pad 2 | GND | +3.3V | GND |

A board made from these files would have the following faults:

1. The CP2102N has no ground connection and its D- line is tied to 3.3 V, so the board would not appear as a USB serial port.
2. The BME280 has no ground connection, so it would not work.
3. C15, the BME280 decoupling capacitor, has both pads on +3.3V, so it does nothing.

**Why DRC did not catch it.** In this project's Board Setup, the `net_conflict` check ("Pad net doesn't match schematic") is set to *Warning*. A DRC run with "Test for parity between PCB and schematic" enabled does list all nine pads, but only under warnings. If you look only at errors, the board appears clean.

### 2. Q3 in the auto-program circuit is wired backwards

Q3 (BC817) is one of the two transistors that let the USB-UART bridge put the ESP32-C3 into download mode. In this version, the schematic and PCB connect Q3's emitter to BOOT and its collector to DTR, which is the reverse of Q2 and of Espressif's reference circuit. Automatic entry into download mode is therefore unreliable. You can still enter it manually with the Boot and Reset buttons.

### 3. The microphone output is on an ADC2 pin

The MAX4466 output (MIC_OUT) is connected to GPIO5, the only ADC2 pin on the ESP32-C3. Espressif's ESP32-C3 errata says ADC2 can become inoperative and recommends using ADC1 instead, and recent versions of ESP-IDF disable ADC2 on this chip by default. ADC2 is also shared with Wi-Fi.

### Fabrication outputs in this repository

The Gerbers (`ESP32PRoject_Gerber/` and `ESP32PRoject_Gerber.zip`) and the placement files (`*.pos`) were generated from this faulty PCB. They are kept for reference only.

## How iteration 2 fixes these issues

Iteration 2 fixes all three issues:

| Issue | Fix in iteration 2 |
|---|---|
| 1. PCB out of sync | The PCB was updated from the schematic. All nine pads are now on their correct nets, the +3.3V tracks to them were removed, and the GND pads, D- and VBUS sense were rerouted. The via under U4's exposed pad now connects to GND. A DRC with schematic parity reports no conflicts on these parts. |
| 2. Q3 backwards | Q3's collector now connects to BOOT and its emitter to DTR, matching Q2 and the Espressif reference circuit. |
| 3. Microphone on ADC2 | MIC_OUT moved from GPIO5 to GPIO0 (ADC1), and CS_SD moved to GPIO5. |

Iteration 2 also includes some improvements that were not fixes for errors:

1. A PTC fuse (F2) on the +5 V output of the charger.
2. A trimmer (R17) to adjust the microphone gain, with new values for R30 (1 kΩ), R31 (22 kΩ) and C19 (10 µF).
3. The light sensor was rearranged so that its output rises with light: Q1's collector now goes to +3.3V and R23 pulls the output down to GND. In iteration 1 the output falls as light increases.
4. Three extra capacitors: C23 (100 nF decoupling) and C24 and C25 (10 nF filtering on the light sensor signal).
5. The Boot and Reset buttons (S1, S2) were rotated 180°, and the right edge of the board moved about 0.8 mm.

All other components are in the same positions as in iteration 1.

## Fixing the PCB errors in this version yourself

If you want to repair iteration 1 as an exercise, I have recorded a short video that shows how to fix the U2 and U4 errors. In outline:

1. Open the PCB and run **Tools → Update PCB from Schematic** (F8). This moves the nine pads to their correct nets.
2. Delete the +3.3V tracks and vias that still end on those pads. After the update they will appear as shorts.
3. Route GND to U4 pads 3, 12 and 21, U2 pads 5 and 7, and C15 pad 2. Route D- from U4 pad 5 to U5 pad 1 as a differential pair with D+, and connect U4 pad 8 to the VBUS sense divider (R12, R13, C11).
4. Run DRC with **Test for parity between PCB and schematic** enabled, and check the warnings as well as the errors. You can also set `net_conflict` to *Error* in Board Setup → Design Rules → Violation Severity.

Issues 2 and 3 are design changes. To fix them, edit the schematic as described in the table above, then repeat steps 1 to 4.

## License

Copyright © 2025, Peter Dalmaris. Licensed under the CERN-OHL-S v2.0. See [LICENSE](LICENSE).
