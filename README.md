
[![Arduino CI](https://github.com/RobTillaart/Max44007/workflows/Arduino%20CI/badge.svg)](https://github.com/marketplace/actions/arduino_ci)
[![Arduino-lint](https://github.com/RobTillaart/Max44007/actions/workflows/arduino-lint.yml/badge.svg)](https://github.com/RobTillaart/Max44007/actions/workflows/arduino-lint.yml)
[![JSON check](https://github.com/RobTillaart/Max44007/actions/workflows/jsoncheck.yml/badge.svg)](https://github.com/RobTillaart/Max44007/actions/workflows/jsoncheck.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/RobTillaart/Max44007/blob/master/LICENSE)
[![GitHub release](https://img.shields.io/github/release/RobTillaart/Max44007.svg?maxAge=3600)](https://github.com/RobTillaart/Max44007/releases)


# Max44007 I2C LUX sensor

Library for Max44007 I2C lux sensor.


## Description

The Max44007 ambient light sensor is an I2C sensor, that has a 22 bit 
dynamic range from 0.025 lux to 104,448 lux.

This library is experimental and created by changing some key values in the
very similar MAX44009 library. As there was no hardware available the library 
is not tested yet. If you do, please share the results (on GitHub)


## Schema breakout Max44007


```cpp
//      breakout Max44007
//
//          +--------+
//      VCC |o       |
//      GND |o       |
//      SCL |o      o| ADDRESS
//      SDA |o      o| -INT
//          +--------+
//
//  ADDRESS:
//  0 = 0x5A
//  1 = 0x5B
//
//  INT:
//  Connect the INT pin to an pull up resistor
//  0 = interrupt
//  1 = no interrupt
//  connect to an LED or an interrupt pin of an Arduino
//
```


## Interface

- **enum class Boolean { True, False }** enum class to prevent bool to be implicitly casted to int.


### Constructor 
  
- **Max44007(uint8_t address, uint8_t dataPin, uint8_t clockPin)** Constructor with dataPin (sda) and clockPin (scl) for ESP32 and ESP8266.
- **Max44007(uint8_t address, begin = Boolean::True)** Constructor for other boards e.g. UNO.
- **Max44007(begin = Boolean::True)** Constructor with default I2C address 0x4A == 74.
- **void configure(uint8_t address, TwoWire \*wire, begin = Boolean::True)** Change I2C interface and address.
- **bool isConnected()** returns true if the device address configured is available on I2C bus.


### Basic 

- **float getLux()** read the sensor and return the value in LUX. If the value is negative, an error has occurred. 
- **int getError()** returns last error.

```cpp
//  ERROR CODES
#define Max44007_OK                     0
#define Max44007_ERROR_WIRE_REQUEST    -10
#define Max44007_ERROR_OVERFLOW        -20
#define Max44007_ERROR_HIGH_BYTE       -30
#define Max44007_ERROR_LOW_BYTE        -31
```


### Configure thresholds

check datasheet for details

- **bool setHighThreshold(float value)** sets the upper threshold for the interrupt 
generation (INT pulls LOW). Works only if INTE bit is set by **enableInterrupt()**. 
Function returns false if the value is out of range.
- **float getHighThreshold()** returns the value set.
- **bool setLowThreshold(float value)** sets the lower threshold for the interrupt 
generation (INT pulls LOW). Works only if INTE bit is set by **enableInterrupt()**. 
Function returns false if the value is out of range.
- **float getLowThreshold()** returns the value set.
- **void setThresholdTimer(uint8_t value)** Time the threshold needs to be exceeded, 
defined in steps of 100ms. 2 seems to be a practical minimum.
- **uint8_t getThresholdTimer()** returns the value set.


### Configure interrupts

check datasheet for details

- **void enableInterrupt()** enables the upper and lower threshold interrupts.
- **void disableInterrupt()** disables the threshold interrupts.
- **bool interruptEnabled()** returns 1 if the interrupt mode is enabled.
- **uint8_t getInterruptStatus()** returns 1 if an interrupt has occurred.


### Configure flags

check datasheet for details

- **void setConfiguration(uint8_t)** writes directly to configuration register. 
**warning** Use with care.
- **uint8_t getConfiguration()** reads the current configuration register.


### Configure sample mode

check datasheet for details

CCR = Current Divisor Ratio.

TIM = Integration time.

- **void setAutomaticMode()** in automatic mode the Max44007 determines the CDR and TIM 
parameters.
- **void setContinuousMode()** continuous mode uses more power than a "single" conversion. 
Advantage is that the latest data is always available fast.
- **void clrContinuousMode()** uses less power so better for LOW power configurations. 
- **void setManualMode(uint8_t CDR, uint8_t TIM)** Set the Current Divisor Ratio and the 
integration time manually. Effectively disable automatic mode.
- **int getIntegrationTime()** returns the set integration time in milliseconds

```
    CDR = Current Divisor Ratio
    CDR = 1 ==> only 1/8th is measured

    TIM = Time Integration Measurement (table)
    000   800ms
    001   400ms
    010   200ms
    011   100ms
    100    50ms       manual only
    101    25ms       manual only
    110    12.5ms     manual only
    111     6.25ms    manual only
```


### Test functions

Function for the conversion math, not meant to be used directly, 
but by making them public they become testable.

- **float convertToLux(uint8_t dataHigh, uint8_t dataLow)** convert intern register 
format to a LUX value. 


## Examples

**Max44007\examples\Max44007_test01**
- use for e.g. UNO (use voltage convertor!)

**Max44007\examples\Max44007_test02**
- will **not** compile for UNO
- use for e.g. ESP32


## Notes

Please be aware this is a **3.3 Volt device** so it should not be connected
to an Arduino or other 5 Volt device directly. Use a level convertor to 
solve this.

Do not forget to connect the address pin as you cannot read the sensor
in a reliable way. As the line will float it will sometimes have the
right address and sometimes not. (been there ;)


## Future

- follow MAX44009
- Merge MAX44007 into MAX44009 class hierarchy.

