# tiny-pi

A project inspired by the Adafruit Pi-Hole.

Features:

- Tiny screen with cool screensavers
- Spotify connect device
- DNS server with optional built in ad blocking

Parts:

- https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/
- https://www.adafruit.com/product/2298
- https://www.adafruit.com/product/2807
- https://www.adafruit.com/product/2256

## Setup

```bash
# Normal setup
$ sudo apt update
$ sudo apt upgrade
$ sudo dpkg --configure -a


# Install screensavers
$ sudo apt install xscreensaver
$ sudo apt install xscreensaver-data-extra xscreensaver-gl-extra

# Configure screensavers
$ xscreensaver

# Enable `ping` command for sonar screensaver
$ sudo chmod +s /usr/lib/xscreensaver/sonar


# Install spotify connect client
sudo apt install -y apt-transport-https curl
curl -sSL https://dtcooper.github.io/raspotify/key.asc | sudo apt-key add -v -
echo 'deb https://dtcooper.github.io/raspotify raspotify main' | sudo tee /etc/apt/sources.list.d/raspotify.list
sudo apt update
sudo apt install raspotify

# Configure client
sudo nano /etc/default/raspotify

# Restart client
sudo systemctl restart raspotify


# Restart for changes to take effect
sudo reboot


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
```
