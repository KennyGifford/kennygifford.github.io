---
layout: page
title: Solar Charged Night Light
subtitle: for teaching STEM in Armenia
---
I collaborated with MIT students Caitlin Ogoe and Kanokwan Tungkitkancharoen to develop a project for the TUMO program.

The goal of the project was to teach basic electronics, coding, and 3D modelling concepts over multiple workshops to students ages 14~20.

I helped design the circuit for a solar charged, battery powered, RGB LED with WiFi control. This circuit was to be enclosed by a 3D printed shroud to project custom shadow patterns onto the ceiling.

### Constraints
* Total budget of \$250
    * 5 kits, so \$50 per kit
* WiFi-enabled MCU 
    * for teaching web application development
* Bright RGB LED
    * enough to project hard shadows on ceiling
* \> 30 mins run time, \< 3 hrs charge time

### Provided Resources
* 3D printers and filament
* Soldering station
* Breadboarding wire

### Parts Selection (per kit)
<table>
  <tr>
    <th>Quantity</th>
    <th>Part</th>
    <th>Details</th>
    <th>Reference</th>
  </tr>
  <tr>
    <td>1</td>
    <td>MCU</td>
    <td>ESP32-WROOM32, USB-C</td>
    <td>
        <a href="https://a.co/d/eCImBkG">Amazon</a>
        <br>
        <a href="https://www.espressif.com/sites/default/files/documentation/esp32-wroom-32_datasheet_en.pdf">Datasheet</a>
    </td>
  </tr>
  <tr>
    <td>1</td>
    <td>RGB LED</td>
    <td>3W, common anode</td>
    <td>
        <a href="https://cdn-shop.adafruit.com/product-files/2530/FD-3RGB-Y2.pdf">Adafruit</a>
    </td>
  </tr>
  <tr>
    <td>1</td>
    <td>Battery Charger</td>
    <td>bq25185, USB-C, solar</td>
    <td>
        <a href="https://www.adafruit.com/product/6091">Adafruit</a>
    </td>
  </tr>
  <tr>
    <td>1</td>
    <td>Battery</td>
    <td>LiPo, 3.7V, 3000mAh</td>
    <td>
        <a href="https://a.co/d/45iwyPS">Amazon</a>
    </td>
  </tr>
  <tr>
    <td>1</td>
    <td>Solar Cell</td>
    <td>5V, 0.6W</td>
    <td>
        <a href="https://www.adafruit.com/product/5856">Adafruit</a>
    </td>
  </tr>
  <tr>
    <td>2</td>
    <td>Breadboard</td>
    <td>400 Point</td>
    <td>
        <a href="https://a.co/d/azNvw7v">Amazon</a>
    </td>
  </tr>
  <tr>
    <td>3</td>
    <td>Transistor</td>
    <td>TIP120, NPN</td>
    <td>
        <a href="https://cdn-shop.adafruit.com/datasheets/TIP120.pdf">Adafruit</a>
    </td>
  </tr>
  <tr>
    <td>-</td>
    <td>Resistors</td>
    <td>variety, 1/2W, 1% tolerance</td>
    <td>
        <a href="https://a.co/d/11eQJZm">Amazon</a>
    </td>
  </tr>
</table>

### Parts Design
*To reduce shipping costs we used Amazon and Adafruit. We prioritized Adafruit for quality, documentation, and support. Orders were placed with little advance, so product availability majorly influenced part selection.*

##### MCU
The basis of this project was "breadboardability" to allow for experimentation and mistakes.

As such, the ESP32-WROOM32 dev board was selected for its pin mounting, as well as the built-in antenna, regulator, low-power modes, and ease of programming via USB-C.

##### LED
The 3W RGB LED was selected for its robustness to possibly allow adhesive mounting, high peak brightness, and wide dispersion over standard LED packages.

##### Charger
The battery charger was selected somewhat arbitrarily as the cheapest and newest product in its category. The status LEDs were crucial for troubleshooting and the USB-C connector with "Power Path to Load" allowed for bypassing the solar cell which was helpful when working indoors. 

##### Battery
The charger constrained the battery selection by supporting a voltage up to 4.2V and recommending a maximum output of 1A. Most available batteries were LiPo at 3.7V and could theoretically support 3.7W of draw - this would be adequate for our 3W LED and 0.264W MCU (80mA average draw at 3.3V). The capacity was calculated from our requirement for 30 minutes run time:

