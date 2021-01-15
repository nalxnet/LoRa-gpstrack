# LoRa-gpstrack

A **LoRa GPS tracker** sensor based on 
 - Arduino Uno and 
 - Dragino LoRa GPS Shield

for use in LoRaWAN networks.

This application transmits
 - current GPS position,
 - altitude and
 - hdop

as soon as a GPS fix is aquired. This data is sent continously using LMIC library. The device is powered by any source that fits to Arduino Uno.

We use this tracker to measure the range of the LoRaWAN Gateways in our network. **The application's output format is compatible for use with [TTN Mapper](http://ttnmapper.org).**

It might suite for any other GPS tracking use case as well. We just included to the values we required but it can easily be extended to more values. Keep in mind that more data means longer transmission times and duty cycles, especially on high spreading factors.

## Installation
What you need:
 - Arduino Uno and 
 - Dragino LoRa GPS Shield
 - power source for powering Arduino

Put together the hardware as described in the vendor's manual: Just put the shield on top of your Arduino Uno.

Create an application in The Things Network Console and add a new device. Use ABP as activation method. Disable Frame Counter Checks.

Change the 
 - NWKSKEY
 - APPSKEY
 - DEVADDR

values in the code according to match the device details you just created in the TTN Console.

Adjust the channel configuration if you use it outside EU.

Compile the Arduino sketch. You need to install the libraries [TinyGPS](https://github.com/neosarchizo/TinyGPS) and [MCCI LoRaWAN LMIC Library](https://github.com/mcci-catena/arduino-lmic). Don't forget to configure the used LoRaWAN frequencies by commenting/uncommenting the corresponding line in project_config/lmic_project_config.h of the LMIC Library in your local libraries directory. For EU868, comment CFG_us915 and uncomment CFG_eu868.

    #define CFG_eu868 1
    //#define CFG_us915 1

Program the application to your Arduino Uno. If you have problems while programming, remove the GPS RX/TX jumpers from the Dragino shield while programming.

In The Things Network Console you can use this code to decode uplink messages:

    function Decoder(bytes, port) {
        // Decode an uplink message from a buffer
        // (array) of bytes to an object of fields.
        var decoded = {};
        // if (port === 1) decoded.led = bytes[0];
        decoded.latitude = ((bytes[0] << 16) >>> 0) + ((bytes[1] << 8) >>> 0) + bytes[2];
        decoded.latitude = (decoded.latitude / 16777215.0 * 180) - 90;
        decoded.longitude = ((bytes[3] << 16) >>> 0) + ((bytes[4] << 8) >>> 0) + bytes[5];
        decoded.longitude = (decoded.longitude / 16777215.0 * 360) - 180;
        var altValue = ((bytes[6] << 8) >>> 0) + bytes[7];
        var sign = bytes[6] & (1 << 7);
        if (sign) {
            decoded.altitude = 0xFFFF0000 | altValue;
        } else {
            decoded.altitude = altValue;
        }
        decoded.hdop = bytes[8] / 10.0;
        return decoded;
    }


### Configuration Options

Optionally you can adjust these values:
- TX_INTERVAL
Defines the pause time in seconds between transmissions (default is 60. You can set it to 0 to get as many data as possible and use the maximum duty cycle allowed.)
 - GPS_INTERVAL
poll interval of GPS DATA
 - lmic_pinmap
PIN mapping matches Dragino LoRa GPS shield. Adjust if your are using a different shield.
 - LMIC_setDrTxpow
Spreading Factor (SF7 -> SF12) and transmission power in dB
 - Debug Mode (DEBUG_SS)
see chapter [Debug mode](#debug-mode)
 - channel operation
by removing the comments from the channel definition you can use all LoRaWAN channels for EU868 suggested by The Things Network.

### Debug mode
Debug mode enables debugging of the reception of GPS data via SoftSerial. On Arduino Uno it is not possible to run debug mode and LoRa transmission at the same time due to memory restrictions. To enable debug mode define DEBUG_SS:

    #define DEBUG_SS

## Integration to ttnmapper.org
Create an TTN Mapper integration in your TTN application. Please use an experiment for testing and remove the experiment string once you are ready to contribute to the main map. See the [TTN Mapper FAQ](https://ttnmapper.org/faq/) for details.

# FAQ & known problems

##### I get an error when trying to program the Arduino, what to do?
Remove the GPS jumpers on the Dragino shield while programming. Don't forget to install them again once programming is finished.

##### I only see packets received by the gateway but not in the application
Check that you have disabled Frame Counter Checks in the device configuration of TTN Console.

##### I want to only transmit on one channel, is this possible?
The LoRa GPS tracker transmits on the 3 default channels of LMIC (EU868). You can disable the unwanted channels by looking for these lines in the code:

    // disable channels not known to single channel gateways
    LMIC_disableChannel(1);  // uncomment to disable channel 1
    LMIC_disableChannel(2);  // uncomment to disable channel 2
    [...]

##### Can I use this code also with other LoRa boards?
This should be possible. You will have to adjust the PIN mapping constant called lmic_pinmap. For GPS a serial port with 9600 baud is used.

# Credits
This script is based on the work by Thomas Telkamp, Matthijs Kooijman and Andreas Spiess.
Thanks for ttnmapper application and code review to JP Meijers.
