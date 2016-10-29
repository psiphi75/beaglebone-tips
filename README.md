# Beaglebone Tips
Tips for the BeagleBone Green/Black.

## Start with a minimal image

### Step 1: Get the image

I start with the smallest image, then install what is required.  Typically I don't use Cloud9 IDE and I have never used X11 (the GUI).  I use the [microSD/Standalone: (console) (BeagleBone/BeagleBone Black/BeagleBone Green)](http://elinux.org/Beagleboard:BeagleBoneBlack_Debian#microSD.2FStandalone:_.28console.29_.28BeagleBone.2FBeagleBone_Black.2FBeagleBone_Green.29) image.  Although this is really bare-bones.  This is just under 400MB in size.

To write the image to the SD card (using a Linux desktop) use the following command:

    xzcat bone-debian-8.5-console-armhf-2016-06-05-2gb.img.xz  | dd of=/dev/mmcblk0

Note that the you should write to the drive (e.g. `/dev/mmcblk0`) not the partition (e.g. `/dev/mmcblk0p1`).

### Step 2: Install the applications you require

Then I install the following components (this will use around 250 MB):

```bash
apt-get update
apt-get dist-upgrade -y

# For node development
apt-get install build-essential device-tree-compiler

# Other tools
apt-get install git mc i2c-tools minicom ppp python

# I only use vim.tiny
cd /usr/bin
ln -s vim.tiny vim
```

We need to figure out which version of nodejs to use:
```bash
apt-cache madison nodejs
```

The install the version we want:
```bash
apt-get install nodejs=0.12.13* 
```

Finish up with:
```bash
apt-get autoremove
apt-get clean
```

The end result is a total installed size of 534 MB.

### Step 3: Get and install the firmware

To load DTOs you will need to follow these instructions:
https://github.com/beagleboard/bb.org-overlays

Once the firmware is installed, in theory you could uninstall many of the above components.


## Make the console fancier

Add the following to ~/.bashrc:

```bash
# ls aliases
alias ls='ls --color=auto'
alias ll='ls -l'
alias la='ls -A'
alias l='ls -CF'

# Coloured prompt
export PS1="\[\033[38;5;1m\]\u\[$(tput sgr0)\]\[\033[38;5;2m\]@\h:\[$(tput bold)\]\[$(tput sgr0)\]\[\033[38;5;15m\]\w\[$(tput sgr0)\] \[$(tput sgr0)\]\[\033[38;5;2m\]\A\[$(tput sgr0)\]\[\033[38;5;15m\] \[$(tput sgr0)\]\[\033[38;5;2m\]\\$\[$(tput sgr0)\]\[\033[38;5;15m\] \[$(tput sgr0)\]"

# Allow quick viewing of the slots
export SLOTS=/sys/devices/platform/bone_capemgr/slots
```


## Potential Issues you may encounter

### The ssh login process takes a while
On the BeagleBone edit `/etc/ssh/sshd_config` and add the following line:

```sh
UseDNS no
```

This removes the reverse DNS lookup (which can take a while).

You need to run to restart the sshd service:

```
service sshd restart
```

### The ethernet over USB does not work

