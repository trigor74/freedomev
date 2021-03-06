# freedomev
FreedomEV repository. Unlocking the full potential of Linux on your Electric Vehicle!
Getting ready to support Model S and X with ARM based MCU.

Newer Model S and X and Model 3 have an Intel based MCU, porting should be doable; other location for persistent launch and storage might be needed as well as other adjustments. If you have root on such a car and would like to explore, contact us. Similar for other Electric Vehicles with Linux running on them.

# Launched!

Official FOSDEM 2019 link: https://fosdem.org/2019/schedule/event/tesla_hacking/

# Wiki
FreedomEV wiki also contains a lot of information: https://www.freedomev.com/wiki

### Disclaimer
This is not made by Tesla Inc., nor officially endorsed (just yet?). We take no responsibility whatsoever for damage, costs, injury or even death caused by this to you or third parties. In certain territories, it might violate local regulations, we don't know about that, and we don't care. We hope FreedomEV will enjoy your car even more.
I am tell my Tesla Service Center about what we are doing here. When the car is serviced, I disable all strange stuff so they might not become too confused (unless they ask for it). Tesla car service people have instructions to remove all visible USB attached stuff to not interfere with possible updates, which is a good thing. Also, I don't claim warranty on stuff I broke myself, Tesla has been very reasonable with respect to that. 
We build this so all changes, apps and additions are on a USB stick. So, when you remove that and reboot the systems, everything is back like it was before. The '/var/freedomevstart' script is the only persistent addition and it stops immediately if the USB stick is not detected.

We hope Tesla will provide - in a not too distant future - a legitimate way for owners to get root on your own car. For example by allowing owners to request a secret ssh token through their web account or car app. Nothing is certain until Elon Musk tweets about it ;)
We also hope other electric car manufacturers will be inspired, and allow the community to grow beyond what we currently imagine.

## Prerequisites
You need root access on the Central Instrument Display of your Tesla Model S or Model X with MCU 2.0 (arm based).
Currently only tested on my Model X. I suppose it will also work on the Model S as the architecture is very similar.
The latest generation of Model S/X and the Model 3 might be more problematic 
_for now_ as they use an Intel based board instead of the ARM based Linux system. If someone has root to such a Tesla, we might get FreedomEV working. Aside from root access, we need some kind of 'persistence across reboot'. On the MCU 1.0 and 2.0 cars this is easily accomplished using the crontab as it reads from a read/writeable /var filesystem.
Model 3 cars are even better closed down and harder to root. Tesla gives high bug bounties for those people finding root exploits and/or persistence across reboots; thus ensuring everybody their cars are safer. These tools and FreedomEV can help security researchers to better analyse and find potential problems.
With root access, FreedomEV can be installed with one command:
```
curl https://raw.githubusercontent.com/jnuyens/freedomev/master/install | bash

```
To fully remove FreedomEV - this will remove the /var/freedomevstart and crontab entry run:
```
curl https://raw.githubusercontent.com/jnuyens/freedomev/master/remove | bash
```

Additionally a USB stick with the 'Ubuntu for NVIDIA Tegra' based image is necessary:

## Installation - easy way: prepare a USB stick from a Linux desktop system
You need a USB stick to insert into the car - best 16 or 32GB, formatted as ext4.
Download and extract the latest image tarball on it as the root user (so the permissions and special files are correct).
Download here: https://www.freedomev.com/FreedomEV-1.0-usbstickimage-extract-to-an-ext4-filesystem.tgz
If you mounted the stick to /mnt/stick:
```
cd /mnt/stick
tar xvf ~/Downloads/FreedomEV-1.0-usbstickimage-extract-to-an-ext4-filesystem.tgz
mv freedomev-1.0-rootfs/* .
mv freedomev-1.0-rootfs/.git .
sync
cd 
umount /mnt/stick
```
It depends on the speed of your USB stick if this will take a while.
Insert the USB stick into the car, it will hopefully be mounted on a subdirectory of /disk/
Go into the chrooted environment:
```
chroot /disk/usb.*/
```
Update to the latest version of FreedomEV (execute in chroot on the stick / directory):
```
git pull 
#webgui for hotspot mode is currently still a git submodule
cd /var/www/html/raspap-webgui
git pull
```

