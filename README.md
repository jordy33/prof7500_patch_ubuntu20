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