I run Debian Linux on my Desktop/Laptop and like to ssh to the BeagleBone via USB.  By default it worked out of the box.  But then it stopped.  Running `mkudevrule.sh` found on the [BeagleBone start page](http://beagleboard.org/getting-started) fixed the problem.

One issue I had is that when I plug the BeagleBone into a USB 3.0 port it has issues with creating network.  But the USB 2.0 ports work.


### Port Scan to find the BeagleBone on the network

**Linux only**

When you have the ethernet cable plugged into the BeagleBone and you want to know what the IP address is, you can port scan it:

```sh
nmap -T4 -F 192.168.2.100-255
```


### Enable Wifi (specifically rtl8192cu)

Sources:
 - http://www.erdahl.io/2016/04/configuring-wifi-on-beagleboardorg.html
 - https://learn.adafruit.com/setting-up-wifi-with-beaglebone-black/configuration
 - https://packages.debian.org/jessie/kernel/firmware-realtek

For my rtl8192cu I get the following message (on kernel 4.1.x):

```Text
[    3.235589] usbcore: registered new interface driver rtl8150
[    7.498277] rtl8192cu: Chip version 0x10
[    7.769925] rtl8192cu: MAC address: e0:3f:49:8f:50:1f
[    7.769952] rtl8192cu: Board Type 0
[    7.771832] rtl_usb: rx_max_size 15360, rx_urb_num 8, in_ep 1
[    7.771982] rtl8192cu: Loading firmware rtlwifi/rtl8192cufw_TMSC.bin
[    7.778692] usb 1-1: Direct firmware load for rtlwifi/rtl8192cufw_TMSC.bin failed with error -2
[    7.778759] usb 1-1: Direct firmware load for rtlwifi/rtl8192cufw.bin failed with error -2
[    7.778772] rtlwifi: Loading alternative firmware rtlwifi/rtl8192cufw.bin
[    7.778779] rtlwifi: Firmware rtlwifi/rtl8192cufw_TMSC.bin not available
[    7.823951] ieee80211 phy0: Selected rate control algorithm 'rtl_rc'
[    7.825363] usbcore: registered new interface driver rtl8192cu
```

The Debian `firmware-realtek` package will fix this issue:
```sh
apt-get install firmware-realtek wireless-tools
```

Then you should get the following result on boot:
```text
[    3.235541] usbcore: registered new interface driver rtl8150
[    7.507096] rtl8192cu: Chip version 0x10
[    7.766588] rtl8192cu: MAC address: e0:3f:49:8f:50:1f
[    7.766616] rtl8192cu: Board Type 0
[    7.767681] rtl_usb: rx_max_size 15360, rx_urb_num 8, in_ep 1
[    7.767828] rtl8192cu: Loading firmware rtlwifi/rtl8192cufw_TMSC.bin
[    7.804861] ieee80211 phy0: Selected rate control algorithm 'rtl_rc'
[    7.806270] usbcore: registered new interface driver rtl8192cu
[   18.603677] rtl8192cu: MAC auto ON okay!
[   18.676509] rtl8192cu: Tx queue select: 0x05
```

Edit `/etc/network/interfaces` and add the following lines:

```text
# WiFi Example
auto wlan0
iface wlan0 inet dhcp
    wpa-ssid "your-wlan-id"
    wpa-psk  "mypassword"
```
Then type:
```sh
ifup wlan0
```

Where `"your-wlan-id"` is the name of your wireless LAN and `"password"` is the password.  

Instead of these last two steps you could use `connmanctl` [see here](https://wiki.archlinux.org/index.php/Connman).

**Instead of `/etc/network/interfaces` you can use `connman` to configure wifi**:

## Use connmanctl to enable wifi

```sh
apt-get install connman
connmanctl
```

Then in `connmanctl` type the following commands:

```sh
enable wifi
scan wifi
agent on
services     # This will give you the list of avaiable services and their keys
```

The list of services will look something like:
```Text
*AO Wired                ethernet_68c90bed2fcc_cable
    Rigi97               wifi_e03f498f501f_526967693937_managed_psk
    Singer-AP            wifi_e03f498f501f_53696e6765722d4150_managed_psk
```

Then we connect to a wifi, still in `connmanctl` type:

```
connect wifi_e03f498f501f_526967693937_managed_psk
```

You will be asked to enter the passphrase.

### I/O (Flash Drive) Performance enhancements for the BeagleBone

[This link has some useful tips](http://jmahler.github.io/linux/2014/02/25/BBB-flash.html), these are summarised below.

#### Periodic TRIM Using cron

Add the following job to a daily cron job:
```sh
fstrim -v /
```

#### I/O Scheduler

Add the following lines to `/etc/rc.local`
```sh
echo noop > /sys/block/mmcblk0/queue/scheduler
```


# Getting I2C working

This is to determine which i2c bus the [3D click](http://www.mikroe.com/click/3d-motion/) is on.

Run:
```sh
i2cdetect -y -r 2
```

note `i2cdetect -y -r 1` or `i2cdetect -y -r 0` also works.  But for me bus 2
worked.  I received an output that implied a few devices are connected.  I got
the following result:

```text
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: 40 -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: 50 51 52 53 UU UU UU UU -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```

To determine which was the 3D click I rebooted and found that if I removed the
3D click the `40` value would disapper:

```text
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: 50 51 52 53 UU UU UU UU -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```

This means that the 3D Motion Click is on Bus 2 with an address of `0x40`. To get it going with JavaScript (bonescript).

```JavaScript
var b = require('octalbonescript');
var address = 0x040
b.i2c.open('/dev/i2c-2', address, function(data){
    console.log(data);
}, function(error, wire){
    if(error){
        console.error(error.message);
        return;
    }
});
```

Or you can use i2c (you need to install this npm module):

```JavaScript
var i2c = require('i2c');
var address = 0x40;
var wire = new i2c(address, {device: '/dev/i2c-2'});
wire.scan(function(err, data) {
    console.log(err, data);
});
```
