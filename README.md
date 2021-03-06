# plantcentre_timer
IoT Assignment Manual where I try to figure out and explain how to activate separate running timers in Arduino to use as trigger, using a Neopixel LEDstrip as demonstration.

## Components
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
- NUMPIXELS is quite long, I changed it to just NUM because we might be using it several times.

### Checkpoint - See if your LEDstrip lights up
- Save the project.
- Upload it by clicking on the arrow button.
- Watch your LEDstrip light up!
Not happening?
- If you changed NUMPIXELS to NUM as well make sure you update it everywhere.
- Make sure all wires are hooked up correctly. If you're using different components, make sure you're using the correct PIN in your code.
- Check if Adafruit Neopixel actually installed in the Library Manager.

## Step 4 - Make a timed function
In order for lights to fire up independently from each other we will need to make a function pause for some given time and reactivate after that. Then we can duplicate and adjust that code.

- Let's first define a new timer function underneath loop() and move it's code to our new function as we won't be looping it regularly:
```
void loop() {

}

void time1() {
 pixels.clear(); 

  for(int i=0; i<NUM; i++) { 
    pixels.setPixelColor(i, pixels.Color(0, 150, 0));
    pixels.show(); 
    delay(DELAYVAL);
  }
}
```
- To call upon this function write `time1();` in the setup() function. Your LEDstrip will go through one loop.
- We can make a function loop itself by making it call upon itself at the end of the function. So add `time1();` add the bottom of itself.

### Delay the function
Here I encountered a problem. Adding a delay() doesn't pause the function it's called upon. It stops the entire code from running for a given time. This works fine if there's only one controller, but running several controllers at once would have the others pause too. 

```
void time1() {
  pixels.clear(); 

  for(int i=0; i<4; i++) { 
    pixels.setPixelColor(i, pixels.Color(0, 150, 0));
    pixels.show(); 
    
    delay(DELAYVAL); // <-- PROBLEM
  }
  time1();
  delay(4000); // <-- PROBLEM
}
```

Let's explore other options to delay a function. A quick Google returns libraries people have already made or Arduino's time function called millis(). Because libraries don't teach you much let's figure out how to use millis(). millis() holds the amount of milliseconds since the machine's bootup. This can be used to see how much time has passed. My original plan of waiting within a function before recalling itself won't work because we need to keep millis() updated. Looks like we're going back to loop().

- Start by deleting old, unnecessary code: in setup() delete `time1();`, in time1() delete the delay()'s and `time1;` as we'll be looping through the loop function again.
- First we need to declare some variables we'll be using as timer interval and to store trigger points in time. Above the setup() function write `const unsigned long interval1 = 3000;` and `unsigned long oldTime1 = 0;`.
- Inside loop() declare a var to store the millis() in `unsigned long curTime = millis();`
- Underneath we'll check if the interval time has passed since last time the function fired:
```
if (curTime - oldTime1 >= interval1) {
    oldTime = curTime;
  }
```
This if statement updates the old time whenever it runs. However, it only runs when interval1 time has passed. We can add code to this if statement to run after interval1.

### Checkpoint - add Serial
Let's see if this works by checking the Serial.
- In setup() add `Serial.begin(9600);`.
- Within the if statement write `Serial.println("check");` and upload the code.
- Open the Serial Monitor with the button top right. **Note:** set the baud to 9600 as well. I got an empty monitor because I forgot the adjust it.
- You'll see "check" written every 3 seconds.

## Step 5 - Integrate Neopixel and duplicate
Now let's see how we can use this timer to affect our Neopixel. We'll divide our Neopixel into 3 sections. Mine's got 13 LED's so I'm taking the number 4.
- Within time1() change NUM to 4, it won't count past that now.
- Underneath the for loop add `pixels.clear();` and `pixels.show();`. You should now see the first 4 LEDs flash every 3 seconds.
- You can copy and paste this code to edit it for a different timer and different LEDs. That would look like this:

```
const unsigned long interval2 = 5000;
unsigned long oldTime2 = 0;

void loop() {
  unsigned long curTime = millis();

  if (curTime - oldTime2 >= interval2) {
    time2();
    oldTime2 = curTime;
  }
}

void time2() {
    for(int i=4; i<8; i++) { 
    pixels.setPixelColor(i, pixels.Color(150, 0, 0));
    pixels.show();   
  }
  pixels.clear();
  pixels.show();
}
```

## Stuck
I figured out how to trigger functions on intervals but not really how to delay functions, however. I would like the LEDs shining for a second and then turning off again. I found a method using an empty`while()` that runs nothing until a certain amount of millis have passed executing the code behind the while loop. I couldn't implement this into my time() functions because these run once and then update the oldTime. They don't get new millis() values so that wouldn't work in my case. 

```
void time1() {
    for(int i=0; i<4; i++) { 
    pixels.setPixelColor(i, pixels.Color(0, 150, 0));
    pixels.show();   
  }

//       V Doesn't get updated
  while (millis() < curTime + 1000){
    
  }
  
  pixels.clear();
  pixels.show();
}
```

## Sources
Adafruit Neopixel Library
Programming Electronics Academy - about millis(): https://www.youtube.com/watch?v=BYKQ9rk0FEQ&ab_channel=ProgrammingElectronicsAcademy
Mads Aasvik - while loop delay - https://dzone.com/articles/arduino-using-millis-instead-of-delay
