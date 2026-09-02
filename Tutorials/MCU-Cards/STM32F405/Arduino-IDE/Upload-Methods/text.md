# Orbit-Q: STM32F405 Upload Methods Guide

Learn how to configure your development environment, select the correct Arduino IDE settings, and program your **Orbit-Q STM32F405** MCU card using **CP2102 UART**, **USB-OTG DFU**, or **ST-Link V2 (SWD)**.

---

## Hardware Overview & Prerequisites

Before flashing firmware using any method, you must install **STM32CubeProgrammer**. Arduino IDE relies on its underlying binaries to communicate with the internal bootloader of the STM32F405.

1. Download and install **[STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html)** from STMicroelectronics.

2. Open **Arduino IDE** and add the STM32 package URL under **File > Preferences > Additional Boards Manager URLs**:

```text
[https://github.com/stm32duino/BoardManagerFiles/raw/main/package_stmicroelectronics_index.json](https://github.com/stm32duino/BoardManagerFiles/raw/main/package_stmicroelectronics_index.json)
```
## Go to Tools > Board > Boards Manager, search for STM32 Cores, and click Install.

## Upload Methods Comparison

| Interface |	Target Port |	Boot Switch |	Upload Method in IDE |	Primary Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **CP2102 UART** |	Baseboard USB-C |	`BOOT / ON` |	STM32CubeProgrammer (Serial) |	Default single-cable flashing & logging |
| **USB-OTG DFU** |	MCU Card Native USB-C |	`BOOT / ON` |	STM32CubeProgrammer (DFU) |	Direct high-speed USB flashing without bridge |
| **ST-Link V2** |	4-Pin SWD Header |	`NORMAL / OFF` | STM32CubeProgrammer (SWD) |	Advanced hardware debugging & recovery |

## 1. CP2102 UART Method (Primary Interface)

**The Orbit-Q carrier features an onboard CP2102 USB-to-UART bridge connected to pins PA9 (TX) and PA10 (RX). This is the recommended interface for everyday development.**

**Arduino IDE Setup**

| Menu Item |	Selected Setting |	Notes |
| :--- | :--- | :---|
| **Board** |	`Generic STM32F4 series` |	Base core package |
| **Board part number** |	`STM32F405RGTx` |	Target MCU variant (1024KB Flash, 192KB RAM) |
| **Upload method** |	`STM32CubeProgrammer (Serial)` |	Flashes via ROM bootloader over UART bridge |
| **Port** |	`Select CP2102 COM Port` |	e.g., COMx on Windows or /dev/ttyUSBx on Linux/Mac |

## Step-by-Step Procedure

- **Connect Cable:** Plug a USB Type-C cable into the **CP2102 USB-C** port on the Orbit-Q baseboard.
- **Set Boot Switch:** Move the Boot Slide Switch to the `BOOT / ON` position.
- **Trigger Reset:** Reset the target processor using either method:
  - **Method A:** Press and release the **RESET (NRST)** button.
  - **Method B:** Toggle the main **Power Switch OFF and back ON.**
- **Select Port:** In Arduino IDE, go to Tools > Port and select the detected CP2102 serial port.
- **Upload Sketch:** Click the Upload arrow button in Arduino IDE (or press `Ctrl + U`).
- **Switch to Normal Execution:** Once flashing completes, move the Boot Slide Switch back to `NORMAL / OFF.`
- **Execute:** Press the **RESET** button (or power cycle the board).

## 2. USB-OTG DFU Method (Direct Native USB)

**The STM32F405 contains a full-speed USB-OTG peripheral (PA11/PA12). In bootloader mode, it acts as a USB DFU (Device Firmware Upgrade) target, bypassing external serial chips.**

## Arduino IDE Setup

| Menu Item | Selected Setting | Notes |
| :--- | :--- | :---|
| **Board** |	Generic STM32F4 series |	Base core package |
| **Board part number** |	STM32F405RGTx |	Target MCU variant |
| **Upload method** |	`STM32CubeProgrammer (DFU)` |	Direct USB bootloader flashing |
| **Port** |	Not Required |	DFU protocol communicates directly without COM port assignment |

## Step-by-Step Procedure

- **Connect Cable:** Plug a USB Type-C cable directly into the **USB-OTG port on the STM32F405 MCU card.**
- **Set Boot Switch:** Move the Boot Slide Switch to the `BOOT / ON` position.
- **Trigger Reset:** Press and release the RESET (NRST) button.
- **Upload Sketch:** Click Upload `(Ctrl + U)` in **Arduino IDE.**
- **Switch to Normal Execution:** Once complete, return the Boot Slide Switch to `NORMAL / OFF`.
- **Execute:** Press the **RESET** button.

!!! warning "Windows USB DFU Driver (Zadig)"
If Arduino IDE reports `No DFU device found` on Windows, download **Zadig**, select Options > List All Devices, locate **STM32 BOOTLOADER**, and replace the driver with WinUSB (v6.1.7600.16385).

## 3. ST-Link V2 / SWD Method (Hardware Debugger)

Flashing via the **SWD (Serial Wire Debug)** interface connects an external ST-Link programmer to the onboard header.

!!! tip "No Boot Switch Toggling Required"
SWD programming works while the board remains in regular operational mode (`NORMAL / OFF`). You do not need to toggle the Boot Slide Switch or force bootloader mode.

### Pinout Connection

| ST-Link V2 Pin | Orbit-Q Header Pin | Description |
| :--- | :--- | :---|
| **3V3** |	3V3	| Target Power Reference |
| **SWCLK** |	PA14	| Serial Wire Clock |
| **SWDIO** |	PA13	| Serial Wire Data Input/Output |
| **GND** |	GND	| Common Ground |

### Arduino IDE Setup

| Menu Item | Selected | SettingNotes |
| :--- | :--- | :---|
| **Board** | `Generic STM32F4 series` | Base core package |
| **Board part number** | `STM32F405RGTx` | Target MCU variant |
| **Upload method** | `STM32CubeProgrammer (SWD)` | Uses ST-Link programmer probe |
| **Port** | **Not Required* | Communicates via ST-Link USB driver |






| Component | Target Pin | Description |
| :--- | :--- | :--- |
| **Onboard User LED** | `PC13` | Green status LED on the STM32F405 MCU card |
| **UART Bridge** | `PA9` (TX) / `PA10` (RX) | CP2102 USB-C interface for flashing and serial log output |

