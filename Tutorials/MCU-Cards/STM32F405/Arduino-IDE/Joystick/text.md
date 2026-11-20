# Joystick Interfacing with ORBIT-Q STM32F405

## 1. Project Overview

This project demonstrates how to interface a two-axis analog joystick module with the ORBIT-Q STM32F405RGT6 development board using Arduino IDE.

The joystick module provides two analog outputs corresponding to horizontal (X-axis) and vertical (Y-axis) movement. It also includes a push-button switch that can be activated by pressing the joystick.

The STM32F405 reads the analog signals and digital button state. The joystick position is represented by a moving indicator on an SSD1306 OLED display, along with the corresponding X-axis and Y-axis values.

The project also includes automatic calibration, a dead zone to reduce unwanted movement near the center position, and an OLED-based visual interface.

## 2. Objectives

- Interface a two-axis analog joystick with the ORBIT-Q STM32F405 development board.
- Read horizontal and vertical analog inputs.
- Detect joystick push-button operations.
- Display joystick position on an OLED display.
- Implement automatic joystick calibration.
- Reduce unwanted movement using a dead zone.
- Gain practical experience with analog and digital GPIO interfacing using STM32.

## 3. Hardware and Software Requirements

### 3.1 Hardware Requirements

| Component | Description |
|---|---|
| Development Board | ORBIT-Q STM32F405RGT6 |
| Input Module | Two-axis analog joystick module |
| Connecting Wires | Jumper wires |
| Breadboard | Mini breadboard |
| Power Supply | External power adapter |
| Programming Connection | USB cable |

### 3.2 Software Requirements

- Arduino IDE
- STM32 board support package

### 3.3 Install Libraries on Arduino-IDE

- Adafruit GFX Library
- Adafruit SSD1306 Library

## 4. Hardware Connections

The joystick module provides five connections:

- GND – Ground
- VCC – Power supply
- VRx – Horizontal analog output
- VRy – Vertical analog output
- SW – Push-button output

The project uses two analog input pins and one digital input pin on the STM32F405.

### 4.1 Joystick Connections

| Joystick Pin | ORBIT-Q STM32F405 Pin | Function |
|---|---|---|
| GND | GND | Ground |
| VCC | 5V | Power supply |
| VRx | PA0 | X-axis analog input |
| VRy | PA1 | Y-axis analog input |
| SW | PC3 | Digital push-button input |

### 4.2 Interface Summary

| Device | Interface | STM32F405 Pins |
|---|---|---|
| Joystick X-axis | Analog Input | PA0 |
| Joystick Y-axis | Analog Input | PA1 |
| Joystick Button | Digital Input | PC3 |

## 5. Joystick Working Principle

A two-axis analog joystick is an input device that detects movement in horizontal and vertical directions.

It generally consists of two potentiometers and a push-button switch.

### 5.1 X-Axis Movement

The horizontal potentiometer generates an analog voltage according to the left and right movement of the joystick.

- Left movement produces a lower reading.
- Center position produces a middle reading.
- Right movement produces a higher reading.

### 5.2 Y-Axis Movement

The vertical potentiometer generates an analog voltage according to the forward and backward movement of the joystick.

- One direction produces a lower reading.
- Center position produces a middle reading.
- The opposite direction produces a higher reading.

The actual positive and negative directions depend on the physical orientation and wiring of the joystick.

### 5.3 Push-Button Operation

The joystick also includes a push-button switch.

When the joystick is pressed, the switch becomes active and the STM32 detects the button operation.

### 5.4 Analog-to-Digital Conversion

The STM32F405 uses a 12-bit ADC to convert analog joystick signals into digital values.

The ADC provides readings from:

- Minimum: 0
- Center: Approximately 2048
- Maximum: 4095

The readings are normalized into a range of -100 to +100 for convenient display.

### 5.5 Automatic Calibration

The project performs an initial calibration when powered on.

During startup:

1. The joystick is kept in its center position.
2. The system measures the initial center values.
3. The initial minimum and maximum ranges are established.
4. The joystick is moved to all four extremes once.
5. The system learns the actual movement range of both axes.

This helps accommodate variations in joystick modules.

### 5.6 Dead Zone

A dead zone is used to ignore small unwanted movements around the joystick's center position.

The current implementation uses a 5% dead zone.

