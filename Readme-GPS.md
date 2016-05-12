# GPS tips and tricks

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

