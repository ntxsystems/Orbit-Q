# HMC5883L Magnetometer Interfacing with ORBIT-Q STM32F405

## 1. Project Overview

This project demonstrates how to interface a HMC5883L-compatible digital magnetometer module with the ORBIT-Q STM32F405RGTX development board using Arduino IDE.

The magnetometer measures the Earth's magnetic field along three axes: X, Y and Z. These measurements are used to determine the magnetic field strength and calculate the heading direction in degrees.

The sensor communicates with the STM32F405 microcontroller through the I²C2 interface. An SSD1306 OLED display connected through I²C1 displays the magnetic field measurements and calculated heading in real time.

**Note:** The physical breakout board used in this project is labelled HMC5883L, but the working implementation uses the QMC5883P-compatible register configuration.

## 2. Objectives

- Interface the HMC5883L-compatible magnetometer with the ORBIT-Q STM32F405 development board.
- Establish I²C communication between the sensor and microcontroller.
- Measure magnetic field values along X, Y and Z axes.
- Calculate magnetic heading in degrees.
- Display magnetic field measurements on an SSD1306 OLED display.
- Understand practical magnetometer interfacing using STM32.
- Gain experience with digital compass applications.

## 3. Hardware and Software Requirements

### 3.1 Hardware Requirements

| Component | Description |
|---|---|
| Development Board | ORBIT-Q STM32F405RGTX |
| Sensor | HMC5883L magnetometer |
| Connecting Wires | Jumper wires |
| Power Supply | External power adapter |
| Programming Connection | USB cable |

### 3.2 Software Requirements

- Arduino IDE
- STM32 board support package

### 3.3 Install Libraries on Arduino-IDE

- Adafruit GFX Library
- Adafruit SSD1306 Library

## 4. Hardware Connections

The project uses two independent I²C communication interfaces.

- I²C2 is used for the magnetometer.
- I²C1 is used for the SSD1306 OLED display.

### 4.1 Magnetometer Connections

| Sensor Pin | ORBIT-Q STM32F405 Pin |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SCL | PB10 (Pin 17) |
| SDA | PB11 (Pin 16) |

### 4.2 Interface Summary

| Device | Interface | SDA | SCL |
|---|---|---|---|
| Magnetometer | I²C2 | PB11 | PB10 |

Both I²C interfaces operate at 400 kHz.

**Note:** SPI communication is not used in this project.

## 5. Magnetometer Working Principle

A digital magnetometer measures the magnetic field surrounding it.

The magnetometer used in this project provides three-axis magnetic field measurements.

### 5.1 X-Axis Measurement

Measures the magnetic field component along the X-axis.

### 5.2 Y-Axis Measurement

Measures the magnetic field component along the Y-axis.

### 5.3 Z-Axis Measurement

Measures the magnetic field component along the Z-axis.

### 5.4 Magnetic Heading

The X and Y magnetic field measurements are used to calculate the heading direction.

The heading is displayed in degrees, with values normalized between 0° and 360°.

The project uses a magnetic declination setting of 0°.

### 5.5 Magnetic Field Units

The magnetic field measurements are displayed in microtesla (µT).

The OLED displays all three axis measurements along with the calculated heading.

## 6. Project Setup and Implementation

The project is implemented using Arduino IDE and the ORBIT-Q STM32F405RGTX development board.

The hardware setup involves:

1. Connecting the magnetometer to the I²C2 interface.
2. Providing power to the development board using an external power adapter.
3. Uploading the Arduino sketch through the appropriate USB programming connection.
4. Observing the magnetic field measurements and heading on the OLED display.

### Hardware Configuration

| Parameter | Configuration |
|---|---|
| Sensor I²C Address | 0x2C |
| Chip Identification | 0x80 |
| Magnetic Field Range | ±8 Gauss |
| Operating Mode | Continuous |
| Output Data Rate | 200 Hz |
| Oversampling Ratio | 8 |
| Downsampling Ratio | 8 |
| OLED Resolution | 128 × 32 |
| OLED Address | 0x3C |
| Magnetic Declination | 0° |

