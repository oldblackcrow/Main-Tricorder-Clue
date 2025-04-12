Hardware:
Adafruit CLUE - nRF52840 Express with Bluetooth LE

Adafruit AMG8833 IR Thermal Camera Breakout (versions 1 & 2)
OR
Adafruit MLX90640 Thermal Camera (version 3 & 5)

-----------------------------
YOU MUST KNOW - when I updated the firmware and libraries on the Clue last month, I could NOT get the Animated GIF to work with the other screens. It worked fine on its own, but as soon as I added a second screen, the GIF was very crunchy and slow. This is why I wanted the Moiré to be software-based in Version 5 instead of relying on an animated image. But if you haven't updated your Clue in a couple years, you should be able to get it to work.
----------------------------

Software:
There are FIVE versions; 
1. Integrates the Moiré animated GIF with 9 internal sensors and the **AMG8833**
2. Integrates the Moiré animated GIF, 9 internal sensors, the **AMG8833**, and Bluetooth to communicate with the Handheld sensors (SGP30 connected to the ItsyBitsy nRF52840 Express).
3. Integrates the Moiré animated GIF, 9 internal sensors, the **MLX90640**, and Bluetooth to communicate with the Handheld sensors (SGP30 connected to the ItsyBitsy nRF52840 Express).
4. Integrates the Moiré animated GIF, 9 internal sensors, the **MLX90640** (interpolated), and Bluetooth to communicate with the Handheld sensors (SGP30 connected to the ItsyBitsy nRF52840 Express), and button beeps.
5. Moiré animated GIF is NO LONGER NEEDED!
   Mode0 - Moiré is now functional by connecting it to the magnetometer! Check out my video that shows this - https://youtu.be/L1sdvRsTv_E
   Mode1 - Light meter displayed in LUX
   Mode2 - Lambda is the same EXCEPT the white LED now only comes on when about a CM away from the sensor... so it doesn't blind you every time you go to that screen!
   Mode3 - dB meter is now nicer looking with a Sensor Sweep/Radar looking screen. Red dots appear as sound is detected with a circular "grid" that shows dB
   Mode4 - ENVIRONMENT Condensed. Now, atmospheric pressure, altitude, temp, humidity, Heat Index, CO2, and VOCs are all on one screen! The CO2 and VOC will turn GREEN when Bluetooth is connected and then pull that data to the Clue. All the sensors also have threshholds that are tied to colow. Nominal - blue background. Warning - Yellow background. Danger - Red background. You still need the ItsyBitsy/SGP30 handheld for this part of the sketch to work.
   Mode5 - Ricci Scalar Field... well, not really. But it's a nice simulation based on the magnetometer and accelerometer. Kinda neat but not really functional.
   Mode6 - MLX90640 is still there looking all nice with minimum and maximum temps at the center of the screen.
   Also, in this version, the back button works no matter what screen you're on and there's a nice double beep for each button press. 

If using versions 1 - 4, Make sure to follow the instructions for the Animated GIF Player (https://learn.adafruit.com/adafruit-clue/animated-gif-player), otherwise, the GIF will not work. 
If you have better luck than me with the Animated GIF, please fork the F out of this and let me know! Thanks!

The animated GIF of the Moire pattern was supplied by CompaniaHill from The Fleet Workshop.

This sketch originated from Adafruit's Clue Sensor Plotter example (https://github.com/adafruit/Adafruit_Arcada/blob/master/examples/full_board_tests/arcada_clue_sensorplotter/arcada_clue_sensorplotter.ino) with their Animated GIF code. 

Versions 1 - 4 integration of the Moire pattern, AMG8833 thermal camera, and Bluetooth was done by notSpock on The Fleet Workshop. Thank you, sir!!

The integration of the MLX90640 and most of Version 5 was created with the help of ChatGPT. Thanks to those folks who made that AI quite useful to me!

Don't forget to subscribe to my YouTube channel so you can keep up with all the updates I make on the Tricorder! https://www.youtube.com/OldBlackCrowCS
