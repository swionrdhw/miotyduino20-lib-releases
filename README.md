<!-- PROJECT LOGO -->
<br />
<p align="center">
  <a href="https://git.swissphone.com/libraries/cpp/mioty-ep-miotyduino20/-/tree/master">
    <img src="https://radiocrafts.com/wp-content/uploads/2020/04/mioty-logo-617x207.png" alt="Logo" width="330" height="120">
  </a>

  <h3 align="center">miotyduino20-lib</h3>
    
  <p align="center">
    Library to capture the MIOTYDuino20 Board into one object.
  </p>
</p>


# MIOTYDuino20 Library
<!-- TABLE OF CONTENTS -->
<details open="open">
    <ol>
        <li><a href="#description">Description</a></li>
        <li><a href="#dependencies">Dependencies</a></li>
        <li><a href="#getting-started">Getting Started</a></li>
        <li><a href="#example">Example</a></li>
        <li><a href="#serial-overview">Serial overview</a></li>
        <li><a href="#methods">Methods</a></li>
        <ul>
            <li><a href="#3v3-aux">AUX 3V3</a></li>
            <li><a href="#step-up-5v">Step Up 5V</a></li>
            <li><a href="#leds">LED's</a></li>
                <ul>
                    <li><a href="#red-led">Red LED</a></li>
                    <li><a href="#green-led">Green LED</a></li>
                    <li><a href="#blue-led">Blue LED</a></li>
                    <li><a href="#toggle-led">Toggle LED('s)</a></li>
                </ul>
            <li><a href="#read-battery-voltage">Read Battery Voltage</a></li>
            <li><a href="#read-analog-pin">Read Analog Pin</a></li>
            <li><a href="#read-button">Read the Button</a></li>
            <li><a href="#sleep">Sleep</a></li>
            <li><a href="#read-hardware-index">Get Hardware Index</a></li>
            <li><a href="#digital-pins">Digital pins</a></li>
            <li><a href="#update-bidistamp-firmware">Update BidiStamp Firmware</a></li>
        </ul>
        <li><a href="#application-layer-payload-format">Application layer payload format</a></li>
        <li><a href="#license">License</a></li>
        <li><a href="#contact">Contact</a></li>
    </ol>
</details>

## Dependencies
- [Arduino IDE](https://www.arduino.cc/en/software)
- [mioty-ep-at-lib](https://git.swissphone.com/libraries/cpp/mioty-ep-at-lib)
- [STM32LowPower](https://github.com/stm32duino/STM32LowPower/archive/master.zip)
- [STM32RTC](https://github.com/stm32duino/STM32RTC/archive/refs/heads/master.zip)

## Description
---

This Arduino library is mainly made for fast Prototyping with the MIOTYDuino20 B board. The MIOTYDuino20 object represents the board. 

On initialization the object configures the MIOTYDuino20, so everything is turned off and with correct handling there shall not be a possibility of a short circuit on a pin.

Additionally the object has methods for managing the standard functions of the board, like the 3.3 Volt regulator or the reading of the battery voltage.

The class inherits from the HardwareSerial class and is automatically to the STlink USART (to PC) connected. All default HardwareSerial functionalities are usable with this class.

For the bidi-stamp is an [MIOTY_EP](https://git.swissphone.com/libraries/cpp/mioty-ep-at-lib) object inside the class available.


## Getting Started
---
To setup the hardware/software environment and to programm the first example on the MIOTYDuino, read: [Getting-Started-MIOTYDuino20](https://www.swissphone.com/download-item/60823).

## Example
---
### Open Example
1. To open an example in Arduino IDE, open File>Examples>MIOTYDuino20
2. Select the example you'd like to open.

### Use Examples
We will go through the [Blink](examples/Blink/Blink.ino) example to show how to use this library.

Firstly, the Library has to be included into the sketch and create an instance of MIOTYDuino20 created.

If your MIOTYDuino has been delivered with an EUI, you can use the default constructor.

```cpp
#include "MIOTYDuino20"

MIOTYDuino20 myDuino();
```
Otherwise you should give the Constructor the details for the connection.

```cpp
#include "MIOTYDuino20"

#define EUI "your EUI"
#define NETKEY "your Networkkey"
#define SHORT "short from the EUI"

MIOTYDuino20 myDuino(EUI,SHORT,NETKEY);
```

Inside the the setup function, to test if the BidiStamp is connected.

```cpp
/*Test if BiDi-Stamp is avalable*/
if(!myDuino.BidiStamp.isAlive()){
    /*... print information*/
    myDuino.println("BidiStamp not connected");
}
```

For demonstration, in the loop function toggle the RGB LED on and of with the 
the [toggleLED](#toggle-led) function.

To toggle all colors of the RGB LED we give the value `4` as an parameter to the function.

```cpp
/*toggle all LEDs*/
myDuino.toggleLED(4);
```

To make the toggle even visible to the human eye, we need a sleep statement.
As an alternative to the sleep function from the library, the delay function is also possible.

```cpp
/*deep sleep for 100ms*/
myDuino.sleep(100);
```

## Serial overview
---

The MIOTYDuino20 class inherits from the HardwareSerial and creates a Serial connection to the PC over the STLink on the UART1.

It also establishes a Serial connection to the BidiStamp over the UART3 by having a MIOTY_EP object member.

```cpp
MIOTY_EP BidiStamp = MIOTY_EP(MIOTY_USART3_RX,MIOTY_USART3_TX);
```

Both communications have a 9600 baudrate and support the default [Serial](https://www.arduino.cc/reference/en/language/functions/communication/serial/) functions.

**Example to pc (over STLink)**:
```cpp
myDuino.println("Hallo Welt");
String stringFromPC = myDuino.readString();
```

**Example to BidiStamp**:
```cpp
myDuino.BidiStamp.print("ATI\r");
String BidiInformation = myDuino.readString();
```

*Serial Over USB*:
--- 
It is possible to use the SerialUSB witch is implemented in the default Arduino environment.

To enable the communication add the following lines on to the end of the `SystemClock_Config` function in the `variant.cpp` file (C:/Users/<username>/AppData/Local/Arduino15/packages/STM32/hardware/stm32/1.9.0/variants/NUCLEO_L452RE/)

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

It's not recommended to use the USB serial for an BidiStamp firmware update, because the estimated update time is around 3.5 hours.

## Methods
---
Short description and intended use of all methods from this library.
### 3V3 AUX
---
For the 3.3V regulator for peripherals, is a method for enabling and disabling of the regulator available.

The 3.3V regulator is by default disabled.

```cpp
void AUX_3V3(bool state)
```

**Takes**:
- `state`: 1/true/HIGH for enable and 0/false/LOW for disable

**Retruns**:
- Nothing

### Step Up 5V
---

For enabling/disabling of the 5V step-up converter. This method is useless if the MIOTYDuino20 Board is on a Micro-USB. Because this method then get overwritten by the Hardware.

```cpp
void Step_Up_5V(bool state)
```
**Takes**:
- `state`: 1/true/HIGH for enable and 0/false/LOW for disable

**Retruns**:
- Nothing


### LEDs
---
There is one RGB LED on the board. each LED can be turned turned on or off.
For each LED color exists one digital function and for all together one toggle function.

#### Red LED
---
```cpp
void redLED(int state)
```
**Takes**:
- `state`: the state of the LED (HIGH/LOW or 1/0 or true/false)

#### Green LED
---
```cpp
void greenLED(int state)
```
**Takes**:
- `state`: the state of the LED (HIGH/LOW or 1/0 or true/false)
#### Blue LED
---
```cpp
void blueLED(int state)
```
**Takes**:
- `state`: the state of the LED (HIGH/LOW or 1/0 or true/false)

#### Toggle LED
One or all LED's can be toggled.

```cpp
void toggleLED(int LED)
```
**Takes**:
- `LED`: number for LED (1: red, 2:green, 3:blue, 4/default:all)

***Note***: if its an unknown number the function toggles all LED's


### Read Battery Voltage
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
\frac{2*[read value]} {1024.0} * 3.3
``` 

### Read Analog Pin
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

### Read Button
---
This function reads the digital value from the Button.
 ```cpp
bool readButton()
 ```
**Returns**:
- `bool`: true if the Button is pressed and false otherwise.

### Sleep
---
As an alternative to the delay function. It's possible to use the sleep method of this module. this method puts the MIOTYDuino20 in to the deep sleep mode for the given amount of milliseconds.

**Important**: This method puts the MIOTYDuino20 immediately in deep sleep mode. 
all independent actions (like println) will be interrupted, until the MIOTYDuino wakes up again. additionally while the MIOTYDuino is in deep sleep, no data over Serial communication can be received.
This can cause undefined behavior in independent actions.

```cpp
void sleep(int ms)
```

**Takes**:
- `ms`: sleep time in milliseconds

**Note**: For this function is the STM32LowPower Library necessary.

### Read Hardware Index
---
***NOT IMPLEMENTED YET***

### Digital Pins
---

#### General
The defined pin names are:

0. `USART2_RX_D0`
0. `USART2_TX_D1`
0. *reserved*
0. `D3`
0. `D4`
0. `D5`
0. `D6`
0. `D7`
0. *reserved*
0. `D9`
0. `SPI_AUX_CS_D10`
0. `MIKRO_SPI_MOSI_D11` 
0. `MIKRO_SPI_MISO_D12`
0. `MIKRO_SPI_SCK_D13`
0. `D14`
0. `D15`

**Note**: The digital IO's  `D2` and `D8` are used for the serial communication to the PC (over STLink).

#### Read
The 16 digital IO's can be read as true or false (1/0 or HIGH/LOW).
Like the [analogRead](#read-analog-pin) method, does this method read only those 16 pins.

```cpp
bool readDigitalIO(int pin)
```

**Takes**:
- `pin`: defined pin number.

**Returns**:
- `bool`: True if the Voltage level is HIGH and false otherwise.

If the pin number is invalid the method returns false regardless of the pins voltage level.

**Note**: When this function is called, the respective pin will automatically changed to an *INPUT*.

### Write
The digital IO's can also be written as digital HIGH or LOW.
The output voltage is 3.3V.

```cpp
void writeDigitalIO(int pin,bool state)
```
**Takes**:
- `pin`: defined pin number.
- `state`: output state of IO (true/false or HIGH/LOW or 1/0).

### Update BidiStamp Firmware
---
The BidiStamp can be update over UART with the XModem protocol.
This Library provides a default entrence into an update mode on boot up.

To enter the BidiStamp update mode, hold down the button during boot up until the blue LED starts blinking.

After the button has been released the blue LED stops blinking. And the Serial tunneling starts.

This will set the serial interface baudrate to the bootloader baudrate and the mioty Duino passes all serial data from the PC (over STLink) directly to the BidiStamp and vice versa.

If the update mode got entered correctly, the BidiStamp sends every second a "C" over the serial interface.

This mode can be exited via a the button.

For a more detailed instruction on How to update the BidiStamp follow the [update guide](!!link-hier-einfügen!!).

## Application layer payload format
---
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

## Licence
---

## Contact
---

Tendai Rondof - [tendai.rondof@swissphone.com](tendai.rondof@swissphone.com)

Christian Gwerder - [christian.gwerder@swissphone.com](christian.gwerder@swissphone.com)

Project Link: [https://git.swissphone.com/libraries/cpp/miotyduino20-lib](https://git.swissphone.com/libraries/cpp/miotyduino20-lib)
