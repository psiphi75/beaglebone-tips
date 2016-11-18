# Installing Edimax AC600 USB driver for the Beaglebone Black/Green 

The [Edimax AC600](http://www.edimax.com/edimax/merchandise/merchandise_detail/data/edimax/au/wireless_adapters_ac600_dual-band/ew-7811utc)
needs special treatment to install the drivers for Debian on the BeagleBone Black or BeagleBone Green.

The instructions are based on those [found here](https://ubuntuforums.org/showthread.php?t=2228244&s=e1c4da1480c7ccc8b73235ff344c14fe&p=13042921#post13042921).

```sh
apt-get update
apt-get install linux-headers-$(uname -r) build-essential git
git clone https://github.com/gnab/rtl8812au.git
cd ~/rtl8812au

# 
# Before we can run `make`, we need to add the `armv7l` directory.  I don't know why it's missing or if it's the correct thing
# to do.  But if fixes a `make` problem and doesn't have any issues.
#
ln -s /usr/src/linux-headers-$(uname -r)/arch/arm /usr/src/linux-headers-$(uname -r)/arch/armv7l 

# Now we can continue and make

make
make install
modprobe 8812au
```
