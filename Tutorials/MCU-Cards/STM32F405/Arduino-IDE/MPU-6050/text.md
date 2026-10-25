# MPU-6050 Sensor Data Filtering Using a Kalman Filter on STM32F405

## 1. Project Overview

This project demonstrates how to read accelerometer and gyroscope measurements from an MPU-6050 sensor using an STM32F405 microcontroller and apply independent one-dimensional Kalman filters to smooth the six sensor readings.

The filtered measurements are displayed on an SSD1306 OLED and printed to the Serial Monitor.

The program preserves the original sensor output format. It does not calculate roll, pitch, or yaw.

## 2. Objectives

- Interface the MPU-6050 with an STM32F405 microcontroller using I²C.
- Read three-axis accelerometer and gyroscope measurements.
- Convert raw sensor data into physical units.
- Apply independent Kalman filters to all six measurement channels.
- Display filtered values on an OLED.
- Print filtered measurements to the Serial Monitor.
- Use an onboard LED as a heartbeat indicator.

## 3. Hardware Requirements

| Component | Purpose |
|---|---|
| STM32F405 development board | Main controller |
| MPU-6050 / GY-521 module | Accelerometer and gyroscope |
| SSD1306 128 × 32 OLED | Sensor data display |
| USB cable and compatible programmer | Programming and debugging |
| Jumper wires | Connections, if required |

**Important:** Confirm the sensor module's supply requirements and I²C voltage compatibility before connecting it.

## 4. Software Requirements

- Arduino IDE
- STM32 board support compatible with the target board
- Adafruit GFX Library
- Adafruit SSD1306 Library
- USB upload/debug setup supported by the development board

The sketch uses the STM32-style pin names `PB11`, `PB10`, `PB7`, `PB6`, and `PC13`. Select a board definition that supports these names and the target STM32F405.

## 5. Hardware Connections

### MPU-6050 to STM32F405

| MPU-6050 pin | STM32F405 connection |
|---|---|
| VCC | Verified compatible supply or 5V at PWR-1|
| GND | GND at PWR-1|
| SDA | PB11 //pin 16|
| SCL | PB10 //pin 17|
| AD0 | Not connected |
| XDA | Not connected |
| XCL | Not connected |
| INT | Not required by this sketch |

### OLED to STM32F405

| OLED signal | STM32F405 connection |
|---|---|
| VCC | Compatible supply for the module |
| GND | GND |
| SDA | PB7 |
| SCL | PB6 |

Check the development board's pinout before wiring. The table describes the connections expected by this program.

## 6. Arduino IDE Setting
1. click tool>Board>STM32 MCU based board>Generic STM32F4 Series
2. Click tool>Board Part Number>Generic STM32F405RGTx

