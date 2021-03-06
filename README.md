# Arduino Uno [Branch]
*NOTE* that this branch contains code for building the radio with an `Arduino Uno` using a slightly different board design. With this board we kept all of the components at 5V so we avoid all of the level-shifters that were required with the micro:bit. The board contains the following components:

 * Arduino Uno
 * [Adafruit Si4713]



![title banner](diagrams/PR_2000_trick.jpg "Public Radio Banner")
# Public Radio
*Public Radio* was designed and fabricated by [New American Public Art](http://www.newamericanpublicart.com/) with electronic design &amp; programming by [Andrew Ringler](https://andrewringler.com/). This repository holds the Arduino code and electronic design documentation that drives the *Public Radio*.  

*Public Radio* is a large outdoor radio that continually plays over-the-air FM radio. It has two dials, one for changing station and one for changing volume. It provides visual feedback with LEDs of the currently selected station as well the current volume level. Additionally it provides audio feedback of the current station and volume by actually playing the selected FM station.

## New Developer Setup
The micro:bit was programmed using the Arduino IDE. If you need to make changes and re-upload the code first power off the radio, disconnect the USB cable from the micro:bit then pull the micro:bit out from the Dragontail connector.

In order to load the program onto a micro:bit you will need to:

 * Install the [Arduino IDE](https://www.arduino.cc/en/Main/Software) software
 * Follow the instructions at Adafruit's [Use Microbit with the Arduino IDE | Add NRF5x Board Support](https://learn.adafruit.com/use-micro-bit-with-arduino/install-board-and-blink#add-nrf5x-board-support-2-7) to add the NRF5x Board Support to your Arduino IDE. 
 * Download the entire repository: [as a zip here](https://github.com/NewAmericanPublicArt/public-radio/archive/master.zip)
 * From Arduino menu `Sketch > Include Library > Manage Libraries…` install the `Adafruit Dotstar` library (tested with v1.0.4) 

Once you have the NRF5x board installed, hook your micro:bit up to your computer via a USB cable. Then double-click the `PublicRadio/PublicRadio.ino` file to load the program in Arduino. Then update the following from Arduino menus `Tools > Board > BBC micro:bit`, `Tools > Softdevice > S110` and `Tools > Port > …BBC micro:bit…`

Then `Sketch > Upload`.

Put the micro:bit back into the Dragontail connector with the LED grid facing up. Power back on the radio.

## Design
*Public Radio* is a large 8x4 foot structure with an acrylic front panel. Two large dials (for channel tuning and volume changing) are mounted flush on the front panel. A curved string of LEDs are mounted on the interior left of the front panel which shine through to the outside providing visual feedback of the currently selected station. Tick marks and station labels (created by CNC cutting the front panel interior) line up with the LEDs. On the front bottom right a small string of LEDs provide visual feedback for the current volume level. Electronics (in waterproof boxes) as well as waterproof speakers are contained within the radio interior. The entire radio is controlled by a single BBC micro:bit.

A Fusion 360 model of the radio structure and components can be downloaded [here](https://a360.co/2QIU2CJ).

### Dials
The dials are attached to axles on the radio interior and on each axle is mounted a single black plexiglass disc with slits laser cut into an outer and an inner ring. The inner ring of slits is 1/2 slit-width out of phase with the outer ring. Two [Vernier VPG-BTD](https://www.vernier.com/products/sensors/vpg-btd/) photogates are mounted per encoder wheel such that one photogate measures the the outer ring of slits and the other photogate measures the inner ring of slits. This combination of using two photogates to measure the inner and outer rings of phase-shifted slits creates an optical incremental directional encoder whose readings we can interpret as a 2-bit Gray code pattern thus resolving the current speed of the axle and the direction the axle is rotating (clockwise versus counterclockwise). For more information on the concept of rotary encoders see [Rotary encoder](https://en.wikipedia.org/wiki/Rotary_encoder) on Wikipedia as well as Jeff Cook's University of Michigan [EECS 461](http://web.eecs.umich.edu/~jfr/embeddedctrls/lectures.html) Embedded Control Systems [Position and Velocity Lecture Notes](http://web.eecs.umich.edu/~jfr/embeddedctrls/files/Lecture3.pdf) (the lecture notes PDF is also available in our resources directory).

Each of the [Vernier VPG-BTD](https://www.vernier.com/products/sensors/vpg-btd/) Photogates is connected to a [Vernier BTA-ELV Analog Protoboard Adapter](https://www.vernier.com/products/accessories/protoboard-adapters/bta-elv/) which is a small breadboard compatible breakout board. We power the board with `5V` then measure pin `DIO0` which is `HIGH` when the photogate is blocked otherwise `LOW`. Since this is a `5V` board we first level-shift the `DIO0` signal down to `3.3V` with a `74AHCT125` level shifter then measure with a GPIO pin on the micro:bit.

We measure the speed and direction of each of the dials. If the station dial is turning clockwise the station increases, if the dial is turning counterclockwise the station decreases. The dials have no hard stops, so upon reaching the maximum station `107.9` while turning clockwise, we restart at the minimum station `86.5`, then continue to increase stations.

The volume dial also has no hard stops, but unlike the station dial, volume does not wrap-around from maximum volume to minimum volume since this would have created a jarring acoustic experience. Instead, we use a different technique we are calling `avocado volume`. Typically, while turning the volume dial clockwise the volume will increase until it reaches the maximum volume, then it will start decreasing, until it reaches the minimum volume, then it will start increasing again. So, if you continuously spin the dial clockwise, the volume of the radio will smoothly ramp up and down forever. Spinning the dials counterclockwise has the same behavior.

### Radio
The actual FM radio is driven by a digital `Si4703` FM tuner board. The micro:bit, on power-on, initializes the FM tuner board to a default station and volume. As the user changes station and volume, using the dials, these internal variables are updated and any changes are sent via `I²C` to the tuner board. The board tunes to the correct FM station and has a small pre-amp and 3.5mm audio jack. We hook a standard stereo audio cable to the board and then connect that to a standard audio amplifier which drives a set of waterproof consumer “deck” speakers.

The `Si4703` board we purchased (and most `Si4703` boards) did not have a separate FM antenna jack, instead the board pulls the signal from the shield of the audio cable. This is a less-than-ideal situation for FM reception. Madis Kaal [lists various options](http://www.nomad.ee/micros/silicon_radio/index.shtml) for dealing with this (also saved to the resources directory as *Si4703_MadisKaal_Antenna.pdf*). I decided to create a “breakout cable” that separates audio from the antenna.

#### Antenna | Audio Breakout Cable
The breakout cable is a stereo 3.5mm audio jack on one end (which attaches to the FM board) and the other end splits into a 3.5mm audio jack and a F-type RF connector (coax). The 3.5mm jack on the Si4703 board connects to a 3-part audio cable the 3 parts being [Tip, Ring and Shield/Sleeve aka TRS](https://en.wikipedia.org/wiki/Phone_connector_(audio)) which correspond to audio left, audio right and [antenna feed line](https://en.wikipedia.org/wiki/Feed_line). In the breakout cable the feedline is connected to the center conductor of the [F connector](https://en.wikipedia.org/wiki/F_connector) and the shield of the F connector is connected to a new ground line running to the board. On the audio side of the breakout cable the tip and ring are connected to the tip and ring again (IE left audio and right audio are as expected) and a new ground line is run to the shield of the audio cable (thus disconnecting the antenna feed line from the audio cable).

I connected the F connector to a standard dipole FM antenna via 75Ω coaxial cable. The audio cable is connected to a standard stereo audio amplifier and outdoor waterproof speakers.

### LEDs
384 `APA-102` LEDs curve along the left side of the front panel providing visual feedback of the current station. A small strip of 37 LEDs is on the bottom right side of the front panel providing feedback of current volume level. For wiring simplicity, all the LEDs are actually wired as a single continuously addressed strip of 421 LEDs with a small 6-inch jumper in between the station and volume LEDs so that to the volume LEDs could be physically and visually separated.

We chose APA-102 LEDs (over say WS2812B aka Adafruit Neopixels) since the APA-102s are a 4-wire spec with a dedicated clock line. Which essentially means they have very loose timing requirements because if you are interrupted by for example an interrupt you can just continue on after the interrupt with the correct clock signal. We actually read the photogates using interrupts so we don't miss any readings and can have very responsive dials. This would not have been possible with WS2812Bs. 

Again, like the Vernier photogates, the APA-102s are a 5V component so we used the 74AHCT125 level shifter to shift our 3.3V output from the micro:bit to the 5V level the APA-102s want. Although many 5V LEDs will sort-of tolerate a 3.3V input it is more robust to supply them with the 5V signal they expect.

### Breadboard Diagram:
Microbit controlling FM Receiver and APA102 LED light string:
![breadboard diagram](diagrams/final_breadboard.png "Breadboard Diagram")

#### Electronics Parts List:
 * 3.3V BBC micro:bit
 * 5V APA102 LED Strip (aka Adafruit Dotstar)
 * 2x 1000 µF, 6.3V Capacitor
 * 3V Si4703 FM Digital Tuner Board
 * 4x 5V Vernier VPG-BTD Photogates
 * 4x 5V Vernier BTA-ELV Analog Protoboard Adapter
 * 2x 74AHCT125 Level Shifter 3V <-> 5V
 * Adafruit DragonTail for micro:bit

The capacitors are attached near the APA-102 strip where it is being powered so that [“the initial onrush of current won't damage the pixels”](https://learn.adafruit.com/adafruit-neopixel-uberguide/powering-neopixels).

<hr>

<a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by-sa/4.0/88x31.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by-sa/4.0/">Creative Commons Attribution-ShareAlike 4.0 International License</a> except where otherwise noted.
