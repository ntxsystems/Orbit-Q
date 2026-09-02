# Orbit-Q: Controlling Onboard WS2812B RGB LEDs

This tutorial covers how to control the onboard array of **10 WS2812B addressable RGB LEDs** using the **Orbit-Q STM32F405** card. We will use the Arduino IDE and the widely supported Adafruit NeoPixel library to send data out via the designated GPIO pin.

---

## 1. Hardware Connections & Jumper Configuration

According to the "Orbit-Q Datasheet.pdf", JP-1 is a 2x2 configuration header used to connect the LED array to the microcontroller and the onboard power supply[cite: 1]. Two jumper caps are required on this header for normal operation[cite: 1]:

*   **Data Jumper:** Connect the **GPIO** pin to the **DI** (Data-In) pin[cite: 1]. The LED array receives serial data from the microcontroller through this GPIO-to-DI connection established by JP1[cite: 1].
*   **Power Jumper:** Connect the **5V IN** pin to the **LED 5V** pin[cite: 1]. Power is supplied from the onboard 5 V regulator through the 5V IN-to-LED 5V connection[cite: 1].
*   Both connections must be present for normal operation[cite: 1].

!!! warning "Check Your Jumpers"
    If the JP-1 jumpers are missing, the LEDs will not receive power from the board's 5V rail or data from the STM32F405, and your code will appear to do nothing.

---

## 2. Installing the Library

We will use the **Adafruit NeoPixel** library to handle the strict timing requirements of the WS2812B protocol.

1. Open the Arduino IDE.
2. Navigate to **Sketch > Include Library > Manage Libraries** (or press `Ctrl + Shift + I`).
3. In the Library Manager search bar, type `Adafruit NeoPixel`.
4. Locate the library by Adafruit and click **Install**.

---

## 3. Arduino Sketch

Copy and paste the following code into your Arduino IDE. This sketch initializes the 10 onboard LEDs and cycles them through basic primary colors in a "wipe" animation.

```cpp
#include <Adafruit_NeoPixel.h>

// Define the STM32 pin connected to the WS2812B data input
#define LED_PIN    PB3

// Define the number of LEDs in the chain
#define LED_COUNT  10

// Initialize the NeoPixel object
// NEO_GRB + NEO_KHZ800 is the standard configuration for WS2812B LEDs
Adafruit_NeoPixel strip(LED_COUNT, LED_PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  // Initialize the NeoPixel library
  strip.begin();
  
  // Set brightness to ~20% (Value from 0 to 255) to limit current draw
  strip.setBrightness(50); 
  
  // Update the strip to ensure all LEDs are turned off on startup
  strip.show(); 
}

void loop() {
  // Wipe Red
  colorWipe(strip.Color(255, 0, 0), 50);
  
  // Wipe Green
  colorWipe(strip.Color(0, 255, 0), 50);
  
  // Wipe Blue
  colorWipe(strip.Color(0, 0, 255), 50);
  
  // Wipe Off (Black)
  colorWipe(strip.Color(0, 0, 0), 50);
}

// Function to fill the dots one after the other with a color
void colorWipe(uint32_t color, int wait) {
  for(int i = 0; i < strip.numPixels(); i++) { 
    strip.setPixelColor(i, color);         // Set pixel 'i' to the specified color
    strip.show();                          // Push the updated color to the hardware
    delay(wait);                           // Pause for a moment
  }
}
```

## 4. Code Explanation

- `Adafruit_NeoPixel strip(...)`: This creates the software object representing your LED strip. We pass it `LED_COUNT` (10) and `LED_PIN` (`PB3`).
- `strip.begin()`: Allocates memory and sets up the STM32 GPIO pin for output.
- `strip.setBrightness(50)`: Caps the maximum brightness (0-255).
- `strip.Color(R, G, B)`: Packs Red, Green, and Blue values (0-255 each) into a single 32-bit variable that the library understands.
- `strip.setPixelColor(index, color)`: Updates the color of a specific LED in the MCU's RAM. `index` starts at `0` (so the 10th LED is index `9`).
- `strip.show()`: The STM32 sends the stored RAM data out to physically update the LEDs. You must call this for changes to become visible.

## 5. Uploading to the Orbit-Q

- Put the Orbit-Q into bootloader mode (Slide Boot Switch to `BOOT / ON`and press `RESET`).
- Select your COM port or USB DFU interface.
- Ensure the board is set to Generic STM32F4 series with part number STM32F405RGTx.
- Click Upload.
- Once complete, switch the Boot Slide Switch back to `NORMAL / OFF` and press `RESET` to run your LED code.

### Troubleshooting
??? bug "LEDs are not turning on or acting erratically"
* Check JP-1 Jumpers: Ensure both the GPIO-to-DI and 5V IN-to-LED 5V jumper caps are securely seated on the JP-1 header.
* Power Issues: If the LEDs turn red or the STM32 resets randomly, the LEDs are drawing too much current from your USB port. Lower strip.setBrightness() in the code to 10 or 20.


