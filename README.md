# POLARIS-RC-SPY-CAR-TRANSMITTER
## A TRANSMITTER FOR POLARIS RC SPY CAR.
![WhatsApp Image 2023-08-31 at 00 55 54](https://github.com/PolarisArm/POLARIS-RC-SPY-CAR-TRANSMITTER/assets/143507006/1f2548b6-76d2-43db-af35-4ecafd60dd1a)

__THIS TRANSMITTER HAVE 6 ANALOG CHANNEL AND 2 DIGITAL CHANNEL__
* 4 OF THE ANALOG CHANNEL ARE BEING USED BY JOYSTICK.
* 2 OF ANALOG CHANNEL USED BY TWO POTENTIOMETER.
* REMAINING TWO CHANNEL IS USED BY TWO DIGITAL SWITCH.JUST ON AND OFF (DIGITAL)
THIS TRANSMITTER INTENDED TO BE USED IN RC CAR, BOAT AND HOBBY AEROPLANE, DRONE(CAN BE USED, BUT NOT RECOMENDED)

__SPECIFICATION__
- OPERATING VOLTAGE - 5.0v FOR ALL SYSTEM AND 3.3v FOR NRF24L01-PA-LNA
- INPUT VOLTAGE: > 6.5v BUT NOT MORE THAN 9v.
- POWER SOURCE:  2X 18650 Li-ION BATTERY IN SERIES PROVIDING 8.4v MAX.
- MICROCONTROLLER: ATMEGA328P
- RF TRANSRECIVER: NRF24L01-PA-LNA
- DISPLAY: SSD1315 0.96inch MONOCHROME OLED DISPLAY 
  
__QUICK WORD FOR NRF24L01__

NRF24L01 is a single-chip 2.4GHz transceiver<br>
It operates in the 2.4 GHz ISM band and offers data rates up to __2 Mbit/s__.<br>
This chip communicates with the processor through SPI pins.<br>
All pins are 5V tolerant (this is the most important feature), but the Vcc pin is not. The maximum voltage on the Vcc pin is limited to 3.3V to 3.4V as an input.<br>
This project uses a slightly updated version of nrf24l01, namely nrf24l01+.<br>
A key aspect of our module is the power amplifier, which is an important feature.<br>
This increases the module's coverage up to __2 km__.<br>

_______________________________________________________________________________________
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
__Make a struct to send data through radio__
```C++
struct CHANNEL_DATA{
  byte throttle;
  byte yaw;
  byte roll;
  byte pitch;
  byte pot1;
  byte pot2;
  byte sw1;
  byte sw2;
};

CHANNEL_DATA data; //Initializing struct
```
In the void setup() section Initialize the radio.
```C++
if (!radio.begin()) {
    Serial.println(F("radio hardware is not responding!!"));
    while (1) {}  // hold in infinite loop
  }
  radio.setDataRate(RF24_250KBPS); // Data sending speed
  radio.setPALevel(RF24_PA_MAX);  // Setting Rf power to max as it will communicate in long distance.
  radio.openWritingPipe(address); // Opening the pipe , in which reciver is listening
  radio.stopListening(); // Because We are transmitting, why should we listen.
```

__Initializing struct with some default value__
```C++
  data.throttle = 127;
  data.yaw      = 127;
  data.roll     = 127;
  data.pitch    = 127;
  data.pot1     = 127;
  data.pot2     = 127;
  data.sw1      = 0;
  data.sw2      = 0;
  ```
__Initializing screen and showing welcome screen and Home screen__
```C++
  display.setRotation(2); // We are using this command as our display is upside down.

  if(!display.begin(SSD1306_SWITCHCAPVCC, SCREEN_ADDRESS)) {
    Serial.println(F("SSD1306 allocation failed"));
    for(;;); // Don't proceed, loop forever
  }
  display.clearDisplay();
  delay(100); 
  display.clearDisplay();
  display.drawBitmap(0,0,WELCOME_SCREEN,128,64,SSD1306_WHITE);
  display.display();
  delay(500); // Showing Welcome screen for 500ms 
  display.clearDisplay();
```
__Loop Section__
Just call these three function one after another.<br>
First send data through rf, using radio.write() command.<br>
```C++
void loop() {
  send_data(); // Sending data through radio
  display_data();  // displaying data in the display
  delayMicroseconds(100); // delay for 100 us
}
```
__Send Data Function__
As we are using byte for our variable we cannot use 10 bit data(0-1023),<br>
because byte is 8bit(0-255).So we have to map our analog read value from<br>
0 - 1023 to 0 - 255.<br>
> A byte stores an 8-bit unsigned number, from 0 to 255.

For this we use arduino map function.
```C++
map(value, fromLow, fromHigh, toLow, toHigh)
```
Parameters:
  * value: the number to map.
  * fromLow: the lower bound of the value’s current range.
  * fromHigh: the upper bound of the value’s current range.
  * toLow: the lower bound of the value’s target range.
  * toHigh: the upper bound of the value’s target range.
```C++
void send_data()
{
  data.throttle = map(analogRead(THROTTLE),0,1023,0,255);
  data.yaw      = map(analogRead(YAW)     ,0,1023,0,255);
  data.roll     = map(analogRead(ROLL)    ,0,1023,0,255);
  data.pitch    = map(analogRead(PITCH)   ,0,1023,0,255);
  data.pot1     = map(analogRead(POT1)    ,0,1023,0,255);  
  data.pot2     = map(analogRead(POT2)    ,0,1023,0,255);
  data.sw1      = digitalRead(SW1);
  data.sw2      = digitalRead(SW2);
  radio.write(&data,sizeof(CHANNEL_DATA)); // here size of gives total size of the data to function.
 // Serial.println("SENT DATA");
}
```

__SHOWING DATA IN THE DISPLAY__
HERE we use draw bitmap function from adafuit gfx library to draw the Home screen.
```C++
void Adafruit_GFX::drawBitmap	(	int16_t 	x, int16_t 	y, const uint8_t 	bitmap[], int16_t 	w, int16_t 	h, uint16_t 	color )
```
Parameters:
  - Top left corner x coordinate
  - y	Top left corner y coordinate
  - bitmap	byte array with monochrome bitmap
  - w	Width of bitmap in pixels
  - h	Height of bitmap in pixels
  - color SSD1306_WHITE and SSD1306_BLACK

To show changing data on the display we use this steps
```C++
      display.setTextSize(1); // First, give this function the text size you want
      display.setTextColor(SSD1306_WHITE); // Add color to your text by using this function. This is a monochrome (black and white two-color) display, so it might just be white here.
      display.setCursor(24,4); // set the position of your text(X,Y)
```


