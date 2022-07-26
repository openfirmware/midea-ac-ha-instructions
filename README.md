# Midea Air Conditioners to Home Assistant

**Update July 2022**: I recommend trying this custom component instead: [midea_ac_lan](https://github.com/georgezhao2010/midea_ac_lan). The automatic configuration takes less than a minute and you will not have to get the token and key manually ([hopefully](https://github.com/georgezhao2010/midea_ac_lan#config-manually)). It installs with [Home Assistant Community Store](https://hacs.xyz/) (HACS), if you use that. Also the instructions mention Home Assistant and the Air Conditioner needing to be on the same sub-network, however I have mine on different subnets and it auto-discovered with no issue!

----

This guide was written 18th of August 2021.

[Home Assistant][HA] is a tool for combining all your "smart" appliances, allowing you to use them on platforms they are not officially supported (such as HomeKit). It runs as a small python server with a web interface (or mobile app) for configuration and display of your devices and local or remote data.

Recent Air Conditioners that have Wi-Fi capability typically connect to a cloud service, and you download a custom app to control it via that cloud. You can also connect it to Home Assistant, and start making smarter automations that run how you want it.

* You could have your own room temperature sensors, and use those as your cooling target instead of the Air Conditioner's built-in temperature sensor.
* Use your own occupancy sensor system (e.g. motion detectors, device trackers) to automatically turn the AC on and off.
* Adjust the maximum fan speed on your own schedule.

Apple Silicon-based Macs that can run iOS apps can be used to run the Midea Air app, which will log the connection details needed for the [midea-ac-py][] custom Home Assistant component.

[HA]: https://www.home-assistant.io
[midea-ac-py]: https://github.com/mac-zhou/midea-ac-py

## Guide

This guide assumes you already have:

* Set up Home Assistant
* Installed HACS or know how to manually install [midea-ac-py][]
* Connected your Air Conditioner via the Midea Air app for iOS or MacOS

Start by installing some Python tools to detect your AC on your network. Using Pip for Python 3+:

```
$ pip3 install msmart
$ midea-discover
INFO:msmart.cli:msmart version: 0.1.25
INFO:msmart.cli:Sending Device Scan Broadcast...
INFO:msmart.cli:Midea Local Data 10.0.0.114 <long string>
INFO:msmart.cli:Decrypt Reply: 10.0.0.114 <long string>
INFO:msmart.lan:Couldn't connect with Device 10.0.0.114:6444
INFO:msmart.cli:*** Found a unsupported device - type: '0xac' - version: V3 - ip: 10.0.0.114 - port: 6444 - id: <id number> - sn: <sn> - ssid: <networkname>
```

Running `midea-discover` should find all your compatible AC units on the same network. If your AC does not show up, then check that it shows up in your Midea Air app.

When your AC does show up, it is okay if it is listed as "unsupported." Note down the ID number and IP address.

**Important:** If your AC has a version of `V2`, then the following part of the guide does not apply. V2 users only need the IP address and ID number: see the [midea-ac-py README][midea-ac-py] for instructions.

For `V3` air conditioners, you will also need a `k1` key and a `token`. The midea-ac-py README covers how to get these keys using Android (or an Android emulator). I found that the Midea Air app for MacOS can also be used.

On an Apple Silicon Mac, download [Midea Air][] from the App Store. Run it at least once to make sure it can connect to your air conditioner and can turn your AC on or off. Quit the app. In the Finder, select the "Go" menu and select "Go to Folder…" (). Enter `~/Library/Containers/Midea Air/Data/Documents/MideaSDKLog`. Inside you will find one or more log files. Open the **newest** one, and find the last instance of `tokenlist` in the document. (If the document doesn't have `tokenlist`, check the next newest one.)

[Midea Air]: https://apps.apple.com/ca/app/midea-air/id1007999530

The `tokenlist` should contain a `key` and a `token`. These are the `k1` and `token` for midea-ac-py. Be sure to find the most recent copy of these keys; I found earlier copies were invalid or expired. When that happens, Home Assistant will show your AC as "unavailable."

These are added to your Home Assistant configuration YAML (web configuration is not yet available):

```yml
climate:
  - platform: midea_ac
    host: 10.0.0.114
    id: 123456789012345
    k1: 64-characters-long
    token: 128-characters-long
```

Once you have saved the configuration file, reload Home Assistant. Go to the admin section of the web interface and select Entities, then search for `climate`. Your Midea-connected AC should be displayed. If it is not, check the Home Assistant log for error messages.

Select the AC entity in HA to customize it, I suggest changing your icon to `hass:air-conditioner` to replace the thermostat icon. While you're here you can also assign the entity to a room.

Select the configuration toggle in the top-right to switch to the AC controls, and verify that they work. You can add Lovelace dashboard entities to control the air conditioner now, just select your new `climate` entity. Here are two examples below.

![thermostat](https://user-images.githubusercontent.com/68210/130004140-c64701e4-a285-43d6-942d-6d69f8436a04.png)  
Above: using the default [Lovelace thermostat](https://www.home-assistant.io/lovelace/thermostat/)

![simple-thermostat](https://user-images.githubusercontent.com/68210/130004148-2ddf893b-dd74-40f8-a991-09a3f029eaec.png)  
Above: Using [simple-thermostat](https://github.com/nervetattoo/simple-thermostat)

## Additional Notes

If you try to set up your AC as offline only (LAN connections only, and reject access to internet), then the AC enters a boot loop and you cannot use local mode ([source](https://github.com/WMP/midea-ac-py/tree/support-8370#wifi-module-version-3)). It seems a [mock cloud server](https://github.com/WMP/midea-ac-py/tree/support-8370#install-fake-cloud-only-if-you-have-protocol-version-3) can be used to fix this, allowing the AC to remain in local mode even when your internet or the cloud service is unavailable.

There is also a method where a USB stick or direct connection to your AC's control board allows [remote control via ESPHome](https://esphome.io/components/climate/midea_ac.html), bypassing cloud requirements completely.
