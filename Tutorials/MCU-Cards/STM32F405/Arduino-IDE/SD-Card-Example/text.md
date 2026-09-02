# Interfacing the MicroSD Card & USB-to-UART Serial Monitoring

**The Orbit-Q docking station integrates a physical **MicroSD card slot** routed via the SPI peripheral interface and a **CP2102 USB-to-UART bridge** connected to the board's serial line. Combining these two systems allows you to perform onboard data logging and view real-time debug outputs directly on your computer's Serial Monitor.**

### 1. Hardware Overview & Jumper Configurations

**MicroSD Card Interface (SPI):**
*   **Protocol:** SPI (Serial Peripheral Interface)
*   **Chip Select (CS) Pin:** `PA4` (as defined in your code)
*   **Hardware Control (JP-5 Header):** The MicroSD slot communicates via the STM32F405's hardware SPI bus. Ensure that the **JP-5** configuration header jumpers are securely placed to bridge the SPI lines (MOSI, MISO, SCK, and CS) between the M.2 module and the onboard card holder.
*   **Power & Activity Indicators:** Verify that the **SL-10** LED indicator is lit, confirming that the MicroSD card socket is receiving its required power supply.

**USB-to-UART Serial Bridge (CP2102):**
*   **Purpose:** Translates UART serial data from the STM32F405 into USB signals for your PC.
*   **Hardware Control (JP-3 Header):** Configure the **JP-3** header to route the MCU's hardware UART TX/RX lines to the CP2102 bridge chip.
*   **Data LEDs:** Look for **SL-6** (TX data transmission) and **SL-7** (RX data reception) indicators to flash when data is moving between the board and your computer.

---

### 2. Software Requirements & Library Installation

To handle file systems on the SD card efficiently, this tutorial uses the optimized **SdFat** library rather than the standard Arduino SD library.

1. In the Arduino IDE, open the Library Manager (**Sketch > Include Library > Manage Libraries...** or press `Ctrl+Shift+I`).
2. Search for `SdFat` by Bill Greiman.
3. Locate the library and click **Install** (install any recommended dependencies if prompted).

---

### 3. Arduino IDE Sketch (Write & Read Test)

Connect your USB cable to the **USB-1** port (connected to the CP2102 chip), select the appropriate COM port in the Arduino IDE, set your board options to the STM32F405, and upload the following tested sketch:

```cpp
#include <SPI.h>
#include <SdFat.h>

#define SD_CS_PIN PA4

SdFat sd;
File file;

void setup() {
  Serial.begin(115200);
  delay(3000);

  Serial.println("STM32F405 USB Serial OK");
  Serial.println("Initializing SD card...");

  pinMode(SD_CS_PIN, OUTPUT);
  digitalWrite(SD_CS_PIN, HIGH);

  SPI.begin();

  if (!sd.begin(SD_CS_PIN, SD_SCK_MHZ(1))) {
    Serial.println("❌ SD init failed");
    sd.initErrorHalt(&Serial);
  }

  Serial.println("✅ SD init OK");

  // ---------------- WRITE ----------------
  Serial.println("Writing to file...");

  file = sd.open("test.txt", FILE_WRITE);
  if (!file) {
    Serial.println("❌ File open for write failed");
    return;
  }

  file.println("Hello from STM32F405!");
  file.println("SD card write + read test");
  file.println("HP 64GB SDXC card");
  file.close();

  Serial.println("✅ Write complete");

  // ---------------- READ ----------------
  Serial.println("Reading file back:");

  file = sd.open("test.txt", FILE_READ);
  if (!file) {
    Serial.println("❌ File open for read failed");
    return;
  }

  while (file.available()) {
    Serial.write(file.read());
  }

  file.close();
  Serial.println("\n✅ Read complete");
}

void loop() {
  // nothing here
}
```

### 4. Viewing Data via the CP2102 Serial Monitor
- Open the Arduino IDE **Serial Monitor** (`Ctrl + Shift + M`).
- Set the baud rate dropdown to `115200` to match `Serial.begin(115200)` in the code.
- Press the physical **RESET** button on the Orbit-Q board to restart the MCU. You should see the initialization text, write confirmations, and the contents of `test.txt` printed directly on the screen.

5. Troubleshooting
!!! failure "SD Initialization Failed or Serial Monitor is Blank"
- **Check JP-5 & JP-3:**Make sure the JP-5 jumpers are set correctly for SPI communication and JP-3 is configured for the CP2102 UART bridge. If JP-3 is open, no text will reach your Serial Monitor.
- **Verify COM Port:** Ensure you have selected the correct COM port associated with the CP2102 USB-to-UART bridge (not the DFU or native USB-OTG port) under Tools > Port.
- **SD Card Formatting:** Ensure your MicroSD card is formatted to FAT16 or FAT32 (exFAT is typically not supported by standard microcontroller FAT libraries).
- **Baud Rate Mismatch:** If the Serial Monitor prints gibberish characters, verify that your monitor baud rate is precisely set to `115200`.