## 7. Arduino IDE Sketch
The OLED uses I²C address `0x3C`, while the MPU-6050 uses I²C address `0x68`. The sketch configures these devices on separate I²C interfaces.
```cpp

#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

#define SCREEN_WIDTH  128
#define SCREEN_HEIGHT 32
#define OLED_RESET    -1
#define OLED_ADDRESS  0x3C

#define MPU_ADDR      0x68
#define WHO_AM_I      0x75
#define PWR_MGMT_1    0x6B
#define ACCEL_CONFIG  0x1C
#define GYRO_CONFIG   0x1B
#define ACCEL_XOUT_H  0x3B

#define STATUS_LED    PC13

TwoWire Wire2(PB11, PB10);
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, OLED_RESET);

bool ledState = false;

// Independent 1D Kalman filter for smoothing each sensor axis.
// This does not calculate roll/pitch or fuse sensor types.
class Kalman1D {
public:
  Kalman1D(float processNoise, float measurementNoise)
    : q(processNoise), r(measurementNoise),
      estimate(0.0f), estimateError(1.0f), initialized(false) {}

  float update(float measurement) {
    if (!initialized) {
      estimate = measurement;
      initialized = true;
      return estimate;
    }
    estimateError += q;
    float kalmanGain = estimateError / (estimateError + r);
    estimate += kalmanGain * (measurement - estimate);
    estimateError *= (1.0f - kalmanGain);
    return estimate;
  }

private:
  float q;
  float r;
  float estimate;
  float estimateError;
  bool initialized;
};

// Starting tuning values; each axis has its own filter.
Kalman1D filterAx(0.08f, 1.0f);
Kalman1D filterAy(0.08f, 1.0f);
Kalman1D filterAz(0.08f, 1.0f);
Kalman1D filterGx(0.5f, 4.0f);
Kalman1D filterGy(0.5f, 4.0f);
Kalman1D filterGz(0.5f, 4.0f);

bool writeRegister(uint8_t reg, uint8_t value) {
  Wire2.beginTransmission(MPU_ADDR);
  Wire2.write(reg);
  Wire2.write(value);
  return Wire2.endTransmission() == 0;
}

bool readRegisters(uint8_t reg, uint8_t *data, uint8_t length) {
  Wire2.beginTransmission(MPU_ADDR);
  Wire2.write(reg);
  if (Wire2.endTransmission(false) != 0) return false;

  uint8_t received = Wire2.requestFrom(MPU_ADDR, length);
  if (received != length) {
    while (Wire2.available()) Wire2.read();
    return false;
  }
  for (uint8_t i = 0; i < length; i++) data[i] = Wire2.read();
  return true;
}

int16_t readInt16(uint8_t highByte, uint8_t lowByte) {
  return (int16_t)((uint16_t)highByte << 8 | lowByte);
}

void showError(const char *message) {
  display.clearDisplay();
  display.setCursor(0, 0);
  display.println("MPU-6050 ERROR");
  display.println(message);
  display.display();
}

void setup() {
  Serial.begin(115200);
  pinMode(STATUS_LED, OUTPUT);
  digitalWrite(STATUS_LED, HIGH); // Active-low LED OFF

  Wire.setSCL(PB6);
  Wire.setSDA(PB7);
  Wire.begin();

  display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDRESS);
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.println("Starting MPU-6050");
  display.display();

  Wire2.begin();
  Wire2.setClock(100000);

  uint8_t identity;
  if (!readRegisters(WHO_AM_I, &identity, 1) || identity != 0x68) {
    showError("Check sensor/I2C");
    while (true) delay(500);
  }
  if (!writeRegister(PWR_MGMT_1, 0x00)) {
    showError("Wake-up failed");
    while (true) delay(500);
  }

  writeRegister(ACCEL_CONFIG, 0x00); // +/- 2g
  writeRegister(GYRO_CONFIG, 0x00);  // +/- 250 degrees/second
  delay(100);
}

void loop() {
  uint8_t data[14];
  if (!readRegisters(ACCEL_XOUT_H, data, 14)) {
    showError("I2C read failed");
    delay(500);
    return;
  }

  int16_t rawAx = readInt16(data[0], data[1]);
  int16_t rawAy = readInt16(data[2], data[3]);
  int16_t rawAz = readInt16(data[4], data[5]);
  int16_t rawGx = readInt16(data[8], data[9]);
  int16_t rawGy = readInt16(data[10], data[11]);
  int16_t rawGz = readInt16(data[12], data[13]);

  float ax = rawAx / 16384.0f * 9.80665f;
  float ay = rawAy / 16384.0f * 9.80665f;
  float az = rawAz / 16384.0f * 9.80665f;
  float gx = rawGx / 131.0f;
  float gy = rawGy / 131.0f;
  float gz = rawGz / 131.0f;

  // Apply Kalman filters only; preserve output names, units and format.
  ax = filterAx.update(ax);
  ay = filterAy.update(ay);
  az = filterAz.update(az);
  gx = filterGx.update(gx);
  gy = filterGy.update(gy);
  gz = filterGz.update(gz);

  display.clearDisplay();
  display.setCursor(0, 0);
  display.print("Ax:"); display.print(ax, 1);
  display.print(" Ay:"); display.println(ay, 1);
  display.print("Az:"); display.println(az, 1);
  display.print("Gx:"); display.print(gx, 1);
  display.print(" Gy:"); display.println(gy, 1);
  display.print("Gz:"); display.println(gz, 1);
  display.display();

  ledState = !ledState;
  digitalWrite(STATUS_LED, ledState ? LOW : HIGH);

  Serial.print("Accel m/s2: ");
  Serial.print(ax); Serial.print(", ");
  Serial.print(ay); Serial.print(", ");
  Serial.println(az);

  Serial.print("Gyro deg/s: ");
  Serial.print(gx); Serial.print(", ");
  Serial.print(gy); Serial.print(", ");
  Serial.println(gz);

  delay(200);
}
```
Compile and Upload the code using usb-cable.

## 8. Uploading and Testing
1. Compile and upload the sketch.
2. Keep the sensor stationary and observe the values.
3. Slowly move the sensor and check whether the filtered measurements respond smoothly.

## 9. Sensor Configuration

The sketch uses the following MPU-6050 registers:

