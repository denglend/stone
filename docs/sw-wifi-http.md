# Wifi & HTTP
One of the major benefits of using the ESP32 is the wifi stack, thus not requiring a separate peripheral for communications.  That said, it took no time at all to realize connecting over HTTP(s) to a variety of destinations without much in the way of parsers or convenience libraries was going to be a significant undertaking.

### Wifi

Connecting ESP32's wifi library is linked in with a quick `#include <WiFi.h>` and the object created with `WiFiClient client;`.

The Wifi library includes event callback functionality, to allow you process changes to the wifi status.
The code I use during the setup function is:

```c++
WiFi.disconnect(true);		//Just in case arduino code restarted  but still connected to wifi?
WiFi.onEvent(WiFiEvent);		//Set up callback
WiFi.begin(ssid, password);		//Connect to wifi
```
My code in the callback function looks like:
```c++
void WiFiEvent(WiFiEvent_t event) {
  switch(event) {
    case SYSTEM_EVENT_WIFI_READY:
      //log the status info to the serial port, for debugging purposes
      logln("INFO","WiFi Event: SYSTEM_EVENT_WIFI_READY"); 
      break;
    case SYSTEM_EVENT_STA_STOP:
      logln("INFO","WiFi Event: SYSTEM_EVENT_STA_STOP");
      break;
    case SYSTEM_EVENT_STA_START:
      logln("INFO","WiFi Event: SYSTEM_EVENT_STA_START");
      break;
    case SYSTEM_EVENT_STA_CONNECTED:
      logln("INFO","WiFi Event: SYSTEM_EVENT_STA_CONNECTED");
      break;
    case SYSTEM_EVENT_STA_DISCONNECTED:
      logln("INFO","WiFi Event: SYSTEM_EVENT_STA_DISCONNECTED");
      IsWiFiConnected = false;		//A global to track wifi connection status
      WiFi.disconnect(true);		//If Wifi became disconnected, it was accidental
      WiFi.begin(ssid, password);	//So, reconnect
      break;
    case SYSTEM_EVENT_STA_GOT_IP:
      Serial.print(" IP: ");Serial.println(WiFi.localIP());	//For debugging purposes
      IsWiFiConnected = true;			//Set global to indicate wifi is connected
      logln("INFO","WiFi Event: SYSTEM_EVENT_STA_GOT_IP");
        ArduinoOTA.begin();  	//This allows receipt of updated code via wifi, rather than serial
      break;
    default:
      logf("INFO","WiFi Unknown Event: %d\n",event);
  }
}
```

Once Wifi is connected, you use `WifiClient.connect()` to connect the the desired host, `WiFiClient.printf()` to send data to the host, and, while `WifiClient.available()` is true, read a byte of data using `WifiClient.read()`.  I created a convenience function to read data from the connected server into a buffer, until a newline was reached (or the buffer is full), since that's frequently needed for HTTP connections.
```c++
void ReadBytesUntilNL(WiFiClient *cl, uint8_t *buf, int ln) {
  int i=0;
  while (i<ln && cl->available() ) {
    buf[i] = cl->read();
    i++;
    if (buf[i-1]=='\r' || buf[i-1] == '\n') {
      if (cl->available() && (cl->peek() == '\r' || cl->peek() == '\n')) cl->read();
      break;
    }
  }
  buf[i]='\0';
  return;
}
```

### HTTP

Reading data from web services over HTTP is both very simple, and very tedious.  Making the request is pretty straightforward.  Here's code from the podcast feed reader:
```c++
//pd->host is the web host to connect to. e.g. rss.art19.com
//pd->page is the file on the host which contains the podcast RSS.  e.g. /the-daily
    if (client.connect(pd->host, 80)) {
      client.printf("GET %s  HTTP/1.1\r\nHost: %s\r\nConnection: close\r\n\r\n",pd->page,pd->host);
  }
  else {
    logln("ERROR","Connect failed for LoadPodcast");
    return 0;
  }
```

Once sending your request, you expect data to be returned from the server, so you can wait until some data is available.  I use a five second timeout.
```c++
  unsigned long timeout = millis();
    while (client.available() == 0) {
      if (millis() - timeout > 5000) {
          logln("ERROR","Response Timeout in LoadPodcast");
          client.stop();
          return 0;
      }
    }
```

Once data is available, it's easy to read it line-by-line, using a  convenience function ala `ReadBytesUntilNL`.  It took me a minute to wrap my brain around the fact that I'd have to read and parse the returned data line by line, rather than read the entire response and parse it, but that's necessary given the constraints of working on a microcontroller.  For instance, while the ESP32 has a relatively generous 520 KB of RAM, the RSS file for The Daily podcast is > 2 MB in size.

To parse each line, I used a slightly modified version of [Nick Gammon's Microcontroller version](https://github.com/nickgammon/Regexp) of the Lua Regular Expression parser.  My minor edit allows you to specify a buffer return size when requesting a CaptureGroup be copied to a buffer, to prevent overflows.

My code to parse a line would look something like:
```c++
	MatchState ms;		//Initialize the regex object
	ReadBytesUntilNL(&client,ReadBuf,READBUF_SIZE-1);	//Read a line from wifi
	 if (ms.Match("<title>(.*)</title>") == REGEXP_MATCHED) {	//If RegEx matches
	 	ms.GetCaptureMax(BUFFER,0,BUFFER_SIZE);	//Load the captured data (i.e. whatever is between the title tags) into the buffer
	 }
```