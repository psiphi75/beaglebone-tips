Get the [Edup EP-N8508GS](http://edupwireless.com/product-1-2-5-mini-wireless-usb-adapter-en/137318) Wifi adapter working on the Beaglebone.  This contains the RTL8812 (8812au) chipset.

**Note: I found this wireless adapter has a very weak signal.**  But instructions are still here for if you have one.

```sh
apt-get install firmware-realtek
```

Then reboot.

Then use [`connmanctl`](https://github.com/psiphi75/beaglebone-tips#use-connmanctl-to-enable-wifi) to enable Wifi.

## Note: BeagleBone Black vs Green

I could not get this going on the BeagleBone Black, my guess is that it uses too much power.  However, it worked fine with the BeagleBone Green.
