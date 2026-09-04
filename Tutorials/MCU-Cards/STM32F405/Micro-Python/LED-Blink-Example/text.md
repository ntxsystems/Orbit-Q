# Orbit-Q: MicroPython Blink LED Example
## **Learn how to configure your environment, set up Thonny IDE, and run the classic MicroPython Blink example on the STM32F405 MCU card on Orbit-Q using direct USB-OTG*

### Hardware Overview & Mapping

The status LED is located directly on the STM32F405 MCU card and connected to pin `PC13`.  

| Component |	Target Pin |	Description |
| --- | --- | --- |
| Onboard User LED | PC13 | Green status LED on the STM32F405 MCU card (Active High) |
| USB-OTG Port | PA11 (USB_DM) / PA12 (USB_DP) | Native STM32F405 USB-OTG interface for MicroPython CDC VCP Serial REPL and file transfer |

### 1. Thonny IDE Setup

**Before running your MicroPython scripts, configure your interpreter options in Thonny IDE:**
| Menu Item | Selected Setting | Notes |
| --- | --- | --- |
| Interpreter Kind | MicroPython (generic) or MicroPython (STM32) | Select the MicroPython runtime target |
| Port or WebREPL | Select STM32 USB CDC / VCP Serial Port | e.g., COMx (USB Serial Device) on Windows or /dev/ttyACM0 on Linux/Mac |

### 2. Connecting via USB-OTG

 **To establish a REPL connection with the STM32F405 running MicroPython over the native USB interface:**
 - Connect Cable: Plug a USB Type-C cable directly into the USB-OTG port on the STM32F405 MCU card.
 - Open Settings: In Thonny IDE, navigate to Tools > Options... and click the Interpreter tab.
 - Select Interpreter: Choose MicroPython (generic) or MicroPython (STM32) from the dropdown list.
 - Select Port: Set the port to the detected STM32 USB Virtual COM Port (VCP / CDC) and click OK.
 - Verify Connection: The Shell panel at the bottom of Thonny should display the MicroPython REPL prompt (>>>).

!!! info "MicroPython USB Firmware Pre-requisite"
This guide assumes MicroPython firmware with native USB VCP stack support is already flashed onto your STM32F405 board. If your board is erased or in DFU mode, flash the MicroPython .dfu or .bin binary using STM32CubeProgrammer over DFU mode before proceeding.

### 3. Code Example

**Create a new file in Thonny IDE (Ctrl + N) and paste the following script:**

# ===========================================================================
# File:         blink_pc13.py
# Description:  Simple LED blink example to verify board functionality.
#               Toggles the onboard LED every 1 second.
# 
# Board:        Orbit-Q Modular Development Board
# Manufacturer: Neotrivotx Systems Pvt. Ltd. (NTX Systems) - Made in India
# MCU:          STM32F405RGT6
# Language:     MicroPython
#
# Hardware:     - Onboard LED connected to pin PC13 (Active High)
# Dependencies: - Built-in 'machine' and 'time' modules
#
# Author:       Siddhant Gupta
# Date:         September 2026
# Version:      1.0
# License:      MIT License
# ===========================================================================

from machine import Pin
import time

# Initialize pin PC13 as an output
# Since the LED is not reversed, value(1) will turn it ON
led = Pin('PC13', Pin.OUT)

print("Starting PC13 Blink Sequence...")

# Main loop
while True:
    led.value(1)      # Turn LED ON
    print("LED ON")
    time.sleep(1)     # Wait for 1 second
    
    led.value(0)      # Turn LED OFF
    print("LED OFF")
    time.sleep(1)     # Wait for 1 second
