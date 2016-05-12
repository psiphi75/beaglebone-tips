# GPS tips and tricks - for Linux

These GPS tips and tricks are based on the [SIM28 GPS unit](http://www.simcom.eu/index.php?m=termekek&prime=1&sub=2&id=0000000244) ([specs](http://www.simcom.eu/media/files/SIM28%20Specification_V1208.pdf)), however, it should work well on other units as well.  But check your documentation first.

This GPS device has the following default settings:

 - UART baud rate of 9600
 - GPS update rate of 1 Hz
 - The following NMEA sentences are output:
 
      - NMEA_SEN_RMC, // GPRMC interval - Recomended Minimum Specific GNSS Sentence
      - NMEA_SEN_GGA, // GPGGA interval - GPS Fix Data
      - NMEA_SEN_GSA, // GPGSA interval - GNSS DOPS and Active Satellites
      - NMEA_SEN_GSV, // GPGSV interval - GNSS Satellites in View

Below are the settings I want:
 - UART baud rate of ~~9600~~ 115200
 - GPS update rate of ~~1 Hz~~ 5 Hz
 - The following NMEA sentences are output:
 
      - NMEA_SEN_RMC, // GPRMC interval - Recomended Minimum Specific GNSS Sentence
      - ~~NMEA_SEN_GGA, // GPGGA interval - GPS Fix Data~~
      - ~~NMEA_SEN_GSA, // GPGSA interval - GNSS DOPS and Active Satellites~~
      - ~~NMEA_SEN_GSV, // GPGSV interval - GNSS Satellites in View~~
      - NMEA_SEN_ZDA, // GPZDA interval – Time & Date

## NMEA

[NMEA sentences](http://www.gpsinformation.org/dale/nmea.htm) are what most GPS devices output, when you turn on the GPS, the output just happens.  You may be able to use the following command to listen to your GPS over a serial port:

```sh
cat /dev/ttyO1
```

However, if the baud rate is not 9600 (in my case) it does not work well.  But you can use `minicom` to do the same:

```sh
minicom -b 9600 -D /dev/ttyO1
```

Note `Ctrl-A,Z,x` will exit `minicom`.

## PMTK

PMTK is language with which you can send commands to the your GPS device, also over the same serial port.  You can send  a command like `$PMTK314,1,1,1,1,1,5,0,0,0,0,0,0,0,0,0,0,0,1,0*2D<CR><LF>` (where `<CR><LF>` are sent in that order) and you should get an acknowledgement like `$PMTK001,314,3*36`.

## Using Node.js to make a very simple 
