# Controlling the Roomba 600 series with Home Automation

# Parts required
[ESP-01 ESP8266](https://surplustronics.co.nz/products/9606-wifi-wireless-module-wireless-esp-01-esp8266-serial-)

[Buck converter](https://surplustronics.co.nz/products/7518-dc-to-dc-step-up-step-down-converter-buck-boost-)

[PNP Transistor](https://www.jaycar.co.nz/2n3906-pnp-transistor/p/ZT2328)

## Original Amazon Parts
ESP-01: https://amzn.to/2qVB2p8

PNP Transistors: https://amzn.to/2FaUfrS

Buck Converters: https://amzn.to/2K7FY33

# Usage
Rewritten to use the new MQTT Vacuum integration in Home Assistant:
https://www.home-assistant.io/integrations/vacuum.mqtt/

Configure in Home Assistant `configuration.yaml` as follows:
```
vacuum:
- platform: mqtt
  name: Roomba
  schema: state
  command_topic: "roomba/commands"
  retain: true
  state_topic: "roomba/state"
  supported_features:
    - start
    - stop
    - return_home
    - battery
```

# References from https://www.crc.id.au/hacking-the-roomba-600/
I was in the market for a new vacuum. My old manual one was on the way out, and I'd heard a lot about Roombas, but never actually seen one. I didn't know what they were like, if they were useful, or how well they worked. I came across this YouTube video from The Hook Up which describes hacking the Roomba's 'Open Interface' to talk to it over a serial link. Considering the base level Roombas were cheaper than a manual vacuum, I thought I'd give it a go.
https://www.youtube.com/watch?v=t2NgA8qYcFI
https://www.youtube.com/channel/UC2gyzKcHbYfqoXA5xbyGXtQ

The good news, the specification is completely documented. https://www.irobotweb.com/-/media/MainSite/PDFs/About/STEM/Create/iRobot_Roomba_600_Open_Interface_Spec.pdf?la=en

The Arduino sketch given on The Hook Up github site was a good start, but I found many issues with it - and me being me - has ended up rewriting and improving just about all of the code given (and not using the Roomba library at all).

The schematic is still the same as The Hook Up's guide - so I'm not going to repeat that information here. If you have a Roomba 600 series and it ends up going to sleep on you quite often, you will need to connect GPIO 0 on the ESP-01S to the BRC pin on the serial interface. When dragged low for a pulse more often than once per minute, this pin stops the Roomba from entering deep sleep. This may not be required on the 500 series (Please leave feedback in the comments!)
https://github.com/thehookup/MQTT-Roomba-ESP01/blob/master/schematic.JPG

When it came to the arduino code, I found many issues.

The Roomba would just stop while cleaning, sound a two tone error and stay put.
The Roomba would go to sleep and not respond to start commands.
An error state could put the Roomba in the wrong baud rate needing a manual reset.
There was no debugging information (via a web interface) to see what was going on.
There was no way to update firmware on the device after you unsoldered the header on the ESP01.
Many more minor issues
The code below allows you to update the firmware in the ESP01 from any https site. If you only use http on your internal network, or you're using an older version of the Arduino ESP8266 SDK without the required BearSSL versions, uncomment the following lines:

//#include <ESP8266HTTPClient.h>

//WiFiClient UpdateClient;
//t_httpUpdate_return result = ESPhttpUpdate.update(UpdateClient, F("http://10.1.1.93/arduino/update/"));
and then comment out the https lines:

BearSSL::WiFiClientSecure UpdateClient;
UpdateClient.setInsecure();
t_httpUpdate_return result = ESPhttpUpdate.update(UpdateClient, F("https://10.1.1.93/arduino/update/"));
This will give you a smaller binary - but you'll lose the ability to use https.

I wrote a script that checks the HTTP_X_ESP8266_STA_MAC header provided by the Arduino update client against a known list, then checks the HTTP_X_ESP8266_SKETCH_MD5 header. If the MD5 of the binary file on the server differs, we return a HTTP 200 OK with the binary file, if they match, then we return a HTTP 304. I'll try to put more of these details online in a different page later on.

If you don't want to do any of the auto-updating, you can just visit the IP address off the ESP01 and click the Update link and upload a new binary that way.

The source code for this project is now in a git repository: View the source

# Credits
https://www.crc.id.au/hacking-the-roomba-600/
https://git.crc.id.au/netwiz/ESP8266_Code/src/branch/master/Roomba

# References
Roomba OI Document: https://cfpm.org/~peter/bfz/iRobot_Roomba_500_Open_Interface_Spec.pdf
