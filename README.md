### Ubuntu 20.04 prof 7500 patch

First of all we are going to install the missing dependencies
```
sudo -i
cd ~
git clone https://github.com/jordy33/ubuntu_20_prof7500_patch_tvheadend.git
cd ~
apt-get update && apt-get upgrade && apt-get install -y build-essential libncurses5-dev gcc libssl-dev grub2 bc bison flex libelf-dev
```

Then we are going to download the kernel from kernel.org
```
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.7.5.tar.xz
tar -xvf linux-5.7.5.tar.xz
```
Then we need to copy the existing config file so that the correct modules for the Hardware of the machine you are compiling the kernel for

You need to enter the command below to find the current kernel config file.
```
ls -l /boot
cp /boot/config-4.XXXX-generic /root/linux-5.7.5/.config
```

Then we are going to compile the kernel firstly you want to cd into the kernel folder
```
cd /root/linux-5.7.5
```

Copy patched file stv0900_core.c the directory
```
cp /root/ubuntu_20_prof7500_patch_tvheadend/svt0900_core.c /root/linux-5.7.5/drivers/media/dvb-frontends/.
```
(Adjust -j8 to the amount of cores the machine has that you are compiling the kernel for)
```
make menuconfig
make -j8 deb-pkg
dpkg -i ../linux-*.deb
update-grub
```
Then you can proceed to rebooting the machine by typing the following
```
reboot
```
Once the machine is rebooted you can confirm the latest kernel by entering the following command
```
uname -a
```

### Install Tvheadend

Language

First run locale to list what locales currently defined for the current user account:
```
$ locale
```
Then generate the missing locale and reconfigure locales to take notice:
```
$ sudo locale-gen "en_US.UTF-8"
```
Generating locales...
  en_US.UTF-8... done
Generation complete.
```
$ sudo dpkg-reconfigure locales
```
Generating locales...
  en_US.UTF-8... up-to-date
Generation complete.

Nothing suggested above worked in my case (Ubuntu Server 12.04LTS). What finally helped was putting to the file /etc/environment:
```
LC_ALL=en_US.UTF-8
LANG=en_US.UTF-8
```
Dependencies
```
sudo aptitude install build-essential git pkg-config libssl-dev bzip2 wget gettext liburiparser-dev libpcre2-dev cmake libdvbcsa-dev debhelper libavahi-client-dev
```
### Setting default python

Listing available python 
```
ls /usr/bin/python*
```

Setting python3 default
```
sudo update-alternatives --config python
```

Configuring
```
./configure --enable-dvbcsa --enable-constcw  --enable-bundle
./Autobuild.sh
cd ..
```

```
sudo dpkg -i tvheadend_4.3*.deb
```

```
 ┌─────────────────────────┤ Configuring tvheadend ├─────────────────────────┐
 │ Tvheadend is accessed via a world reachable web interface (assuming your  │ 
 │ host is reachable from the internet). To protect from malicious use you   │ 
 │ must create an initial account to be able to login to Tvheadend. This     │ 
 │ account can not be changed nor deleted from within Tvheadend itself. If   │ 
 │ you want to change the superuser account you need to reconfigure the      │ 
 │ Tvheadend package.                                                        │ 
 │                                                                           │ 
 │ Choose a username for Tvheadend administrator.  
```

Choose: admin and password


it will show:
```
 | Tvheadend is accessed via a world reachable web interface (assuming your  │ 
 │ host is reachable from the internet). To protect from malicious use you   │ 
 │ must create an initial account to be able to login to Tvheadend. This     │ 
 │ account can not be changed nor deleted from within Tvheadend itself. If   │ 
 │ you want to change the superuser account you need to reconfigure the      │ 
 │ Tvheadend package. 
```

Next:
```
 │ After installation Tvheadend can be accessed via HTTP on port 9981. From  │ 
 │ this machine you can point your web-browser to http://localhost:9981/.    │ 
 │                                                                           │ 
 │ If you want to completely remove configuration, use your package          │ 
 │ managers --purge option, e.g, apt-get remove --purge tvheadend* 
Follow Directions give password Enter Tvheadend

http://192.168.1.84:9981. (use user and password previously selected)
```

### Create universal access to tvheadend
Create user * with password * and allow network: 192.168.0.0/24 and allow all permisions.
In TVH change in General/Base/Authentication type= Plain Insecure:


### Install FFmpeg
```
sudo apt install ffmpeg
ffmpeg -version
```


### Create a mux for NBC
In tvh add an IPTV network (no automatic).  Add a mux for that network and use a URL like this....

```
pipe:///usr/bin/ffmpeg -loglevel fatal -re -i http://admin:trinity@192.168.1.84:9981/stream/channel/43e4470dded1a2ca5c5c2d80f8df844b?ticket=8f09d95d8f714ff2b8054366f6d5c9f23013a7be -c:v copy -filter_complex [0:1][0:2][0:3]amerge=inputs=3,pan=5.1|FL=c0|FR=c1|FC=c2|LFE=c3|BL=c4|BR=c5 -c:a eac3 -f mpegts pipe:1
```

(Get the http link openning the link to open the channel)

It'll scan in a new service and you just map that to a new channel.  

### Install OSCAM

```
cd /home/pi
git clone https://github.com/oscam-emu/oscam-patched
cd oscam-patched
git checkout oscam11546-emu798
```

for options:
```
./config.sh | more
```

Compile
```
make USE_LIBCRYPTO=1
cd Distribution
sudo mv oscam-1.20_svn11546-798-arm-linux-gnueabihf /usr/local/bin/oscam
```
Create directories
```
cd ~
mkdir .oscam
cd oscam
mkdir tmp
```

