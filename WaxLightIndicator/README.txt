Matt McFarland
February 2015, Electric Imp Wax Light Prototype

This project would not have been possible without referencing Tombrew’s Instructable Project “NeoWeather: Ambient Weather Indicator”

README summary:

This project uses an electric imp to help skiers and snowboarders conveniently and instantaneously know which wax to put on their skis/boards for the conditions at any given location. Because different temperature and snow conditions change the optimal type of wax the user should apply to their ski/snowboard, users frequently have to reference temperature and waxing charts before they wax. A real-time indicator would be incredibly handy and fun to have around the wax room! The Imp also alerts the user to snow and dangerous windchill conditions. 

The Electric Imp board (April Breakout) is attached to a RGB LED (Grove Chainable RGB LED), a white LED and a blue LED. The Electric Imp, upon receiving weather information from the agent on the Electric Imp server, changes the color of the RGB LED to the appropriate color of wax the user should apply to their equipment for their specified location. The user can change locations by visiting the agent server hosted on the Electric Imp website. The white LED illuminates if it is currently snowing at the target location, and blue LED illuminates in proportion to the temperature to warn the user about potentially dangerous wind chills (< 10 F). 

Operation:

The user must connect the Electric Imp device to the RGB LED, snow LED, windchill LED and an external power source. (Because this project illuminates the LEDs continuously, an external power source other than a battery is recommended.) Once the Electric Imp is powered, the user should connect it to their wifi network and update the agent code with a WeatherUnderground WebAPI developer key. To obtain a (free) developer key, apply on the following website http://www.wunderground.com/weather/api . After the Imp is connected to the network, the user should set the specified location visiting the agent server on the Electric Imp website. While the Imp is hooked up the network, the agent will push weather updates to the Imp and the Imp will light the LEDs accordingly.

For the given temperature T (F), the RGB LED will indicate the following wax types appropriate for the current conditions.

T > 32 	: yellow
32 > T > 28 	: red
28 > T > 23	: purple
23 > T > 11	: blue
11 > T 	: green

If the current conditions string fetched from Weather Underground contains the word “snow,” the snow LED will illuminate. If the windchill value provided by Weather Underground is less than 0 F, then the windchill warning LED will illuminate. 

If the windchill is below 10F, the Imp will illuminate the windchill warning LED. The strength of the LED’s light will increase as the windchill decreases. At -20 F, the intensity of the LED maximizes and remains there for any colder windchill.

Code Description:

Agent Code:

Function prepWebPage(): This function was taken from Tom’s NeoWeather project. It prepares a webpage for the user to change the specified location for the weather gathering. This function works in tandem with the http.onrequest( ) to handle location changes from the user. (These functions were taken from the NeoWeather project.)

Function getConditions(): Conditions are refreshed once every fifteen minutes (900 seconds). This getConditions function is called either when that fifteen minutes expires on the agent or the imp requests information upon restart. The agent requests weather information from Weather Underground using their web API (the developer key the user provides) and then sends the weather, temperature, and windchill information to the device (and logs that on the server).

Device Code:

The device controls the white and blue LEDs by simply configuring two pins as GPIO output pins. To illuminate these LEDs, the Imp sinks current through these GPIO pins. The Imp must communicate with the RGB LED to set the color. To communicate with the RGB LED’s controlling IC, the Imp uses a simple SPI data transmitter and clock signal (configured for 100 kHz data transfer rate).

Function setcolor( blue, green, red)
This function handles communication to the RGB LED. The exact protocol is spelled out in the #### data sheet and is summarized as follows. The data packet contains 32 bits. 

1st byte: 1 1 B7’ B6’ G7’ G6’ R7’ R6’
2nd byte: unsigned, 8 bit integer for blue pixel (lower 8 bits of blue arg)
3rd byte: unsigned, 8 bit integer for green pixel (lower 8 bits of green arg)
4th byte: unsigned, 8 bit integer for red pixel (lower 8 bits of red arg)

