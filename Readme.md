# OctoPrint + Motion Ubuntu 18.04 Setup Guide

Working in Ubuntu 18.04 LTS

Original OctoPrint Guide : https://discourse.octoprint.org/t/setting-up-octoprint-on-a-raspberry-pi-running-raspbian/2337

## Description

This guide was created because following the original guide lead to some problems. Plus I wanted Ubuntu running Octoprint and a true webcam stream (not time-lapse photos). Performance is much better than a RaspPI.

Everything was configured in this order from a fresh desktop installation.
I chose to autologon my main user because security isn't a real concern here.
Desktop was chosen over Server because I wanted to use the computer for other clicky pointy things in my garage.

## Updates

First thing's first, update:

```
sudo apt-get update && sudo apt-get -y upgrade
```

## Install Drivers

This auto installer is really nice for getting all system drivers.

```
sudo ubuntu-drivers autoinstall
```

## Allow USB Access

Add `$USER` to dialout group allowing USB / Serial access:

```
sudo adduser $USER dialout
```

Save `.\config-files\50-myusb.rules` to `/etc/udev/rules.d/50-myusb.rules`

Reboot: `sudo reboot now`

## Install Octoprint

Directly from the guide. Installs latest version:

```
cd ~
sudo apt update
sudo apt install python-pip python-dev python-setuptools python-virtualenv git libyaml-dev build-essential
mkdir OctoPrint && cd OctoPrint
virtualenv venv
source venv/bin/activate
pip install pip --upgrade
pip install https://get.octoprint.org/latest
```

Then exit out of `venv` and start in terminal with:

```
~/OctoPrint/venv/bin/octoprint serve
```

Browse to `http://0.0.0.0:5000/`

## GCODE

In OctoPrint, click the settings wrench and find GCODE. Then add these scripts as needed.

#### Start

```
G21 ;metric values
G90 ;absolute positioning
M82 ;set extruder to absolute mode
M107 ;start with the fan off
G28 ;home
M190 S70 ;bed temperature wait
M109 S220 ;extruder temperature wait
G0 X20 Y20 F15000 ;bring extruder to the front
G1 Z25 F15000 ;move the platform down 25mm
G92 E0 ;zero the extruded length
G1 F200 E55 ;extrude 55mm of feed stock
G92 E0 ;zero the extruded length again
G1 F15000 ; set travel speed
G0 X20 Y50 F15000 ; move head back
```

#### Print Complete or Canceled

```
M107 ;fans off
G91 ;relative positioning
G1 E-1 F200 ;retract the filament a bit before lifting the nozzle, to release some of the pressure
G1 Z+5 E-25 F15000 ;move Z up a bit and retract filament even more
G28 ;Home
M84 ;steppers off
G90 ;absolute positioning
M140 S50 ;bed temperature don't wait
M104 S75 ;extruder temperature don't wait
```

## Motion / Configure Webcam

Install with:

```
sudo apt-get install motion
mkdir ~/.motion
```

Copy `.\config-files\motion.conf` into `~/.motion/`

Reboot: `sudo reboot now`

Start in terminal with: `motion`

Browse to http://system.ip.address:8080 (not localhost)

## System Startup

Create `~/startup`

Move `.\config-files\startup\` to `~/startup/`

Assign Execute Permissions:

```
chmod u+x ~/startup/start_motion.sh
chmod u+x ~/startup/start_octoprint.sh
```

Test with:

```
cd ~/startup
./start_motion.sh
./start_octoprint.sh
```

Browse to `Startup Applications` and add these scripts.

## Conclusion

If everything was successful, issue a reboot. When the server reboots, OctoPrint should be running at `http://0.0.0.0:5000` and a webcam stream should be running at `http://system.ip.address:8080`

## Optional (not tested)

Install NetGear Wifi Drivers : https://ubuntuforums.org/showthread.php?t=1806839