This helps prevent small analog fluctuations from continuously moving the display indicator.

## 6. Project Setup and Implementation

The project is implemented using Arduino IDE and the ORBIT-Q STM32F405RGT6 development board.

The hardware setup involves:

1. Connecting the joystick's VCC and GND pins.
2. Connecting VRx to PA0.
3. Connecting VRy to PA1.
4. Connecting the push-button output to PC3.
5. Powering the development board using the external power adapter.
6. Uploading the Arduino sketch.
7. Moving the joystick and observing the OLED output.

## 7. Arduino-IDE Sketch

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH   128
#define SCREEN_HEIGHT  32
#define OLED_ADDR      0x3C

#define JOY_X_PIN      PA0     // VRx (analog)
#define JOY_Y_PIN      PA1     // VRy (analog)
#define JOY_SW_PIN     PC3     // SW  (digital, active low)

#define INVERT_X       false   // flip if the dot moves the wrong way
#define INVERT_Y       false
#define DEADZONE       5       // percent, ignores small drift around center
#define ADC_MAX        4095    // 12-bit
#define START_SPAN     300     // initial +/- range before you swing the stick

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

int centerX = ADC_MAX / 2;
int centerY = ADC_MAX / 2;
int minX, maxX, minY, maxY;

int readAvg(uint8_t pin) {
  long sum = 0;
  for (int i = 0; i < 32; i++) {
    sum += analogRead(pin);
    delay(2);
  }
  return sum / 32;
}

// Returns -100 .. +100, each side scaled by its own measured travel
int scaleAxis(int raw, int center, int mn, int mx) {
  long v;
  if (raw >= center) v = (long)(raw - center) * 100 / max(1, mx - center);
  else               v = (long)(raw - center) * 100 / max(1, center - mn);
  if (v > 100)  v = 100;
  if (v < -100) v = -100;
  if (abs(v) < DEADZONE) v = 0;
  return (int)v;
}

void setup() {
  Wire.setSDA(PB7);
  Wire.setSCL(PB6);
  Wire.begin();
  Wire.setClock(400000);

  analogReadResolution(12);
  pinMode(JOY_SW_PIN, INPUT_PULLUP);

  if (!display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDR)) {
    while (1);
  }
  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);
  display.setTextSize(1);
  display.setCursor(0, 0);
  display.println(F("Joystick calibrating"));
  display.println(F("Leave stick centered"));
  display.display();

  delay(500);
  centerX = readAvg(JOY_X_PIN);
  centerY = readAvg(JOY_Y_PIN);

  minX = centerX - START_SPAN;  maxX = centerX + START_SPAN;
  minY = centerY - START_SPAN;  maxY = centerY + START_SPAN;

  display.clearDisplay();
  display.setCursor(0, 0);
  display.println(F("Swing stick to all"));
  display.println(F("4 extremes once"));
  display.display();
  delay(1500);
}

