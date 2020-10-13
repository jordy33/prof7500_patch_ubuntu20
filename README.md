### Ubuntu 20.04 prof 7500 patch

First of all we are going to install the missing dependencies
```
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
cp /boot/config-4.4.0-21-generic /root/linux-5.7.5/.config
```

Then we are going to compile the kernel firstly you want to cd into the kernel folder
```
cd /root/linux-5.7.5
```

Copy patched file stv0900_core.c the directory
```
cp ./svt0900_core.c /root/linux-5.7.5/drivers/media/dvb-frontends/.
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



