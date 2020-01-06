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

# Parts References:

- https://cults3d.com
- https://tinkercad.com
- https://thingiverse.com

# Materials

- Essentium

# PID settings

[PID Tutorial](https://www.thorlabs.com/tutorials.cfm?tabID=5DFCA308-D07E-46C9-BAA0-4DEFC5C40C3E)


| Parameter Increased | Rise Time      | Overshoot      | Settling Time  | Steady-State Error     | Stability |
| ---                 | ---            | ---            | ---            | ---                    | ---       |
| $K_p$               | Decrease       | Increase       | Small Change   | Decrease               | Degrade   |
| $K_i$               | Decrease       | Increase       | Increase       | Decrease Significantly | Degrade   |
| $K_d$               | Minor Decrease | Minor Decrease | Minor Decrease | No Effect              | Improve (for small Kd) |


**Tuning**

In general the gains of P, I, and D will need to be adjusted by the user in
order to best servo the system. While there is not a static set of rules for
what the values should be for any specific system, following the general
procedures should help in tuning a circuit to match one’s system and
environment. In general a PID circuit will typically overshoot the SP value
slightly and then quickly damp out to reach the SP value.

Manual tuning of the gain settings is the simplest method for setting the PID
controls. However, this procedure is done actively (the PID controller turned
on and properly attached to the system) and requires some amount of experience
to fully integrate. To tune your PID controller manually, first the integral
and derivative gains are set to zero. Increase the proportional gain until you
observe oscillation in the output. Your proportional gain should then be set to
roughly half this value. After the proportional gain is set, increase the
integral gain until any offset is corrected for on a time scale appropriate for
your system. If you increase this gain too much, you will observe significant
overshoot of the SP value and instability in the circuit. Once the integral
gain is set, the derivative gain can then be increased. Derivative gain will
reduce overshoot and damp the system quickly to the SP value. If you increase
the derivative gain too much, you will see large overshoot (due to the circuit
being too slow to respond). By playing with the gain settings, you can maximize
the performance of your PID circuit, resulting in a circuit that quickly
responds to changes in the system and effectively damps out oscillation about
the SP value.

| Control Type  | Kp         | Ki              | Kd            |
| ---           | ---        | ---             | ---           | 
| P             | $0.50 K_u$ | -               | -             |
| PI            | $0.45 K_u$ | $1.2 K_p / P_u$ | -             |
| PID           | $0.60 K_u$ | $2   K_p / P_u$ | $K_p P_u / 8$ |

While manual tuning can be very effective at setting a PID circuit for your
specific system, it does require some amount of experience and understanding of
PID circuits and response. The Ziegler-Nichols method for PID tuning offers a
bit more structured guide to setting PID values. Again, you’ll want to set the
integral and derivative gain to zero. Increase the proportional gain until the
circuit starts to oscillate. We will call this gain level Ku. The oscillation
will have a period of Pu. Gains are for various control circuits are then given
below in the chart.

