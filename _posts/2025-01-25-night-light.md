---
layout: post
title: Solar Charged Night Light
subtitle: for teaching STEM in Armenia
toc: true
tags: [ESP32]
---
This winter I collaborated with MIT students Caitlin Ogoe and Kanokwan Tungkitkancharoen to develop a project for the TUMO program.

The goal of the project was to teach basic electronics, coding, and 3D modelling concepts over multiple workshops to students ages 14~20.

I helped design the circuit for a solar charged, battery powered, WiFi controlled, RGB night light. This circuit was to be enclosed by a 3D printed shroud to project custom shadow patterns which were designed by the students.

This is a picture Kano took from Armenia after she helped the students design the enclosures. You can see three of the completed circuits.<br>
![finished_night_lights](/assets/night_light/finished_night_lights.png)<br>

### Constraints
* Total budget of \$250
    * 5 kits, so \$50 per kit
* WiFi-enabled MCU 
    * for teaching web application development
* Bright RGB LED
    * enough to project hard shadows on ceiling
* \> 30 mins run time, \< 3 hrs charge time

### Workshop Resources
* 3D printers and filament
* Soldering station
* Breadboarding wire

### Parts Selection
<table>
  <tr>
    <th>Qt/kit</th>
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

## Parts Design
*To reduce shipping costs we used Amazon and Adafruit. We prioritized Adafruit for quality, documentation, and support. Orders were placed with little advance, so product availability majorly influenced part selection.*

#### MCU
We chose a ESP32-WROOM dev board because of the built-in antenna, regulator, low-power modes, and ease of programming via USB-C.

#### LEDs
We selected a 3W RGB LED for its large heat sink (for possible adhesive mounting), high peak brightness, and wide dispersion over standard LED packages.

#### Charger
To charge the battery we picked an Adafruit charging board. It has status LEDs, a USB-C connector, and "Power Path to Load" for bypassing the solar panel - which is helpful when working indoors.

#### Battery
After choosing the battery charger, we had constraints for the battery itelf. The charger supports a voltage up to 4.2V and recommends a maximum output of 1A. Most commonly available batteries are LiPo at 3.7V and can theoretically support 3.7W of draw. We know this is adequate because our main load is the 3W LED and the MCU should only occasionally need about 264mW☆.

>☆ from the datasheet the ESP32-WROOM uses 80mA on average.<br>
At 3.3V this is: 0.08A * 3.3V = 0.264W

We found the battery capacity required for 30 minutes of run time to be about 441mAh☆.

>☆ (3W + 0.264W) ÷ 3.7V × 0.5 hrs = 441mAh

But, batteries with much higher capacity are cheap so we went with 3000mAh regardless. In theory this could give us about 3.5 hrs of run time.

#### Solar Panel
The charger also recommends using a 5~7V solar panel. But, in order to charge 30 mins worth of run time within 3 hrs of charging we also needed to calculate the solar panel wattage. The minimum wattage required was about 0.544W☆.

>☆ 3.264W × 0.5 hrs ÷ 3 hrs = 0.544W

So, we chose a cheap 5V 0.6W solar panel. 

#### Transistors
We could not find adequate MOSFETs to ship in time, so we opted for TIP120 transistors which were available. They were cheap and were recommended for use with high power LEDs, perfect! Right? 

Unfortunately, our desire for precise current/brightness control of the LEDs and the low voltages we had available to us made using these a bit of a headache...

## Circuit Design - PWM
![pwm_circuit](/assets/night_light/pwm_circuit.png)<br>
This is the basic circuit to control and drive the LEDs with PWM.

V<sub>CC</sub> is sourced from the load pins of the battery charger, and should be the same as the battery voltage, so around 3.7V.

R1-R3 must be calculated to normalize the brightness of each color channel.
R4-R6 must be calculated for proper transistor base current and switching.

From the LED datasheet we find a few necessary plots:
#### Fig. 1 - Luminosity vs. Forward Current
![luminosity_vs_current](/assets/night_light/luminosity_vs_current.png)<br>
From this plot we find the brightness of each LED according to the current flowing through them.

We’ll arbitrarily pick 100mA for easier math and approximately 14 lumens of brightness per LED, thus 42 total lumens, which should be adequate for a night light.

#### Fig. 2 - Forward Current vs. Forward Voltage
![current_vs_voltage](/assets/night_light/current_vs_voltage.png)<br>
The LEDs have nonlinear internal resistance, so from this plot we use our desired current to find the voltage drop across each LED in the circuit. 

The different colors of LEDs also have different characteristics, so at 100mA we expect 2.1V drop over the red LED and 3.05V drop over the blue and green LEDs.

#### Fig. 3 - Collector-Emitter Voltage vs. Current
![transistor_voltage_vs_current](/assets/night_light/transistor_voltage_vs_current.png)<br>
From this plot we can find the voltage drop across the transistor’s collector and emitter pins. At the desired current of 100mA, this drop is 0.7V.

We can also find how much current is required at the base to drive this collector current. The collector current must be 250 times the base current, thus we require 100mA / 250 = 0.4mA at the base.

