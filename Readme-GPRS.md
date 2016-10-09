# M95 FA GPRS Modem

This page describes how to get the Quectel M95 modem working on the BeagleBone Black/Green.

Excellent instructions on getting the MikroBUS cape configured and installed:
http://www.libstock.com/projects/view/1525/remote-weather-station

AT Command reference for M95:
http://www.sos.sk/a_info/resource/c/quectel/M95_ATC_V1.0.pdf


Helpful reference (but not necessary):
http://ciudadoscura.com/beaglebonecape/downloads.html
http://ciudadoscura.com/beaglebonecape/downloads/trackingCapeRefManual.pdf

### Increasing the modem data rate

The default modem rate appears to be 9600 baud for the M95 FA GPRS modem on the
BBB.  This is the the communication rate (speed in bits per second) between the
GPRS modem and the BBB.   It is possible to increase this rate to faster rates.
This is not the data rate between the GPRS modem and your cellphone provider's
cell towers.  However, the very slow speed 9600 baud will definitely cause a
bottleneck when there is a lot of data being transferred.

Here I describe how to increase the speed of M95 FA GRPS communication rate
with the BBB.

```sh
minicom -b 9600 -D /dev/ttyO4
```

The console is very primitive, but it works. Typing AT then return into the
minicom console should return OK.  This means that the modem is responding to
your command.

```text
AT
OK
```

Type in `AT+IPR?`.  This will list what data rate the modem set to.  A value of 0
means that it's auto-detect.  Read the Quectel GSM UART Application Note for
more details.

```text
AT+IPR?
+IPR: 0

OK
```

`AT+IPR=?` will list all the available data rates.  The first list is the list of
detectable data rates, the second list is the possible data rates you can set.

```text
AT+IPR=?
+IPR: (4800,9600,19200,38400,57600,115200),(0,75,150,300,600,1200,2400,4800,9600,14400,19200,28800,38400,57600,11)

OK
```

This is the key part.  This will set the data rate.  Note that this setting will
not be saved after reboot.  See below for saving this.

```text
AT+IPR=115200
OK
```

Now you will be at a higher data rate.  However, you will have to exit minicom
because all AT commands will not work, because the BBB is set to 9600 baud.
Press CTRL-A Z, followed by X to exit minicom.

Check that it works at 115200 baud by running minicom again:

```sh
minicom -b 115200 -D /dev/ttyO4
```

Type AT, and you should get OK

Type AT&W and this will save value, even across reboots.

```text
AT&W
OK
```

#### Links:
http://www.quectel.com/product/prodetail.aspx?id=7
    Quectel_GSM_UART_Application_Note_V1.2.pdf
    Quectel_M95_AT_Commands_Manual_V3.2.pdf

## Connecting to the Internet

The GPRS modem we have needs to be connected to the Internet. To do this we use
the Debian package called `ppp`.  `ppp` can be scripted to let us easily connect
to the Internet.

The instructions here are based on the following link: http://www.acmesystems.it/ppp

### Install and configure `ppp`

```sh
apt-get install ppp
```

Replace the `/etc/chatscripts/pap` contents with the following:

```Text
TIMEOUT      60
ABORT        BUSY
ABORT        VOICE
ABORT        "ERROR"
ABORT        "NO CARRIER"
ABORT        "NO DIALTONE"
ABORT        "NO DIAL TONE"
""            ATZ
OK           AT+CGDCONT=1,"IP","internet","0.0.0.0",0,0
OK           ATDT\T
CONNECT      ""
```

*Important note* : The line above with `AT+CGDCONT=1` is the WAP connection
string.  Each provider has their own value, in my case it is `internet`.

Replace the `/etc/ppp/peers/provider` file with the following:

```sh
# MUST CHANGE: replace myusername@realm with the PPP login name given to
# your by your provider.
# There should be a matching entry with the password in /etc/ppp/pap-secrets
# and/or /etc/ppp/chap-secrets.
user ""
password ""


# MUST CHANGE: replace ******** with the phone number of your provider.
# The /etc/chatscripts/pap chat script may be modified to change the
# modem initialization string.
connect "/usr/sbin/chat -v -f /etc/chatscripts/pap -T *99***1#"

# Serial device to which the modem is connected.
/dev/ttyO4

# Speed of the serial line.
115200

# Assumes that your IP address is allocated dynamically by the ISP.
noipdefault
# Try to get the name server addresses from the ISP.
usepeerdns
# Use this connection as the default route.
defaultroute

# Makes pppd "dial again" when the connection is lost.
persist

# Do not ask the remote to authenticate.
noauth

noccp
-vj

debug
```

*Note* : The last `debug` option is optional.  This allows us to get more
debug information if you have issues.

To monitor the ppp connection you can enter the following command to print out
the ppp messages as the happen:

```sh
tail -f /var/log/messages
```

### Loading /dev/ttyO4 using node

```JavaScript
var obs = require('octalbonescript');
obs.serial.enable('/dev/ttyO4', function(){
    console.log('enabled serial');
});
```

