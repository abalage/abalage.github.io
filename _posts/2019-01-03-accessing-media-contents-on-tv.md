---
title: "Accessing media contents on TV"
date: 2019-01-03 14:54:23 +0000
categories: [entertainment]
---

We already able to access our NFS exports on our Linux workstations or client machines. But how to access them on a TV?

There are two TVs in this article, one comes with an Ethernet connector and has [DLNA](https://en.wikipedia.org/wiki/Digital_Living_Network_Alliance) functionality, while the other although also has "Smart TV" capability, but it is limited to a single USB storage device and does not support DLNA. Both of them are Samsung devices, but it does not really matter.

Let see how to access the media contents on them. I tell no lies, NFS will not play any role in it.

## Smart TV with Ethernet connection

This one is easier than the other case. The only problem I had with this TV is that it does not support NFS exports therefore I cannot make it work out of the box. I need something to "translate" between the NFS share and the TV and that is where DLNA comes into the picture.

DLNA itself is a standard for sharing multimedia contents between different devices, like smartphones, TV, tablets, etc. It contains multiple parts responsible for different tasks. One of them is DMS (Digital Media Server) which makes the media contents available for our TV. This is the the task what our server will do with the help of MiniDLNA.

In a nutshell, we will install MiniDLNA and set up where it can find our media contents and how it should categorize the contents. It will operate as a network service therefore we need to open up a firewall port so the TV can access it. It is not rocket science, there are many [guides](https://wiki.archlinux.org/index.php/ReadyMedia) over the Internet about it already. So let's see how to do it.

### Setup MiniDLNA (ReadyMedia)

1. As MiniDLNA is not in the default repositories of openSUSE, we need to install it from [packman](https://en.opensuse.org/Additional_package_repositories#Packman) repository:

```bash
test@server:~> sudo zypper ar -cfp 90 http://ftp.gwdg.de/pub/linux/misc/packman/suse/openSUSE_Leap_15.0/ packman
Adding repository 'packman' ...........[done]
Repository 'packman' successfully added

URI         : http://ftp.gwdg.de/pub/linux/misc/packman/suse/openSUSE_Leap_15.0/
Enabled     : Yes
GPG Check   : Yes
Autorefresh : Yes
Priority    : 90 (raised priority)

Repository priorities in effect: (See 'zypper lr -P' for details)
      90 (raised priority)  :  1 repository  
      99 (default priority) :  4 repositories
test@server:~> sudo zypper ref
Retrieving repository 'packman' metadata --------[\]

New repository or package signing key received:

  Repository:       packman
  Key Name:         PackMan Project (signing key) <packman@links2linux.de>
  Key Fingerprint:  F8875B88 0D518B6B 8C530D13 45A1D067 1ABD1AFB
  Key Created:      Mon Sep 15 22:18:00 2014
  Key Expires:      Thu Sep 12 22:17:21 2024
  Rpm Name:         gpg-pubkey-1abd1afb-54176598

Do you want to reject the key, trust temporarily, or trust always? [r/t/a/? shows all options] (r): a
Retrieving repository 'packman' metadata .........[done]
Building repository 'packman' cache ........[done]
All repositories have been refreshed.
test@server:~> sudo zypper in -y minidlna
```

1. Edit its configuration file which is located in **/etc/sysconfig/minidlna** to contain the directory paths. Here is an example with helpful comments:

```
# Here you can define your media directories you want scanned
#
# Set this to the directories you want scanned.
#
# * if you want to restrict a media_dir to specific content types, you
#   can prepend the types, followed by a comma, to the directory:
#
#   + "A" for audio  (eg. A,/home/jmaggard/Music)
#   + "V" for video  (eg. V,/home/jmaggard/Videos)
#   + "P" for images (eg. P,/home/jmaggard/Pictures)
#   + "PV" for pictures and video (eg. AV,/home/jmaggard/digital_camera)
#
# * if you want to scan multiple directories, you have to separate
#   the directories with a vertical bar:
#
#   eq. A,/home/jmaggard/Music | V,/home/jmaggard/Videos | P,/home/jmaggard/Pictures
#
# Note: the default is /mnt
media_dir="A,/srv/nfs/music | V,/srv/nfs/videos | P,/srv/nfs/pictures"
```

1. Open up the firewall port so the TV can access it. The default port is TCP 8200.

```bash
test@server:~> sudo firewall-cmd --add-port=8200/tcp --zone=internal
test@server:~> sudo firewall-cmd --add-port=8200/tcp --zone=internal --permanent
```

Note: We need to issue both commands if we want to commit the rule and make it permanent. If we forget to do the later one, the rule will vanish upon a reboot of the server.

1. Enable the service.

```bash
test@server:~> sudo systemctl enable --now minidlna
```

1. Assuming that your TV is in the same firewall zone then you should be able to browse the multimedia contents on your TV. The default friendly name of the server is "MiniDLNA", that is what you need to look for unless you renamed it in advance.

## A TV without Ethernet connectivity

I have an about 8 year old FullHD television. Although it has some limited "Smart" functionality, it is limited to storage devices which can be plugged in via USB. It was acceptable a couple of years ago as I had an old 60GB SATA HDD that I used to transfer media contents from my PC to the hard drive. The problems I had with this solution were that the TV was choosy regarding the media formats it was willing to replay so I had to transcode some formats, and replaying pictures was not really user friendly. I knew I do not want to replace the TV as it works well. So I replaced the media player device to a Raspberry Pi 2 and installed OpenELEC then later LibreELEC on it. This is what I still use these days.

### These are the reasons why I found it better than having DLNA.

- A small computer which has hardware acceleration support for most popular file formats.

- It consumes very little power so it can be literally turned on 24/7.

- The device comes with excellent software support, both regarding the hardware and the media center functionality. It basically runs on Linux.

- The media center software is capable to replay almost any media formats on the world.<sup>1 </sup>

- I does have maintained support for Youtube and many others.

- The media player and the device itself can be controlled via the TV's remote thanks to HDMI-CEC support

- Network connectivity (cable Ethernet)

- Very little maintenance cost. The media software can be even auto-updated.

The reason why I said this costs money is that you need to buy the device, the power supply, a case and a small SD card (2GB is more than enough). This is not exceptional, because other solutions may also involve investing into a hardware.

The installation of LibreELEC is already documented on the [project's wiki](https://wiki.libreelec.tv/installing_libreelec_on_the_rpi), I will not duplicate that content here.

<sup>1</sup> I still have issues with 3gp formats as it causes LibreELEC to completely freeze. But I accept it as the files are more than a decade old and I could transcode them if needed into a supported format.

## Verdict

Both solutions are still used in my family. Setting them up requires some technical skills, but operating them is just a matter of using the TV's remote. I like that I have the power to use the software I like. I even plan to upgrade LibreELEC from OSMC to have more control over the device and for example collect the logs of the system in a central log server. I do not regret the choice of going with the Raspberry Pi and not using some Android device.