However, this is only the minimum current required so to guarantee saturation we will use a base current of 5 times this, so 2mA.

#### Fig. 4 - ESP32 DC Characteristics
![esp32_dc_characteristics](/assets/night_light/esp32_dc_characteristics.png)<br>
The dev board provides the ESP32 with regulated V<sub>DD</sub> = 3.3V. So, from this table we find that the GPIO pins can output at least 0.8 × 3.3 = 2.64V, and can typically source 40mA.

We’d like 2mA of current at the base, so this current is adequate.

#### Finding R1-R3
To determine R1-R3 we will use Ohm’s law, R = V ÷ I.

We know we want 100mA of LED current, and that V<sub>CC</sub> is 3.7V.
We also found that the red LED has 2.1V drop at 100mA and the green and blue LEDs have 3.05V drop.
Finally, we found the transistor has a collector-to-emitter voltage drop, V<sub>CE</sub>, of 0.7V at 100mA.

The value for R1 is (3.7 - 2.1 - 0.7)V ÷ 100mA = 9Ω ~ 10Ω.
The value for R2 and R3 is (3.7 - 3.05 - 0.7)V ÷ 100mA = -0.05Ω ~ 0Ω.

For R1, we’re all set since we have 10Ω resistors.
However, for R2 and R3 it seems we barely have enough voltage to drive 100mA through the green and blue LEDs...

We could decide to step down our current and recalculate, but we're not going for 100% color accuracy and we're not even going to get it with this method anyways. This is due to our big assumption of V<sub>CC</sub> equalling exactly 3.7V when really this depends on the voltage of the battery and is not regulated to be fixed at 3.7V.

So, we'll simply use a 0Ω or 1Ω resistor to try not to restrict current too much while still designing a point of failure (this 1/2W resistor) besides the LED itself incase of over-current.

#### Finding R4-R6
We know I<sub>B</sub> = 2mA, and that the GPIO pins can output at least 2.64V, but will most likely output 3.3V.

One last thing to consider is the base-to-emitter voltage drop, V<sub>BE</sub>, of the transistor from Fig. 3 which is 1.25V at 100mA.

So, the value for resistors R4-R6 is (3.3 - 1.25)V ÷ 2mA = 1,025Ω ~ 1kΩ.

Finally, we have the complete PWM circuit:<br>
![pwm_circuit_complete](/assets/night_light/pwm_circuit_complete.png)<br>

## Circuit Design - Complete
With the PWM circuit designed, all that is left is to  power and control it.<br>
![solar_power_circuit](/assets/night_light/solar_power_circuit.png)<br>

#### Charger
We'll connect the LOAD pins to the LEDs and ESP32, the DCIN pins to the solar panel, and the BATT pins to the battery.

#### Battery

>I only realized after getting the batteries that their JST connector was smaller than that on the charger... oops!
So, I cut off the connector and attempted to use a screw terminal. However, the wire gauge was too small for the terminal to bite. I also realized this was the inferior "leaf-spring" terminal type and it could hardly bite 22 gauge wire either. As a last resort I soldered the wires directly to the screw terminal, and they seemed firmly attached.
But that's not all, because the screw terminal also had a wider pitch than 0.1", but fortunately it still fit the breadboard when it was placed diagonally. Phew.

After connecting the battery to the charger the power indicator lights up. Good sign!<br>
![battery_to_charger](/assets/night_light/battery_to_charger.jpg)<br>

#### Solar Panel
We connect the charger to the breadboard rails with the LOAD pins, and then connect the solar panel to the rails. 

>I had soldered 22 gauge wire directly to the solar panel pads and this wire fit snuggly into the breadboard without the need for a connector.

The PWR LED remained on while the CHRG LED remained off because we were indoors.<br>
![panel_to_charger](/assets/night_light/panel_to_charger.jpg)<br>

But, after plugging into USB-C we saw the charging indicator turn on, sweet!<br>
![charger_is_charging](/assets/night_light/charger_is_charging.jpg)<br>

#### MCU
Then we connected the ESP32 dev-board to the rails and saw the PWR indicator turn on.<br>
![esp_to_charger](/assets/night_light/esp_to_charger.jpg)<br>

#### LEDs
Finally, we followed the PWM circuit from above to wire up the RGB LEDs using the appropriate resistors.

The V<sub>CC</sub> in the circuit comes from the LOAD pins of the charger, and should be 3.7V when the battery is nearly full.

We connected GPIO pins 26, 27, and 25 to the Red, Green, and Blue cathodes respectively.

Below is the slight mess of the final circuit. You can't see the LED itself, but it was off because the base pins of the transistors were not being powered when no code was being ran (i.e. no connection from the cathodes to ground was made).<br>
![completed_circuit](/assets/night_light/completed_circuit.JPG)<br>

*Unfortunately I don't have pictures of the LEDs illuminated because I wasn't planning to post about this project and I didn't take any before they left for the trip. But, I heard the projects turned out great for the students!*
