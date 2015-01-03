RaspberryPiSetup
================

What I did to set up my home Raspberry Pi as a headless server.

After playing around with the latest Raspbian (from the Raspberry Pi
foundation NOOBS image), I chose to use the Pidora image because it
contains more updated software.

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

When it's done, I popped the microSD card back out and popped it in to
have access to the partitions:

    [ksylvan@ksylvan-t420 ~]$ df | grep media
    /dev/mmcblk0p2 1967044   1727772     119652  94% /runmedia/ksylvan/rootfs
    /dev/mmcblk0p1   51082     22874      28208  45% /runmedia/ksylvan/BOOT

The image is 2GB, with a small boot partition and the root filesystem.

## Set up the headless file

Based on the information here:
http://zenit.senecac.on.ca/wiki/index.php/Pidora-Headless-Mode

In the BOOT partition of the micro-sd card, put the following in a
file named "headless":

    RESIZE
    SWAP=1024

This resizes the rootfs to the available space on the device (in my
case, 32GB).

Unmount the partitions now:

    $ cd
    $ sync
    $ umount /run/media/ksylvan/*

Now it's safe to pop out the card and pop it into the RPi.

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

Use the instructions from here:
http://fedoraproject.org/wiki/Networking/CLI#Wifi

You can use this command to scan the wireless networks:

    nmcli device wifi list

And then do this:

    nmcli device wifi connect YourNetworkSSID password NotTheRealPassword

This will connect to the wifi network named YourNetworkSSID and create
the needed files in /etc/sysconfig/network-scripts/ (a file called
ifcfg-YourNetworkSSID and keys-YourNetworkSSID).

Use the "pifconfig" command to verify that things are working.

Now we can reboot again to ensure that the system auto-connects to the
wifi network.

At this point, we have a system configured to connect to the WiFi
network.

## Problems with the Pidora "headless" mode

The way the Pidora headless mode works is that it runs /usr/bin/headon
early in the boot process. The intent is to be able to login to your
RPi without having to have a monitor and keyboard handy.

The /usr/bin/headon script does a few things if it finds the
/boot/headless file:

1. Start the sshd service and stop and disable the firstboot graphical
service.

2. Set up the rootfs-resize service and runs it (if the /boot/headless
file contains the RESIZE directive).

3. Set up /etc/sysconfig/network-scripts/ifcfg-eth0

4. restart NetworkManager

5. flash the IP address (using the RPi LED) and read the IP address
(through the speakers).

What this means is that if you typically don't have your ethernet
cable plugged into the device (or only intend to use it on Wireless
only), the boot process will have to time out before going on.

In any case, all of this network file setup and network restarts take
time.

So, in order to create a truly headless server that boots fast, I did
a few things:

1. Remove /boot/headless

    [root@pidora ~]# rm /boot/headless
    rm: remove regular file ‘/boot/headless’? y
    [root@pidora ~]#

So now /usr/bin/headon does nothing.

2. Set the default target to multi-user.target (no graphical desktop,
saves some memory).

    [root@pidora ~]# systemctl set-default multi-user.target

3. Use "ssh -X" and system-config-date to set the timezone (not
strictly necessary).

4. Next, protect the WIFI network setup from being overwritten.

I made the ifcfg-YourNetworkSSID and keys-YourNetworkSSID files
read-only (even to root)

    [root@pidora ~]# cd /etc/sysconfig/network-scripts/
    [root@pidora network-scripts]# chattr +i *YourNetworkSSID*
    [root@pidora network-scripts]# echo testing >> keys-YourNetworkSSID
    -bash: keys-YourNetworkSSID: Permission denied
    [root@pidora network-scripts]#

Now, I shut down the RPi, removed the ethernet cable, and restarted
it.

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

## Disable Wifi Dongle power management

Every once in a while, it seems the Wifi speed is very very slow.

The command "iwconfig wlan0" showed that powe management was on, which
might be responsible for the occasionally slow wireless network
connection.

I followed the instructions here:
https://ask.fedoraproject.org/en/question/8168/proper-disabling-of-wifi-power-management/
with one major change. The iwconfig script needed to be called with an
explicit path.

    [root@pidora ~]# cat /etc/NetworkManager/dispatcher.d/02-wlan-powersave-off
    #!/bin/sh
    IF=$1
    STATUS=$2
    me=$(basename $0)
    if [ "${IF}" = "wlan0" ] && [ "${STATUS}" = "up" ]; then
       /sbin/iwconfig ${IF} power off
       echo "${IF}: turning off powersave mode to prevent constant reconnections"
    fi
    [root@pidora ~]# chmod +x /etc/NetworkManager/dispatcher.d/02-wlan-powersave-off

Then rebooted and verified that wifi power management is off:

    [root@pidora ~]# journalctl --no-pager --reverse | grep 02-wlan-powersave-off 
    Dec 31 16:00:31 pidora.local nm-dispatcher.action[230]: 02-wlan-powersave-off: wlan0: turning off powersave mode to prevent constant reconnections
    [root@pidora ~]# iwlist wlan0 power 
    wlan0     Current mode:off

At this point, I shutdown the RPi, and put it in a corner of my
office, connected to power. The RPi now serves as an always-on
headless server.

## Adding Google Two-Step authentication for SSH on Pidora

Date: 1/2/2015

A day after setting up and placing my RPi on the net (via
port-forwarding using the uVerse gateway), I am seeing messages like
this when logging into the server:

    Last failed login: Fri Jan  2 10:56:51 PST 2015 from 122.225.97.77 on ssh:notty
    There were 876 failed login attempts since the last successful login.
    Last login: Fri Jan  2 10:46:45 2015 from 209.66.74.34

Obviously, the box is under attack by someone who is trying to ssh
into it.

So I added the google-authenticator to the SSH login process. I can
still use password-less key-based ssh logins, but attackers will have
to go through the google-authenticator steps before getting in.

I used this web page as a reference:
http://www.sbprojects.com/projects/raspberrypi/ga.php

Start by installing the google-authenticator RPM:

    [root@pidora log]# yum install google-authenticator
    Loaded plugins: langpacks, refresh-packagekit
    Resolving Dependencies
    --> Running transaction check
    ---> Package google-authenticator.armv6hl 0:1.0-0.gita096a62.fc20.2 will be installed
    --> Finished Dependency Resolution

The package in Fedora contains both the PAM module and the
google-authenticator binary.

Then run the google-authenticator binary and answer "y" to all questions:

    [root@pidora ~]# google-authenticator

    Do you want authentication tokens to be time-based (y/n) y

The program outputs a QR code you can scan on your phone using the
Google Authenticator app on iOS and Android, or you can manually add
the info with the data it displays.

Next, enable its use in the sshd PAM setup:

    [root@pidora ~]# echo 'auth required pam_google_authenticator.so' >> /etc/pam.d/sshd

And edit the file /etc/ssh/sshd_config to set
ChallengeResponseAuthentication to yes (that line is commented out by
default).

Now, restart the sshd service:

    [root@pidora ~]# systemctl restart sshd.service

Do not disconnect from your session till you have verified that it
works! In my case, I verified that my existing key-based
authentication worked, then I tried logging in from another machine:

    kayvan@pox[~]:501$ ssh root@XXXXXXXXXXX
    Password:
    Verification code:
    Last login: Fri Jan  2 12:20:16 2015 from 209.66.74.34
    [root@pidora ~]#

So even if someone manages to break the password (very little chance
of that, but still), they have to put in the time-based verification
code.

## Shellinabox and Lighttpd

I adapted the Raspbian-based tutorial here:
http://blog.remibergsma.com/2013/03/15/always-available-linux-terminal-shell-in-a-box-on-raspberry-pi/

Start by installing lighttpd and shellinabox:

    [root@pidora ~]# yum install lighttpd shellinaboz
    [...]
    Installed:
      lighttpd.armv6hl 0:1.4.35-1.fc20
       shellinabox.armv6hl 0:2.14-25.git88822c1.fc20
    Dependency Installed:
       compat-lua-libs.armv6hl 0:5.1.4-5.fc20
    Complete!

Choose your favorite settings from the options documented via "man
shellinaboxd" to put in /etc/sysconfig/shellinaboxd, but you'll have
to include --disable-ssl and --localhost-only.

These are the options I picked:

    [root@pidora ~]# echo 'OPTS="--no-beep -s /terminal:SSH --disable-ssl --localhost-only"' >> /etc/sysconfig/shellinaboxd

And enable and start the service:

    [root@pidora ~]# systemctl enable shellinaboxd.service
    ln -s '/usr/lib/systemd/system/shellinaboxd.service' '/etc/systemd/system/multi-user.target.wants/shellinaboxd.service'
    [root@pidora ~]# systemctl start shellinaboxd.service

If you do "netstat -ntl", you should see something like the following,
indicating that shellinabox daemon is listening on localhost:4200

    [root@pidora ~]# netstat -ntl
    Active Internet connections (only servers)
    Proto Recv-Q Send-Q Local Address           Foreign Address         State
    tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
    tcp        0      0 127.0.0.1:4200          0.0.0.0:*               LISTEN
    tcp6       0      0 :::22                   :::*                    LISTEN

Next, we will be setting up lighttpd to do a reverse proxy and provide the
SSL functionality).

First, add the following to the end of /etc/lighttpd/lighttpd.conf:

    proxy.server = (
     "/terminal" =>
      ( (
        "host" => "127.0.0.1",
        "port" => 4200
      ) )
    )

Next, we must enable the proxy.server functionality. In
/etc/lighttpd/modules.conf, modify the server.modules section to read
as follows:

    server.modules = (
      "mod_access",
    #  "mod_alias",
    #  "mod_auth",
    #  "mod_evasive",
    #  "mod_redirect",
    #  "mod_rewrite",
    #  "mod_setenv",
    #  "mod_usertrack",
      "mod_proxy",
    )

Then enable and start the lighttpd server:

    [root@pidora ~]# systemctl enable lighttpd.service
    ln -s '/usr/lib/systemd/system/lighttpd.service' '/etc/systemd/system/multi-user.target.wants/lighttpd.service'
    [root@pidora ~]# systemctl start lighttpd.service

Now you should be able to connect to your Raspberry Pi IP address
inside your network by opening http://yourpi/terminal and you should
see a login prompt.  If you installed the google-authenticator for
2-factor authentication of ssh login (as I have), then you will see a
"Verification code:" prompt as well!

I would not recommend making this available to the wider network,
though, not without the SSL piece we will set up next. Following the
exact same instructions I found in Remy Bergsma's blog, we will set up
the self-signed certificate:

    # openssl genrsa -des3 -out mypidora.key.passwd 2048

This will ask you for a password, twice. Pick something easy, like
"password", because in the next step we remove the password.

    # openssl rsa -in mypidora.key.passwd -out mypidora.key
    Enter pass phrase for mypidora.key.passwd:
    writing RSA key

If you don't remove the pass phrase, you will have to enter it each
time the web server restarts, which won't work for your headless server.

    # openssl req -new -key mypidora.key -out mypidora.csr

Answer the questions with reasonable values and you will have a
"mypidra.csr" file to use in the next step:

    # openssl x509 -in mypidora.csr -out mypidora.pem -req -signkey mypidora.key -days 3650
    # cat mypidora.key >> mypidora.pem

The "mypidora.pem" file now contains the self signed certificate,
which we can use for lighttpd.

    # mkdir /etc/lighttpd/ssl
    # cp mypidora.pem /etc/lighttpd/ssl

And now add the following to the /etc/lighttpd/lighttpd.conf file:

    $SERVER["socket"] == "192.168.1.234:443" {
      ssl.engine = "enable"
      ssl.pemfile = "/etc/lighttpd/ssl/mypidora.pem"
      server.name = "YOURSERVERNAME"
      server.errorlog = "/var/log/lighttpd/mypidora_serror.log"
      accesslog.filename = "/var/log/lighttpd/mypidora_saccess.log"
    }

YOURSERVERNAME is whatever you specified earlier when you generated
the self-signed certificate.

Then restart lighttpd again:

    # systemctl restart lighttpd.service

Now you should be able to navigate to https://yourpi/terminal and
ignore the warning in your browser and you will have encrypted traffic
to your ssh session in a browser window.

With the extra security of the google-authenticator, I chose not to
add the .htaccess based login (as Remy did in his post).
