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
    ---> Package bash.armv6hl 0:4.2.48-2.fc20 will be an update
    ---> Package libtool-ltdl.armv6hl 0:2.4.2-24.fc20 will be updated
    ---> Package libtool-ltdl.armv6hl 0:2.4.2-26.fc20 will be an update
    ---> Package raspberrypi-kernel.armv6hl 0:3.12.26-1.20140808git4ab8abb.rpfr20 will be installed
    ---> Package raspberrypi-kernel-devel.armv6hl 0:3.12.23-2.20140626git25673c3.rpfr20 will be updated
    ---> Package raspberrypi-kernel-devel.armv6hl 0:3.12.26-1.20140808git4ab8abb.rpfr20 will be an update
    ---> Package raspberrypi-kernel-headers.armv6hl 0:3.12.23-2.20140626git25673c3.rpfr20 will be updated
    ---> Package raspberrypi-kernel-headers.armv6hl 0:3.12.26-1.20140808git4ab8abb.rpfr20 will be an update
    ---> Package raspberrypi-vc-demo-source.armv6hl 0:20140630git1682505-19.rpfr20 will be updated
    ---> Package raspberrypi-vc-demo-source.armv6hl 0:20140808gitdf36e8d-2.rpfr20 will be an update
    ---> Package raspberrypi-vc-firmware.armv6hl 0:20140630git1682505-19.rpfr20 will be updated
    ---> Package raspberrypi-vc-firmware.armv6hl 0:20140808gitdf36e8d-2.rpfr20 will be an update
    ---> Package raspberrypi-vc-libs.armv6hl 0:20140630git1682505-19.rpfr20 will be updated
    ---> Package raspberrypi-vc-libs.armv6hl 0:20140808gitdf36e8d-2.rpfr20 will be an update
    ---> Package raspberrypi-vc-libs-devel.armv6hl 0:20140630git1682505-19.rpfr20 will be updated
    ---> Package raspberrypi-vc-libs-devel.armv6hl 0:20140808gitdf36e8d-2.rpfr20 will be an update
    ---> Package raspberrypi-vc-static.armv6hl 0:20140630git1682505-19.rpfr20 will be updated
    ---> Package raspberrypi-vc-static.armv6hl 0:20140808gitdf36e8d-2.rpfr20 will be an update
    ---> Package raspberrypi-vc-utils.armv6hl 0:20140630git1682505-19.rpfr20 will be updated
    ---> Package raspberrypi-vc-utils.armv6hl 0:20140808gitdf36e8d-2.rpfr20 will be an update
    --> Finished Dependency Resolution
    
    Dependencies Resolved
    
    ================================================================================
     Package                    Arch    Version           Repository           Size
    ================================================================================
    Installing:
     raspberrypi-kernel         armv6hl 3.12.26-1.20140808git4ab8abb.rpfr20
                                                          pidora-rpfr-updates  11 M
    Updating:
     bash                       armv6hl 4.2.48-2.fc20     pidora-updates      921 k
     libtool-ltdl               armv6hl 2.4.2-26.fc20     pidora-updates       45 k
     raspberrypi-kernel-devel   armv6hl 3.12.26-1.20140808git4ab8abb.rpfr20
                                                          pidora-rpfr-updates 7.9 M
     raspberrypi-kernel-headers armv6hl 3.12.26-1.20140808git4ab8abb.rpfr20
                                                          pidora-rpfr-updates 830 k
     raspberrypi-vc-demo-source armv6hl 20140808gitdf36e8d-2.rpfr20
                                                          pidora-rpfr-updates  30 M
     raspberrypi-vc-firmware    armv6hl 20140808gitdf36e8d-2.rpfr20
                                                          pidora-rpfr-updates 3.0 M
     raspberrypi-vc-libs        armv6hl 20140808gitdf36e8d-2.rpfr20
                                                          pidora-rpfr-updates 397 k
     raspberrypi-vc-libs-devel  armv6hl 20140808gitdf36e8d-2.rpfr20
                                                          pidora-rpfr-updates 290 k
     raspberrypi-vc-static      armv6hl 20140808gitdf36e8d-2.rpfr20
                                                          pidora-rpfr-updates 8.4 k
     raspberrypi-vc-utils       armv6hl 20140808gitdf36e8d-2.rpfr20
                                                          pidora-rpfr-updates 143 k
    
    Transaction Summary
    ================================================================================
    Install   1 Package
    Upgrade  10 Packages
    
    Total download size: 55 M
    Is this ok [y/d/N]: y
    Downloading packages:
    No Presto metadata available for pidora-rpfr-updates
    No Presto metadata available for pidora-updates
    warning: /var/cache/yum/arm/20/pidora-updates/packages/libtool-ltdl-2.4.2-26.fc20.armv6hl.rpm: Header V4 DSA/SHA1 Signature, key ID 67fd194f: NOKEY
    Public key for libtool-ltdl-2.4.2-26.fc20.armv6hl.rpm is not installed
    (1/11): libtool-ltdl-2.4.2-26.fc20.armv6hl.rpm              |  45 kB  00:04     
    (2/11): bash-4.2.48-2.fc20.armv6hl.rpm                      | 921 kB  00:05     
    Public key for raspberrypi-kernel-3.12.26-1.20140808git4ab8abb.rpfr20.armv6hl.rpm is not installed
    (3/11): raspberrypi-kernel-3.12.26-1.20140808git4ab8abb.rpf |  11 MB  00:16     
    (4/11): raspberrypi-kernel-devel-3.12.26-1.20140808git4ab8a | 7.9 MB  00:17     
    (5/11): raspberrypi-kernel-headers-3.12.26-1.20140808git4ab | 830 kB  00:01     
    (6/11): raspberrypi-vc-firmware-20140808gitdf36e8d-2.rpfr20 | 3.0 MB  00:03     
    (7/11): raspberrypi-vc-libs-20140808gitdf36e8d-2.rpfr20.arm | 397 kB  00:00     
    (8/11): raspberrypi-vc-libs-devel-20140808gitdf36e8d-2.rpfr | 290 kB  00:00     
    (9/11): raspberrypi-vc-static-20140808gitdf36e8d-2.rpfr20.a | 8.4 kB  00:00     
    (10/11): raspberrypi-vc-utils-20140808gitdf36e8d-2.rpfr20.a | 143 kB  00:01     
    (11/11): raspberrypi-vc-demo-source-20140808gitdf36e8d-2.rp |  30 MB  00:34     
    --------------------------------------------------------------------------------
    Total                                              1.0 MB/s |  55 MB  00:53     
    Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-pidora-20
    Importing GPG key 0x67FD194F:
     Userid     : "pidora (fedora20v6) <seneca.sigul@gmail.com>"
     Fingerprint: a01d 420a bd0c ff14 7858 14bb 4483 4c0f 67fd 194f
     Package    : pidora-release-20-8.rpfr20.noarch (@pidora-rpfr/bluesky)
     From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-pidora-20
    Is this ok [y/N]: y
    Running transaction check
    Running transaction test
    Transaction test succeeded
    Running transaction (shutdown inhibited)
      Updating   : raspberrypi-vc-firmware-20140808gitdf36e8d-2.rpfr20.armv    1/21 
      Updating   : raspberrypi-vc-libs-20140808gitdf36e8d-2.rpfr20.armv6hl     2/21 
      Updating   : raspberrypi-vc-demo-source-20140808gitdf36e8d-2.rpfr20.a    3/21 
      Updating   : raspberrypi-vc-libs-devel-20140808gitdf36e8d-2.rpfr20.ar    4/21 
      Updating   : raspberrypi-vc-static-20140808gitdf36e8d-2.rpfr20.armv6h    5/21 
      Updating   : raspberrypi-kernel-devel-3.12.26-1.20140808git4ab8abb.rp    6/21 
      Updating   : raspberrypi-kernel-headers-3.12.26-1.20140808git4ab8abb.    7/21 
      Updating   : bash-4.2.48-2.fc20.armv6hl                                  8/21 
      Installing : raspberrypi-kernel-3.12.26-1.20140808git4ab8abb.rpfr20.a    9/21 
      Updating   : raspberrypi-vc-utils-20140808gitdf36e8d-2.rpfr20.armv6hl   10/21 
      Updating   : libtool-ltdl-2.4.2-26.fc20.armv6hl                         11/21 
      Cleanup    : raspberrypi-vc-utils-20140630git1682505-19.rpfr20.armv6h   12/21 
      Cleanup    : raspberrypi-vc-static-20140630git1682505-19.rpfr20.armv6   13/21 
      Cleanup    : raspberrypi-vc-libs-devel-20140630git1682505-19.rpfr20.a   14/21 
      Cleanup    : raspberrypi-vc-demo-source-20140630git1682505-19.rpfr20.   15/21 
      Cleanup    : raspberrypi-vc-libs-20140630git1682505-19.rpfr20.armv6hl   16/21 
      Cleanup    : raspberrypi-vc-firmware-20140630git1682505-19.rpfr20.arm   17/21 
      Cleanup    : bash-4.2.47-2.fc20.armv6hl                                 18/21 
      Cleanup    : raspberrypi-kernel-devel-3.12.23-2.20140626git25673c3.rp   19/21 
      Cleanup    : raspberrypi-kernel-headers-3.12.23-2.20140626git25673c3.   20/21 
      Cleanup    : libtool-ltdl-2.4.2-24.fc20.armv6hl                         21/21 
      Verifying  : raspberrypi-vc-demo-source-20140808gitdf36e8d-2.rpfr20.a    1/21 
      Verifying  : raspberrypi-vc-libs-devel-20140808gitdf36e8d-2.rpfr20.ar    2/21 
      Verifying  : libtool-ltdl-2.4.2-26.fc20.armv6hl                          3/21 
      Verifying  : raspberrypi-vc-static-20140808gitdf36e8d-2.rpfr20.armv6h    4/21 
      Verifying  : raspberrypi-kernel-3.12.26-1.20140808git4ab8abb.rpfr20.a    5/21 
      Verifying  : raspberrypi-kernel-headers-3.12.26-1.20140808git4ab8abb.    6/21 
      Verifying  : raspberrypi-vc-utils-20140808gitdf36e8d-2.rpfr20.armv6hl    7/21 
      Verifying  : raspberrypi-vc-libs-20140808gitdf36e8d-2.rpfr20.armv6hl     8/21 
      Verifying  : raspberrypi-vc-firmware-20140808gitdf36e8d-2.rpfr20.armv    9/21 
      Verifying  : raspberrypi-kernel-devel-3.12.26-1.20140808git4ab8abb.rp   10/21 
      Verifying  : bash-4.2.48-2.fc20.armv6hl                                 11/21 
      Verifying  : libtool-ltdl-2.4.2-24.fc20.armv6hl                         12/21 
      Verifying  : raspberrypi-kernel-headers-3.12.23-2.20140626git25673c3.   13/21 
      Verifying  : raspberrypi-vc-libs-20140630git1682505-19.rpfr20.armv6hl   14/21 
      Verifying  : raspberrypi-vc-demo-source-20140630git1682505-19.rpfr20.   15/21 
      Verifying  : raspberrypi-vc-libs-devel-20140630git1682505-19.rpfr20.a   16/21 
      Verifying  : raspberrypi-kernel-devel-3.12.23-2.20140626git25673c3.rp   17/21 
      Verifying  : raspberrypi-vc-static-20140630git1682505-19.rpfr20.armv6   18/21 
      Verifying  : raspberrypi-vc-firmware-20140630git1682505-19.rpfr20.arm   19/21 
      Verifying  : raspberrypi-vc-utils-20140630git1682505-19.rpfr20.armv6h   20/21 
      Verifying  : bash-4.2.47-2.fc20.armv6hl                                 21/21 
    
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

This will connect to the wifi network and create a file named /etc/sysconfig/network-scripts/keys-NotTheRealNetworkName.

    nmcli connection edit con-name wlan0

Then use the various sections to set the SSID, etc.

The "nmcli" commands end up generating the /etc/sysconfig/network-scripts/ifcfg-wlan0 containing the
following (redacted) content:

    [root@pidora ~]# cat /etc/sysconfig/network-scripts/ifcfg-wlan0 
    ESSID="NotTheRealNetworkName"
    MODE=Managed
    KEY_MGMT=WPA-PSK
    TYPE=Wireless
    BOOTPROTO=dhcp
    DEFROUTE=yes
    IPV4_FAILURE_FATAL=no
    IPV6INIT=yes
    IPV6_AUTOCONF=yes
    IPV6_DEFROUTE=yes
    IPV6_PEERDNS=yes
    IPV6_PEERROUTES=yes
    IPV6_FAILURE_FATAL=no
    NAME=wlan0
    UUID=c4b66b09-abda-400f-8bbf-00b4b2bb9844
    ONBOOT=yes
    PEERDNS=yes
    PEERROUTES=yes

Use the "pifconfig" command (the renamed ifconfig command, don't ask me why!) to verify that things are working.
