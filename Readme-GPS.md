# GPS tips and tricks - for Linux

These GPS tips and tricks are based on the [SIM28 GPS unit](http://www.simcom.eu/index.php?m=termekek&prime=1&sub=2&id=0000000244) ([specs](http://www.simcom.eu/media/files/SIM28%20Specification_V1208.pdf)), however, it should work well on other units as well.  But check your documentation first.

The SIM28 NMEA and PMTK commands are documented [here](http://www.vis-plus.ee/pdf/SIM28@SIM68R@SIM68V_NMEA_Messages_Specification_V1.01.pdf).

This GPS device has the following **default settings:**

 - UART baud rate of 9600
 - GPS update rate of 1 Hz
 - The following NMEA sentences are output:

      - NMEA_SEN_RMC, // GPRMC interval - Recomended Minimum Specific GNSS Sentence
      - NMEA_SEN_GGA, // GPGGA interval - GPS Fix Data
      - NMEA_SEN_GSA, // GPGSA interval - GNSS DOPS and Active Satellites
      - NMEA_SEN_GSV, // GPGSV interval - GNSS Satellites in View

Below are the settings **I want:**
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

The `251` (PMTK_SET_NMEA_BAUDRATE) command can be used to change the baud rate.  We want to change this to 115200 baud.

```sh
node pmtk_output.js "PMTK251,115200" > /dev/ttyO1
```

You should notice that it is much quicker now.  You will have to restart your serial port monitoring software.


## Reduce the cruft that gets sent

I am only interested in getting the following information:
 - position (lat, long, alt)
 - velocity and direction of travel

The `GGA` and `RMC` NMEA sentences provide this information.  The PMTK `314` (PMTK_API_SET_NMEA_OUTPUT) command allows you to set the NMEA output.  For the SIM28 GPS module this means I disable (set value to `0`) the other NMEA output and set the others to once every cycle.  This means once a second if you have the NMEA output rate at 1 Hz.

Documentation states that `RMC` is the 2nd option and `GGA` is the 4th.

```sh
node pmtk_output.js "PMTK314,0,1,0,1,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0" > /dev/ttyO1
```

The response should be something like `$PMTK001,314,3*36`.

To reset this option to default, use the following command:

```sh
node pmtk_output.js "PMTK314,-1" > /dev/ttyO1
```

## Update the rate of NMEA sentences

I would like to have NMEA sentences spat out at 10 Hz (10 times per second). This is simple to do, but the documentation is not clear.  So, this is where my knowledge gets fuzzy, because the documentation is not clear.

The [SIM28 Hardware Design document](http://www.sabreadv.com/wp-content/uploads/SIM28_Hardware-Design_V1.06.pdf) explains
that EASY Mode (Embedded Assist System) works only at 1 Hz, it implies it conflicts with higher frequencies.  So we
turn off EASY Mode, but this is not documented how to do this.  But it is documented
[here](https://cdn-shop.adafruit.com/datasheets/PMTK_A11.pdf), under PMTK command `869` (PMTK_CMD_EASY_ENABLE).  

**Disable EASY Mode:**
```sh
node pmtk_output.js "PMTK869,1,0" > /dev/ttyO1
```

Once we have done that we can use PMTK command `220` (PMTK_SET_POS_FIX) to set the NMEA update rate.  The value is in
ms.  The documenation states it must be 200 ms or greater, but it seems to work at 100 ms.  The
[specs](http://www.simcom.eu/media/files/SIM28%20Specification_V1208.pdf) also state that it should work at 10 Hz (100
ms frequency).

**Set to 10 Hz update rate:**
```sh
node pmtk_output.js "PMTK220,100" > /dev/ttyO1
```

You should see the update increase significantly.

Reset to default:

```sh
node pmtk_output.js "PMTK220,1000" > /dev/ttyO1
```

### u-blox NEO-6M

I have installed the u-blox NEO-6M, I have found this (much more reliable)[http://stackoverflow.com/questions/38864087/while-moving-gps-lat-long-values-intermittantly-dont-change] than the SIM28 GPS module I was using before.

In a nutshell, to get u-blox NEO-6M configured (using Linux, not Windows) without a USB to serial port converter, we take the followings steps:
1) Install CrossOver and u-center on your Linux box.
2) Install socat and run it on the BeagleBone.
3) Connect to the GPS via the BeagleBone via socat using u-center.


**1) Installing u-center on Linux:** However, the u-blox NEO-6M comes with it's own configuration software (u-center) in Windows.  I don't want to reboot into Windows, so I used [CrossOver for Linux](https://www.codeweavers.com/products/crossover-linux) (I tried Wine but it did not work) to run u-center.  

**2) Install `socat`:** this is a magic tool that pipes the serial data over the network:

```
apt-get install socat
```

Note: (These instructions)[https://kurtraschke.com/2014/11/rpi-gps] are very helpful.

Run `socat` using the following command:

```
socat tcp-l:2000,reuseaddr,fork file:/dev/ttyO4,nonblock,waitlock=/var/run/ttyO4.lock,b9600,iexten=0,raw
```

This opens port 2000 and creates a two-way pipe for `/dev/tty04` (my u-blox GPS).  u-center will be able to connect to this.

**3) Connect using u-center**: In the *Reciever -> Network connection -> New* menu enter the following details: `tcp://192.168.43.18:2000` and ta da, just like magic the GPS values will start streaming through.
