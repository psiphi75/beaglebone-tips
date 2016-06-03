Get the RTL8812 (8812au) working on the Beaglebone

```sh
apt-get install linux-headers-4.1.18-ti-r56 build-essential git

cd /lib/modules/4.1.18-ti-r56/build
ln -s /usr/src/linux-headers-4.1.18-ti-r56 build

git clone https://github.com/abperiasamy/rtl8812AU_8821AU_linux
cd rtl8812AU_8821AU_linux
```

Edit `Makefile` and turn on pi and off x86

Turn off `CONFIG_PLATFORM_I386_PC = n`

Turn on `CONFIG_PLATFORM_ARM_RPI = y`

```sh
make
make install
```

Then use [`connmanctl`](https://github.com/psiphi75/beaglebone-tips#use-connmanctl-to-enable-wifi) to enable Wifi.
