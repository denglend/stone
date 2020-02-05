# Misc Software Notes
#### OTA Firmware Upgrades
One side effect of wireless charging and no cords or cables is that upgrading the software must also be done wirelessly, at least once the device is fully assembled.  Thankfully, there's an arduino library that makes it relatively easy to do so: [ArduinoOTA](https://github.com/jandrassy/ArduinoOTA).

OTA Upgrades are as straightforward as including something along the lines of the following in your setup function
```C
ArduinoOTA.setPassword(OTA_PASSWORD);
  ArduinoOTA
      .onStart([]() {
      logln("INFO","Starting OTA");
    })
    .onEnd([]() {
      logln("INFO","OTA Complete");
    })
    .onProgress([](unsigned int progress, unsigned int total) {
      logf("INFO","OTA Progress: %u%%\n", (progress / (total / 100)));
    })
    .onError([](ota_error_t error) {
      logf("ERROR","OTA Error[%u]: ", error);
      if (error == OTA_AUTH_ERROR) {logln("ERROR","Auth Failed");}
      else if (error == OTA_BEGIN_ERROR) {logln("ERROR","Begin Failed");}
      else if (error == OTA_CONNECT_ERROR) {logln("ERROR","Connect Failed");}
      else if (error == OTA_RECEIVE_ERROR) {logln("ERROR","Receive Failed");}
      else if (error == OTA_END_ERROR) {logln("ERROR","End Failed");}
    });
```
and then `if (IsWiFiConnected) ArduinoOTA.handle();` in your main loop.

I found that sometimes the Arduino IDE failed to recognize the network host with OTA available, but it would usually work after a couple of tries.
