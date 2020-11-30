# tiny-pi

A project inspired by the Adafruit Pi-Hole.

Features:

- Tiny screen with cool screensavers
- DNS server with optional built in ad blocking
- Spotify connect device
- Airplay audio device
- Multi-room audio server

Parts:

- https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/
- [Adafruit 320x240 2.8" TFT](https://www.adafruit.com/product/2298)
- https://www.adafruit.com/product/2807
- https://www.adafruit.com/product/2256

Useful links:

- https://learn.adafruit.com/adafruit-pitft-28-inch-resistive-touchscreen-display-raspberry-pi/easy-install-2
- https://learn.adafruit.com/pi-hole-ad-pitft-tft-detection-display/raspberry-pi-setup
- https://github.com/badaix/snapcast
  - https://github.com/badaix/snapcast/blob/master/doc/install.md#debian
  - https://github.com/skalavala/Multi-Room-Audio-Centralized-Audio-for-Home/blob/master/Install%20Snapcast%20Server.md
- [Smokeping configuration docs](https://oss.oetiker.ch/smokeping/doc/smokeping_config.en.html)
- [Setting up cacti](http://www.greatwhitewifi.com/2016/02/12/network-monitoring-with-raspberry-pi-part-1-cacti/)
- [Snapcast guide](https://www.technicallywizardry.com/speaker-multi-room-wireless-receiver/)
- [Shairport](https://github.com/mikebrady/shairport-sync)

## Setup

```bash
# Normal setup
$ sudo apt update
$ sudo apt upgrade
$ sudo dpkg --configure -a

# ---- ---- ---- ----

# Install screensavers
$ sudo apt install xscreensaver
$ sudo apt install xscreensaver-data-extra xscreensaver-gl-extra

# Configure screensavers
$ xscreensaver

# Enable `ping` command for sonar screensaver
$ sudo chmod +s /usr/lib/xscreensaver/sonar

# ---- ---- ---- ----

# Install spotify connect client
sudo apt install -y apt-transport-https curl
curl -sSL https://dtcooper.github.io/raspotify/key.asc | sudo apt-key add -v -
echo 'deb https://dtcooper.github.io/raspotify raspotify main' | sudo tee /etc/apt/sources.list.d/raspotify.list
sudo apt update
sudo apt install raspotify

# Configure client
sudo nano /etc/default/raspotify

# Set correct hardware output
# See: https://github.com/dtcooper/raspotify/issues/31
sudo nano /lib/systemd/system/raspotify.service
sudo systemctl daemon-reload

# Restart client
sudo systemctl restart raspotify

# Restart for changes to take effect
sudo reboot

# ---- ---- ---- ----

# For Adafruit TFT hats: 
# https://learn.adafruit.com/adafruit-pitft-28-inch-resistive-touchscreen-display-raspberry-pi/easy-install-2

# If using the adafruit 2.8" capacitive touch hat
cd ~
sudo pip3 install --upgrade adafruit-python-shell click==7.0
sudo apt-get install -y git
git clone https://github.com/adafruit/Raspberry-Pi-Installer-Scripts.git
cd Raspberry-Pi-Installer-Scripts

# If the rotation is not correct, you can run this command again
sudo python3 adafruit-pitft.py --display=28r --rotation=90 --install-type=fbcp

# ---- ---- ---- ----

# Install PiHole DNS server
curl -sSL https://install.pi-hole.net | sudo bash

# Update PiHole to use port 81
# Change `server.port = 80` to `server.port = 81`
sudo nano /etc/lighttpd/lighttpd.conf
# Restart lighttpd
sudo service lighttpd restart

# ---- ---- ---- ----

# Install smokeping server monitor

# First install apache, needed to serve smokeping (cgi)
sudo apt install apache2
# Enable apache CGI
sudo a2enmod cgi
# Restart apache
sudo systemctl restart apache2

# Now install smokeping
# TODO: Configure for emailing on server down
sudo apt install smokeping

# Adjust desired delay and ping count, this is the default if not otherwise specified by probe
# Recommended: step=300 pings=5
sudo nano /etc/smokeping/config.d/Database

# Configure probes
# See recommendation below
sudo nano /etc/smokeping/config.d/Probes

# Add your desired targets
sudo nano /etc/smokeping/config.d/Targets

# Restart the service
sudo systemctl restart smokeping

# ---- ---- ---- ----

# Install cacti network monitor
# Note: The password you use during setup will be the admin password on the web interface
sudo apt install cacti

# ---- ---- ---- ----

# Install audio server
# Note: Find URL of latest version
wget https://github.com/badaix/snapcast/releases/download/v0.22.0/snapserver_0.22.0-1_armhf.deb

# Note: Must use absolute path
sudo apt install /home/pi/snapserver_0.22.0-1_armhf.deb

# FIXME: This is not persisted across reboots
# Set up alsa loopback device. Local audio output should be directed here.
sudo modprobe snd-aloop
# Find the loopback device with
aplay -l

# FIXME: librespot crashes when outputting to an active loopback
# See: https://github.com/librespot-org/librespot/issues/542
# Update spotify client to use the loopback device
sudo nano /lib/systemd/system/raspotify.service
sudo systemctl daemon-reload
sudo systemctl restart raspotify

# Need to test if respotify still goes through headphones in this case.
# Probably need to route loopback to hardware as well with a system service
# alsaloop -C hw:LOOPBACK,OUTPUT -P hw:HEADPHONES,0

# Add loopback to snapcast server
sudo nano /etc/snapserver.conf
sudo systemctl restart snapserver
```

## Files

### `/etc/smokeping/config.d/Database`
```
*** Database ***

step     = 300
pings    = 5
```

### `/etc/smokeping/config.d/Probes`
```txt
*** Probes ***

+ FPing

binary = /usr/bin/fping

++ FPingLarge
packetsize = 1000
step = 600
pings = 4

++ FPingNormal
packetsize = 56
step = 300
pings = 4

++ FPingLongDelay
packetsize = 56
step = 900
pings = 4

```
