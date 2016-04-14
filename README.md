# beaglebone-tips
Tips for the BeagleBone Green/Black.

## Reduce the size of the image

First start with the smallest image.  I use the [microSD/Standalone: (console) (BeagleBone/BeagleBone Black/BeagleBone Green)](http://elinux.org/Beagleboard:BeagleBoneBlack_Debian#microSD.2FStandalone:_.28console.29_.28BeagleBone.2FBeagleBone_Black.2FBeagleBone_Green.29) image.  Although this is really bare-bones.  This is just under 400MB in size.

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
apt-get install nodejs=0.10.42~dfsg-0rcnee2~bpo80+20160217+1  nodejs-legacy
```

Finish up with:
```bash
apt-get autoremove
apt-get clean
```

To load DTOs you will need to follow these instructions:
https://github.com/beagleboard/bb.org-overlays


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
