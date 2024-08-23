Hardware:
Adafruit CLUE - nRF52840 Express with Bluetooth LE

Adafruit AMG8833 IR Thermal Camera Breakout (versions 1 & 2)
OR
Adafruit MLX90640 Thermal Camera (version 3 & 4)


Software:
There are four versions; 
1. Integrates the Moiré animated GIF with 9 internal sensors and the AMG8833
2. Integrates the Moiré animated GIF, 9 internal sensors, the AMG8833, and Bluetooth to communicate with the Handheld sensors (SGP30 connected to the ItsyBitsy nRF52840 Express).
3. Integrates the Moiré animated GIF, 9 internal sensors, the MLX90640, and Bluetooth to communicate with the Handheld sensors (SGP30 connected to the ItsyBitsy nRF52840 Express).
4. Integrates the Moiré animated GIF, 9 internal sensors, the MLX90640 (interpolated), and Bluetooth to communicate with the Handheld sensors (SGP30 connected to the ItsyBitsy nRF52840 Express), and button beeps.

Make sure to follow the instructions for the Animated GIF Player (https://learn.adafruit.com/adafruit-clue/animated-gif-player), otherwise, the GIF will not work.

The animated GIF of the Moire pattern was supplied by CompaniaHill from The Fleet Workshop.

This sketch originated from Adafruit's Clue Sensor Plotter example (https://github.com/adafruit/Adafruit_Arcada/blob/master/examples/full_board_tests/arcada_clue_sensorplotter/arcada_clue_sensorplotter.ino) with their Animated GIF code. 

The integration of the Moire pattern, AMG8833 thermal camera, and Bluetooth was done by notSpock on The Fleet Workshop. Thank you, sir!!

The integration of the MLX90640 was with the help of ChatGPT. Thanks to those folks who made that AI quite useful to me!