And test it out! (see below)

## Installation instructions - manual installation
We start out from the NVIDIA Ubuntu image
Extract it on a USB stick, use the mounting/chroot script and install extra packages (we don't use them all yet).
Ensure you are connected to WiFi to not pull too much extra data over the Tesla network.
``` 
apt-get install flex bison liblzo2-dev texinfo zlib1g-dev zlib1g ncurses-dev g++ texinfo subversion m4 gettext texi2html liblzo2-2 libacl1 libacl1-dev libglib2.0-dev autoconf automake libtool git libexif-dev notify-osd ruby-notify zenity x11-apps kde-runtime oneko xpenguins xphoon xjokes kaffeine marble libreadline-dev libc-dev libncurses5-dev libreadline-dev libsqlite3-dev libgtk2.0-dev libagg-dev libfribidi-dev nmap wireshark tcpdump firefox i3 fbi imagemagick mplayer hostapd dnsmasq bridge-utils dstat screen tmux chromium-browser mpg123 feh vim qv4l2 qv4l2 kde-style-breeze-qt4 locate links wget festival festvox-kallpc16k festvox-kdlpc16k  festvox-us-slt-hts festvox-don festvox-en1 festvox-rablpc16k  festvox-us1  festvox-us2 festvox-us3 xterm konsole xdotool x11-utils xauth xserver-xephyr
```

On your usb stick you can go to the / filesystem directory and 
```
git clone http://www.github.com/jnuyens/freedomev .
```
For easy access to the applications, adjust your path:
```
echo "export PATH=$PATH:/freedomev/tools" >> ~/.bash_profile
source ~/.bash_profile
```

### Test it out:
```
cid-message "FreedomEV Upgrade Initiated, prepare to be Pan Galactic Gargleblasted!"
```
This should show this message on your central display

### Lets test some more:
```
moonshine.sh
```
This should make the colors on you Instrument Cluster (behind the steering wheel) fade slowly


## How to contribute:
Have a look at the various directories in /freedomev/apps-available they all have a similar structure. These are the apps which are currently available in FreedomEV. Each app is using its own directory. If an app is enabled, a symbolic link is created in the /freedomev/apps directory to the /freedomev/apps-available directory.
Adding an app or feature is simple.
The most complete way to start out, is by looking at the 0template app in the /freedomev/apps-inprogress/ directory. In this directory apps are located which are not ready 'for prime time' yet, you can see if there's something you can contribute to.
These are the files and directories in the 0template directory, most of them are optional:
* optional file activation
  This file is run every time FreedomEV is activated. The USB stick looses power whenever the system goes to a certain sleep state. This file starts the required services as soon as FreedomEV becomes active again.
* optional file deactivation
  This file is run when FreedomEV detects that the USB stick containing the root filesystem is no longer there. At activation this script is copied to the not-chrooted /tmp/freedomev/deactivation-applicationname so it will still be available. Here we can kill processes which can't run properly anyway as their root filesystem is gone.
* description.json
  This file contains the full name of the application as how it will be displayed into the configuration screen. As well as a description, as well as the possibility to hide this application in the web interface. The 00core of FreedomEV provides the core functionality such as the configuration screen, this is uses the 'hidden' option, so it cannot be disabled by accident. Other apps might make use of it.
* optional directories: everyminute, everyfiveminutes, everyhour, everyday and everymonth can contain executable scripts. They will be run at those time intervals. Suitable for most periodic tasks.


Have fun!
Email me for questions/additions. Pull requests welcome.
jasper at linux.com

Interesting related projects:
http://github.com/lunars/tesla

