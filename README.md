<div align="center" markdown>

# ORBIT-Q
**One Platform. Multiple MCU Possibilities.**

![Status](https://img.shields.io/badge/Status-Active%20Development-success?style=for-the-badge)
![STM32](https://img.shields.io/badge/STM32-03234C?style=for-the-badge&logo=stmicroelectronics&logoColor=00ECFF)
![Arduino](https://img.shields.io/badge/Arduino-00979D?style=for-the-badge&logo=arduino&logoColor=white)
![MicroPython](https://img.shields.io/badge/MicroPython-2B2728?style=for-the-badge&logo=micropython&logoColor=white)

</div>

---

ORBIT-Q is a modular embedded development platform built around a 75-position M.2 E-Key connector for compatible microcontroller modules. It combines power management, debugging, USB-to-UART, display, storage, RGB LEDs, and I/O expansion into a single development platform for rapid prototyping, embedded software development, and hardware validation.

![Orbit-Q Board](images/vibe3d-render-64e02aa3.jpg)



| **Manufacturer** | NTX Systems Pvt. Ltd. |
| :--- | :--- |
| **Current MCU Cards** | STM32F103 (Cortex-M3), STM32F405 (Cortex-M4, hardware FPU) |
| **Roadmap** | ESP32, RP2040, CDAC Vega (RISC-V), nRF54, FPGA |
| **Programming** | C / C++ (Arduino Framework), MicroPython |
| **Hardware Design** | Proprietary / Closed-hardware |
| **Documentation & Drivers** | Open-source (MIT License) |

---
## Contents

- [Features](#features)
- [Hardware Architecture](#hardware-architecture)
- [Power Management](#power-management)
- [User Interfaces](#user-interfaces)
  - [OLED Display](#oled-display)
  - [RGB LEDs](#rgb-leds)
- [USB-to-UART](#usb-to-uart)
- [M.2 E-Key Interface](#m2-e-key-interface)
- [I/O Expansion](#io-expansion)
- [MicroSD Card](#microsd-card)
- [ST-LINK Debugger](#st-link-debugger)
- [Development](#development)
- [Documentation](#documentation)
- [License](#license)

## Features

- Modular M.2 E-Key MCU architecture
- 75-position M.2 E-Key connector
- 6 V – 16.8 V DC input range
- 5 V @ 3 A and 3.3 V @ 3 A regulated power rails
- On-board STM32F103-based ST-LINK debugger
- CP2102 USB-to-UART bridge
- 128 × 32 monochrome OLED display
- 10 × WS2812B individually addressable RGB LEDs
- On-board microSD card interface
- 68 accessible I/O signals
- SPI, I²C, UART, PWM, ADC, USB and SWD interfaces
- Jumper-configurable peripheral connections
- Standard 2.54 mm expansion headers

## Hardware Architecture

ORBIT-Q separates the microcontroller module from the main development platform.

The M.2 E-Key connector acts as the central interface between the installed microcontroller module and the board's power, communication, debugging, storage, display, and expansion subsystems. This allows compatible MCU modules to be changed while retaining the same development platform.

## Power Management

ORBIT-Q uses two independent high-efficiency switching regulators to provide:

| Rail  | Output | Maximum Current |
|-------|--------|------------------|
| 5 V   | 5.0 V  | 3 A              |
| 3.3 V | 3.3 V  | 3 A              |

The board accepts power through either a DC barrel jack or dedicated battery input pads, with an operating input range of 6 V to 16.8 V. Dedicated power headers are also provided for external circuits and peripherals.

## User Interfaces

### OLED Display

ORBIT-Q includes an onboard 128 × 32 monochrome OLED connected through I²C.

It can be used for:

- System status
- Menus
- Sensor data
- Debug information
- User-defined graphics

A jumper-based interface allows the OLED connection to be isolated from the MCU module.

### RGB LEDs

The board includes 10 WS2812B individually addressable RGB LEDs for:

- Visual feedback
- Status indication
- Diagnostics
- User applications

The LED interface uses a single-wire digital data connection with configurable power and signal routing.

## USB-to-UART

A CP2102-GMR USB-to-UART bridge provides a USB serial interface between the host computer and the installed MCU module.

The interface supports:

- Firmware logging
- Serial communication
- Command-line interaction
- Debugging

Connection is provided through a USB Type-C connector.

## M.2 E-Key Interface

The ORBIT-Q platform uses a 75-position M.2 E-Key connector as its primary microcontroller interface.

The connector provides access to:

- Power
- GPIO
- UART
- SPI
- I²C
- USB
- SWD
- Other MCU signals

The installed module acts as the central processing element for the onboard subsystems and expansion interfaces.

## I/O Expansion

Three expansion headers provide access to 68 I/O signals from the installed microcontroller module.

Supported signals include:

- Digital GPIO
- UART
- SPI
- I²C
- PWM
- ADC
- USB
- SWD
- Power and Ground

The headers use standard 2.54 mm spacing for prototyping and external hardware connections.

## MicroSD Card

ORBIT-Q includes an onboard microSD card interface using SPI.

It can be used for:

- Data logging
- Firmware storage
- Configuration files
- Removable storage
- Embedded application data

The interface is connected to the MCU module through dedicated SPI signals.

## ST-LINK Debugger

An onboard STM32F103-based ST-LINK debugger provides programming and debugging for the installed MCU module through Serial Wire Debug (SWD).

The debugger uses a dedicated USB Type-C connector and operates independently from the USB-to-UART bridge and MCU USB interface.

## Development

ORBIT-Q is designed for:

- Embedded software development
- Rapid prototyping
- Hardware validation
- Sensor and peripheral experimentation
- Data logging
- Custom embedded applications

The modular architecture allows developers to work with compatible MCU modules without redesigning the entire development platform.

## Documentation

For detailed hardware information, pin assignments, jumper configurations, electrical specifications, and subsystem operation, see the [ORBIT-Q Datasheet & Hardware Reference Manual](docs/ORBIT-Q-Datasheet.pdf).

Additional resources:

- [Getting Started](docs/getting-started.md)
- [Arduino](docs/arduino.md)
- [MicroPython](docs/micropython.md)
- [Projects](docs/projects.md)

## License

See the [LICENSE](LICENSE) file for details.
---

![Orbit-Q Board](<images/Yellow and Blue Modern Logistics Company Profile A4 Document.png>)

---

## Hardware Architecture & Ecosystem

<div class="grid cards" markdown>

-   :material-expansion-card-variant: **The Infrastructure Carrier**

    ---

    High-current power distribution (5V @ 3A and 3.3V @ 3A), dual USB Type-C interfaces, visual status displays, and full peripheral integration.

-   :material-chip: **The Swappable MCU Core**

    ---

    The target processor and minimal support circuitry on a 75-position M.2 E-Key footprint.

-   :material-sitemap: **Custom M.2 Pinout Mapping**

    ---

    Mechanically a standard M.2 connector, but electrically routes SWD debugging lines, dual power rails, SPI, I²C, UART, and 68 raw GPIO signals to dedicated expansion headers.

</div>

## MCU Card Ecosystem

<div class="grid cards" markdown>

-   :material-check-decagram: **STM32F103 Card**

    ---

    Initial validation card, ARM Cortex-M3, used to verify power distribution and interconnect signals.

-   :material-rocket-launch: **STM32F405 Card**

    ---

    Flagship high-performance target, 168 MHz ARM Cortex-M4 with hardware FPU, for real-time DSP, motor control, and sensor processing.

-   :material-router-wireless: **Planned Roadmap Targets**

    ---

    ESP32, RP2040, CDAC Vega (RISC-V), nRF54, and FPGA module cards, at roughly 60–90 day intervals. *(Roadmap items, not shipping yet.)*

</div>

!!! note "Beyond the Dev Board"
    The same MCU card standard extends to at least four carrier form factors — an education baseboard, a robotics mini carrier, a wireless-first carrier, and a modular drone baseboard — all electrically and mechanically compatible with the cards above.

---

![Orbit-Q Board](images/example-code-001.jpeg)

---

## Why STM32, and Why Modular

Arduino and ESP32 dominate India's embedded landscape on cost and tutorial volume; STM32 stays underused at the learning stage despite being the more relevant architecture for real unmanned/embedded systems work. Raspberry Pi, meanwhile, is typically chosen for AI/compute showcase projects rather than MCU-level work.

The comparison that matters isn't bare board price — it's total cost to a working setup, once an external programmer, power supply, and breadboarded peripherals are accounted for. ORBIT-Q folds those into the board itself, and the M.2 card system means moving from a validation-tier MCU to a flagship-tier one doesn't require rebuilding the setup from scratch.

---

ORBIT-Q separates infrastructure from compute. The carrier provides onboard power delivery, debugging, display, and storage — the MCU itself is the only part that swaps, via a 75-position M.2 E-Key slot with a custom pinout mapping. Moving from a validation-tier MCU card to a flagship-tier one requires no rewiring and no new tooling.

-   :material-tools: **Integrated Development Infrastructure**

    ---


## License

All documentation, sample code, and software drivers in this repository are released under the [MIT License](LICENSE). Hardware designs, layout files, and product specifications remain the proprietary IP of **NTX Systems Pvt. Ltd.**