(3W + 0.264W) / 3.7V * 0.5 hrs = 441mAh

Yet, batteries with much higher capacity were cheapest so the 3000mAh LiPo battery was selected.

##### Solar Panel
The charger also recommended a 5~7V solar panel. But, with our constraint of charging enough for 30 mins of run time in 3 hrs of charge time we had to calculate the minimum generated wattage:

3.264W * 0.5 hrs / 3 hrs = 0.544W

So, the cheapest 5V 0.6W solar panel was selected.

##### Transistors
Because of a lack of availiability in adequate MOSFETs, we opted for TIP120 transistors for cheapness and because they were recommended for high power LEDs. However, our desire for precise current/brightness control of the LEDs at low voltages caused them to be a bit of a headache to use...

### Circuit Design - PWM Control of the LEDs
![pwm_circuit](/assets/night_light/pwm_circuit.png)
This is the basic circuit to control and drive the LEDs with PWM.

VCC is sourced from the load pins of the battery charger, and should be the same as the battery voltage, so around 3.7V.

R1-R3 must be calculated to normalize the brightness of each color channel.
R4-R6 must be calculated for proper transistor base current and switching.

From the LED datasheet we find a few necessary plots:
##### Fig. 1 - Luminosity vs. Forward Current
![luminosity_vs_current](/assets/night_light/luminosity_vs_current.png)
From this plot we find the brightness of each LED according to the current flowing through them.

We’ll use 100mA for easier math and approximately 14 lumens of brightness per LED, thus 42 total lumens, which should be adequate for a night light.

##### Fig. 2 - Forward Current vs. Forward Voltage
![current_vs_voltage](/assets/night_light/current_vs_voltage.png)
The LEDs have nonlinear internal resistance, so from this plot we use our desired current to find the voltage drop across each LED in the circuit. 

The different colors of LEDs also have different characteristics, so at 100mA we expect 2.1V drop over the red LED and 3.05V drop over the blue and green LEDs.

##### Fig. 3 - Collector-Emitter Voltage vs. Current
![transistor_voltage_vs_current](/assets/night_light/transistor_voltage_vs_current.png)
From this plot we can find the voltage drop across the transistor’s collector and emitter pins. At the desired current of 100mA, this drop is 0.7V.

We can also find how much current is required at the base to drive this collector current. The collector current must be 250 times the base current, thus we require 100mA / 250 = 0.4mA at the base.

However, this is only the minimum current required so to guarantee saturation we will use a base current of 5 times this, so 2mA.

##### Fig. 4 - ESP32 DC Characteristics
![esp32_dc_characteristics](/assets/night_light/esp32_dc_characteristics.png)
The dev-board provides the ESP32 with regulated VDD = 3.3V.
So, from this table we find that the GPIO pins can output at least 0.8 × 3.3 = 2.64V, and can typically source 40mA.
We’d like 2mA of current at the base, so this current is adequate.

##### Finding R1-R3
To determine R1-R3 we will use Ohm’s law, R = V/I.

We know we want 100mA of LED current, and that VCC is 3.7V.
We also found that the red LED has 2.1V drop at 100mA and the green and blue LEDs have 3.05V drop.
Finally, we found the transistor has a collector-to-emitter voltage drop, VCE, of 0.7V at 100mA.

The value for R1 is (3.7 - 2.1 - 0.7)V / 100mA = 9Ω ~= 10Ω.
The value for R2 and R3 is (3.7 - 3.05 - 0.7)V / 100mA = -0.05Ω ~= 0.

For R1, we’re good since we have 10Ω resistors.
However, for R2 and R3 we barely have enough voltage to drive 100mA through the green and blue LEDs so we will use a 0Ω or 1Ω resistor as nothing more than a 1/2W fuse.

##### Finding R4-R6
We know IB = 2mA, and that the GPIO pins can output at least 2.64V, but will most likely output 3.3V.

One last thing to consider is the base-to-emitter voltage drop, VBE, of the transistor from Fig. 3 which is 1.25V at 100mA.

So, the value for resistors R4-R6 is (3.3 - 1.25)V / 2mA = 1,025Ω ~= 1kΩ.

Finally, we have the complete PWM circuit:
![pwm_circuit_complete](/assets/night_light/pwm_circuit_complete.png)