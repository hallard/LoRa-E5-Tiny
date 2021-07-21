# LoRa-E5-Tiny

<img src="https://github.com/hallard/LoRa-E5-Tiny/blob/main/pictures/LoRa-E5-Tiny-top.png" width="50%" height="50%"> <img src="https://github.com/hallard/LoRa-E5-Tiny/blob/main/pictures/LoRa-E5-Tiny-top.png" width="50%" height="50%">

Based on [LoRa-E5](https://www.seeedstudio.com/LoRa-E5-Wireless-Module-p-4745.html) from Seedstudio, but I wanted something really Tiny so I removed loy of stuff and left only JTAG prog, Serial and I2C Stemma QWIIC connector, and of course cell coin handler.

> :warning: **This board is experimental** : Use at your own risk, 

> :eyes: take a look on this excellent [reading](https://www.rs-online.com/designspark/can-i-prolong-my-coin-cell-battery-life-with-a-capacitor) on how to use capacitor to prolong cell coin batteries and understand the risk. I will use for EU8868 so for peaks approx 40mA, 3 times less than in the article so I guess it could works with 2 x 220uF or 470uF capacitors. Challenge would be to find them in 1206 footprint format.

I'm using mainly to flash custom firmware in it, and not using AT default firmware.

> :warning: **These boards have not been received** so I can't confirm they works but I'm confident on this point

## Features

- LoRa-E5 Module
- FTDI SMD 6 pads edge connector (:warning: **use 3.3V FTDI One, not 5V**)
- JTAG SMD 6 pads edge connector to flash module (PA13-SWDIO / PA14-SWCLK / PB3-SWO / RESET)
- Green Led on PB13
- Red Led on PA9
- 2 Tactile Switches (boot and reset)
- u-FL Antenna connector
- Stemma QWIIC I2C connector
- 2 x 1206 footprint for big capacitors see [reading](https://www.rs-online.com/designspark/can-i-prolong-my-coin-cell-battery-life-with-a-capacitor)
- CR2450 Cell coin battery size
- CR2450 battery holder

## Detailed Description

No specific documentation for now, it's just a kind of wiring helper as schematic.

I also assume that you are familiar with all LoRaWAN stuff, all setup/infrastructure/network server/provisionning and other are out of scope of this repository.

## Schematics

<img src="https://github.com/hallard/LoRa-E5-Tiny/blob/main/pictures/LoRa-E5-Tiny-sch.png">

## Boards 

You can order board on [oshpark](https://oshpark.com). 

- [V1.0](https://oshpark.com/shared_projects/xUa1Y94Z) 

It's a pitty after several discuss with OSHPark that I can't have any rewards for each people ordering my boards, this would allow me to order free PCB for shared projects and create new ones. For information my shared boards generated a total of **$285 162.00** orders at PCBs.io in 4 years, not bad at all :-), but looks like they have gone :sob:

Hoping one day OSHparks will thanks me giving them this market. 

### Assembled boards

**Top & bottom side V1.0**

<img src="https://github.com/hallard/LoRa-E5-Tiny/blob/main/pictures/LoRa-E5-Tiny-top.png">
<img src="https://github.com/hallard/LoRa-E5-Tiny/blob/main/pictures/LoRa-E5-Tiny-bot.png">

### Bill Of Material

Nothing fancy, due to size constraint, components are 0603/1206 and can be ordered almost anywhere (digikey, mouser, radiospare, ...). 
use only what you need dependings on what you want to do. 

> :memo: I2C pullup may not needed, most QWIIC/Steamma boards have their own. 

Check Seeed format [BOM](https://github.com/hallard/LoRa-E5-Tiny/blob/main/LoRa-E5-Tiny-BOM.xlsx) File, check on [Seeed OPL](https://www.seeedstudio.com/opl.html) for manufacturer SKU match.

## Firmware

Before flashing any custom firmware, I strongly advise to test the board with default AT-Firmware to get the keys (even if you can use your own of course). 

Do do this, use 3.3V (and NOT 5V) FTDI USB/Serial adapter, I love this one from [sparkun](https://www.sparkfun.com/products/14050)
<img src="https://cdn.sparkfun.com//assets/parts/1/1/8/8/8/14050-01.jpg" width="50%" height="50%">

Main idea is to connect with something like this idea (Thanks [@mharizanov](https://github.com/mharizanov) for this awesome simple efficient idea) and could be dual row male headers also.

<img src="https://harizanov.com/wp-content/uploads/2013/03/IMG_20150208_16352922.jpg">

- Connect FTDI on the 6 pins edge header of the board (if PCB is 2mm thickness should works without soldering)
- use them a terminal application and open the port on your computer corresponding to the FTDI device
- set terminal settings to 9600 bauds 8 bits no parity 1 stop bit (8N1)
- check with at command `AT` device should anwser ``+AT: OK``

then get keys of the device

```
AT 
+AT: OK
AT+ID 
+ID: DevAddr, 24:90:05:44
+ID: DevEui, 2C:F7:F1:20:24:90:05:44
+ID: AppEui, 80:00:00:00:00:00:00:06
```

### Provision device on Network Server

For testing I'm always using The Things Network (TTN).
So next step is to provision this new device to TTN with the above keys (no need DevAddr) and get APPKEY from TTN (random generate) then get the key issued from TTN (we'll use it later below)

### Compile and flash Firmware

You can flash the board with excellent [mbed-os](https://os.mbed.com/mbed-os/) framework. 
Easy way is to use [mbed studio IDE](https://os.mbed.com/studio/). 
We added this board into [stm32customtargets](https://github.com/ARMmbed/stm32customtargets), don't hesitate to read the [readme](https://github.com/ARMmbed/stm32customtargets/blob/master/README.md). 
Finally the main firmware [mbed-os-example-lorawan](https://github.com/ARMmbed/mbed-os-example-lorawan) program.

Once IDE installed: 

- use `file` / `import program` and them import the example with URL `https://github.com/ARMmbed/mbed-os-example-lorawan`
- right click in the project name and select `Add Library` and enter `https://github.com/ARMmbed/stm32customtargets`
- open the file `custom_targets.json` from folder `stm32customtargets` and copy whole contents
- paste copied contents in the main root folder file `custom_targets.json` (yes replace the whole file) 
- open the file `mbed_app.json` and change parameters on the section `target_overrides`
    - LoRaWAN parameters such as frequency plan, OTAA, Duty Cycle, ...
    - replace keys with the ones you got from above step `lora.device-eui`, `lora.application-eui` and `lora.application-key`
- add the following section near the end of the file `mbed_app.json`.

```json
        "LORA_E5_TINY": {
            "stm32wl-lora-driver.debug_tx": "PA_9",
            "stm32wl-lora-driver.debug_rx": "PB_13"
        }
```

Then on IDE select target "LORA_E5_TINY", build and flash with your favorite programmer (I'm using STLink) with GND/SWDIO/SWDCLK/RESET connected. 

Pay attention, that 1st time you need to erase SeeeStudio original firmware, make sure the Read Out Protection of the device is AA. If it is shown as BB, select AA and click Apply. See the end of this [section](https://wiki.seeedstudio.com/LoRa_E5_Dev_Board/#24-modify-your-device-eui-application-eui-application-key-and-your-lorawan-region) on how to do that with STM32CubeProgrammer.

### Build and Flash

From IDE you can build the example. If you plug your STLink while project opened, mbed ide will ask you if you want to set it up for this project/target, once approved you can compile, flash and even debug from mbed ide (need some tools installed, [read](https://os.mbed.com/docs/mbed-studio/current/monitor-debug/debugging-with-mbed-studio.html), very nice.


<!--<img src="https://github.com/hallard/LoRa-E5-Breakout/blob/main/pictures/mbed-ide.png">-->

You can also see logs with the FTDI adapter and any Serial terminal set to 115200 bauds 8 bits no parity 1 stop bit (8N1)

```
Mbed LoRaWANStack initialized 
 CONFIRMED message retries : 3 
 Adaptive data  rate (ADR) - Enabled 
 Connection - In Progress ...
 Connection - Successful 
 Dummy Sensor Value = 3 
 23 bytes scheduled for transmission 
 Message Sent to Network Server 
 Dummy Sensor Value = 5 
 23 bytes scheduled for transmission 
 Message Sent to Network Server 
 Dummy Sensor Value = 7 
 23 bytes scheduled for transmission 
```

Green LED will be on when on receive mode and Red when sending data.

## License

<img alt="Creative Commons Attribution-NonCommercial 4.0" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png">   

This work is licensed under a [Creative Commons Attribution-NonCommercial 4.0 International License](http://creativecommons.org/licenses/by-nc/4.0/)    

