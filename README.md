<!-- PROJECT LOGO -->
<br/>
<p align="center">
  <a href="https://github.com/swionrdhw/miotyduino20-lib-releases">
    <img src="mioty-logo-617x207.png" alt="Logo" width="330" height="120">
  </a>

  <h3 align="center">miotyduino20-lib</h3>

  <p align="center">
    Library to capture the MIOTYDuino20 Board into one object.
  </p>
</p>

# 1. MIOTYDuino20 Library
<!-- TABLE OF CONTENTS -->
<details open="open">

- [1. MIOTYDuino20 Library](#1-miotyduino20-library)
- [2. Dependencies](#2-dependencies)
- [3. Description](#3-description)
- [4. Getting Started](#4-getting-started)
- [5. Example](#5-example)
  - [5.1. Open Example](#51-open-example)
  - [5.2. Usage Examples](#52-usage-examples)
- [6. Serial overview](#6-serial-overview)
- [7. Defined Pins](#7-defined-pins)
- [8. Functions](#8-functions)
  - [8.1. MIOTYDuino20 Constructor](#81-miotyduino20-constructor)
  - [8.2. Init mYON](#82-init-myon)
  - [8.3. Use mYON function](#83-use-myon-function)
  - [8.4. 3V3 AUX](#84-3v3-aux)
  - [8.5. Step Up 5V](#85-step-up-5v)
  - [8.6. LEDs](#86-leds)
    - [8.6.1. Red LED](#861-red-led)
    - [8.6.2. Green LED](#862-green-led)
    - [8.6.3. Blue LED](#863-blue-led)
    - [8.6.4. Toggle LED](#864-toggle-led)
  - [8.7. Read Battery Voltage](#87-read-battery-voltage)
  - [8.8. Read Analog Pin](#88-read-analog-pin)
  - [8.9. Read Button](#89-read-button)
  - [8.10. Sleep](#810-sleep)
  - [8.11. Digital Pins](#811-digital-pins)
  - [8.12. m.YON mode selection](#812-myon-mode-selection)
- [9. Application layer payload format](#9-application-layer-payload-format)
- [10. License](#10-license)
- [11. Contact](#11-contact)

</details>

# 2. Dependencies

- [Arduino IDE](https://www.arduino.cc/en/software)
- [mioty-ep-at-lib](https://github.com/swionrdhw/mioty-ep-at-lib-releases)
- [STM32LowPower](https://github.com/stm32duino/STM32LowPower/archive/master.zip)
- [STM32RTC](https://github.com/stm32duino/STM32RTC/archive/refs/heads/master.zip)

# 3. Description

This Arduino library is mainly made for fast Prototyping with the MIOTYDuino20 B board. The MIOTYDuino20 object represents the board with all default peripherals.

On initialization the object configures the MIOTYDuino20, so everything is turned off and with correct handling there shall not be a possibility of a short circuit on a pin.

Additionally the object has methods for managing the standard functions of the board, like the 3.3 Volt regulator or the reading of the battery voltage.

The class inherits from the HardwareSerial class and is automatically to the STLink USART (to PC) connected. All default HardwareSerial functionalities are usable with this class.

For the mYON is an [MIOTY_EP](https://github.com/swionrdhw/mioty-ep-at-lib-releases) object inside the class available.

# 4. Getting Started

To setup the hardware/software environment and to program the first example on the MIOTYDuino, read: [Getting-Started-MIOTYDuino20](https://www.swissphone.com/download-item/60823).

# 5. Example

## 5.1. Open Example

1. To open an example in Arduino IDE, open File>Examples>MIOTYDuino20
2. Select the example you'd like to open.

## 5.2. Usage Examples

This part will go through the [Blink](examples/Blink/Blink.ino) example, to show how to use this library.

Firstly, the Library has to be included into the sketch and an instance of MIOTYDuino20 created.

```cpp
#include "MIOTYDuino20"

MIOTYDuino20 myDuino();
```

Inside the the setup function, first initialize the m.YON.
If the m.YON has been delivered with an EUI, short and networkkey, the default initialization function can be called.

```cpp
void setup(){
  /*init m.YON*/
  myDuino.mYON.init();
```

Otherwise it is recommended to set the EUI/shortaddress/networkkey via the string option of the init function.

```cpp
#define EUI "your EUI"
#define NWK "your Networkkey"
#define SHORT "short from the EUI"

void setup(){
  /*init mYON*/
  myDuino.mYON.init(EUI, SHORT, NETKEY);
```

To test if the m.YON was connected in the initialization connected the wasAlive function can be called.

```cpp
/*Test if mYON is available*/
if(!myDuino.mYON.wasAlive()){
    /*... print information*/
    myDuino.println("mYON not connected");
}
```

For demonstration, in the loop function toggle the RGB LED on and of with the
the [toggleLED](#toggle-led) function.

To toggle all colors of the RGB LED we give no parameter to the function.

```cpp
/*toggle all LEDs*/
myDuino.toggleLED();
```

To make the toggle even visible to the human eye, we need a sleep statement.
As an alternative to the sleep function from the library, the standard Arduino delay function is also possible.

```cpp
/*deep sleep for 100ms*/
myDuino.sleep(100);
```

# 6. Serial overview

The MIOTYDuino20 class inherits from the HardwareSerial and creates a Serial connection to the PC over the STLink on the UART1.

It also establishes a Serial connection to the m.YON over the UART3 by having a MIOTY_EP object member. this baudrate changes to 115200 if the m.YON is in the bootloader mode.

The Serial connection to the PC has a default 115200 baudrate and supports all standard [Serial](https://www.arduino.cc/reference/en/language/functions/communication/serial/) functions.

**Example to pc (over STLink)**:

```cpp
myDuino.println("Hallo Welt");
String stringFromPC = myDuino.readString();
```

**Example to m.YON**:

```cpp
myDuino.mYON.print("ATI\r");
String BidiInformation = myDuino.readString();
```

*Serial Over USB*:

---
It is possible to use the SerialUSB witch is implemented in the default Arduino environment.

To enable the communication add the following lines on to the end of the `SystemClock_Config` function in the `variant.cpp` file (C:/Users/"user_name"/AppData/Local/Arduino15/packages/STM32/hardware/stm32/1.9.0/variants/NUCLEO_L452RE/)

```cpp
//////////////////////////////////////////////////////////////////////////
  RCC_PeriphCLKInitTypeDef PeriphClkInit = {};
  PeriphClkInit.PeriphClockSelection = RCC_PERIPHCLK_USB;
  PeriphClkInit.UsbClockSelection = RCC_USBCLKSOURCE_PLLSAI1;
  PeriphClkInit.PLLSAI1.PLLSAI1Source = RCC_PLLSOURCE_MSI;
  PeriphClkInit.PLLSAI1.PLLSAI1M = 1;
  PeriphClkInit.PLLSAI1.PLLSAI1N = 24;
  PeriphClkInit.PLLSAI1.PLLSAI1P = RCC_PLLP_DIV7;
  PeriphClkInit.PLLSAI1.PLLSAI1Q = RCC_PLLQ_DIV2;
  PeriphClkInit.PLLSAI1.PLLSAI1R = RCC_PLLR_DIV2;
  PeriphClkInit.PLLSAI1.PLLSAI1ClockOut = RCC_PLLSAI1_48M2CLK;
  if (HAL_RCCEx_PeriphCLKConfig(&PeriphClkInit) != HAL_OK) {
    Error_Handler();
  }
  ///////////////////////////////////////////////////////////////////////////
```

Inside the Arduino IDE the Upload method has to be changed to CDC (generic 'Serial' supersede U(S)ART)

Usage example:

```cpp
SerialUSB.begin(115200);
SerialUSB.println("Hello from USB");
```

After the Upload the mioty duino will connect with the USB on a different COM port and recognized as an 'STMicroelectronics Virtual COM port'.

USB Serial is not used by this library, but user can add it to run side by side with other serial ports.

It's not recommended to use the USB serial for an m.YON firmware update, because the estimated update time is around 3.5 hours.

# 7. Defined Pins

All pins of the miotyDuino20 board are defined in the `MIOTYDuino20_B_Pins.h`.

**List of defined pins and description:**

| Pin Name                |Pin description                                                        |
|-------------------------|-----------------------------------------------------------------------|
| `H_3V3_AUX_EN`          |   Enable pin of external 3V3 output (high active)                     |
| `L_5V_STEP_UP_EN`       |   Enable pin of 5V stepup converter (low active)                      |
| `L_BIDI_RST`            |   Reset pin of m.YON (low active)                                 |
| `MIKRO_RST`             |   Reset pin of the MikroBUS                                           |
| `GROVE_A0`              |   Analog pin of of the MikroBUS                                       |
| `GROVE_A1`              |   Not connected / Analog input                                        |
| `GROVE_A2`              |   I/O 1 pin of of the Grove connector #3                              |
| `GROVE_A3`              |   I/O 2 pin of of the Grove connector #3                              |
| `A4`                    |   Analog Input pin                                                    |
| `A5`                    |   Analog Input pin                                                    |
| `D15`                   |   Digital output pin                                                  |
| `D14`                   |   Digital output pin                                                  |
| `MIKRO_SPI_SCK_D13`     |   MikroBUS SPI clock pin                                              |
| `MIKRO_SPI_MISO_D12`    |   MikroBUS SPI master in slave out pin                                |
| `MIKRO_SPI_MOSI_D11`    |   MikroBUS SPI master out slave in pin                                |
| `SPI_AUX_CS_D10`        |   MikroBUS SPI chip select pin                                        |
| `D9`                    |   Digital output pin                                                  |
| `USART1_TX_D8`          |   Digital output pin or USART #1 transmit pin                         |
| `D7`                    |   Digital output pin                                                  |
| `D6`                    |   Digital output pin                                                  |
| `D5`                    |   Digital output pin                                                  |
| `D4`                    |   Digital output pin                                                  |
| `D3`                    |   Digital output pin                                                  |
| `USART1_RX_D2`          |   Digital output pin or USART #1 receive pin                          |
| `USART2_TX_D1`          |   Digital output pin or USART #2 transmit pin                         |
| `USART2_RX_D0`          |   Digital output pin or USART #1 receive pin                          |
| `H_OC1_EN`              |   Open collector transistor #1 enable pin (high active)               |
| `H_OC2_EN`              |   Open collector transistor #2 enable pin (high active)               |
| `BUTTON`                |   On board button input pin                                           |
| `LED_RED`               |   Red segment of on board RGB LED                                     |
| `LED_GREEN`             |   Green segment of on board RGB LED                                   |
| `LED_BLUE`              |   Blue segment of on board RGB LED                                    |
| `BATT_DIVIDER_EN`       |   Enable of battery voltage divider (to measure the battery voltage)  |
| `MIOTY_USART3_TX`       |   Transmit pin from m.YON                                         |
| `SPI_MIOTY_CS`          |   Chip select of m.YON                                            |

# 8. Functions

Short description and intended use of all functions from this library.

## 8.1. MIOTYDuino20 Constructor

The Constructor initializes the Serial communication to the PC with the default baudrate of 115200 if no other baudrate is given.

Additionally the constructor defines all in-/output pins.

default as Output:

- all LED's (undef.)
- 3.3V enable pin (HIGH)
- Battery measuring enable pin (LOW)
- Pin for 5V stepup (HIGH)
- Opencollector 1&2 (LOW)
- Hardware index enable (LOW)

default as Input:

- Button
- Battery lvl reading pin
- Hardware index reading pin
- digital I/O's
- Analog Pins

```cpp
MIOTYDuino20(long baudrate/*= DEFAULT_BAUDRATE*/)
```

***Takes:***

- `baudrate`: baudrate of serial to STLink (optional)

## 8.2. Init mYON

---
The function calls the init function of the m.YON.
This can take about 6s.

If the EUI, ShortAddress and Networkkey should be changed, the information can be given to the function.

**If only the Networkkey should be changed, the `setNetworkkey` function from the mioty-at-ep-lib should be called**.

***Note:*** The m.YON is not usable until it is initialized.

```cpp
void mYON.init(String EUI /*= ""*/, 
              String SHORT /*= ""*/, 
              String NWK /*= ""*/)
```

***Takes:***

- `EUI`: Endpoint EUI64 (eg. "5C335CF667340010") (optional)
- `SHORT`: Short 2-Byte representation of EUI64 (eg. "0010")(optional)
- `NWK`: Network key (eg. "9ee5e6d365b04679e21bd34146aa5ea1")(optional)

## 8.3. Use mYON function

---
To use function from [mioty-ep-at-lib](https://github.com/swionrdhw/mioty-ep-at-lib-releases) there has to be a mYON in front of the function call.

See example:

```cpp
bool mYON.isAlive();
```

## 8.4. 3V3 AUX

---
For the 3.3V regulator for peripherals, is a method for enabling and disabling of the regulator available.

The 3.3V regulator is by default enabled.

```cpp
void external3V3(bool state)
```

**Takes**:

- `state`: 1/true/HIGH for enable and 0/false/LOW for disable

## 8.5. Step Up 5V

---
For enabling/disabling of the 5V step-up converter. This method is useless if the MIOTYDuino20 Board is on a Micro-USB. Because this method gets overwritten by the Hardware.

```cpp
void external5V(bool state)
```

**Takes**:

- `state`: 1/true/HIGH for enable and 0/false/LOW for disable

## 8.6. LEDs

---
There is one RGB LED on the board. each LED can be turned turned on or off.
For each LED color exists one digital function and for all together one toggle function.

### 8.6.1. Red LED

---

```cpp
void redLED(int state)
```

**Takes**:

- `state`: the state of the LED (HIGH/LOW or 1/0 or true/false)

### 8.6.2. Green LED

---

```cpp
void greenLED(int state)
```

**Takes**:

- `state`: the state of the LED (HIGH/LOW or 1/0 or true/false)

### 8.6.3. Blue LED

---

```cpp
void blueLED(int state)
```

**Takes**:

- `state`: the state of the LED (HIGH/LOW or 1/0 or true/false)

### 8.6.4. Toggle LED

One or all LED's can be toggled.

```cpp
void toggleLED(int LED)
```

**Takes**:

- `LED`: number for LED (LED_RED, LED_GREEN, LED_BLUE, default:all)

***Note***: if its an unknown number the function toggles all LED's

## 8.7. Read Battery Voltage

---
The MIOTYDuino20 is able to read the voltage of it's battery.
For this function the Hardware-Voltage divider gets activated and after a 100ms waiting period the value gets read via the `analogRead` function from the Arduino extension.

```cpp
float readBatteryVoltage()
```

**Returns**:

- `float`: Voltage value in Volts.

It has a resolution of 3.22mV and the voltage gets calculated with the formula:

```math
\frac{2*[read value]} {1024.0} * 3.3V
```

## 8.8. Read Analog Pin

---
There are 6 analog pins available.
Only those 6 pins can be read by this function.

```cpp
int readAnalog(int pin)
```

**Takes**:

- `pin`: an integer representation of the pin.

For the integer are defines from the *MIOTYDuino20_B_Pins.h* available.
The possible defines are:

0. `GROVE_A0`
1. `GROVE_A1`
2. `GROVE_A2`
3. `GROVE_A3`
4. `A4`
5. `A5`

**Returns**:

- `int`: read 5-Bit (0-1024) value from the ADC.

If an invalid pin is given, the method return -1.

## 8.9. Read Button

---
This function reads the digital value from the Button.

 ```cpp
bool readButton()
 ```

**Returns**:

- `bool`: true if the Button is pressed and false otherwise.

## 8.10. Sleep

---
As an alternative to the delay function. It's possible to use the sleep method of this module. this method puts the MIOTYDuino20 in to the deep sleep mode for the given amount of milliseconds.
This reduces power consumption during downtime

**Important**:
Actions like print/println will be [flushed](https://www.arduino.cc/reference/de/language/functions/communication/serial/flush/) before the MIOTYDuino20 enters sleep mode. additionally while the MIOTYDuino20is in  sleep, no data over Serial communication can be received.
This can cause undefined behavior in interrupt depended function.

```cpp
void sleep(int ms)
```

**Takes**:

- `ms`: sleep time in milliseconds

**Note**: This function does sometimes not work properly.
Use delay as a workaround.

## 8.11. Digital Pins

---

The defined pin names are:

0. `USART2_RX_D0`
1. `USART2_TX_D1`
2. *reserved*
3. `D3`
4. `D4`
5. `D5`
6. `D6`
7. `D7`
8. *reserved*
9. `D9`
10. `SPI_AUX_CS_D10`
11. `MIKRO_SPI_MOSI_D11`
12. `MIKRO_SPI_MISO_D12`
13. `MIKRO_SPI_SCK_D13`
14. `D14`
15. `D15`

**Note**: The digital IOs  `D2` and `D8` are used for the serial communication to the PC (over STLink).

The 16 digital IOs can be read as true or false (1/0 or HIGH/LOW).
Like the [analogRead](#read-analog-pin) method, does this method read only those 16 pins.

The digital IO's can also be written as digital HIGH or LOW.
The output voltage is 3.3V.

```cpp
bool digitalIO(int pin,int state = -1);
```

**Takes**:

- `pin`: defined pin number.
- `state`: HIGH, LOW or -1 (default -1).

**Returns**:

- `bool`: True if the Voltage level is HIGH or the output was set successfully.
          False if the Voltage was LOW or the pin doesn't exist.

If the pin number is invalid the method returns false regardless of the pins voltage level.

**Note**: When a pin is set with this function it gets set as output.

## 8.12. m.YON mode selection

---
The m.YON can be update over UART with the XModem protocol.
Additionally a serial tunneling mode exists as well.
This Library provides a default entrance into both modes during boot up.

**To select the m.YON mode, hold down the button during boot up until the LED starts blinking in purple.**

After the button has been released, the LED stops blinking and the miotyduino waits 5 seconds.

During the selection process the led blinks in the color of the currently selected mode.

- `red LED`: the duino exits the mode selection and initialize normally.
- `blue LED`: enters the update mode for a firmware update.
- `green LED`: Serial Tunneling (direct access to the m.YON over the UART).

As an alternative the software update function can be used to enter the update mode:

```cpp
void enterMiotyModuleUpdateMode();
```

This will set the serial interface baudrate to the bootloader baudrate and the mioty Duino passes all serial data from the PC (over STLink) directly to the m.YON and vice versa.

If the update mode got entered correctly, the m.YON sends every second a "C" over the serial interface.

This mode can be exited via the button or automatically after an XModem file has been sent.

For a more detailed instruction on How to update the m.YON follow the [update guide](https://www.swissphone.com/download-item/61943).

# 9. Application layer payload format

The mioty Application Center has the option to add a payload format which allows automatic processing of uplink data.

This allows to send and decode different length of payload with one endpoint.
To make that possible an endpoint type has to be added on the station.

for example, the MIOTY_BME280 example uses the "5c335cf982b21f6b" application layer payload format:

```json
{
  "version": "2.0",
  "typeEui": "5c335cf982b21f6b",
  "meta": {
      "name": "miotyduino wheater sensor temp_humidity_pressure_battery",
      "vendor": "Swissphone"
  },
  "component": {
      "battery": {
        "func": "$/256*1.3+3.0", 
        "littleEndian": false, 
        "size": 8, 
        "type": "uint", 
        "unit": "V"
      },
      "humidity": {
        "func": "", 
        "littleEndian": false, 
        "size": 8, 
        "type": "uint", 
        "unit": "%"
      },
      "pressure": {
        "func": "$/10", 
        "littleEndian": false, 
        "size": 16, 
        "type": "uint", 
        "unit": "hPa"
      },
      "temperature": {
        "func": "$/10-273.15", 
        "littleEndian": false, 
        "size": 16, 
        "type": "uint", 
        "unit": "°C"
      }
  },
  "uplink": [
      {
      "id": 0,
      "payload": [
          {"component": "temperature","name": "temperature"},
          {"component": "humidity","name": "humidity"},
          {"component": "pressure","name": "pressure"}
      ]
      },
      {
          "id": 192,
          "payload": [
              {"component": "temperature","name": "temperature"},
              {"component": "humidity","name": "humidity"},
              {"component": "pressure","name": "pressure"},
              {"component": "battery","name": "battery"}
          ]
      }
  ]
}
```

# 10. License

# 11. Contact

Mioty Team - [mioty@swissphone.com](mioty@swissphone.com)

Project Link: [https://github.com/swionrdhw/miotyduino20-lib-releases](https://github.com/swionrdhw/miotyduino20-lib-releases)