**Hardware identification note:** The physical breakout board carries an HMC5883L marking. The uploaded working sketch uses the QMC5883P-compatible address, identification value and register configuration. The exact chip fitted to the breakout should be confirmed from its component marking or manufacturer documentation.

## 7. Uploading the Program

Follow these steps to upload the program:

1. Open Arduino IDE.
2. Install the required Adafruit libraries.
3. Open the HMC5883L`.ino` file.
4. Select the appropriate STM32F405 board configuration.
5. Select the appropriate upload method.
6. Connect the ORBIT-Q development board to the computer using USB.
7. Compile and upload the sketch.
8. Power the board using the external power adapter.
9. Observe the OLED display for live magnetic field measurements and heading.

## 8. Arduino-IDE Sketch

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH    128
#define SCREEN_HEIGHT   32
#define OLED_ADDR       0x3C
#define MAG_ADDR        0x2C
#define DECLINATION_DEG 0.0f

// QMC5883P registers
#define REG_CHIPID  0x00      // expect 0x80
#define REG_DATA    0x01      // X LSB, X MSB, Y LSB, Y MSB, Z LSB, Z MSB
#define REG_STATUS  0x09      // bit0 = DRDY
#define REG_CTRL1   0x0A
#define REG_CTRL2   0x0B

#define LSB_PER_GAUSS 3750.0f // +/-8 G range

TwoWire Wire2(PB11, PB10);    // SDA, SCL (I2C2)
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

bool writeReg(uint8_t reg, uint8_t val) {
  Wire2.beginTransmission(MAG_ADDR);
  Wire2.write(reg);
  Wire2.write(val);
  return Wire2.endTransmission() == 0;
}

bool readRegs(uint8_t reg, uint8_t *buf, uint8_t len) {
  Wire2.beginTransmission(MAG_ADDR);
  Wire2.write(reg);
  if (Wire2.endTransmission(false) != 0) return false;
  if (Wire2.requestFrom((uint8_t)MAG_ADDR, len) != len) return false;
  for (uint8_t i = 0; i < len; i++) buf[i] = Wire2.read();
  return true;
}

void showError(const __FlashStringHelper *l1, const __FlashStringHelper *l2) {
  display.clearDisplay();
  display.setCursor(0, 0);
  display.println(l1);
  display.println(l2);
  display.display();
  while (1);
}

void setup() {
  Wire.setSDA(PB7);
  Wire.setSCL(PB6);
  Wire.begin();
  Wire.setClock(400000);

  Wire2.begin();
  Wire2.setClock(400000);

  if (!display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDR)) {
    while (1);
  }
  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);
  display.setTextSize(1);

  uint8_t id;
  if (!readRegs(REG_CHIPID, &id, 1)) {
    showError(F("Mag read failed"), F("Check I2C2"));
  }
  if (id != 0x80) {
    display.clearDisplay();
    display.setCursor(0, 0);
    display.println(F("Unexpected chip ID"));
    display.print(F("ID = 0x")); display.println(id, HEX);
    display.display();
    while (1);
  }

  writeReg(REG_CTRL2, 0x08);   // range +/-8 G, set/reset on
  writeReg(REG_CTRL1, 0xCF);   // continuous mode, 200 Hz, OSR 8, downsample 8
  delay(10);
}

