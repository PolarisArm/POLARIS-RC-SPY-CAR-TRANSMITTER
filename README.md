# POLARIS-RC-SPY-CAR-TRANSMITTER
## A TRANSMITTER FOR POLARIS RC SPY CAR.
![WhatsApp Image 2023-08-31 at 00 55 54](https://github.com/PolarisArm/POLARIS-RC-SPY-CAR-TRANSMITTER/assets/143507006/1f2548b6-76d2-43db-af35-4ecafd60dd1a)

## INCLUDE LIBRARIES TO YOUR ARDUINO SKETCH
__Adafruit GFX LIBRARY__ : https://github.com/adafruit/Adafruit-GFX-Library.git <br>
__Adafruit SSD1306__ : https://github.com/adafruit/Adafruit_SSD1306.git <br>
__RF24__ : https://github.com/nRF24/RF24.git <br>

________________________________________________________________________________________

__Now define this variable__

```C++
#define CE         9
#define CSN       10
#define ROLL      A1
#define PITCH     A0
#define YAW       A2
#define THROTTLE  A3
#define SW1        5
#define SW2        6
#define POT1      A6
#define POT2      A7
#define TRCAL      3
#define YACAL      7
#define PICAL      4
#define ROCAL      8

```
__CE(CLOCK ENABlE) & CSN(CHIP SELECT NOT)__ are SPI pin and for nrf24l01.<br>
Throttle(Up & Down) & Yaw(Left & Right) is first joystick(Left).<br>
Pitch(Up & Down) & Roll(Left & Right) is first joystick(Left).<br>
Two potentiometer is POT1 & POT2. Two toggle switch is SW1(Left) & SW2(Right).<br>

__Now Initiate (0.96 inch) SSD1315 OLED DISPLAY__
This screen communicate atmega328p via I2C protocol.<br?
I2C pin <br>
* SCL A5 <br>
* SDA A4 <br>

```C++
#define SCREEN_WIDTH 128 // OLED display width, in pixels
#define SCREEN_HEIGHT 64 // OLED display height, in pixels

#define OLED_RESET     -1 // Reset pin # (or -1 if sharing Arduino reset pin)
#define SCREEN_ADDRESS 0x3C
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT);
```
> USE 0x3C for Screen address, Unless it will not work.<br>

__NOW INITIALIZE RADIO MODULE__
```C++
const uint64_t address = 0XE8E8F0F0E1LL; //This address will be same in reciver
RF24 radio(CE,CSN);
```
