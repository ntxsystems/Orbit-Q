# Orbit-Q STM32F405 Development Platform - Pinout & Reference Guide

This document provides the complete hardware pinout, peripheral mappings, and configuration guidelines for the **Orbit-Q** development board featuring the **STM32F405RGT6** M.2 module.

---

## 1. Complete M.2 Pinout Mapping (Pins 1–68)

| Pin (Left) | Signal / Function | Pin (Right) | Signal / Function |
| :---: | :--- | :---: | :--- |
| **1** | VCC (3.3V / VIN) | **2** | VCC (3.3V / VIN) |
| **3** | GND | **4** | GND |
| **5** | PA14 (SWD / SWCLK) | **6** | PA13 (SWD / SWDIO) |
| **7** | GND | **8** | PA12 (USB-OTG / CAN1_TX) |
| **9** | PA11 (USB-OTG / CAN1_RX) | **10** | GND |
| **11** | PB15 (SPI2_MOSI) | **12** | PB14 (SPI2_MISO) |
| **13** | PB13 (SPI2_SCK) | **14** | PB12 (SPI2_NSS / CS) |
| **15** | GND | **16** | PB11 (I2C2_SDA / UART3_RX) |
| **17** | PB10 (I2C2_SCL / UART3_TX) | **18** | PB1 |
| **19** | PB0 | **20** | PC5 |
| **21** | PC4 | **22** | PA7 (SPI1_MOSI / TIM3_CH2) |
| **23** | PA6 (SPI1_MISO / TIM3_CH1) | **24** | PA5 (SPI1_SCK) |
| **25** | PA4 (SPI1_NSS / SD Card CS) | **26** | PA3 (USART2_RX) |
| **27** | GND | **28** | PA2 (USART2_TX) |
| **29** | PA1 (UART4_RX) | **30** | PA0 (UART4_TX / WKUP) |
| **31** | PC3 | **32** | PC2 |
| **33** | PC1 | **34** | PC0 |
| **35** | VCC | **36** | VCC |
| **37** | GND | **38** | GND |
| **39** | N.C. | **40** | N.C. |
| **41** | N.C. | **42** | N.C. |
| **43** | GND | **44** | PC6 (USART6_TX) |
| **45** | PC7 (USART6_RX) | **46** | PC8 |
| **47** | PC9 (I2C3_SDA) | **48** | N.C. |
| **49** | PC14 (OSC32_IN) | **50** | PC15 (OSC32_OUT) |
| **51** | PB9 | **52** | PB8 (TIM4_CH3) |
| **53** | BOOT0 | **54** | PB7 (I2C1_SDA / TIM4_CH2) |
| **55** | GND | **56** | PB6 (I2C1_SCL / TIM4_CH1) |
| **57** | PB5 | **58** | PB4 |
| **59** | PB3 (JTDO / TRACESWO) | **60** | PD2 (UART5_RX) |
| **61** | PC12 (SPI3_MOSI / UART5_TX) | **62** | PC11 (SPI3_MISO / UART3_RX) |
| **63** | PC10 (SPI3_SCK) | **64** | PA15 (SPI3_NSS) |
| **65** | PA10 (USART1_RX / TIM1_CH3) | **66** | PA9 (USART1_TX / TIM1_CH2) |
| **67** | GND | **68** | PA8 (I2C3_SCL / TIM1_CH1) |

---

## 2. Onboard Peripherals & Default Pin Mapping

*   **Onboard OLED Display ($128\times32$, SSD1306):**
    *   **Protocol:** I2C1 (`PB6` = SCL, `PB7` = SDA)
    *   **I2C Address:** `0x3C`
    *   **Isolation Jumper:** **JP-2** (Bridge to connect MCU to display)
*   **MicroSD Card Slot:**
    *   **Protocol:** Hardware SPI1 (`PA4` = CS, `PA5` = SCK, `PA6` = MISO, `PA7` = MOSI)
    *   **Isolation Jumper:** **JP-5**
*   **USB-to-UART Bridge (CP2102):**
    *   **Isolation Jumper:** **JP-3**
*   **Onboard ST-LINK Debugger:**
    *   **Protocol:** SWD (`PA13` = SWDIO, `PA14` = SWCLK via Header A)

---

## 3. Arduino IDE Setup & Configuration

1. **Add Board Manager URL:**
   Open Arduino IDE, go to **File > Preferences**, and paste the following URL into *Additional board manager URLs*:
   ```text
   [https://github.com/stm32duino/BoardManagerFiles/raw/main/package_stmicroelectronics_index.json](https://github.com/stm32duino/BoardManagerFiles/raw/main/package_stmicroelectronics_index.json)
