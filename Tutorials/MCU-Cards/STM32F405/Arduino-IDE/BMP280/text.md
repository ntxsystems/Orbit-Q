# BMP280 Temperature and Pressure Monitoring Using ORBIT-Q STM32F405

## 1. Project Overview

This project demonstrates interfacing a BMP280 digital temperature and barometric pressure sensor with the ORBIT-Q STM32F405RGT6 development board.

The BMP280 measures temperature and atmospheric pressure. The system also calculates estimated altitude using atmospheric pressure.

The sensor communicates with the microcontroller through I²C2, while an SSD1306 OLED display uses I²C1 to display the measurements in real time.

The BMP280 module has six pins, but only four connections are used in this project for I²C communication.

## 2. Objectives

- Interface the BMP280 sensor with the ORBIT-Q STM32F405 development board.
- Establish I²C communication between the sensor and microcontroller.
- Measure temperature and atmospheric pressure.
- Calculate estimated altitude.
- Display sensor readings on an OLED display.
- Understand practical sensor interfacing using STM32.

## 3. Hardware and Software Requirements

### Hardware Requirements

| Component | Description |
|---|---|
| Development Board | ORBIT-Q STM32F405RGT6 |
| Sensor | BMP280 six-pin module |
| Display | SSD1306 OLED, 128 × 32 pixels |
| Connecting Wires | Jumper wires |
| Power Supply | External power adapter |
| Programming Connection | USB cable |

### Software Requirements

- Arduino IDE
- STM32 board support package

### Install Libraries in Arduino-IDE

- Adafruit BMP280 Library
- Adafruit GFX Library
- Adafruit SSD1306 Library

## 4. Hardware Connections

The project uses two independent I²C interfaces.

- I²C2 for BMP280.

### 4.1 BMP280 Sensor Connections

| BMP280 Pin | ORBIT-Q STM32F405 Pin |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SCL | PB10 (Pin 17) |
| SDA | PB11 (Pin 16) |

**Note:** Although the BMP280 module has six pins, only four connections are used in this project.

### 4.3 Interface Summary

| Device | Interface | SDA | SCL |
|---|---|---|---|
| BMP280 | I²C2 | PB11 | PB10 |
| OLED | I²C1 | PB7 | PB6 |

Both I²C interfaces operate at 400 kHz. 

## 5. Working Principle

The BMP280 is a digital environmental sensor manufactured by Bosch Sensortec.

It measures two primary parameters:

**Temperature:** Measures the surrounding temperature in degrees Celsius (°C).

**Atmospheric Pressure:** Measures atmospheric pressure in hectopascals (hPa).

**Estimated Altitude:** Uses atmospheric pressure and a reference sea-level pressure to estimate altitude in metres (m).

The measured values are displayed on the SSD1306 OLED screen.

The project uses a default sea-level pressure reference of 1013.25 hPa. This value can be adjusted according to local sea-level pressure for improved altitude estimation.

## 6. Project Setup and Implementation

The project is implemented using Arduino IDE and the ORBIT-Q STM32F405RGT6 development board.

The hardware setup involves:

1. Connecting the BMP280 sensor to the I²C2 interface.
2. Connecting the OLED display to the I²C1 interface.
3. Providing power to the development board using an external power adapter.
4. Uploading the Arduino sketch through the appropriate USB programming connection.
5. Observing the sensor measurements on the OLED display.

## 7. Uploading the Program

1. Open Arduino IDE.
2. Install the required libraries.
3. Open the BMP280 project `.ino` file.
4. Select the appropriate STM32F405 board configuration.
5. Select the appropriate upload method.
6. Connect the ORBIT-Q board to the computer.
7. Compile and upload the program.
8. Power the board using the external power adapter.
9. Observe the OLED display for live measurements.

