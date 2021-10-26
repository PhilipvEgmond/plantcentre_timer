# plantcentre_timer
IoT Assignment Manual where I try to figure out and explain how to activate separate running timers in Arduino to use as trigger, using a Neopixel LEDstrip as demonstration.

## Materials
* NodeMCU
* RGB LEDStrip
* USB Cabel

## Step 1 - Connect the LEDstrip
- Connect wire 5V to pin 3V3.
- Wire Din to D5.
- Wire GND to GND.

## Step 2 - Install the Adafruit Neopixel library on Arduino
Using the Adafruit Neopixel library we can easily connect to and control our LEDstrip.

- Open Arduino and press **Sketch < Include Library > Manage Libraries...**
- Search and install "Adafruit Neopixel".

## Step 3 - Setup a Neopixel project
The Neopixel library comes with a few examples we can handily make use of.
We're going to open a basic example, to find it:

- Press **File < Examples < Adafruit Neopixel < simple**
- Read the comments to familiarize yourself with the workings of the library.
- To make the code easier to read remove any comments and code you won't need.

This is the code I ended up with:
```
#include <Adafruit_NeoPixel.h>

#define PIN        6 
#define NUMPIXELS 16 
#define DELAYVAL 500

Adafruit_NeoPixel pixels(NUMPIXELS, PIN, NEO_GRB + NEO_KHZ800);

void setup() {
  pixels.begin(); 
}

void loop() {
  pixels.clear(); 

  for(int i=0; i<NUMPIXELS; i++) { 
    pixels.setPixelColor(i, pixels.Color(0, 150, 0));
    pixels.show(); 
    delay(DELAYVAL);
  }
}
```
### Personalise the code
- The PIN we're using is D5, change it's value in the code.
- NUMPIXELS is the amount of pixels on your LEDstrip, change this to your own amount.
- NUMPIXELS is quite long, I changed it to just NUM because you might be using it several times. 
**NOTE:** Whenever you change a variable make sure to change it throughout the entirety of the code.

## Step 4 - See if the code and LEDstrip work
- Save the project.
- Upload it by clicking on the arrow button.
