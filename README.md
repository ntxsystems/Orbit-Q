<div align="center" markdown>

# ORBIT-Q

### The Modular Embedded Development Workstation

**Official documentation, datasheets, pinout guides, and code examples for the Orbit-Q development board by NTX SYSTEMS.**

</div>

[View on GitHub :material-github:](https://github.com/ntxsystems/Orbit-Q){ .md-button }

---

![Orbit-Q Board](images/image_9cbed628.png)

## Why ORBIT-Q Exists
Most major hardware ecosystems have a reference dev board behind them: Raspberry Pi for the UK, the ESP series out of China, ST Nucleo and Arduino out of Europe, Teensy/Adafruit/SparkFun out of the US. India didn't have an equivalent of its own — that was the starting gap.

But a dev board on its own isn't a reason to build anything; the interesting part was working out why that gap existed, not just that it did. Looking at what people in India actually reach for:

- **Arduino and ESP32 dominate** — they're cheap, and the volume of online projects/tutorials makes them the path of least resistance.
- **ST Nucleo sees some use, but rarely** — despite STM32 being the MCU family behind a large share of real unmanned/embedded systems work, it's underused at the learning stage.
- **Raspberry Pi shows up for a different reason** — it's a full SBC, not strictly a dev board, and it gets picked when a project wants to show AI/advanced-compute capability rather than do MCU-level embedded work.

Many development boards are built around a single processor and provide only the essentials needed to get that processor running. As projects become more ambitious, users often end up adding programmers, power supplies, USB interfaces, displays, storage and other peripheral modules around the board. Moving to a different processor usually means changing the entire setup as well.

!!! tip "The Orbit-Q Philosophy"
    **ORBIT-Q was designed to remove that fragmentation:** provide the development infrastructure in one reusable platform, while making the compute module itself replaceable.

---

![Orbit-Q Board](<images/Yellow and Blue Modern Logistics Company Profile A4 Document.png>)

---

## Designing for the Market Reality

That's the problem ORBIT-Q was built to close, not just "make a board." The design decisions map directly to the findings above:

- **Onboard ST-Link + CP2102:** Removes the external-programmer step that keeps people off STM32; the CP2102 UART bridge isn't STM32-specific, so it's useful with other MCU cards too.
- **Proper onboard power delivery:** (24W, dual-rail) removes the "every project needs its own supply" problem common to all the boards above.
- **OLED display, addressable LEDs, microSD:** These are the peripherals that show up in most real projects anyway (status output, visual feedback, data logging), so they're built in rather than breadboarded each time.
- **M.2 swappable MCU card:** The real fragmentation problem isn't Arduino vs. STM32 vs. ESP32, it's that switching between them means rebuilding your whole setup. Making the MCU itself the only part that swaps solves that directly.

The goal wasn't just to close the gap — it was to close it in a way that actually fits the market. A price-sensitive market doesn't just want a cheaper board; it wants a lower total cost to get to a working setup. Comparing bare board prices misses that most of the real cost is external — a programmer, a power supply, breadboarded peripherals for every project. Folding those into one board changes the comparison from "board vs. board" to **"total setup cost vs. total setup cost,"** and that's where the price argument actually holds.

The card system does something similar for the learning curve. On Arduino or ESP32, moving to a more capable architecture usually means starting over on a new board, new wiring, new tooling. On ORBIT-Q, the same carrier carries you from a basic validation-tier MCU card to a flagship-tier one — the STM32F103 card to the STM32F405 and beyond — without rebuilding anything. That matters specifically because STM32 is the more relevant architecture for real unmanned/embedded work and stays underlearned for exactly the setup-friction reasons above.

> The result is a single board that carries everything a project usually needs onboard, with the MCU as the one modular part — the same model a desktop PC uses: change the part that needs changing (CPU/GPU), keep everything else (power supply, case, peripherals) as-is.

---

![Orbit-Q Board](images/example-code-001.jpeg)

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

    Wireless targets (ESP32 / nRF series), RISC-V (CDAC Vega), and FPGA module cards. *(Roadmap items, not shipping yet)*.

</div>

---

## Full Technical Specifications

| Parameter | Specification |
| :--- | :--- |
| **Input Voltage Range** | 6.0 V – 16.8 V DC (Recommended 12 V DC) |
| **Absolute Max Input** | 20.0 V DC |
| **Power Supply Inputs** | DC barrel jack + battery solder pads (mutually exclusive) |
| **Recommended Supply** | 12 V DC, 2.5 A or greater |
| **PWR-1 Rail** | 5.0 V @ 3 A (OLED, LEDs, SD card slot, M.2 slot) |
| **PWR-2 Rail** | 3.3 V @ 3 A (independent, reserved for external expansion) |
| **MCU Interface** | 75-position M.2 E-Key slot, custom pinout mapping |
| **GPIO Breakout** | 68 pins across 3 headers (A, B, C) |
| **Status Display** | 128×32 monochrome OLED, I²C (default address 0x3C) |
| **Addressable Lighting** | 10× WS2812B addressable RGB LEDs (5V rail) |
| **Data Storage** | Onboard microSD, SPI interface, 3.3V LDO |
| **USB-to-UART Bridge** | CP2102, USB Type-C, USB 2.0 Full Speed |
| **Hardware Debugger** | ST-Link V2 compatible (dedicated STM32F103), USB-C SWD port |

---

<video autoplay muted loop playsinline width="60%">
  <source src="/Orbit-Q/images/videos/orbit-q-video-001.mp4" type="video/mp4">
</video>

---

## Quick Resources

* 📄 **Datasheet:** [Download Orbit-Q Datasheet PDF](./Orbit-Q%20Datasheet.pdf)
* 📷 **Board Photos:** [Browse high-resolution imagery on GitHub](https://github.com/ntxsystems/Orbit-Q/tree/main/images)

---

## Overview

Orbit-Q is a modular development board engineered by **NTX Systems Pvt. Ltd.** for rapid prototyping across embedded systems, IoT, and robotics projects.

* **Manufacturer:** NTX Systems Pvt. Ltd.
* **Programming Languages:** C / C++ (Arduino Framework), MicroPython
* **Documentation & Drivers:** Open-source
* **Hardware Design:** Proprietary / Closed-hardware

---

## License

All documentation, sample code, and software drivers in this repository are released under the [MIT License](LICENSE). Hardware designs, layout files, and product specifications remain the proprietary IP of **NTX Systems Pvt. Ltd.**
