# Orbit-Q: MicroPython Blink LED Example
## *Learn how to configure your environment, set up Thonny IDE, and run the classic MicroPython Blink example on the STM32F405 MCU card on Orbit-Q using direct USB-OTG*

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
```cpp
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
```

### 4. Run and Save

- Run Interactively: Click the Run current script button (or press F5) in Thonny to start execution immediately.
- Save to Run on Power-Up:
   - Go to File > Save as...
   - Select MicroPython device when prompted.
   - Name the file main.py and click OK. (MicroPython automatically executes main.py upon power-up/reset).

<div align="center" markdown>

<video autoplay muted loop playsinline width="30%">
  <source src="/Orbit-Q/Tutorials/MCU-Cards/STM32F405/Micro-Python/LED-Blink-Example/media/Blink-pc13-video-python.mp4">
</video>

</div>

!!! success "Expected Output"
The green status LED on your STM32F405 MCU card will turn ON for 1 second and OFF for 1 second in a continuous loop, while "LED ON" and "LED OFF" messages print sequentially in the Thonny Shell.  

### Troubleshooting
??? bug "REPL Connection Failed or USB Device Not Detected"
- Wrong USB Port: Ensure the USB Type-C cable is connected directly to the USB-OTG port on the STM32F405 MCU card rather than an external bridge or power-only port.
- Device Busy / REPL Locked: Click the red Stop / Restart backend button in Thonny or press Ctrl + C in the Shell to interrupt any running code blocking the REPL.
- USB CDC Driver Issues: On Windows, verify under Device Manager > Ports (COM & LPT) that the device appears as a USB Serial Device (VCP). If missing, check USB cable power/data capabilities.
- Standalone Execution: To run the code autonomously without being tethered to Thonny, ensure the file is named specifically as main.py directly on the MicroPython filesystem.
