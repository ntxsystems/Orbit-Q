# Interfacing the Onboard OLED Display (I2C)

The Orbit-Q features a built-in $128\times32$ monochrome OLED display. Because it is integrated directly into the docking station, you only need to bridge the I2C connections using the onboard jumpers and install the appropriate display libraries to start rendering text.

### 1. Hardware Setup & I2C1 Pin Mapping

In the Arduino IDE, the standard `Wire` library automatically defaults to the **I2C1** peripheral. On the STM32F405RGT6 microcontroller, this default hardware I2C interface is mapped to:
*   **PB6** (SCL - Serial Clock)
*   **PB7** (SDA - Serial Data)

To route these MCU pins to the onboard OLED display, you must configure the **JP-2** header block.

1.  **Bridge the I2C Lines:** Place two jumper caps on the **JP-2** configuration header. This physically connects the MCU's PB6 and PB7 pins to the OLED's SCL and SDA lines.
2.  **Verify Power:** Once the board is powered via USB or the main supply, verify that the **SL-3** LED indicator is illuminated. This confirms the OLED module is receiving 3.3V power.

### 2. Installing the Required Libraries

The screen requires specific drivers to interpret the commands sent over I2C. We will use Adafruit's standard libraries.

1. In the Arduino IDE, go to **Sketch > Include Library > Manage Libraries...** (or press `Ctrl+Shift+I`).
2. In the Library Manager search bar, type `Adafruit SSD1306`.
3. Locate the **Adafruit SSD1306** library in the list and click **Install**. 
4. A prompt will appear asking if you want to install missing dependencies (such as `Adafruit GFX Library` and `Adafruit BusIO`). Click **Install All**. 

### 3. Arduino IDE Sketch

This basic sketch initializes the display using the default I2C1 interface at address `0x3C` and prints a welcome message.

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 32 // OLED display height, in pixels

// Declaration for an SSD1306 display connected to I2C (SDA, SCL pins)
// The &Wire object automatically uses I2C1 (PB6=SCL, PB7=SDA) on the STM32F405
#define OLED_RESET     -1 // -1 indicates the display shares the MCU reset pin
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

void setup() {
  Serial.begin(115200);

  // SSD1306_SWITCHCAPVCC = generate display voltage from 3.3V internally
  // 0x3C is the standard I2C address for 128x32 OLEDs
  if(!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) { 
    Serial.println(F("SSD1306 allocation failed. Check JP-2 jumpers!"));
    for(;;); // Don't proceed, loop forever
  }

  // Clear the buffer
  display.clearDisplay();

  // Configure text properties
  display.setTextSize(1);              // Normal 1:1 pixel scale
  display.setTextColor(SSD1306_WHITE); // Draw white text
  display.setCursor(0,0);              // Start at top-left corner
  
  // Print message to the buffer
  display.println(F("Hello, Orbit-Q!"));
  display.println(F("I2C1 is online."));

  // Push the buffer to the physical screen
  display.display();
}

void loop() {
  // Static display test - nothing to loop
}
```

!!! tip "Run the Built-in Adafruit Example"
    You can also test the screen's graphics capabilities by opening the built-in library example: navigate to **File > Examples > Adafruit SSD1306 > ssd1306_128x32_i2c**. Because the Orbit-Q uses the default I2C pins and display address (`0x3C`), this example sketch works perfectly out-of-the-box without any code modifications!

### Troubleshooting
!!! failure "OLED Screen Remains Blank"
- Check JP-2: Ensure the jumper caps are securely placed on both the SCL and SDA pins of the JP-2 header. Without these, PB6 and PB7 are not connected to the screen.
- Verify I2C Address: The Orbit-Q OLED uses address 0x3C. If you accidentally modify the code to use 0x3D (which is common for larger $128\times64$ displays), the initialization will fail.
- Power Check: Ensure the SL-3 LED is on. If it's off, check your board's main power configuration (JP-6).