void loop() {
  int rawX = analogRead(JOY_X_PIN);
  int rawY = analogRead(JOY_Y_PIN);
  bool pressed = (digitalRead(JOY_SW_PIN) == LOW);

  // Learn the real travel of each axis
  if (rawX < minX) minX = rawX;
  if (rawX > maxX) maxX = rawX;
  if (rawY < minY) minY = rawY;
  if (rawY > maxY) maxY = rawY;

  int x = scaleAxis(rawX, centerX, minX, maxX);
  int y = scaleAxis(rawY, centerY, minY, maxY);
  if (INVERT_X) x = -x;
  if (INVERT_Y) y = -y;

  // Dot position inside the 32x32 box (up on the screen = positive Y)
  int dotX = 16 + (x * 11) / 100;
  int dotY = 16 - (y * 11) / 100;

  display.clearDisplay();

  // Box with crosshair
  display.drawRect(0, 0, 32, 32, SSD1306_WHITE);
  display.drawFastHLine(0, 16, 32, SSD1306_WHITE);
  display.drawFastVLine(16, 0, 32, SSD1306_WHITE);
  if (pressed) display.fillCircle(dotX, dotY, 3, SSD1306_WHITE);
  else         display.drawCircle(dotX, dotY, 3, SSD1306_WHITE);

  // Numbers
  display.setCursor(40, 0);
  display.print(F("X: ")); display.println(x);
  display.setCursor(40, 8);
  display.print(F("Y: ")); display.println(y);
  display.setCursor(40, 16);
  display.print(rawX); display.print(','); display.println(rawY);
  display.setCursor(40, 24);
  display.println(pressed ? F("SW: PRESSED") : F("SW: -"));

  display.display();
  delay(20);
}
```

### Display Features

The OLED interface provides the following information:

- A rectangular movement area with a crosshair.
- A circular indicator representing the joystick position.
- Normalized X-axis values.
- Normalized Y-axis values.
- Raw ADC readings for both axes.
- Push-button status.

When the joystick is pressed, the circular indicator changes from an outline to a filled circle.

## 8. Uploading the Program

Follow these steps to upload the program:

1. Open Arduino IDE.
2. Install the required Adafruit libraries.
3. Open `Joystick-stm32f405-orbit-q.ino`.
4. Select the appropriate STM32F405 board configuration.
5. Select the appropriate upload method.
6. Connect the ORBIT-Q development board to the computer using USB.
7. Compile and upload the sketch.
8. Power the board using the external power adapter.
9. Keep the joystick centered during initial calibration.
10. Move the joystick to all four extremes once.
11. Observe the movement indicator and numerical readings on the OLED.

## 9. Testing and Experimental Results

The joystick module was tested using the ORBIT-Q STM32F405RGT6 development board.

The system successfully reads the joystick's horizontal and vertical movements and displays the corresponding position on the SSD1306 OLED.

The push-button operation is also detected and represented through a change in the display indicator.

### 9.1 Testing Results

| Test | Expected Result | Status |
|---|---|---|
| Analog Input – X-axis | Horizontal movement detected | Passed |
| Analog Input – Y-axis | Vertical movement detected | Passed |
| Center Calibration | Initial center position established | Passed |
| Range Calibration | Four movement extremes detected | Passed |
| Dead Zone | Small center fluctuations ignored | Passed |
| Push Button | Button press detected | Passed |
| OLED Display | Joystick position displayed | Passed |
| Position Indicator | Indicator moves with joystick | Passed |

### 9.2 Observed Output

The OLED displays:

| Parameter | Display Format |
|---|---|
| X-axis | -100 to +100 |
| Y-axis | -100 to +100 |
| Raw X | ADC reading (0–4095) |
| Raw Y | ADC reading (0–4095) |
| Switch | PRESSED / - |

The joystick position is represented visually using a circular indicator inside a 32 × 32 pixel movement area.

### Result

The joystick module was successfully interfaced with the ORBIT-Q STM32F405 development board. Horizontal movement, vertical movement and push-button operations were detected and displayed on the OLED screen..


## 10. Troubleshooting

| Problem | Possible Solution |
|---|---|
| Joystick readings not changing | Check VRx and VRy connections. |
| X-axis moving incorrectly | Check PA0 connection and axis orientation. |
| Y-axis moving incorrectly | Check PA1 connection and axis orientation. |
| Indicator not centered | Restart the board with the joystick centered. |
| Indicator moves without input | Check the dead zone and analog connections. |
| Button not detected | Check the SW connection to PC3 and ground. |
| OLED not displaying output | Check OLED power, I²C connections and address. |
| Compilation errors | Verify STM32 board support and installed libraries. |

## 11. Applications

- Robotics and robot control systems.
- Gaming controllers.
- RC vehicle control systems.
- Robotic arm movement.
- Menu navigation systems.
- Embedded user interfaces.
- Drone control interfaces.
- Human-machine interaction systems.

## 12. Future Improvements

- Wireless joystick control using Bluetooth or Wi-Fi.
- Integration with robotic motors and servos.
- Remote-controlled vehicle development.
- Multiple joystick support.
- Custom joystick calibration settings.
- Integration with robotics and IoT applications.
- Wireless control of embedded devices.

## 13. Conclusion

The two-axis analog joystick module was successfully interfaced with the ORBIT-Q STM32F405RGTX development board using Arduino IDE.

The project demonstrates practical implementation of analog input reading, digital button detection, ADC conversion, automatic calibration and OLED-based visualization.

The successful implementation provides practical experience in embedded systems development and human-machine interaction.

This project can serve as a foundation for developing robotic controllers, gaming interfaces and remote-control applications.


