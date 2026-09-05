# Orbit-Q Visualizer V2: Build & Code Walkthrough

*This tutorial explains `Orbit-Q_Visualizer_v2.ino`: a 6-mode ambient display running on the ORBIT-Q board with an STM32F405RGT6 MCU card, driving a 128×32 I2C OLED and a 10-pixel WS2812B strip as one coordinated visual surface, not two separate animations. If you're following along to build this yourself, read the "Core Concept" section first — every mode in the code only makes sense once you understand that one idea.*

### 1. What this demo actually does
Six modes cycle automatically, ~90 seconds total, then loop forever:

| # | Mode | OLED shows | LED strip shows
| --- | --- | --- | --- |
| 0 | Wave Cross | scrolling waveform | the same waveform function, sampled past the right edge of the screen |
| 1 | Orbit Sweep | a dot orbiting with a fading trail | a comet at the matching angular position |
| 2 | Plasma Bloom | a dithered 2D plasma field | the same field, sampled off-screen |
| 3 | Radar Sweep | rotating sweep line + 4 decaying blips | a sharp point mirroring the sweep angle, flashing on blip hits |
| 4 | VU Spectrum | 8 bars driven by synthesized envelopes (not real audio — there's no mic on this board) | a level-meter bar with a decaying peak-hold marker |
| 5 | Matrix Cascade | falling column drops | a flash on the mapped LED the instant a column's drop crosses the bottom edge |

Transitions fade in/out over 600 ms so mode changes aren't an abrupt cut.

### 2. Hardware you need
- ORBIT-Q board with the STM32F405RGT6 MCU card (Cortex-M4F core, hardware single-precision FPU)
- SSD1306 128×32 OLED, I2C, address `0x3C`.
- WS2812B strip/ring, 10 pixels
- ST-Link (or your usual ORBIT-Q upload path)



