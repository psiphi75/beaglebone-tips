
# Getting the BeagleBone Internet connected via USB with Linux as your host

These instructions are to get the BeagleBone (Black / Green) connected to the Internet using only the USB connection.  
These instructions are [based on these](https://elementztechblog.wordpress.com/2014/12/22/sharing-internet-using-network-over-usb-in-beaglebone-black/).


## On the Linux Host

```sh
sudo su

# These values are for my host, your values may be different.  Use `ifconfig` to figure them out.

# The NIC of your desktop
DESKTOP_NIC="wlo1"  

# Find the NIC of your BeagleBone
BB_IP="192.168.7.1"
BB_NIC=$(ifconfig | grep -B1 ${BB_IP} | grep -o "^\w*" | head -1 )

ifconfig ${BB_NIC} ${BB_IP}
iptables --table nat --append POSTROUTING --out-interface ${DESKTOP_NIC} -j MASQUERADE
iptables --append FORWARD --in-interface ${BB_NIC}  -j ACCEPT
echo 1 > /proc/sys/net/ipv4/ip_forward
```

## On the BeagleBone

```sh
ifconfig usb0 192.168.7.2
route add default gw 192.168.7.1

# This last step is only required if you can't connect with the above two instructions
echo "nameserver 1.1.1.1" >> /etc/resolv.conf
```