The data bits are latched on the rising edge of the clock signal. The clock should idle low. The Imp achieves this makeshift protocol by creating a 4 byte blob and then filling that blob one byte at a time. This blob is send out on the SPI line configured earlier, but the Imp must send a clear data (all zeros) packet before and after the data packet containing the color information.

Note: since Squirrel naturally passes around integers of 32 bit size, the user may try and write values larger than 255 to the setcolor function. If this is the case, the device will send a pixel value of 255 instead of the larger input argument. If the argument is less than zero, the imp will write a pixel value of zero for that color argument. If the argument is 255 or less and zero or greater, the argument is unchanged and the lowest 8 bits are written to the blob data packet.

Function handleweather( weather )
This is the call back from for when the agent sends weather information to the imp. If the weather string contains the word “snow,” the Imp sinks current into on the snow indicator LED.

Function handletemp( temp )
This is the callback function handling the temperature information sent from the agent. Depending on the value of the temperature, the Imp sets the RGB LED to a particular color to indicate the appropriate wax for the conditions.

Function handlewc( windchill )
This callback function handles the windchill. If the windchill is below 10 F, the Imp will sink current on the windchill warning LED in proportion to the temperature. Once the windchill reaches -20 F, the LED will reach full intensity and remain at that intensity as the temperature drops. 

Schematic and Materials:

Electric Imp + April Breakout Board x 1
Grove Chainable RGB LED x 1
LS7408 Quad AND gate chip x 1
Blue LED x 1
White LED x 1
330 ohm resister x 2

See attached Schematic.jpg for electrical layout

Note on Communicating with P9813 chip:

The P9813 chip that receives the communication packet from the Imp and sets the color of the RGB LED operates on a 5V power scheme. The chip also expects the data and clock signals at 5V. This presents a problem when trying to interface the P9813 chip with the Electric Imp because the Imp operates on a 3.3V output architecture. While the April breakout board can provide the 5V power signal the RGB LED and IC require off of the Vin pin (if provided 5.0 volts), the GPIO pins used to send communication signals over spi189 will fail to register with the P9813 chip. In order to translate the 3.3V Imp output to a 5V signal, the prototype design uses a HD74LS08P Quad-And gate chip to increase the Imp’s signal voltage. The AND gate provides the necessary voltage translation because one of the AND port’s is wired to the Vin signal (so always “1”), and the other port receives the signal from the Imp. When the Imp’s signal line rises, the AND gate’s output also rises. Since the LS08 chip sees any voltage greater than 2.0V as high, it will effectively turn the Imp’s 3.3V signal into a 5V output signal. This is done to both the Imp’s data line and the clock line to the RGB LED. (There are many ways to solve this voltage translation problem. The LS08 chip was just a part that the designer had handy. One concern with this implementation is that the AND gate does not forward the IMP’s signal changes instantaneously. There is a time delay on the order of 20 ns for transitions. Using a clock speed of 50 MHz or greater might present issues. To be safe, this prototype uses a clock rate of 100 kHz.)

Future Development

This prototype could be improved in several ways. First, the system would greatly benefit from some wind speed indicator. Using an LED bar indicator could easily fulfill this functionality. Next, users typically like to know sunup/sundown conditions before they head for skiing/while planning their outdoor activities. Adding sunrise and sunset times on 4-digit, 7-segment displays would increase convenience and relevance of system. The less the user has to reference other forms of information like the forecast on their phones/computers, the more powerful this ski information system becomes.

Resources and Acknowledgements:

- Instructable by Tom at Electric Imp: “NeoWeather: Ambient Weather Indicator”:
http://www.instructables.com/id/NeoWeather-Ambient-Weather-Indicator/

- Handling https requests tutorial: on Electric Imp website:
http://electricimp.com/docs/hardware/imp003evb/weather/

- Seeed Wiki on Grove Chainable RGB LED
http://www.seeedstudio.com/wiki/Grove_-_Chainable_RGB_LED

- Weather Underground Web API:
http://www.wunderground.com/weather/api

- Data sheet for P9813 IC

- Data sheet for 74LS08 Quad AND gate