## 8. Arduino-IDE Sketch
```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <Adafruit_BMP280.h>

#define SCREEN_WIDTH  128
#define SCREEN_HEIGHT 32
#define OLED_ADDR     0x3C      // some 128x32 modules are 0x3D
#define SEA_LEVEL_HPA 1013.25   // set to local QNH for correct altitude

// I2C1 -> OLED (default Wire)
// I2C2 -> BMP280
TwoWire Wire2(PB11, PB10);      // SDA, SCL

Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);
Adafruit_BMP280  bmp(&Wire2);

void setup() {
  // I2C1 for OLED
  Wire.setSDA(PB7);
  Wire.setSCL(PB6);
  Wire.begin();
  Wire.setClock(400000);

  // I2C2 for BMP280
  Wire2.begin();
  Wire2.setClock(400000);

  if (!display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDR)) {
    while (1);  // OLED not found
  }
  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);
  display.setTextSize(1);
  display.setCursor(0, 0);

  // BMP280 address is 0x76 (SDO=GND) or 0x77 (SDO=VCC)
  bool ok = bmp.begin(0x76);
  if (!ok) ok = bmp.begin(0x77);

  if (!ok) {
    display.println(F("BMP280 not found"));
    display.println(F("Check I2C2 PB10/PB11"));
    display.display();
    while (1);
  }

  bmp.setSampling(Adafruit_BMP280::MODE_NORMAL,
                  Adafruit_BMP280::SAMPLING_X2,    // temperature
                  Adafruit_BMP280::SAMPLING_X16,   // pressure
                  Adafruit_BMP280::FILTER_X16,
                  Adafruit_BMP280::STANDBY_MS_500);
}

void loop() {
  float t = bmp.readTemperature();           // deg C
  float p = bmp.readPressure() / 100.0F;     // hPa
  float a = bmp.readAltitude(SEA_LEVEL_HPA); // m

  display.clearDisplay();
  display.setCursor(0, 0);
  display.print(F("Temp : ")); display.print(t, 2); display.println(F(" C"));
  display.print(F("Press: ")); display.print(p, 2); display.println(F(" hPa"));
  display.print(F("Alt  : ")); display.print(a, 1); display.println(F(" m"));
  display.display();

  delay(500);
}
```

## 9. Testing and Experimental Results

The BMP280 sensor was tested using the ORBIT-Q STM32F405RGT6 development board.

After connecting the sensor and powering the board, the sensor measurements were displayed on the SSD1306 OLED.

### 9.1 Observed Output

The demonstration setup showed approximately the following readings:

| Parameter | Observed Reading |
|---|---:|
| Temperature | 27.03 °C |
| Atmospheric Pressure | 1003.5 hPa |
| Estimated Altitude | 79.9 m |

*These readings are examples from one demonstration. Actual measurements may vary depending on environmental conditions and the configured sea-level pressure.*

### 9.2 Testing Results

| Test | Expected Result | Status |
|---|---|---|
| Sensor Detection | BMP280 detected successfully | Passed |
| I²C Communication | Sensor communicates with STM32F405 | Passed |
| Temperature Measurement | Temperature displayed in °C | Passed |
| Pressure Measurement | Pressure displayed in hPa | Passed |
| Altitude Estimation | Estimated altitude displayed in metres | Passed |
| OLED Display | Measurements visible on display | Passed |

### Result

The BMP280 sensor was successfully interfaced with the ORBIT-Q STM32F405 development board. Temperature, atmospheric pressure and estimated altitude were displayed on the OLED screen.

## 10 Hardware Setup and Demonstration

### 10.1 Complete Hardware Setup

Add the photograph showing the complete ORBIT-Q development board and connected BMP280 sensor.

**Image:** `images/bmp280-hardware-setup.jpg`

### 10.2 BMP280 Sensor Connections

Add a close-up photograph showing the four connected wires between the BMP280 module and development board.

**Image:** `images/bmp280-sensor-connections.jpg`

### 10.3 Live OLED Output

Add the photograph showing the sensor measurements on the OLED display.

**Image:** `images/bmp280-oled-output.jpg`

### 10.4 Working Demonstration Video

Add the recorded video demonstrating the working project.

**Video:** `Videos/bmp280-working-demo.mp4`

## 11. Troubleshooting

| Problem | Possible Solution |
|---|---|
| BMP280 not detected | Check VCC, GND, SDA and SCL connections. |
| I²C communication failure | Verify PB10 and PB11 pin assignments. |
| Sensor not detected at 0x76 | Check whether the module uses address 0x77. |
| OLED not displaying output | Check OLED power, connections and I²C address. |
| Incorrect temperature readings | Check sensor placement and power supply stability. |
| Incorrect altitude readings | Verify the configured sea-level pressure reference. |
| Compilation errors | Check the STM32 board configuration and installed libraries. |

**Important:** Verify the CSB and SDO pin configuration required for I²C operation when using a different BMP280 breakout module. Check supply voltage and logic-level compatibility before powering the hardware.

## 12. Applications

- Weather monitoring systems.
- Environmental monitoring devices.
- Altitude estimation systems.
- Indoor navigation systems.
- Drone and UAV applications.
- Portable pressure measurement devices.
- IoT-based environmental monitoring.

## 13. Future Improvements

- Wireless sensor data transmission using Wi-Fi or Bluetooth.
- Cloud-based environmental monitoring.
- Data logging of temperature and pressure.
- Graphical representation of sensor readings.
- Integration with additional environmental sensors.
- Development of a portable weather monitoring device.

## 14. Conclusion

The BMP280 sensor was successfully interfaced with the ORBIT-Q STM32F405RGT6 development board using Arduino IDE.

The project demonstrates practical implementation of I²C communication, temperature measurement, atmospheric pressure measurement, altitude estimation and OLED display integration.

This project provides a foundation for developing more advanced environmental monitoring and IoT-based applications.

