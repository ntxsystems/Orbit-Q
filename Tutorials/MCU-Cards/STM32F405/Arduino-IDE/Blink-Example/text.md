# Orbit-Q: Blink LED Example

Learn how to configure your environment, set up the bootloader, and upload the classic Blink example to the **STM32F405** MCU card on Orbit-Q using the Arduino IDE.

---

## Hardware Overview & Mapping

The status LED is located directly on the STM32F405 MCU card and connected to pin `PC13`.

| Component | Target Pin | Description |
| :--- | :--- | :--- |
| **Onboard User LED** | `PC13` | Green status LED on the STM32F405 MCU card |
| **UART Bridge** | `PA9` (TX) / `PA10` (RX) | CP2102 USB-C interface for flashing and serial log output |

---

## 1. Arduino IDE Setup

Before writing or compiling your code, configure your toolchain options under **Tools** in the Arduino IDE:

| Menu Item | Selected Setting | Notes |
| :--- | :--- | :--- |
| **Board** | `Generic STM32F4 series` | Core STM32 architecture package |
| **Board part number** | `STM32F405RGTx` | Target MCU variant (1024KB Flash, 192KB RAM) |
| **Upload method** | `STM32CubeProgrammer (Serial)` | Flashes via ROM bootloader over the CP2102 UART bridge |
| **Port** | Select CP2102 COM Port | e.g., `COMx` on Windows or `/dev/ttyUSBx` on Linux/Mac |

---

## 2. Entering Bootloader Mode

To allow the STM32F405 internal bootloader to accept code over UART via the CP2102 bridge, perform the following hardware sequence:

1. **Connect Cable:** Plug a USB Type-C cable into the **CP2102 USB-C port** on the Orbit-Q baseboard.
2. **Set Boot Switch:** Move the **Boot Slide Switch** to the `BOOT / ON` position.
3. **Trigger Reset:** Reset the target processor using either method:
   * **Method A:** Press and release the **RESET (NRST)** button.
   * **Method B:** Toggle the main **Power Switch** OFF and back ON (or power cycle the supply rail).
4. **Select Port:** In Arduino IDE, go to **Tools > Port** and select the detected CP2102 serial port.

!!! info "Flexibility"
    While this tutorial uses the **CP2102 UART** bridge for single-cable flashing and serial monitoring, the board also supports **ST-Link V2 (SWD)** and **USB-OTG (DFU)** upload methods.

---

## 3. Code Example

Open a new sketch in Arduino IDE or go to **File > Examples > 01.Basics > Blink**, then update the code as shown below:
```cpp
/*
  Blink

  Turns an LED on for one second, then off for one second, repeatedly.

  On the Orbit-Q STM32F405 MCU Card by NTX Systems, the on-board status LED
  is connected to digital pin PC13. We define LED_BUILTIN as PC13 to target
  this specific pin on the hardware platform.

  modified 8 May 2014
  by Scott Fitzgerald
  modified 2 Sep 2016
  by Arturo Guadalupi
  modified 8 Sep 2016
  by Colby Newman
  modified for Orbit-Q STM32F405
  by NTX Systems Pvt. Ltd.

  This example code is in the public domain.

  https://ntxsystems.github.io/Orbit-Q/
*/

// Define the onboard LED pin for Orbit-Q STM32F405
#define LED_BUILTIN PC13

// the setup function runs once when you press reset or power the board
void setup() {
  // initialize digital pin PC13 as an output.
  pinMode(LED_BUILTIN, OUTPUT);
}

// the loop function runs over and over again forever
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);                       // wait for a second
  digitalWrite(LED_BUILTIN, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);                       // wait for a second
}
```
---

## 4. Flash and Run
Upload Sketch: Click the Upload arrow button in Arduino IDE (or press Ctrl + U).

Switch to Normal Execution: Once flashing completes, move the Boot Slide Switch back to the NORMAL / OFF position.  

Execute: Press the RESET button (or power cycle the board).  

!!! success "Expected Output"
The green status LED on your STM32F405 MCU card will turn ON for 1 second and OFF for 1 second in a continuous loop.  

## Troubleshooting  
??? bug "Upload Failed or LED Not Blinking"
* Failed to Init Serial Bootloader: Verify the Boot Slide Switch was set to BOOT / ON before you pressed the RESET button or power-cycled the board.
* Program Doesn't Run After Upload: Ensure you toggled the Boot Slide Switch back to NORMAL / OFF and pressed RESET after uploading.
* Board Not Detected: Check that the M.2 MCU card is properly seated and locked into the carrier slot.
