RaspberryPiSetup
================

What I did to set up my home Raspberry Pi as a headless server.

After playing around with the latest Raspbian (from the Raspberry Pi foundation NOOBS image), I chose
to use the Pidora image because it contains more updated software.

## Install Pidora F20 (as of 12/30/14)

1. Start by installing F20 Pidora (from
http://www.raspberrypi.org/downloads/ using the Pidora remix torrent
file) here: http://downloads.raspberrypi.org/pidora_latest.torrent

2. I inserted a 32GB micro-sd card into my Fedora 20 laptop, then
unmounted the mount(s):

    $ umount /run/media/ksylvan/*

3. installed the fedora-arm-installer and ran it.

    sudo yum install fedora-arm-installer
    fedora-arm-installer

Selet the sdcard device, install the image. This takes a few minutes.
The fedora-arm-installer will unzip the Pidora-2014-3.zip, verify the
md5sum of the image, then write it to the device.

When it's done, I popped the microSD card back out and popped it in to have
access to the partitions:

    [ksylvan@ksylvan-t420 ~]$ df | grep media
    /dev/mmcblk0p2 1967044   1727772     119652  94% /runmedia/ksylvan/rootfs
    /dev/mmcblk0p1   51082     22874      28208  45% /runmedia/ksylvan/BOOT

The image is 2GB, with a small boot partition and the root filesystem.

## Set up the headless file

Based on the information here: http://zenit.senecac.on.ca/wiki/index.php/Pidora-Headless-Mode

In the BOOT partition of the micro-sd card, put the following in a file named "headless":

    RESIZE
    SWAP=1024

This resizes the rootfs to the available space on the device (in my case, 32GB).

umount the partitions now:

    $ cd
    $ sync
    $ umount /run/media/ksylvan/*

now it's safe to pop out the card and pop it into the RPi.

Next, I connect the RPi via a wired cable to my uVerse router. On the
uVerse router page and boot it, I watch and see that a new device
called "pidora" shows up on the network.

## SSH to the server and update its software

    [ksylvan@ksylvan-t420 ~]$ ssh root@pidora
    The authenticity of host 'pidora (2602:306:309a:96f0:ba27:ebff:fea2:3afe)' can't be established.
    ECDSA key fingerprint is ef:82:48:79:0e:e0:59:db:38:89:35:d7:01:03:ec:23.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added 'pidora,2602:306:309a:96f0:ba27:ebff:fea2:3afe' (ECDSA) to the list of known hosts.
    root@pidora's password:
    [root@pidora ~]#

The root password by default is "raspberrypi", changed immediately:

    root@pidora ~]# passwd
    Changing password for user root.
    New password: 
    Retype new password: 
    passwd: all authentication tokens updated successfully.

Then "yum update":

    [root@pidora ~]# yum update
    Loaded plugins: langpacks, refresh-packagekit
    [...]
    Installed:
      raspberrypi-kernel.armv6hl 0:3.12.26-1.20140808git4ab8abb.rpfr20
    Updated:
      bash.armv6hl 0:4.2.48-2.fc20                                                  
      libtool-ltdl.armv6hl 0:2.4.2-26.fc20                                          
      raspberrypi-kernel-devel.armv6hl 0:3.12.26-1.20140808git4ab8abb.rpfr20        
      raspberrypi-kernel-headers.armv6hl 0:3.12.26-1.20140808git4ab8abb.rpfr20      
      raspberrypi-vc-demo-source.armv6hl 0:20140808gitdf36e8d-2.rpfr20              
      raspberrypi-vc-firmware.armv6hl 0:20140808gitdf36e8d-2.rpfr20                 
      raspberrypi-vc-libs.armv6hl 0:20140808gitdf36e8d-2.rpfr20                     
      raspberrypi-vc-libs-devel.armv6hl 0:20140808gitdf36e8d-2.rpfr20               
      raspberrypi-vc-static.armv6hl 0:20140808gitdf36e8d-2.rpfr20                   
      raspberrypi-vc-utils.armv6hl 0:20140808gitdf36e8d-2.rpfr20                    
    Complete!
    [root@pidora ~]# 

This took about 10-15 minutes to complete. Once done, I rebooted
(because of the kernel upgrade).

    [root@pidora ~]# shutdown -r now
    Connection to pidora closed by remote host.
    Connection to pidora closed.

## Wireless setup.

Use the instructions from here: http://fedoraproject.org/wiki/Networking/CLI#Wifi

You can use this command to scan the wireless networks:

    nmcli device wifi list

And then do this:

    nmcli device wifi connect YourNetworkSSID password NotTheRealPassword

This will connect to the wifi network named YourNetworkSSID and create
the needed files in /etc/sysconfig/network-scripts/ (a file called
ifcfg-YourNetworkSSID and keys-YourNetworkSSID).

Use the "pifconfig" command to verify that things are working.

Now we can reboot again to ensure that the system auto-connects to the wifi network.

At this point, we have a system configured to connect to the WiFi network.

## Problems with the Pidora "headless" mode

The way the Pidora headless mode works is that it runs /usr/bin/headon early in the boot
process. The intent is to be able to login to your RPi without having to have a monitor
and keyboard handy.

The /usr/bin/headon script does a few things if it finds the /boot/headless file:

1. Start the sshd service and stop and disable the firstboot graphical
service.

2. Set up the rootfs-resize service and runs it (if the /boot/headless file contains the
RESIZE directive).

3. Set up /etc/sysconfig/network-scripts/ifcfg-eth0

4. restart NetworkManager

5. flash the IP address (using the RPi LED) and read the IP address (through the
speakers).

What this means is that if you typically don't have your ethernet
cable plugged into the device (or only intend to use it on Wireless
only), the boot process will have to time out before going on.

In any case, all of this network file setup and network restarts take time.

So, in order to create a truly headless server that boots fast, I did
a few things:

1. Remove /boot/headless

    [root@pidora ~]# rm /boot/headless
    rm: remove regular file ‘/boot/headless’? y
    [root@pidora ~]#

So now /usr/bin/headon does nothing.

2. Set the default target to multi-user.target (no graphical desktop, saves some memory).

    [root@pidora ~]# systemctl set-default multi-user.target

3. Use "ssh -X" and system-config-date to set the timezone (not strictly necessary).

4. Next, protect the WIFI network setup from being overwritten.

I made the ifcfg-YourNetworkSSID and keys-YourNetworkSSID files read-only (even
to root)

    [root@pidora ~]# cd /etc/sysconfig/network-scripts/
    [root@pidora network-scripts]# chattr +i *YourNetworkSSID*
    [root@pidora network-scripts]# echo testing >> keys-YourNetworkSSID
    -bash: keys-YourNetworkSSID: Permission denied
    [root@pidora network-scripts]#

Now, I shut down the pi, removed the ethernet cable, and restarted it.

## Fast booting headless (and portable) server

Now I have a Raspberry Pi running Pidora that is a truly portable,
untethered, wifi-connected server.

I timed its startup, and from the time I plug the power cable in, to
the time that the blue LED lights up on the wifi USB dongle is under
30 seconds!

One last note, you will probably want to set up your wireless router
to give your Raspberry Pi a static IP address. I went into the web
interface for my uVerse router and gave my Pi a fixed allocated IP
address, which also makes it possible to forward ssh or https traffic
to it.
