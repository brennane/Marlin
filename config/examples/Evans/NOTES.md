# RAMPS 1.4 Board:

![Board][pinout] ([Pinout][input])

[pinout]: https://reprap.org/wiki/File:Arduinomega1-4connectors.png>

Hint: When testing, connect black common to USB connector or an AUX ground pin.

# RAMP Fan Extender


~~~
                   > GND   P11 <
+-----------------+---------------------+
| NC  | +5V | GND |  GND | VIN | Fan0 + | > FAN-OUT <
| NC  | +5V | GND |   R    R R |      - |
| D6  | +5V | GND |   R        |      + |
| D11 | +5V | GND |   R    R R |      - | > GND <
+-----------------+---------------------+
~~~

Pin-11 and Ground goes in top.  Top and bottom side pins are used to the extruder fan.


See `Configuration_adv.h` 

~~~
#define E0_AUTO_FAN_PIN 11 /* EVANS: was -1 */
#define EXTRUDER_AUTO_FAN_TEMPERATURE 50
#define EXTRUDER_AUTO_FAN_SPEED 255   // 255 == full speed
~~~



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

From 
[Thorlabs PID Tutorial](https://www.thorlabs.com/tutorials.cfm?tabID=5DFCA308-D07E-46C9-BAA0-4DEFC5C40C3E)
and 
[RepRap PID Tuning](https://reprap.org/wiki/PID_Tuning)



| Parameter Increased | Rise Time      | Overshoot      | Settling Time  | Steady-State Error     | Stability |
| ---                 | ---            | ---            | ---            | ---                    | ---       |
| $K_p$               | Decrease       | Increase       | Small Change   | Decrease               | Degrade   |
| $K_i$               | Decrease       | Increase       | Increase       | Decrease Significantly | Degrade   |
| $K_d$               | Minor Decrease | Minor Decrease | Minor Decrease | No Effect              | Improve (for small Kd) |


**Tuning**

The default Marlin M303 calculates a set of Ziegler-Nichols "Classic"
parameters based on the Ku (Ultimate Gain) and the Pu (Ultimate Period), where
the Ku and Pu are determined by searching for a biased BANG-BANG oscillation
around an average power level that produces oscillations centered on the
setpoint. 

In general a PID circuit will typically overshoot the SP value slightly and
then quickly damp out to reach the SP value.

## manual tuning:

* To tune your PID controller manually, first the integral
  and derivative gains are set to zero. 
* Increase the proportional gain until you observe oscillation in the output. 
  Your proportional gain should then be set to roughly half this value. 
* Increase the integral gain until any offset is corrected for on a time scale
  appropriate for your system.  If you increase this gain too much, you will
  observe significant overshoot of the SP value and instability in the circuit. 
* Derivative gain can then be increased. Derivative gain will
  reduce overshoot and damp the system quickly to the SP value. If you increase
  the derivative gain too much, you will see large overshoot (due to the
  circuit being too slow to respond). 

## Ziegler-Nochols

The Ziegler-Nichols method for PID tuning offers a bit more structured guide to
setting PID values. Again, you’ll want to set the integral and derivative gain
to zero. Increase the proportional gain until the circuit starts to oscillate.
We will call this gain level Ku. The oscillation will have a period of Pu.
Gains are for various control circuits are then given below in the chart.

| Control Type  | Kp         | Ki              | Kd            |
| ---           | ---        | ---             | ---           | 
| P             | $0.50 K_u$ | -               | -             |
| PI            | $0.45 K_u$ | $1.2 K_p / P_u$ | -             |
| PID           | $0.60 K_u$ | $2   K_p / P_u$ | $K_p P_u / 8$ |

# Current Start G-Code

~~~
;; EXTRUDER SECTION
;; M92 E90; E98.2 ; [cold extrude 120mm from E90]
; M92 E196;
; M200 D0 ; force linear extrusion calc 

;; PID TUNING 
;; M303 E0 C10 Swww
;; M221 S85

M301 E0 P10 I0.95 D35
;; M301 E0 P11.75 I0.9 D35.00  ;; 218C
;; M301 E0 P11.37 I0.7998 D40.98 ;; 220C
;; M301 E0 P8.92 I0.59 D33.62  ;; 250C
;; M301 E0 P9.06 I0.63 D32.65  ;; 290C
M304 P109 I21 D143
M503 ; dump settings to log

;; FLOW RATE
;; M221 S70

;; debugging - "set position"
; G92 X50 Y50 Z50
; G1 Pos || G1 +/- RelPos
; G1 X Y [F4800]
; G1 Z [F120]

;; CALIBRATATION ;;
; M302         ; report current cold extrusion state
; M302 P0      ; enable cold extrusion checking
; M302 P1      ; disable cold extrusion checking
~~~

# Printer Prefs (Repetier Host)

- usbmodem40xxxx
- baud 2500000
- stop bits 1
- parity none
- xfer ascii
- rcv cache size 63
- timeout 40s

# AnyCubic Base

Size: 215mm x 215mm

1. PLA:50-70℃
2. Flexible filament: 50-70℃
3. ABS:100-125℃ ( 110℃ is recommended )
4. PC:100-130℃ ( 120℃ is recommended )
5. Nelon:90-120℃ ( 110℃ is recommended )
6. PP:100-130℃ ( 120℃ is recommended )
7. PETG: 50-70℃

# Titan AERO update

1. Moved Xstop from XMIN to XMAX location on RAMPS board
2. New Motor:
   - Step Angle: 0.9°
   - Rated Current: 1.68A
   - Body length: 40mm
   - Faceplate: NEMA 17
   - Motor cable length: 1m
3. ref.  <https://e3d-online.dozuki.com/c/Titan_Aero_Firmware_Guides>
4. motor steps/mm -> 837 due to 0.9 motor _and_ adding gear.
5. will need to redo PID tuning, using aero thermistor
6. themistor #define TEMP_SENSOR_0 5


# PINDAv2

Per [this document](https://github.com/ultimachine/Einsy-Rambo/blob/1.1a/board/Project%20Outputs/Schematic%20Prints_Einsy%20Rambo_1.1a.PDF)
the PINDAv2 has the following layout:

J15
Pin 1: Z-THERM -> D47
Pin 2: Z-MIN   -> D32
Pin 3: GND
Pin 4: VCC 

Pindav2 should also work at 5V.

Newer marlin has support for the temp sensor. (this config is not there yet)

Any arguments left out of G29 will use your configured defaults.

By default G28 disables bed leveling. Follow with M420 S to turn leveling
on, or use RESTORE_LEVELING_AFTER_G28 to automatically keep leveling on
after G28.

To save time and machine wear, save your matrix to EEPROM with M500 and in
your slicer’s “Starting G-code” replace G29 with M420 S1 to enable your
last-saved matrix.


# UBL Mesh

Ref.  <https://www.3dmakerengineering.com/blogs/3d-printing/unified-bed-leveling-marlin>

This takes a mesh and saves it to memory

~~~
M190 S{material_bed_temperature}
G28
G29 P1
G29 P3
G29 F10
G29 S1
G29 A
G29 L1
M500
~~~



~~
G29 L1 / M420 L1 ; load mesh from EEPROM
; M420 Z1 ; enable fade?
M420 V1 T0 ; show mesh
~~
