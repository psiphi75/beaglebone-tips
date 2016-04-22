# Beaglebone Tips
Tips for the BeagleBone Green/Black.

## Start with a minimal image

### Step 1: Get the image

I start with the smallest image, then install what is required.  Typically I don't use Cloud9 IDE and I have never used X11 (the GUI).  I use the [microSD/Standalone: (console) (BeagleBone/BeagleBone Black/BeagleBone Green)](http://elinux.org/Beagleboard:BeagleBoneBlack_Debian#microSD.2FStandalone:_.28console.29_.28BeagleBone.2FBeagleBone_Black.2FBeagleBone_Green.29) image.  Although this is really bare-bones.  This is just under 400MB in size.

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
apt-get install nodejs=0.10.42*  nodejs-legacy npm
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
