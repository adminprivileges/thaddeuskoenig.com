---
date: '2023-11-23T12:00:00-05:00'
draft: false
title: 'Cheap Temp/Humidity Sensing in HAS'
tags:
  - HomeAssistant
  - Xiaomi
  - ESPHome
  - ESP32
---
If your house is like mine you have some rooms that run a little hotter than others (especially if you have servers running). Because of this the need to monitor temperature readings in rooms became a little more important, now there's a multitude of different ways to tackle this problem but this is by far the cheapest (although more time consuming on the front end) and doesn't require any cloud services to keep it running. I'd be unfair if I didn't mention some other good alternatives that use Zigbee/ZWave if you don't want to go through all this (these are not affiliate links, im not that important). 
- [Aotech ZWave temperature, humidity, dew point sensor](https://www.amazon.com/ZWave-temperature-humidity-point-sensor/dp/B0987MNJZ1/ref=sr_1_10_mod_primary_new?crid=AVZC1YK57TY6&keywords=aeotec+sensor&qid=1661607657&sbo=RZvfv%2F%2FHxDF%2BO5021pAnSA%3D%3D&sprefix=aotech+senso%2Caps%2C117&sr=8-10)
- [Aqara Temperature and Humidity Sensor](https://www.amazon.com/Aqara-WSDCGQ11LM-Temperature-Humidity-Sensor/dp/B07D37FKGY/ref=sr_1_4?crid=2GXB43O9BWG3J&keywords=aotech+temperature&qid=1661607636&sprefix=aotech+tempuratuer%2Caps%2C110&sr=8-4)

## Requirements
In order to pull this off youre going to need the following. 

- Xiaomi Mijia Bluetooth Temperature and Humidity Sensor (i get mine from [cloudfree.shop](https://cloudfree.shop/product/xiaomi-mijia-bluetooth-temperature-and-humidity-sensor/) but you can get them off [amazon](https://www.amazon.com/saikang-Bluetooth-Thermometer-Wireless-Hygrometer/dp/B083Y1D8WB/ref=sr_1_6?keywords=Xiaomi+Mijia+Bluetooth+Temperature+and+Humidity+Sensor&qid=1661608010&sr=8-6) or [aliexpress](https://www.aliexpress.com/item/3256801527318530.html?spm=a2g0o.productlist.0.0.1b672cfdInDh9Z&algo_pvid=e2967338-fa15-402a-a457-34fcf9438464&algo_exp_id=e2967338-fa15-402a-a457-34fcf9438464-0&pdp_ext_f=%7B%22sku_id%22%3A%2212000017260496317%22%7D&pdp_npi=2%40dis%21USD%217.98%214.79%21%21%21%21%21%402103143616616080951855349e0d1d%2112000017260496317%21sea&curPageLogUid=DJN2vstg5ACj)).
- A development board compatable with ESPhome (i used the [ESP-WROOM-32](https://www.amazon.com/dp/B08D5ZD528?psc=1&ref=ppx_yo2ov_dt_b_product_details)).
- A microusb chord for power/data transfer.
- A Bluetooth capable device (this wont work on iPhone).   

## Xiaomi
So this is going to be the most time-consuming part, although its really not that bad in case these instructions go out of date check out the github page for the flashing tool https://github.com/pvvx/ATC_MiThermometer#getting-started . 

1. To begin, youre going to need to be using google chrome or edge from there youre going to need to  go to your chrome settings and enable the Experimental Web Platform features by pasting this URL into your address bar `chrome://flags/#enable-experimental-web-platform-features` make sure to restart the browser after. 
2. From there you're going to want to open the [OTA Flasher](https://pvvx.github.io/ATC_MiThermometer/TelinkMiFlasher.html), click the Connect button, and you should see bluetooth devices. Yours should be named something like  LYWSD03MMC . 
3. After your connection is established click the Do Activation button and this will start pulling your keys (technically you could stop here and grab the keys for the Xiaomi Home Assistant plugin, but the ESP method is much better for continuity purposes).
4. After that, click Custom Firmware to flash the new firmware it should disconnect when its finished.
    - If you run into issues like I did, you can go to the [alternate flasher](https://pvvx.github.io/ATC_MiThermometer/TelinkOTA.html) and download the [latest ATC version](https://github.com/atc1441/ATC_MiThermometer/releases) from Github and install it here. 
5. If successful, when you turn your device off and on (by removing and reinstalling the battery) it should show you the last 6 digits of its MAC Address (the OUI is A4:C1:38) remember this because you'll .need it. 
6. From here you should have everything you need from the Xiaomi sensor.

## ESP Home
1. Install the ESPHome add-on in Home Assistant
2. Once you have it installed click the Open Web UI option if this is your first device it will open a setup wizard (evrything in there is spelled out for you very clearly), if not just click New Device. 
3. From here, you need to have your board connected to your computer in a browser that supports WebSerial (Chrome is probably your best bet).
4. I recommend using the configuration file created for you by ESPHome, from there we can add what we need to after.
5. From here, just make sure to validate and install your config onto your board. 
6. In the `Devices & Services` option under the Home Assistant settings, you can go to ESPHome and set your device to the room of your choosing. 
7. After that, paste the following template into your configuration file with your device MAC and preferred names (The spacing may be off due to the site, so make sure to validate and clean build)
```
esp32_ble_tracker:
sensor:
  - platform: atc_mithermometer
    mac_address: "A4:C1:38:XX:XX:XX"
    temperature:
      name: ["Example Name"]
    humidity:
        name: ["Example Name"]
    battery_level:
        name: ["Example Name"]
    battery_voltage:
        name: ["Example Name"]
    signal_strength:
        name: ["Example Name"]
```
 - [Important Note: If you add multiple devices, the `esp32_ble_tracker` and `sensor:`options are only specified once, and new devices begin at `- platform:`] 

From here, your sensors should show up in your home asstant dashboard in the same room as your ESPHome. You can add as many sensors as your home can handle you'll just have to buy a new board for every 8 or so devices (i believe that's what the Bluetooth threshold is). 