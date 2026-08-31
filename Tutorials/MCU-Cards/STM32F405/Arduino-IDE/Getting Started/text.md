# Start-Up Manual: STM32 in Arduino IDE

To use the STM32F series microcontroller with the Arduino IDE, you need to install the STM32duino core and configure the correct upload method.

---

## 1. Configure Arduino IDE

Open the Preferences window by navigating to **File > Preferences**.

![Step 1](images/image-01.png)

---

## 2. Add Board URL

In the **"Additional Boards Manager URLs"** field, paste the official STM32 index link:

```text
[https://github.com/stm32duino/BoardManagerFiles/raw/main/package_stmicroelectronics_index.json](https://github.com/stm32duino/BoardManagerFiles/raw/main/package_stmicroelectronics_index.json)

```

![Step 1](images/image-02.png)

---

## 3. Install Boards

Go to Tools > Board > Boards Manager.

Search for "STM32 MCU based boards".

Click Install.

![Step 1](images/image-03.png)

---

## 4. Install Support Tools (Optional but Recommended)

Download and install STM32CubeProgrammer from STMicroelectronics. It acts as a bridge for uploading code and ensures you have the necessary USB/SWD drivers installed.

[Download STM32CubeProgrammer from STMicroelectronics](https://www.st.com/en/development-tools/stm32cubeprog.html)

---

## 5. Select Your Board and Settings 

Once installed, configure your hardware settings under the Tools menu:

Board: Select STM32 MCU based boards - Your Series, e.g., Generic STM32F4 series].

Board Part Number: Choose your specific chip (e.g., Generic F405RGTx).

![Step 1](images/image-04.png)

Upload Method: This depends on your hardware connection:

ST-Link: Best for debugging; requires an ST-Link V2 programmer.

STM32CubeProgrammer (Serial): Used with a USB-to-TTL adapter.

STM32CubeProgrammer (DFU): Used if your board has a native USB port and DFU bootloader.

