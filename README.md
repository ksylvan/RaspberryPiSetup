RaspberryPiSetup
================

What I did to set up my home Raspberry Pi as a headless server.

After playing around with the latest Raspbian (from the Raspberry Pi foundation NOOBS image), I chose
to use the Pidora image because it contains more updated software.

## Install Pidora F20 (as of 12/30/14)

1. Start by installing F20 Pidora (from http://www.raspberrypi.org/downloads/ using the Pidora remix torrent file)
2. On my Fedora 21 laptop, I installed the fedora-arm-installer:

    sudo yum install fedora-arm-installer
    fedora-arm-installer

Then, selet the sdcard device, install the image.

## Set up the headless file

Based on the information here: http://zenit.senecac.on.ca/wiki/index.php/Pidora-Headless-Mode

In the BOOT partition of the micro-sd card, put the following in a file named "headless":

    RESIZE
    SWAP=1024

This resizes the rootfs to the available space on the device (in my case, 32GB).

Then I plug the pi via a wired cable to my uVerse router. On the uVerse router page,
I watch and see that a new device called "pidora" shows up on the network.

## SSH to the server and update its software

    $ ssh pidora -l root
    root@pidora's password:

The root password by default is "raspberrypi", changed immediately:

    root@pidora ~]# passwd
    Changing password for user root.
    New password: 
    Retype new password: 
    passwd: all authentication tokens updated successfully.

Then "yum update":

    [root@pidora ~]# yum update
    Loaded plugins: langpacks, refresh-packagekit
    pidora                                                      | 3.9 kB  00:00     
    pidora-rpfr-updates                                         | 3.7 kB  00:00     
    pidora-updates                                              | 3.8 kB  00:00     
    (1/5): pidora-rpfr-updates/20/arm/primary_db                |  20 kB  00:10     
    (2/5): pidora-updates/20/arm/group_gz                       | 394 kB  00:12     
    (3/5): pidora-rpfr-updates/20/arm/group_gz                  | 394 kB  00:13     
    (4/5): pidora-updates/20/arm/primary_db                     | 6.1 MB  00:26     
    (5/5): pidora/20/arm/primary_db                             |  16 MB  00:33     
    pidora/20/arm/group_gz                                      | 394 kB  00:07     
    Resolving Dependencies
    --> Running transaction check
    ---> Package bash.armv6hl 0:4.2.47-2.fc20 will be updated
    [...]
    Transaction Summary
    ================================================================================
    Install   1 Package
    Upgrade  10 Packages
    Total download size: 55 M
    Is this ok [y/d/N]: y
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

Then rebooted (because of the kernel upgrade).

## Wireless setup.

Use the instructions from here: http://fedoraproject.org/wiki/Networking/CLI#Wifi

Basically, first do this:

    nmcli device wifi connect NotTheRealNetworkName password NotTheRealPassword

This will connect to the wifi network and create a file the needed files in /etc/sysconfig/network-scripts/ (a file called ifcfg-NotTheRealNetworkName and keys-NotTheRealNetworkName).

Use the "pifconfig" command (the renamed ifconfig command, don't ask me why!) to verify that things are working.
