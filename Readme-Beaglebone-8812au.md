Get the [Edup EP-N8508GS](http://edupwireless.com/product-1-2-5-mini-wireless-usb-adapter-en/137318) Wifi adapter working on the Beaglebone.  This contains the RTL8812 (8812au) chipset.

**Note: I found this wireless adapter has a very weak signal.**  But instructions are still here for if you have one.

```sh

#
# You may need to change '4.1.18-ti-r56' to the version of Linux you are currently running.
#

apt-get install linux-headers-4.1.18-ti-r56 build-essential git

cd /lib/modules/4.1.18-ti-r56/build
ln -s /usr/src/linux-headers-4.1.18-ti-r56 build

cd ~
git clone https://github.com/abperiasamy/rtl8812AU_8821AU_linux
cd rtl8812AU_8821AU_linux
```

Edit `Makefile` and turn on pi and off x86

Turn off `CONFIG_PLATFORM_I386_PC = n`

Turn on `CONFIG_PLATFORM_ARM_RPI = y`

```sh
make
make install

modprobe rtlwifi
```



Then use [`connmanctl`](https://github.com/psiphi75/beaglebone-tips#use-connmanctl-to-enable-wifi) to enable Wifi.
