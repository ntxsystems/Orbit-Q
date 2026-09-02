# Orbit-Q: Blink LED Example

Learn how to compile and upload your first "Hello World" program—blinking an LED—on the Orbit-Q platform using the Arduino IDE.

---

## Overview

The Blink example verifies that your toolchain, driver setup, and hardware programming interface (ST-Link or CP2102 UART bridge) are functioning correctly.

!!! note "Hardware Compatibility"
    This tutorial applies to both the **STM32F103** and **STM32F405** MCU cards mounted on the Orbit-Q Infrastructure Carrier.

---

## Hardware Mapping

| Component | Default Pin | Logic Active State | Description |
| :--- | :--- | :--- | :--- |
| **Status LED** | `PC13` | Active LOW (`LOW` = ON) | Onboard user LED on the target MCU Card |
| **CP2102 UART** | `PA9` (TX) / `PA10` (RX) | 3.3V Logic | Serial debug output via primary USB-C port |

---

## Code Example

=== "Arduino IDE (C++)"

    ```cpp
    /*
     * Orbit-Q: Simple Blink Example
     * Target: STM32 MCU Card (STM32F405 / STM32F103)
     * Manufacturer: NTX Systems Pvt. Ltd.
     */

    #define USER_LED PC13  // Target GPIO for MCU Card Status LED

    void setup() {
      // Configure LED pin as digital output
      pinMode(USER_LED, OUTPUT);
      
      // Initialize hardware UART for debug messages via CP2102
      Serial.begin(115200);
      Serial.println("Orbit-Q System Initialized.");
    }

    void loop() {
      // Active LOW: Pulling GPIO LOW turns the LED ON
      digitalWrite(USER_LED, LOW);
      Serial.println("LED State: ON");
      delay(500);

      // Pulling GPIO HIGH turns the LED OFF
      digitalWrite(USER_LED, HIGH);
      Serial.println("LED State: OFF");
      delay(500);
    }
    ```

=== "MicroPython"

    ```python
    # Orbit-Q: MicroPython Blink Example
    # Target: STM32 MCU Card

    import machine
    import time

    # Initialize PC13 as Output
    led = machine.Pin("PC13", machine.Pin.OUT)

    print("Orbit-Q MicroPython Blink Loop Running...")

    while True:
        led.value(0)  # Turn LED ON (Active Low)
        time.sleep(0.5)
        led.value(1)  # Turn LED OFF
        time.sleep(0.5)
    ```

---

## Step-by-Step Upload Guide

1. **Connect Hardware:** Plug a USB Type-C cable into the Orbit-Q base board and connect it to your PC.
2. **Configure Board Settings in Arduino IDE:**
   * Go to **Tools > Board > STM32 Board Series**.
   * Select your target processor series (e.g., *Generic STM32F4 series* or *Generic STM32F1 series*).
   * Under **Board part number**, select your card's specific variant (e.g., *STM32F405RGTx*).
3. **Select Upload Method:**
   * **ST-Link SWD (Recommended):** Set **Tools > Upload method** to `STM32CubeProgrammer (SWD)`.
   * **CP2102 UART Bootloader:** Set **Tools > Upload method** to `STM32CubeProgrammer (Serial)` and select the COM port corresponding to the CP2102 bridge.
4. **Compile and Upload:** Press **Ctrl + U** (or click the arrow icon in Arduino IDE).

!!! success "Expected Result"
    Once flashing completes, the status LED on your MCU card will blink at 1 Hz (500 ms ON, 500 ms OFF). Open the **Serial Monitor** at `115200` baud to observe live debugging output.

---

## Troubleshooting

??? bug "Troubleshooting Connection Issues"
    * **Target Not Found (SWD):** Ensure the M.2 MCU card is fully seated and secured in the slot on the Orbit-Q base board.
    * **No Serial Output:** Confirm that your Serial Monitor baud rate is set to `115200` and that the CP2102 USB-C cable is connected.
