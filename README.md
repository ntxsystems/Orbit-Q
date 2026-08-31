# Orbit-Q: Modular Development Board

Official documentation, datasheets, pinout guides, and code examples for the Orbit-Q development board by NTX SYSTEMS

![Orbit-Q Board](images/image_9cbed628.png)

# Why ORBIT-Q Exists

Most major hardware ecosystems have a reference dev board behind them: Raspberry Pi for the UK, the ESP series out of China, ST Nucleo and Arduino out of Europe, Teensy/Adafruit/SparkFun out of the US. India didn't have an equivalent of its own — that was the starting gap.

But a dev board on its own isn't a reason to build anything; the interesting part was working out why that gap existed, not just that it did. Looking at what people in India actually reach for:

- **Arduino and ESP32 dominate** — they're cheap, and the volume of online projects/tutorials makes them the path of least resistance.

- **ST Nucleo sees some use, but rarely** — despite STM32 being the MCU family behind a large share of real unmanned/embedded systems work, it's underused at the learning stage.

- **Raspberry Pi shows up for a different reason** — it's a full SBC, not strictly a dev board, and it gets picked when a project wants to show AI/advanced-compute capability rather than do MCU-level embedded work.

Many development boards are built around a single processor and provide only the essentials needed to get that processor running. As projects become more ambitious, users often end up adding programmers, power supplies, USB interfaces, displays, storage and other peripheral modules around the board. Moving to a different processor usually means changing the entire setup as well.

**ORBIT-Q was designed to remove that fragmentation: provide the development infrastructure in one reusable platform, while making the compute module itself replaceable.**

---

![Orbit-Q Board](images/Yellow%20and%20Blue%20Modern%20Logistics%20Company%20Profile%20A4%20Document.png)

---
That's the problem ORBIT-Q was built to close, not just "make a board." The design decisions map directly to the findings above:

- Onboard ST-Link + CP2102 — removes the external-programmer step that keeps people off STM32; the CP2102 UART bridge isn't STM32-specific, so it's useful with other MCU cards too.
- Proper onboard power delivery (24W, dual-rail) — removes the "every project needs its own supply" problem common to all the boards above.
- OLED display, addressable LEDs, microSD — these are the peripherals that show up in most real projects anyway (status output, visual feedback, data logging), so they're built in rather than breadboarded each time.
- M.2 swappable MCU card — the real fragmentation problem isn't Arduino vs. STM32 vs. ESP32, it's that switching between them means rebuilding your whole setup. Making the MCU itself the only part that swaps solves that directly.The goal wasn't just to close the gap — it was to close it in a way that actually fits the market. A price-sensitive market doesn't just want a cheaper board; it wants a lower total cost to get to a working setup. Comparing bare board prices misses that most of the real cost is external — a programmer, a power supply, breadboarded peripherals for every project. Folding those into one board changes the comparison from "board vs. board" to "total setup cost vs. total setup cost, " and that's where the price argument actually holds.
## Quick Resources

The card system does something similar for the learning curve. On Arduino or ESP32, moving to a more capable architecture usually means starting over on a new board, new wiring, new tooling. On ORBIT-Q, the same carrier carries you from a basic validation-tier MCU card to a flagship-tier one — the STM32F103 card to the STM32F405 and beyond — without rebuilding anything. That matters specifically because STM32 is the more relevant architecture for real unmanned/embedded work and stays underlearned for exactly the setup-friction reasons above.The result is a single board that carries everything a project usually needs onboard, with the MCU as the one modular part — the same model a desktop PC uses: change the part that needs changing (CPU/GPU), keep everything else (power supply, case, peripherals) as-is.



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

## Key Features

* **Modular Form Factor:** Easily integrable with expansion modules and breadboards.
* **Flexible Power Supply:** USB-C powered with onboard regulation.
* **Peripherals:** Digital GPIO, ADC, PWM, I2C, SPI, and UART interfaces.

---

## License

All documentation, sample code, and software drivers in this repository are released under the [MIT License](LICENSE). Hardware designs, layout files, and product specifications remain the proprietary IP of **NTX Systems Pvt. Ltd.**
