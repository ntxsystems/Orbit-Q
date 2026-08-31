# Start-Up Manual: STM32 in Arduino IDE

To use the STM32F series microcontroller with the Arduino IDE, you need to install the STM32duino core and configure the correct upload method.

---

## 1. Configure Arduino IDE

Open the Preferences window by navigating to **File > Preferences**.

![Arduino Preferences Menu](../images/image-01.png)

---

## 2. Add Board URL

In the **"Additional Boards Manager URLs"** field, paste the official STM32 index link:

```text
[https://github.com/stm32duino/BoardManagerFiles/raw/main/package_stmicroelectronics_index.json](https://github.com/stm32duino/BoardManagerFiles/raw/main/package_stmicroelectronics_index.json)
