# ESP32 Specific

There were a few valuable benefits the ESP32 had over older microprocessors for features specific to this project.  Among them were Deep Sleep and NVM.

### Deep Sleep

The ESP32 has a variety of sleep modes; including a Deep Sleep which powers down the CPU and RAM, optionally keeping a low power co-processor active.  Even if the co-processor is kept powered, the overall consumption is extremely low, ideal for a battery-powered application such as this.  Unfortunately, the ESP32 dev board I used (Adafruit's HUZZAH32) is not optimized for low power consumption, so the Deep Sleep current draw is dwarfed by other quiescient current that's drawn by the board (the USB to serial converter, specifically).  That said, in Deep Sleep with hourly wakeups, the device is still good for several days without a recharge, drawing about 9 mA while in deep sleep.

Here's my code to enter Deep Sleep:

```c++
void DeepSleep(uint64_t sec) {
  if (sec == 0) {						//If # of seconds not provided...
    RealTime.Update();						//...Get the current real world time
    if (!RealTime.IsTimeSet()) {
      sec = DEEPSLEEP_DEFAULT_MINUTES*60;
    }
    else {							//...Assuming real world time was successfully loaded
    								//...Set wakeup for the next hour (e.g. 6:00 AM if it's currently 5:25)
      sec = (60 - RealTime.timeinfo.tm_min) * 60 + (60 - RealTime.timeinfo.tm_sec);
      if (sec < DEEPSLEEP_MINIMUM_SEC) {			//...Unless the next hour is almost upon us
        sec += 60*60;						//...In which case, wake up in an hour from now
      }
    }
  }
  logf("INFO","Deep Sleeping for %"PRIu64" seconds (%"PRIu64"\n",sec,sec*1000000);
  esp_sleep_enable_timer_wakeup(sec * 1000000);	//Wake up when the timer has expired (i.e. every hour)
  unsigned long long mask = 0;
  for (int i=0;i<DEEPSLEEP_PIN_COUNT;i++) {
    mask |= 1ULL << DeepSleepPinList[i];
  }
  esp_sleep_enable_ext1_wakeup(mask, ESP_EXT1_WAKEUP_ANY_HIGH); //...Or wake up if any button is pressed
  esp_deep_sleep_pd_config(ESP_PD_DOMAIN_RTC_PERIPH, ESP_PD_OPTION_ON);
  beam.HardShutdown();				//Turn off the LEDs during sleep
  esp_wifi_disconnect();			//...and disconnect the wifi
  delay(2000);            			//...but given enough time to cleanly disconnect
  esp_deep_sleep_start();			//Go to sleep
  logln("DEBUG","assert never");		//This line never runs
}
```

The functionality here is that the device wakes up at the top of every hour to update the screen, and also wakes up if any button is pressed, to react to the button press.

Tip: By its name, you can surmise that `ESP_EXT1_WAKEUP_ANY_HIGH` indicates to the RTC controller to wake up the main CPU if any of the selected GPIO lines go high.  I use this to wake up the CPU if any button is pressed.  One lesson learned: there is no such thing as `ESP_EXT1_WAKEUP_ANY_LOW`.  (There is a `ESP_EXT1_WAKEUP_ALL_LOW` which does exactly what it sounds like).  I originally wired my buttons pulled up, and then had to rewire and code them all when I implemented Deep Sleep, so that they were high on button press instead of low.

To react appropriately to the wakeup source, you need to know what caused the wakeup from Deep Sleep.  I used something along these lines, in the setup function:


```C++
switch (esp_sleep_get_wakeup_cause()) {
	case ESP_SLEEP_WAKEUP_TIMER: {
		//Wakeup was from hourly timer
		//Download updated weather, podcasts, etc. over wifi
		//Update the e-ink
		//Go back to sleep
		break;
      }
	case ESP_SLEEP_WAKEUP_EXT1: {
		//Wakeup was from a button press
		uint64_t wakeup_pin_mask = esp_sleep_get_ext1_wakeup_status();
		if (wakeup_pin_mask != 0) {
		int pin = __builtin_ffsll(wakeup_pin_mask) - 1;
		b.SimulateButtonPress(pin);
		} else {
			Serial.println("Wake up from unknown GPIO");
		}
		break;
	}
	case ESP_SLEEP_WAKEUP_UNDEFINED:
	default: {
    		// Not a deep sleep reset
      }
```
 
 
### NVM
Given that RAM is cleared every time Deep Sleep is entered, there needs to be a non-volitile storage location for anything that needs to persist across resets.  In this case, that's data like: podcast play history, API tokens, etc.
 
Conveniently, the ESP32 includes a facility to use some of the built-in flash memory for this purpose.  The flow I use is that each class's `begin` function (called during initial setup()) loads data from NVM into the class's data structure.  Any time data is refreshed or altered, it is then saved back to NVM.
 
The ESP32 codebase includes some convenience functions for reading and writing data of different types from/to NVM.  For instance, here's the code to load the Thermostat data from NVM upon startup:

```C++
int EcoBeeObj:: LoadFromNVM() {
  int r = 0;
  r += preferences.getBytes("EB.RefreshToken",RefreshToken,ECOBEE_TOKEN_SIZE);
  CurTemp = preferences.getFloat("EB.CurTemp",0);
  HoldHeat = preferences.getInt("EB.HoldHeat",0);
  HoldCool = preferences.getInt("EB.HoldCool",0);
  CurCool = preferences.getInt("EB.CurCool",0);
  CurHeat = preferences.getInt("EB.CurHeat",0);
  CurMode = preferences.getChar("EB.CurMode",'\0');
  r += CurTemp;
  return r;					//r == 0 upon failure
}
```
 