| Register | Address | Function |
|---|---:|---|
| `WHO_AM_I` | `0x75` | Checks the sensor identity |
| `PWR_MGMT_1` | `0x6B` | Wakes the sensor |
| `ACCEL_CONFIG` | `0x1C` | Configures the accelerometer range |
| `GYRO_CONFIG` | `0x1B` | Configures the gyroscope range |
| `ACCEL_XOUT_H` | `0x3B` | Starting register for sensor output data |

The selected measurement ranges are ±2 g for acceleration and ±250 degrees per second for angular velocity.

The program reads 14 consecutive bytes starting at `0x3B`. These include accelerometer, temperature, and gyroscope registers. The temperature measurement is read but not used in the displayed output.

## 10. Data Conversion

### Accelerometer

At the selected ±2 g range, the nominal sensitivity is 16,384 counts per g.

The program converts raw readings into metres per second squared:

`acceleration = raw_value / 16384.0 × 9.80665`

The resulting channels are `Ax`, `Ay`, and `Az`, measured in m/s².

### Gyroscope

At the selected ±250°/s range, the nominal sensitivity is 131 counts per degree per second.

The conversion is:

`angular_velocity = raw_value / 131.0`

The resulting channels are `Gx`, `Gy`, and `Gz`, measured in degrees per second.

These conversions depend on the configured sensor ranges.

## 11. Kalman Filter Implementation

The sketch applies six independent one-dimensional Kalman filters: one for each accelerometer and gyroscope axis.

Each filter uses two tuning parameters:

- **Process noise (`q`):** Controls how much the estimated value is allowed to change between measurements.
- **Measurement noise (`r`):** Controls how strongly the filter trusts each new measurement.

The implementation follows a scalar Kalman-filter update:

1. **Predict:** Increase the estimated uncertainty by the process noise.
2. **Calculate gain:** Determine the relative weight of the new measurement.
3. **Correct:** Move the estimate toward the measured value.
4. **Update uncertainty:** Reduce the estimated uncertainty after incorporating the measurement.

The initial parameters in the sketch are:

| Channels | Process noise (`q`) | Measurement noise (`r`) |
|---|---:|---:|
| Ax, Ay, Az | 0.08 | 1.0 |
| Gx, Gy, Gz | 0.5 | 4.0 |

These are starting values, not calibrated noise measurements. They may need adjustment for the sensor, mounting, and desired response speed.

This implementation smooths each axis independently. It does not perform sensor fusion or calculate orientation angles. A constant bias or drift may remain after filtering.

## 12. Program Execution Flow

1. Initialize the Serial Monitor at 115200 baud.
2. Configure the onboard status LED.
3. Initialize the OLED on the first I²C interface.
4. Initialize the MPU-6050 on the second I²C interface.
5. Check the sensor identity.
6. Wake the sensor and configure its measurement ranges.
7. Read the accelerometer and gyroscope registers.
8. Convert raw values into physical units.
9. Apply the six independent Kalman filters.
10. Display the filtered readings and print them to Serial.
11. Toggle the heartbeat LED.
12. Repeat after a 200 ms delay.

If the sensor identity check or a register read fails, the program displays an error message.


A stationary accelerometer normally measures gravity along the vertical axis, with the sign depending on sensor orientation. Gyroscope readings may show small offsets even when the board is stationary.

## 13. Expected Output

### Serial Monitor

The output format remains:

```text
Accel m/s2: Ax, Ay, Az
Gyro deg/s: Gx, Gy, Gz
```

The actual output consists of numeric filtered readings separated by commas.

### OLED

The existing labels are retained:

- Ax and Ay
- Az
- Gx and Gy
- Gz

The sketch preserves the original OLED drawing commands. Since the display is only 128 × 32 pixels, all six printed lines may not be visible simultaneously. A future display-layout improvement could address this without changing the sensor calculations.

## 12. Troubleshooting

| Problem | What to check |
|---|---|
| Sensor identity check fails | Verify power, GND, SDA/SCL wiring, address, and I²C interface |
| OLED remains blank | Check OLED power, address `0x3C`, wiring, and initialization |
| Compilation errors | Check STM32 board support and required libraries |
| No Serial output | Check the selected port and baud rate of 115200 |
| Readings fluctuate | Keep the sensor still, check wiring, and tune the filter parameters |
| Filter responds too slowly | Consider increasing process noise (`q`) or reducing measurement noise (`r`) |
| Readings show a persistent offset | Check sensor bias and calibration; filtering alone does not remove bias |


## 13. Conclusion

This project demonstrates MPU-6050 interfacing, I²C communication, physical-unit conversion, basic Kalman filtering, OLED display output, and serial debugging on an STM32F405 platform.

The main result is a version of the original sensor-reading program that retains the existing output format while applying independent filters to the six accelerometer and gyroscope measurements.

