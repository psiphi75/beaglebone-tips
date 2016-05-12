# GPS tips and tricks - for Linux

These GPS tips and tricks are based on the [SIM28 GPS unit](http://www.simcom.eu/index.php?m=termekek&prime=1&sub=2&id=0000000244) ([specs](http://www.simcom.eu/media/files/SIM28%20Specification_V1208.pdf)), however, it should work well on other units as well.  But check your documentation first.

The SIM28 NMEA and PMTK commands are documented [here](http://www.vis-plus.ee/pdf/SIM28@SIM68R@SIM68V_NMEA_Messages_Specification_V1.01.pdf).

This GPS device has the following default settings:

 - UART baud rate of 9600
 - GPS update rate of 1 Hz
 - The following NMEA sentences **are output**:
 
      - NMEA_SEN_RMC, // GPRMC interval - Recomended Minimum Specific GNSS Sentence
      - NMEA_SEN_GGA, // GPGGA interval - GPS Fix Data
      - NMEA_SEN_GSA, // GPGSA interval - GNSS DOPS and Active Satellites
      - NMEA_SEN_GSV, // GPGSV interval - GNSS Satellites in View

Below are the settings **I want**:
 - UART baud rate of **115200**
 - GPS update rate of **5 Hz**
 - The following NMEA sentences are output:
 
      - NMEA_SEN_RMC, // **GPRMC** interval - Recomended Minimum Specific GNSS Sentence
      - NMEA_SEN_ZDA, // **GPZDA** interval – Time & Date

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

PMTK is language with which you can send commands to the your GPS device, also over the same serial port.  You can send  a command like `$PMTK314,1,1,1,1,1,5,0,0,0,0,0,0,0,0,0,0,0,1,0*2D<CR><LF>` 
where:
 -  `$PMTK`: is the start of the string,
 -  `314`: is the PMTK command ID, this particular command is the PMTK_API_SET_NMEA_OUTPUT, which sets how frequently to display the NMEA commands;
 -  `,1,1,1,1,1,5,0,0,0,0,0,0,0,0,0,0,0,1,0`: Is the command argument;
 -  `*2D`: is the checksum;
 -  `<CR><LF>`: are sent in that order.

And you should get an acknowledgement like `$PMTK001,314,3*36`.

## Using Node.js to make a very simple 

Create the `pmtk_output.js` file.

```JavaScript
var arg = process.argv[2];
process.stdout.write('$' + arg + '*' + updateChecksum(arg) + '\r\n');

// http://www.hhhh.org/wiml/proj/nmeaxor.html
// Compute the MTK checksum and display it
function updateChecksum(cmd)
{
  // Compute the checksum by XORing all the character values in the string.
  var checksum = 0;
  for(var i = 0; i < cmd.length; i++) {
    checksum = checksum ^ cmd.charCodeAt(i);
  }

  // Convert it to hexadecimal (base-16, upper case, most significant nybble first).
  var hexsum = Number(checksum).toString(16).toUpperCase();
  if (hexsum.length < 2) {
    hexsum = ("00" + hexsum).slice(-2);
  }
  return hexsum;
}
```

Then you can run it using the following command:

```sh
node pmtk_output.js "PMTK314,1,1,1,1,1,5,0,0,0,0,0,0,0,0,0,0,0,1,0"
```

This outputs `$PMTK314,1,1,1,1,1,5,0,0,0,0,0,0,0,0,0,0,0,1,0*2D`.  The GPS device will respond with `$PMTK001,314,3*36`.


You can then redirect the output to the serial port (in my case `/dev/ttyO1`).  Open two different terminals.  First we display the serial output (NMEA output).

```sh
# Terminal 1
cat /dev/ttyO1
```

Then in the other terminal we use to send commands:

```sh
# Terminal 2
node pmtk_output.js "PMTK314,1,1,1,1,1,5,0,0,0,0,0,0,0,0,0,0,0,1,0" > /dev/ttyO1
```

## Change the baud rate of the GPS

The `251` (`PMTK_SET_NMEA_BAUDRATE`) command can be used to change the baud rate.  We want to change this to 115200 baud.

```sh
node pmtk_output.js "PMTK251,115200" > /dev/ttyO1
```

You should notice that it is much quicker now.  You will have to restart your serial port monitoring software.

node output.js "PMTK869,1,0" > /dev/ttyO1
node output.js "PMTK869,1,1" > /dev/ttyO1
node output.js "PMTK251,115200" > /dev/ttyO1
node output.js "PMTK000" > /dev/ttyO1
node output.js "PMTK220,200" > /dev/ttyO1
node output.js "PMTK220,100" > /dev/ttyO1
node output.js "PMTK220,1000" > /dev/ttyO1
node output.js "PMTK251,9600" > /dev/ttyO1
node output.js "PMTK314,1,1,1,1,1,5,0,0,0,0,0,0,0,0,0,0,0,1,0" > /dev/ttyO1


