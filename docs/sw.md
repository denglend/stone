# Software
To some extent, the software choices were dictated by the hardware choices I made.  I selected the ESP32 microcontroller because it 
1. Supported wifi
2. Had enough RAM to buffer the 3-color e-ink screen
3. had sufficient I/O for the peripherals I needed
4. Was low-power enough to run from battery, and
5. was readily available with enough support and documentation.

Given the choice of the ESP32, I could choose to code for it natively, to code in C++ using the Arduino IDE, or to use Micropython.  Micropython was tempting, but it was still rough enough around the edges that I was concerned I'd run into issues using some peripherals or microcontroller features.  So, the code is written in C++ in the Arduino environment, with a combination of existing and custom-written libraries.

It was a while since I wrote much in C++, so it took a little getting used to. A few unitialized variables and segfaults later, it did end up being a pretty convenient language and environment to work in.

Here's a brief overview of the code structure:

 - Main Project Files
 	- **mproj**: This is the main project file; Arduino setup function and main loop
 	- **button**: A convenience class for managing the state of the nine buttons on the device
 	- **ecobee**: Code to communicate with the Ecobee Thermostat API
 	- **ifttt**: Code to communicate with If This Then That (used to call a cellphone to "find" it)
 	- **menu**: A convience class to display the I/O menu structure that's displayed on the LEDs
 	- **menu_callbacks**: The logic that defines what happens when you interact with the menus
 	- **podcast**: Code to interact with podcasts (load them, parse for new episodes, send to speaker, etc.)
 	- **pushover**: Code to communicate with Pushover service (used to push notification to cellphone to "find" it)
 	- **screen**: Draws the screen components (menu, weather, etc.)
 	- **screendata**: Bitmaps used for screen drawing
 	- **util**: A variety of misc utility functions
 	- **weather**: Reads weather data from Weather Underground
 - Custom Libraries
 	- **beam**
 	- **epd4in2b**
 - Custom Tools
 	- **imgconv.py**
 - Off-the-shelf Libraries
 
 	