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

The STM32F405 contains a full-speed USB-OTG peripheral. In bootloader mode, it acts as a USB DFU (Device Firmware Upgrade) target, bypassing external serial chips.

### Arduino IDE Setup

| Menu Item | Selected Setting | Notes |
| :--- | :--- | :---|
| **Board** | Generic STM32F4 series | Base core package |
| **Board part number** | STM32F405RGTx | Target MCU variant |
| **Upload method** | `STM32CubeProgrammer (DFU)` | Direct USB bootloader flashing |
| **Port** | **Select the active USB port** | USB-OTG requires the appropriate port selection to communicate |

### Step-by-Step Procedure

1. **Configure Power (JP-6):** Determine your power source for the flash process. 
    * If powering the module directly from the USB cable, set the **JP-6** jumper to the **MCU VIN USB 5V** position. 
    * If using the board's main power supply, set the **JP-6** jumper to the **+5V MCU VIN** position.
2. **Connect Cable:** Plug a USB Type-C cable into the **MCU USB** connector on the Orbit-Q docking station. This connector routes directly to the USB DP (Pin 8) and USB DN (Pin 9) signals of the M.2 connector.
3. **Verify Power:** Check the status indicators. **LED-9** should illuminate to confirm 5V is present on the MCU USB connector, and **LED-8** should illuminate to confirm the onboard 3.3V LDO is supplying power to the M.2 module.
4. **Set Boot Switch:** Move the Boot Slide Switch to the `BOOT / ON` position (this ties the BOOT-0 signal, which is exposed on I/O PIN Header C at Pin 53, high).
5. **Trigger Reset:** Press and release the RESET (NRST) button.
6. **Upload Sketch:** Click Upload `(Ctrl + U)` in the **Arduino IDE**.
7. **Switch to Normal Execution:** Once complete, return the Boot Slide Switch to `NORMAL / OFF`.
8. **Execute:** Press the **RESET** button.

!!! warning "Windows USB DFU Driver (Zadig)"
    If Arduino IDE reports `No DFU device found` on Windows, download **Zadig**, select Options > List All Devices, locate **STM32 BOOTLOADER**, and replace the driver with WinUSB (v6.1.7600.16385).

## 3. ST-Link V2 / SWD Method (Hardware Debugger)

Flashing via the SWD (Serial Wire Debug) interface utilizes the Orbit-Q development platform's onboard ST-LINK debugger, which is based on an STM32F103 microcontroller. This provides a dedicated programming and debugging interface for the installed M.2 microcontroller module without the need for additional external hardware.

!!! tip "No External Wiring or Boot Switch Toggling Required"
    The Serial Wire Debug lines are internally connected (SWCLK to PIN 5 and SWDIO to PIN 6 on Header A), meaning no jumper configuration is required during normal operation. Additionally, SWD programming works seamlessly while the board remains in its standard operational mode (NORMAL / OFF).

### Arduino IDE Setup

| Menu Item | Selected Setting | Notes |
| :--- | :--- | :--- |
| **Board** | Generic STM32F4 series | Base core package |
| **Board part number** | STM32F405RGTx | Target MCU variant |
| **Upload method** | STM32CubeProgrammer (SWD) | Uses the onboard ST-LINK debugger |
| **Port** | Not Required | Communicates via the ST-Link USB driver |

### Step-by-Step Procedure

1. **Connect to Debugger:** Connect the **USB-3** port to your host computer using a USB Type-C cable. This dedicated port operates independently of the USB-to-UART bridge.
2. **Set Boot Switch:** Ensure the Boot Slide Switch is set to `NORMAL / OFF.`
3. **Upload Sketch:** Click **Upload** (`Ctrl + U`). Supported development environments will automatically detect the onboard ST-LINK, flash the target M.2 module, and trigger a software reset to execute your code.

### Troubleshooting
??? bug "Upload Failed or Board Not Responding"
- **Failed to Init Serial Bootloader (CP2102):** Verify the Boot Slide Switch was set to `BOOT / ON` before you pressed RESET. Close the Arduino Serial Monitor prior to uploading, as an open serial connection locks the COM port.
- **Program Doesn't Run After Upload:** Ensure you toggled the Boot Slide Switch back to NORMAL / OFF and pressed RESET after uploading over CP2102 or USB-OTG.
- **DFU Device Not Detected:** Verify your USB cable supports data (not power-only) and is connected directly to the native USB-OTG port.
- **STM32CubeProgrammer Not Found:** Ensure STM32CubeProgrammer is installed in its default system path. Restart Arduino IDE after installation so system environment variables refresh.



| Component | Target Pin | Description |
| :--- | :--- | :--- |
| **Onboard User LED** | `PC13` | Green status LED on the STM32F405 MCU card |
| **UART Bridge** | `PA9` (TX) / `PA10` (RX) | CP2102 USB-C interface for flashing and serial log output |

