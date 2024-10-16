# esphome-brunnen

## PCB

You can find a KiCad Model in this repository. The PCB can be mounted (aka glued) to the battery case of the [Waveshare Solar Power Manager D](https://www.waveshare.com/solar-power-manager-d.htm). Using the spacers that come with the Solar board, you can mount it to the PCB. 

### Parts

| Number | Component | Position on PCB|
| --------- | ------ | -------- |
| 1 | Solar Panel, 6-24V, see below | | |
| 1 | [Waveshare Solar Power Manager D](https://www.waveshare.com/solar-power-manager-d.htm)|
| 1 | MT3608 step up module | | |
| 1 | D1 Mini ESP32 with headers ||
| 1 | TL-136 Sensor, 4-20mA, 24V ||
| 1 | DS18B20 Sensor, waterproof ||
| 1 | JST-XH 2 Pin cable | |
| 2 | JST-XH 3 Pin cable | |
| 1 | JST-XH 4 Pin cable | |
| 1 | BC547 | Q1 |
| 1 | BC557 | Q2 |
| 1 | tantalum capacitor 35V  1uF | C1 |
| 1 | JST-XH 2 Pin Header | J3 |
| 2 | JST-XH 3 Pin Header | J1, J4 |
| 1 | JST-XH 4 Pin Header | J2 |
| 1 | DS18B20 Sensor, TO-92 package | U2 |
| 1 | Resistor, 4.7k | R1 |
| 1 | Resistor, 33k | R2 |
| 1 | Resistor, 100k | R3 |
| 2 | Resistor, 150k | R4, R5 (see note below) |
| 2 | Resistor, 1k | R6, R9 |
| 1 | Resistor, 150k | R7 |
| 1 | Resistor, 27k | R8 |
| 1 | Resistor, 10k | R10 |

### D1 Mini ESP32

Before adding the D1 Mini to your PCB (i highly recommend using headers), you need to install the software to the board. All subsequent updates can be done using OTA but the first installation requires an USB connection.

> [!CAUTION]
> DO NEVER CONNECT YOUR D1 BOARD VIA USB WHEN IT'S MOUNTED ON THE PCB

### Solar Panel

You can use any solar panel that provides between 6-24V. Choose wattage according to the placement of your panel. 

The resistors `R4` and `R5` are used to split the voltage of the solar panel into a range, that can be measured with an ESP32 gpio pin (max 3.3V). By using equally sized resistors, the voltage is cut by half, so this is fine for 6V panels.
Change the resistors to the following values, when using other panels, also change the multiply factor of the `solar_voltage` sensor in `brunnen.yaml`.

> [!WARNING]
> Some solar panels provide a slightly higher voltage than specified (e.g. 14V instead of 12V). To prevent any damage on the GPIO pin, use the resistor and multiply value of the next higher rated panel if you are not sure. This reduces the accuracy of the measured voltage a bit, but might save your D1.

| Panel | R4 | R5 | max V @ GPIO | `multiply` |
| ----- | -- | -- | ------|---------- |
| 6V | 150k | 150k| 3V |`multiply: 2`|
| 12V | 150k | 47k| 2.86|`multiply: 4.2`|
| 18V | 150k | 33k| 3.25V|`multiply: 5.5`|
| 24V | 150k | 22k| 3.07V |`multiply: 7.8`|

### MT3608 Step up Module

The pressure sensor requires 24V which are generated by a step up module. The module is disabled when the ESP32 is in deepsleep using a BC547 transistor to reduce the power consumption.

Before adding the MT3608 to your circuit, you need to measure and change the output voltage to 24V, use a multimeter for that task.

### Assembly

* Solder the 2 Pin JST-XH cable to your pressure sensor
* Solder one 3 Pin JST-XH cable to your waterproof DS18B20 sensor
* Solder one 1 Pin JST-XH cable to your MT3608 step-up
    * VIN
    * GND
    * VOUT
* Attach one 4 Pin JST-XH cable to your Waveshare Board:
    * GND
    * 5V
    * BAT+ (use the free terminal)
    * SOL+ (add cable together with solar panel to terminal)
* Add solar panel to Waveshare Board
* Attach batteries
* Solder all remaining parts to the PCB
* Check parts once again
* Connect J2 to JST-XH cable coming from Waveshare Board, your ESP32 should power up now
* Connect all remaining JST-XH cables
    * J1 - waterproof DS18B20
    * J3 - Pressure sensor
    * J4 - MT3608 Module

You should now see some sensor readings and can head over to the calibration of your pressure sensor. 

Note: to make the connections a bit more sturdy, you can also remove the terminals and solder all cables to the Waveshare Board

## Installation

### secrets.yaml

Create a `secrets.yaml` file which contains your WiFi-credentials:

```
wifi_ssid: "YOUR_SSID"
wifi_password: "YOUR_PASSWORD"
```

### Initial upload to ESP32

Attach your D1 Mini using USB to your computer, find out the correct COM port and run:

```
esphome run --device COM4 .\brunnen.yaml
```

The ESP32 should then reboot and connect to your wifi, note down it's IP.

> [!CAUTION]
> DO NEVER CONNECT YOUR D1 BOARD VIA USB WHEN IT'S MOUNTED TO THE PCB

### OTA Updates

When the initial upload is done, you can update your ESP32 using OTA updates. This does not require you to remove the board from the PCB. 

```
esphome run --device 10.0.50.132 .\brunnen.yaml
```

### D18B20 Sensors

Each of the D18B20 temperature sensors has a unique address that needs to be configured. Esphome logs all available devices when starting, you just need to figure out, which sensor is for the case and which one for water. 

```
[20:37:31][C][gpio.one_wire:020]: GPIO 1-wire bus:
[20:37:31][C][gpio.one_wire:021]:   Pin: GPIO5
[20:37:31][C][gpio.one_wire:080]:   Found devices:
[20:37:31][C][gpio.one_wire:082]:     0x5d4a131c35646128 (DS18B20)
[20:37:31][C][gpio.one_wire:082]:     0x8f00000e7dbdd128 (DS18B20)
```

### Pressure Sensor calibration

The pressure sensor gives around 4-20mA when submerged into water. Although this gives us a rough estimation, I recommend to calibrate the sensor with some known values. 

Use the log output to determine the values read by your ESP32, this is what we are looking for:

```
'Wasserstand Voltage': Sending state 0.27872 V with 2 decimals of accuracy
```

First note down the voltage when the sensor is not in water (basically 0cm). Then add as many data points as seem reasonable (e.g. one per meter).

When submerging the sensor, give it a minute or two so the value stabilizes.

Now update the mapping table to reflect your measurements
```
      - calibrate_linear:
            - 0.28 -> 0.0 # 0cm
            - 0.60 -> 1.2 # 120cm
            - 0.75 -> 2.2 # 220cm
            - 0.91 -> 3.2 # 320cm
            - 1.32 -> 4.2 # 420cm
```


## Home Assistant Integration 

Finally you can add the Board to your Home Assistant setup and configure a switch that enables/disables deep-sleep.

`Settings` > `Devices & Services` > `Add integration` > `esphome` > `Enter IP of your node`

### Deep Sleep Helper

By default the ESP32 board does not go to deep sleep. To allow OTA updates, a toggle in Home Assistant is used, that enables / disables sleep. As soon as the board wakes up, it checks if the toggle is enabled and only if so will sleep again. 

If you ever want to do an OTA update and your ESP32 is sleeping, disable the toggle, wait until it wakes up for the next time and you can proceed.

Create the toggle in Home Assistant:

`Settings` > `Devices & Services` > `Helpers` > `Create Helper` > `Toggle` 

Name the Helper `Enable ESPHome Deepsleep` (it's important that the id of the new helper matches the one configured in `brunnen.yaml`).

# Acknowledgement

* Thanks to [r0oland](https://github.com/r0oland) for the [ESP32 mini KiCad Library](https://github.com/r0oland/ESP32_mini_KiCad_Library)
* Thanks to [tathamoodie](https://github.com/tathamoddie) for his [blogpost]( https://tatham.blog/2021/02/06/esphome-batteries-deep-sleep-and-over-the-air-updates/) about deep sleep 