void loop() {
  uint8_t st;
  if (!readRegs(REG_STATUS, &st, 1)) {
    showError(F("Status read failed"), F("Check wiring"));
  }
  if (!(st & 0x01)) {          // no new data yet
    delay(2);
    return;
  }

  uint8_t b[6];
  if (!readRegs(REG_DATA, b, 6)) {
    showError(F("Data read failed"), F("Check wiring"));
  }

  int16_t rx = (int16_t)((b[1] << 8) | b[0]);
  int16_t ry = (int16_t)((b[3] << 8) | b[2]);
  int16_t rz = (int16_t)((b[5] << 8) | b[4]);

  float x = rx / LSB_PER_GAUSS * 100.0f;   // microtesla
  float y = ry / LSB_PER_GAUSS * 100.0f;
  float z = rz / LSB_PER_GAUSS * 100.0f;

  float heading = atan2f((float)ry, (float)rx) * 180.0f / PI + DECLINATION_DEG;
  if (heading < 0)    heading += 360.0f;
  if (heading >= 360) heading -= 360.0f;

  display.clearDisplay();
  display.setCursor(0, 0);
  display.print(F("X: ")); display.print(x, 1); display.println(F(" uT"));
  display.print(F("Y: ")); display.print(y, 1); display.println(F(" uT"));
  display.print(F("Z: ")); display.print(z, 1); display.println(F(" uT"));
  display.print(F("Hdg: ")); display.print(heading, 1); display.println(F(" deg"));
  display.display();

  delay(100);
}
```

## 9. Testing and Experimental Results

The magnetometer was tested using the ORBIT-Q STM32F405RGT6 development board.

After connecting the sensor and powering the board, the magnetometer successfully communicated with the microcontroller through the I²C2 interface.

The sensor provided magnetic field measurements along all three axes, which were displayed on the SSD1306 OLED.

### 9.1 Displayed Parameters

| Parameter | Unit | Description |
|---|---|---|
| X-Axis | µT | Magnetic field along X-axis |
| Y-Axis | µT | Magnetic field along Y-axis |
| Z-Axis | µT | Magnetic field along Z-axis |
| Heading | Degrees (°) | Calculated magnetic direction |

### 9.2 Testing Results

| Test | Expected Result | Status |
|---|---|---|
| I²C1 Initialization | OLED communication established | Passed |
| I²C2 Initialization | Magnetometer communication established | Passed |
| Sensor Identification | Expected chip ID detected | Passed |
| X-Axis Measurement | Magnetic field displayed | Passed |
| Y-Axis Measurement | Magnetic field displayed | Passed |
| Z-Axis Measurement | Magnetic field displayed | Passed |
| Heading Calculation | Heading displayed in degrees | Passed |
| OLED Display | Measurements visible on display | Passed |

### Result

The magnetometer was successfully interfaced with the ORBIT-Q STM32F405 development board. Magnetic field measurements along the X, Y and Z axes and the calculated heading were displayed on the OLED screen.

## 10. Troubleshooting

| Problem | Possible Solution |
|---|---|
| Magnetometer not detected | Check VCC, GND, SDA and SCL connections. |
| I²C communication failure | Verify PB10 and PB11 pin assignments. |
| Unexpected chip ID | Verify the actual sensor chip and compatible register configuration. |
| Incorrect magnetic field readings | Check sensor orientation and nearby magnetic interference. |
| Unstable heading | Keep the sensor away from motors, magnets and current-carrying wires. |
| OLED not displaying output | Check the power of the board or turn on/off the power of board. |
| Compilation errors | Check the STM32 board configuration and installed libraries. |

**Important:** Magnetic readings can be affected by nearby metal objects, magnets, motors and electrical equipment. Test the sensor away from these sources for more reliable measurements.

## 11. Applications

- Digital compass systems.
- Navigation devices.
- Robotics and autonomous systems.
- Drone and UAV navigation.
- Electronic orientation detection.
- Magnetic field monitoring.
- Embedded navigation systems.
- IoT-based sensing applications.

## 12. Future Improvements

- Magnetic sensor calibration.
- Hard-iron and soft-iron error compensation.
- Tilt-compensated compass functionality.
- Integration with accelerometer and gyroscope sensors.
- Wireless transmission of magnetic field measurements.
- Data logging and graphical visualization.
- Integration into autonomous robotic navigation systems.

## 13. Conclusion

The magnetometer module was successfully interfaced with the ORBIT-Q STM32F405RGT6 development board using Arduino IDE.

The project demonstrates practical implementation of I²C communication, three-axis magnetic field measurement, heading calculation and OLED display integration.

The successful implementation provides practical experience in magnetometer interfacing and embedded navigation applications.

This project can serve as a foundation for developing digital compasses, robotic navigation systems and other magnetic sensing applications.



