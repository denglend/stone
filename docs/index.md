Several years ago, my wife and I took a trip to Italy for our honeymoon. The trip was full of beautiful cities, impressive landscapes, and lot sof good food.  One thing that stuck in my brain years later was the Slow Food Movement. As an innoculation against the . I do find myself in fast food restaurants from time to time, but the idea of stepping out of the high speed rapids, even for a few moments, is an appealing idea.

At some point, I realized that I felt the same way about technology, and especially cellphones. Cellphones can act like fast food: even when you set out to do something important or purposeful (eat a meal, check the weather), your enjoyment experience can be hijacked by the bombardment of speed and information.

So, in that spirit, I set out to build a Slow Tech device, as a birthday present for my wife. The goal was the provide more enjoyable way to access some of the benefits of technology, without providing a pathway to getting sucked into Facebook for hours, twitter wars, or mindless video games.

The result was the following device:

| Front | Back |
|---|---|
| ![Front](images/IMG_1378.JPG)  | ![Back](images/IMG_1377.JPG) |

The device has a two-color e-ink screen which only updates once per hour, as well as a ephemeral LED display that activates when input is needed.

It charges wirelessly, so there are no cables or plugs.  The aesthetic was intended to evoke an old radio.

It allows you to: view the current and upcoming weather. View the indoor temperature, and adjust the thermostat. Listen to podcasts or the radio on Sonos speakers, find your cellphone, and open the garage door.

The following pages describe the design and technical choices in the creation of this device. The software and design are fully open source, and available [here](https://github.com/denglend/stone) here, with the caveat that this was created as a one-of-a-kind device, and so some tinkering with the software and hardware should be expected if attempting to reproduce.

 - Overview
	 - [Design Choices](design-choices.md)
	 - [Parts list](parts-list.md)
 - Software
	 - [Wifi & HTTP](sw-wifi-http.md)
	 - HTTPS
	 - EcoBee
	 - IFTTT
	 - Sonos
	 - Pushover
	 - [Weather Underground](sw-weather.md)
	 - [ESP32 Specific (Deep Sleep, NVM)](sw-esp32.md)
	 - [Misc (OTA)](sw-misc.md)
 - Electronics
	 - Garage Door Opener
	 - E-Ink  --- (half updates b/c not enough current when wifi connecting?)
	 - Beam
	 - Buttons
 - Case Hardware
	 - Design & Files
	 - Assembly 