Create file
```
vim ~/.oscam/oscam.conf
```
And Insert the following
```
# oscam.conf generated automatically by Streamboard OSCAM 1.20-unstable_svn SVN r11392
# Read more: http://www.streamboard.tv/svn/oscam/trunk/Distribution/doc/txt/oscam.conf.txt

[global]
logfile                       = stdout
fallbacktimeout               = 3
bindwait                      = 5
initial_debuglevel            = 64
nice                          = -1
maxlogsize                    = 200
waitforcards                  = 0
preferlocalcards              = 1
lb_mode                       = 1
lb_save                       = 100
lb_savepath                   = /home/pi/.oscam/oscam.stat
lb_auto_timeout_t             = 26000

[cache]

[streamrelay]
stream_relay_enabled          = 0

[dvbapi]
enabled                       = 1
au                            = 1
pmt_mode                      = 4
listen_port                   = 9001
delayer                       = 60
ecminfo_type                  = 4
user                          = tvheadend
read_sdt                      = 1
extended_cw_api               = 1
boxtype                       = pc

[webif]
httpport                      = 8888
httprefresh                   = 2
httppollrefresh               = 2
httpallowed                   = 127.0.0.1,192.168.0.0-192.168.254.254
```

Create the file:
```
vim ~/.oscam/oscam.server
```

And Insert the following

```
[reader]
label                         = emulator
protocol                      = emu
device                        = emulator
caid                          = 090F,0500,1801,0604,2600,FFFF,0E00,4AE1,1010
detect                        = cd
disablecrccws_only_for        = 0E00:000000
ident                         = 0D00:0000C0;0D02:0000A0;0500:023800,021110,007400;0E00:000000;1801:000000;1010:000000
group                         = 1
emmcache                      = 2,0,2,0
emu_auproviders               = 0E00:000000
auprovid                      = 000E00

[reader]
label                         = SoftCam.Key
protocol                      = constcw
device                        = /home/pi/.oscam/SoftCam.Key
caid                          = 0D00,0D02,090F,0500,1801,0604,2600,FFFF,0E00,4AE1,1010
ident                         = 1801:000000,001101;0604:000000;2600:000000;FFFF:000000;0E00:000000;4AE1:000000;1010:000000
group                         = 1

```

Create the file:
```
vim ~/.oscam/oscam.services
```

And insert the following:
```
# oscam.services generated automatically by Streamboard OSCAM 1.20_svn SVN r11441
# Read more: http://www.streamboard.tv/svn/oscam/trunk/Distribution/doc/txt/oscam.services.txt

[afn]
caid                          = 0E00
provid                        = 000000
srvid                         = 0004,0005,0006,0007,0008,0009,012C,0098,0096,04B1,03E9,04B2,00AD,00B7,00B9,00AF,019E,0193,019D,0192,0191,004A,004C,009D,00A0,004B,0097,000F,0047,0048,0003
```

Create the file:
```
vim ~/.oscam/oscam.user
```

And insert the following:
```
[account]
user                          = tvheadend
pwd                           = tvheadend
monlevel                      = 4
au                            = 1
group                         = 1,2,3,4,5,6,7,8,9,11,12,13,14,15,16,17,18,19,21,22,23,24,25,26,27,28,29,31,32,33,34,35,36,37,38
```

Create Deamon at run boot
```
sudo vim /etc/init.d/oscam
```
Insert the following
```
#!/bin/sh
### BEGIN INIT INFO
# Provides: oscam
# Required-Start: $local_fs $network $remote_fs
# Required-Stop: $local_fs $network $remote_fs
# Default-Start:  2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: start and stop service oscam
# Description: oscam
### END INIT INFO

DAEMON=/usr/local/bin/oscam
PIDFILE=/var/run/oscam.pid
DAEMON_OPTS="-p 4096 -r 2 -B ${PIDFILE} -c /home/pi/.oscam -t /home/pi/.oscam/tmp"

test -x ${DAEMON} || exit 0

. /lib/lsb/init-functions

case "$1" in
  start)
    log_daemon_msg "Starting OScam..."
    /sbin/start-stop-daemon --start --quiet --background --name oscam --exec ${DAEMON} -- ${DAEMON_OPTS}
    log_end_msg $?
    ;;
  stop)
    log_daemon_msg "Stopping OScam..."
    /sbin/start-stop-daemon -R 5 --stop --name oscam --exec ${DAEMON}
    log_end_msg $?
    ;;
  restart)
    $0 stop
    $0 start
    ;;
  force-reload)
    $0 stop
    /bin/kill -9 `pidof oscam`
    /usr/bin/killall -9 oscam
    $0 start
    ;;
  *)
    echo "Usage: /etc/init.d/oscam {start|stop|restart|force-reload}"
    exit 1
    ;;
esac
```

To “enable” the script and the starting up of oscam on reboot, use the following commands on Ubuntu/Debian:
```
sudo chmod +x /etc/init.d/oscam
sudo update-rc.d oscam defaults
```
To force detect service
```
sudo systemctl daemon-reload
sudo systemctl enable oscam
sudo systemctl start oscam
```

To test:
```
http://192.168.1.82:8888
```
sudo systemctl start oscam

Go to Tvheadend and 
Configure CAs
```
Configuration/CAs/+Add/CAPMT
Enabled: [x]
Name: emu
Mode: OSCam net protocol (rev >= 10389)
Camd.socket filename / IP Address (TCP mode): 127.0.0.1 
Listen / Connect port: 9001
```

Click Save
