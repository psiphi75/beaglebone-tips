
# Getting the BeagleBone Internet connected via USB with Linux as your host

These instructions are to get the BeagleBone (Black / Green) connected to the Internet using only the USB connection.  
These instructions are [based on these](https://elementztechblog.wordpress.com/2014/12/22/sharing-internet-using-network-over-usb-in-beaglebone-black/).


## On the Linux Host

```sh
sudo su

# These values are for my host, your values may be different.  Use `ifconfig` to figure them out.
WLAN="wlo1"
USB="enx88c2556ba65f"

ifconfig ${USB} 192.168.7.1
iptables --table nat --append POSTROUTING --out-interface ${WLAN} -j MASQUERADE
iptables --append FORWARD --in-interface ${USB} -j ACCEPT
echo 1 > /proc/sys/net/ipv4/ip_forward
```

## On the BeagleBone

```sh
ifconfig usb0 192.168.7.2
route add default gw 192.168.7.1

# This last step is only required if you can't connect with the above two instructions
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
```
