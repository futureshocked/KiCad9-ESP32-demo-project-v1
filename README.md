# KiCad 9 ESP32 demo project, iteration 1 (archived)

> **For building, studying or modifying the board, use iteration 2:**
> **[KiCad9-ESP32-demo-project-v2-KiCad10](https://github.com/futureshocked/KiCad9-ESP32-demo-project-v2-KiCad10)**
>
> This repository holds iteration 1 in two versions: the original files from the lectures, which contain known errors, and a focused repair of the most important of those errors.

This repository holds the first iteration of the ESP32 sensor board that I design in the KiCad 9 video lectures. I keep it so that you can follow the lectures with the same files you see on screen, and so that the history of the project is preserved.

## Versions in this repository

| Version | Where to find it | KiCad version | Contents |
|---|---|---|---|
| **v1.1-repaired** | The `main` branch, or the [v1.1-repaired release](https://github.com/futureshocked/KiCad9-ESP32-demo-project-v1/releases/tag/v1.1-repaired) | KiCad 10 or later | Iteration 1 with the focused repair shown in the repair video |
| **v1.0-original** | The [v1.0-original release](https://github.com/futureshocked/KiCad9-ESP32-demo-project-v1/releases/tag/v1.0-original) | KiCad 9 or later | Iteration 1 exactly as it was at the end of the original lectures, including all the errors described below |

If you are following the original lectures in KiCad 9, download **v1.0-original** from its release page. The repaired files were saved in KiCad 10 and cannot be opened in KiCad 9.

## About the project

The board is a battery-powered environmental sensor built around the ESP32-C3. It is a four-layer PCB, about 80 × 54 mm, and includes:

1. An ESP32-C3-WROOM-02 module (U3) with Wi-Fi and Bluetooth LE.
2. A USB-C connector with a CP2102N USB-to-UART bridge (U4), USBLC6-2SC6 ESD protection (U5) and boot/reset circuitry (Q2, Q3).
3. An MCP73871 Li-ion charger with power-path management (U1) and an LM1117 3.3 V regulator (VR1).
4. A BME280 temperature, humidity and pressure sensor (U2).
5. A TEMT6000 ambient light sensor (Q1).
6. An electret microphone with a MAX4466 amplifier (U7, MK1).
7. A microSD card slot (J3) and a W25Q32 SPI flash chip (U6).
8. A header for an I2C OLED display, status LEDs, and Boot and Reset buttons.

## Two repositories, and why

There are two repositories for this project:

| Repository | Contents | Use it to |
|---|---|---|
| **KiCad9-ESP32-demo-project-v1** (this one) | Iteration 1: the original KiCad 9 files and a focused repair | Follow the original lectures and the repair video, and see how the project developed |
| **[KiCad9-ESP32-demo-project-v2-KiCad10](https://github.com/futureshocked/KiCad9-ESP32-demo-project-v2-KiCad10)** | Iteration 2, corrected and updated in KiCad 10 | Study, modify or manufacture the board |

I have kept both so that the history of the project stays intact, while anyone who wants to experiment with the design, or have it made, has a clean and correct version to start from.

## The focused repair (v1.1-repaired)

After a student reported errors in the iteration 1 PCB, I recorded a video that repairs iteration 1. As I explain in the video, this is a **focused repair** of U2, U4 and the microphone output (MIC_OUT). The remaining repairs, and several improvements, are made in iteration 2, and I do not plan any further work on iteration 1.

The repair made these changes:

1. **U2, U4, U5 and C15 are connected correctly** (issue 1 below). The PCB was updated from the schematic, the +3.3V tracks to the wrong pads were removed, and the pads were rerouted to GND, USB D- and VBUS sense. D- is routed at 0.2 mm next to D+.
2. **MIC_OUT moved from GPIO5 to GPIO0** (issue 3 below). The schematic and PCB were changed so that MIC_OUT goes to U3 pin 18 (GPIO0, on ADC1) and CS_SD goes to U3 pin 4 (GPIO5).

A DRC of the repaired board, with "Test for parity between PCB and schematic" enabled, reports no net conflicts and no unconnected pads.

The repair deliberately leaves the following as they are. If you need any of them, use iteration 2:

1. **The automatic reset and download mode circuit** (issue 2 below) is unchanged and still does not work. See [Uploading firmware](#uploading-firmware).
2. **The improvements made in iteration 2** are not included.
3. **The mounting hole and the two logos were removed.** Running Update PCB from Schematic with "Delete footprints with no symbols" ticked deleted the 2.5 mm mounting hole at the left of the board, the TE Connectivity logo and the KiCad logo, because none of them has a schematic symbol. They were not put back. Iteration 2 has all three.
4. **The fabrication files are from the original board.** The Gerbers (`ESP32PRoject_Gerber/` and `ESP32PRoject_Gerber.zip`) and the placement files (`*.pos`) were generated before the repair and still contain the errors. Do not send them to a fabricator.
5. **The `net_conflict` check is still set to Warning** in Board Setup. See issue 1 for why this matters.

## Issues found in the original iteration 1

### 1. The PCB is out of sync with the schematic (U2, U4, U5, C15)

**Status: fixed in v1.1-repaired and in iteration 2.**

The schematic is correct, but the PCB was not updated from it. Nine pads that should be on GND, USB D- or the VBUS sense net are on +3.3V instead, and +3.3V tracks are routed to them:

| Part and pad | Pin function | Net in the original PCB | Correct net |
|---|---|---|---|
| U4 (CP2102N) pad 3 | GND | +3.3V | GND |
| U4 pad 12 | GND | +3.3V | GND |
| U4 pad 21 (exposed pad) | GND | +3.3V | GND |
| U4 pad 5 | USB D- | +3.3V | /ESP32-C3-02/USB_D- |
| U4 pad 8 | VBUS sense | +3.3V | Net-(U4-VBUS) |
| U5 (USBLC6) pad 1 | I/O1, D- side | +3.3V | /ESP32-C3-02/USB_D- |
| U2 (BME280) pad 5 | SDO (I2C address select) | +3.3V | GND |
| U2 pad 7 | GND | +3.3V | GND |
| C15 pad 2 | GND | +3.3V | GND |

A board made from the original files would have the following faults:

1. All three of the CP2102N's ground pins are on +3.3V and its D- line is tied to 3.3 V, so the board would not appear as a USB serial port.
2. The BME280's pin 1 is correctly grounded, but its other ground pin, pin 7, is on +3.3V. If the two ground pins are joined inside the package, as ground pins normally are, this shorts the 3.3 V supply to ground through the sensor, which would likely damage it.
3. The BME280's SDO pin (pin 5) sets its I2C address. The schematic ties it to GND for address 0x76, but on the PCB it is on +3.3V, which selects address 0x77.
4. C15, the BME280 decoupling capacitor, has both pads on +3.3V, so it does nothing.

**Why DRC did not catch it.** In this project's Board Setup, the `net_conflict` check ("Pad net doesn't match schematic") is set to *Warning*. A DRC run with "Test for parity between PCB and schematic" enabled does list all nine pads, but only under warnings. If you look only at errors, the board appears clean.

### 2. Automatic reset and download mode do not work

**Status: not fixed in v1.1-repaired. Partly fixed in iteration 2, but still not working.**

The auto-program circuit (Q2, Q3, R33, R34) is meant to let the CP2102N's DTR and RTS signals reset the ESP32-C3 and put it into download mode, so that you can upload firmware without pressing any buttons. It has two faults:

1. **Q3 is wired backwards.** The schematic and PCB connect Q3's emitter to BOOT and its collector to DTR, which is the reverse of Q2 and of Espressif's reference circuit. Iteration 2 corrects this.
2. **The CP2102N has no DTR output.** This design uses the 20-pin QFN package of the CP2102N, which has no DTR pin. The net named DTR is connected to U4 pin 15, which is CTS, an input (see table 5.3 of the CP2102N datasheet in `Datasheets/`). Nothing drives the DTR net. This is the same in iteration 2, and I have not yet decided whether to change it.

Neither automatic reset nor automatic download mode can be relied on. DRC cannot detect either fault, because the schematic and PCB agree with each other. See [Uploading firmware](#uploading-firmware) for two ways to program the board anyway.

### 3. The microphone output is on an ADC2 pin

**Status: fixed in v1.1-repaired and in iteration 2.**

In the original files, the MAX4466 output (MIC_OUT) is connected to GPIO5, the only ADC2 pin on the ESP32-C3. Espressif's ESP32-C3 errata says ADC2 can become inoperative and recommends using ADC1 instead, and recent versions of ESP-IDF disable ADC2 on this chip by default. ADC2 is also shared with Wi-Fi. The fix moves MIC_OUT to GPIO0 (ADC1) and CS_SD to GPIO5.

## Improvements in iteration 2

Besides the fixes above, iteration 2 includes these changes, which are not part of the iteration 1 repair:

1. A PTC fuse (F2) on the +5 V output of the charger.
2. A trimmer (R17) to adjust the microphone gain, with new values for R30 (1 kΩ), R31 (22 kΩ) and C19 (10 µF).
3. The light sensor was rearranged so that its output rises with light: Q1's collector now goes to +3.3V and R23 pulls the output down to GND. In iteration 1 the output falls as light increases.
4. Three extra capacitors: C23 (100 nF decoupling) and C24 and C25 (10 nF filtering on the light sensor signal).
5. The Boot and Reset buttons (S1, S2) were rotated 180°, and the right edge of the board moved about 0.8 mm.

## Repairing the original files yourself

If you want to repeat the repair as an exercise, download **v1.0-original** and follow the repair video. In outline:

1. Open the PCB and run **Tools → Update PCB from Schematic** (F8). This moves the nine pads to their correct nets. **Untick "Delete footprints with no symbols"** in the update dialog, or the mounting hole and the two logos will be deleted.
2. Delete the +3.3V tracks and vias that still end on those pads. After the update they will appear as shorts.
3. Route GND to U4 pads 3, 12 and 21, U2 pads 5 and 7, and C15 pad 2. Route D- from U4 pad 5 to U5 pad 1 as a differential pair with D+, and connect U4 pad 8 to the VBUS sense divider (R12, R13, C11).
4. To fix the microphone output, swap the MIC_OUT and CS_SD connections on U3 in the schematic, update the PCB again, and reroute both nets.
5. Run DRC with **Test for parity between PCB and schematic** enabled, and check the warnings as well as the errors. You can also set `net_conflict` to *Error* in Board Setup → Design Rules → Violation Severity.

## Uploading firmware

Because automatic reset and download mode do not work, use one of these two methods to upload firmware. Both work on every version of iteration 1 and on iteration 2:

1. **Through the USB-C connector and the CP2102N, with the buttons.** Hold **Boot** (S1), press and release **Reset** (S2), then release **Boot**. The ESP32-C3 starts in download mode and you can upload. Press **Reset** afterwards to run the new firmware. Serial monitoring through the CP2102N works normally.
2. **Through the ESP32-C3's built-in USB Serial/JTAG controller.** The ESP32-C3 has its own USB interface on GPIO18 (D-) and GPIO19 (D+). On this board these pins are brought out to header **J5**: pin 3 is D-, pin 4 is D+ and pin 5 is GND. Wire a USB cable's D-, D+ and GND to those pins and power the board from the USB-C connector or the battery. **Do not connect the cable's 5 V wire to J5**: J5 pin 1 is the 3.3 V rail. Through this interface, esptool and the Arduino and ESP-IDF tools can reset the chip and enter download mode without the buttons, and you get a serial console and JTAG debugging as well. For this to keep working, your firmware must not use GPIO18 and GPIO19 as ordinary I/O.

## License

Copyright © 2025–2026, Peter Dalmaris. Licensed under the CERN-OHL-S v2.0. See [LICENSE](LICENSE).
