# RAMPS 1.4 Board:

![Board][pinout] ([Pinout][input])

[pinout]: https://reprap.org/wiki/File:Arduinomega1-4connectors.png>

Hint: When testing, connect black common to USB connector or an AUX ground pin.

# Motors:

The motors are 1.8 degrees and rated at 1.2A 12V

- DRV8825: Imax = 2 * Vref   (which means if Imax = 1.5A, then Vref is to be set to 1.5/2 V = 0.75V)
- A4988:   Imax = 2.5 * Vref // with Rcs = 0,05 Ohm

<https://blog.prusaprinters.org/calculator/>

## A4988

- STEPS: `{ 80, 80, 4000, 90 }`

### DRV8825

- 80% power - 0.475 Vref  -- was this but might be causing issues
- 83% power - 0.500 Vref
- 92% power - 0.550 Vref  -- 2019.12.16 using this
- 96% power - 0.575 Vref
- 100% power - 0.600

- STEPS: `{ 160, 160, 8000, 196 }`

Find out the Vref location on the breakout board - Pololu comes with two
locations: a via close to the chip and the center point of the potentiometer;
clones mostly only provide the center point of the potentiometer

# Optical Endstops

Adding in cheap endstops from amazon (middle picture at) 
<http://marlinfw.org/docs/hardware/endstops.html> the end-stops need
to no longer invert the signal since NC (safety feature) works.

Reminder:

- "V" is Volt (+) and connects to RAMPS "+"
- "G" is Ground (=) and connects to RAMPS "-"
- "S" is Singal (-) and connects to RAMPS "S"

