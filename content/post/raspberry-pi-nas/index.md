+++
draft = "false"
author = "Arjdroid"
title = "Setting up a NAS on my Raspberry Pi"
date = "2020-10-23"
description = "This blog post details the steps I took in setting up a N.A.S on my Raspberry Pi."
tags = [
    "raspberrypi",
    "nas",
    "networking",
    "storage",
]
categories = [
    "raspberrypi",
    "nas",
    "networking",
    "storage",
]
series = ["RaspberryPi"]
aliases = ["Setting-up-a-pi-nas"]
image = "raspberrypi4nas.jpg"
+++

> **Disclaimer**: If you plan on using this article as a guide for setting up your OWN raspberry pi nas (which I hope you do 😁), **PLEASE** read the entire article before going through with this process (It will help clear any doubts you have as well as ensure that you can go through with this process). Also, do remember that **I AM NOT RESPONSIBLE** for any damage that YOU may cause to yourself or your own hardware including, but not limited to: bricked devices, dead SD cards, thermonuclear war, or you getting fired because the alarm app failed. YOU are choosing to make these modifications, and if you point the finger at me for messing up your hardware, I will laugh at you. All jokes aside though, this process is pretty solid, and shouldn't break anything unless you *seriously* mess it up.

## Introduction

So, this project all started when my old faithful Windows PC started giving up so I had to temporarily work from my Macbook.
It was fine for the most part except that I didn't have access to most of the files on my 2 Terrabyte hard drive, that was a real problem because those files would've saved me a lot of time...

So, my father and I decided to setup a N.A.S!

Easy right? Not really...

There are a lot of options in terms of N.A.S storage devices, you could buy one of those pre-configured N.A.S boxes with their propreitary software and hardware which, if they break, you're out of luck, they're also often times really expensive!
So, what better alternative to closed sources and proprietary hardware than to use open source!
Enter: Raspberry Pi; a tiny credit card sized fully functioning computer that is great for D.I.Y projects.
So it was decided that the Raspberry Pi4 would be the brains of the operation, N.A.S operation, at least at this small scale, isn't a really compute heavy process so we decreed that a 2GB Model of the Raspberry Pi4 would be sufficient for the task.

> ### Prerequisites:
> * A working Raspberry Pi (Any Version), with all of it's required hardware like microSD card, USB power brick, etc.
> * Some sort of mass storage device (Like an HDD or SSD or USB Flash Drive, etc.)
> * A working brain

## Raspberry Pi Setup

So, the first thing we had to do was configure the Raspberry Pi4, it was pretty simple, we had a 16GB microSD card and we flashed Raspberry Pi OS onto it (that's what they call Raspbian now) and we attached it to a TV with a keyboard and mouse as well as an Ethernet connection to our router and configured Raspberry Pi OS with all the usual stuff;

```bash
sudo raspi-config
```

And viola! A pseudo command-line text-based G.U.I appears which allows you to configure most of the Raspberry Pi's settings with arrow keys, a spacebar and an enter key. I'm skipping the configuration part;

> ### If you don't know how to configure your Raspberry Pi:
>
> I highly recommend that you check out **[this](https://projects.raspberrypi.org/en/projects/raspberry-pi-setting-up)** article by the raspberrypi foundation about setting up your Pi, it will be greatly helpful!

## Hard Drive Configuration


Now, all we have got to do is connect to it via SSH; (**THIS IS ONLY IF YOU DO NOT HAVE A KEYBOARD AND MOUSE AND MONITOR CONNECTED TO THE PI, IF YOU DO HAVE THEM CONNECTED, SKIP THE SSH STEP, YOU CAN PERFORM ALL YOUR COMMANDS ON THE TERMINAL APP IN THE PI**)

On your own laptop/desktop terminal:

```bash
ssh username@ipa.ddr.ess.opi
```

Remember to replace `username` with your pi's username, most of the time it'll be `pi` and replace `ipa.ddr.ess.opi` with the ip address of your pi, it'll look something like this, `192.168.1.180`, to be sure of what it is, you can access

And enter your credentials!

Now, we have to connect our hard drive/ssd/usbflashdrive/etc. to the Raspberry Pi, in our case, it was a 100GB hard drive which was connected to the Raspberry Pi with a SATA Adapter and 2 USB 3.0 cables.

After that, we must check whether the Pi actually detected the Hard Drive or not. For this you must have privileged access to your Raspberry Pi;

```bash
sudo fdisk -l
```

The result should be something like this:
```bash
Disk /dev/sda: 93.2 GiB, 107374127424 bytes, 123513245 sectors
Disk model: HDD100G
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x29035alr

Device    Boot Start           End    Sectors   Size  Id  Type
/dev/sda1       2048  107374127424  123513245  93.2G   9  HPFS/NTFS/exFAT
```

You would see many drives and each will have information in a format like this, the way you would identify your external hard drive would be by the size, so there would be many disks with different names like `/dev/sda` and `/dev/sbd` and things like that but next to the name, there is the size, `93.2 GiB` or something similar, your drive may be 100Gigabytes but the size shown in the terminal will be smaller like 93.2Gigabytes but it should be close enough for you to identify it.

So suppose you had connected a 1TB Hard Drive, it may say the size is something like 932 Gigabytes so be on the lookout for that.

Under the disk information, it would give `Device` information, each `Device` is a volume or partition on the drive, each partition is named with the name of the disk suffixed with the partition number if it is a logical primary partition.

In the sample partition I gave was `/dev/sda1`, it is the only partition in the drive and as you can see, it's `End` byte is the last byte in the whole drive so it's size encompasses the entire drive.

This is the volume that you will be mounting using as the 'drive'.

### Incase Your Drive Is Un-Initialised (Doesn't Have Any Partitions / Volumes) OR You Want To Start Off Fresh
> Run the partition editor for that drive;
>
> ```bash
> sudo fdisk -l /dev/sda
> ```
>
> Then, press 'd' to delete every partition in the drive, just in case, **YOU WILL LOSE YOUR DATA IF THERE WAS ANY PRESENT**.
>
> Then, press 'n' to create a new partition, you can just keep pressing enter to all the questions, that will make you enter the default configuration details for the partition.
>
> This will create a single partition that encompasses the entire drive.
>
> This partition will have the default linux ext4 filesystem, if you wish to use NTFS, you should know how to do that, don't worry though, ext4 is still readable to Windows!
>
> If you want to create more than one partition, you should know how to do that.
>
> Now check whether you've actually gotten a new volume on the drive.
>
> ```bash
> sudo fdisk -l
> ```
>
> And be on the lookout for a result similar to the one previously mentioned;
> ```
> Device    Boot Start           End    Sectors   Size  Id  Type
> /dev/sda1       2048  107374127424  123513245  93.2G   9  HPFS/NTFS/exFAT
> ```

## Mounting The Volume


Now that you know the name of the volume that you want to mount, in this case `/dev/sda1`, you're going to want to mount it!

In Raspberry Pi OS, and most other distros I've used, there is a directory in the root of the drive known as `/mnt`, this directory can be used as the parent directory under which external drives can be mounted.

Before we mount our drive volume, we must create a folder for it to use as a mount point. To do this we must first change directories into the `/mnt` directory;

```bash
cd /mnt
```

Then, we must create a new directory to use as a mount point, you can name it anything but in this case, it's HDD100G;

```bash
mkdir HDD100G
```

Now, we are ready to mount the volume onto our new mount point!

```bash
sudo mount /dev/sda1 /mnt/HDD100G
```

It is a good idea to recheck the name of the volume before mounting it;

```bash
sudo fdisk -l
```

Great! Now you've mounted your drive!

Incase you need to unmount the drive (if you're disconnecting the cable, PLEASE unmount before doing so.)

```bash
sudo umount /dev/sda1
```

## Setting Up The SMBD (samba share) Server


An SMBD (samba share) server allows file sharing because it appears as a network drive which can be accessed by any device on the network that is permitted to access it.

We first have to install the smbd server but before doing that, we must update our repositories;

```bash
sudo apt update
```

You could also run;

```bash
sudo apt upgrade
```

But running that might be quite timetaking and ideally you should've done that when you were first setting up the Raspberry Pi

Now that our repositories are up to date, we can install the samba program;

```bash
sudo apt install samba samba-common
```

If it prompts you for anything, just say 'Yes'

Now you should edit `smb.conf`, the file that contains the samaba server configuration;

```bash
sudo nano /etc/samba/smb.conf
```

This will open up the nano text editor which will allow you to modify the `smb.conf` file. Now, you want to go to the end of the file you can do this either with `CTRL + END` or you can just keep spamming the down arrow key if that isn't possible.

At the bottom of the `smb.conf` file, you should enter the details as such;

```bash
[HDD100G]
path = /mnt/HDD100G
writeable = yes
create mask = 0777
directory mask = 0777
public = yes
```

Remember to replace `HDD100G` with the names that you have used. The writeable toggle is pretty self-explanatory, the create and directory mask values are values that allow all smb users to read, modify and delete the files on that drive, if you wish to set a different set of values for the masks, a quick [insert your preffered search engine]() search should help you out, although you should know that these masks should work perfectly fine for you. The public toggle means that in your LAN, the network location will be visible under the Pi's hostname, in this case, raspberrypi.

Now, we need to add a password for the current smb share user, in this case, pi. This username and password will allow you to access the smb share files. To add a password to the current user, pi do this;

```bash
sudo smbpasswd -a pi
```

You can now enter the password you want to enter, this password doesn't have to be the same password as your user password for accessing the system.

You can only add an smbpasswd for a user that is already existing in the system. If you want to add a new user in the system apart from pi, do this;

```bash
sudo adduser user
```

Replace user with a username of your choice.

Once you've done `sudo adduser user`, you can also add that user to the smb server;

```bash
sudo smbpasswd -a user
```

And repeat the same process that you did to add the pi user to the smb share server.

Once you're finished with all your samba configuration, you need to restart the samba service;

```bash
sudo systemctl restart smbd
```

Now your samba share server should be ready!

To access the drive from a Windows system, you must go to the File Explorer and in the address bar, you can enter `\\\raspberrypi\HDD100G` replace `raspberrypi` with the hostname of your raspberry pi OR the IP address of the Raspberry Pi and `HDD100G` with the name entered as the header for your smb.conf entry.

To access the drive from a Unix system (MacOS, Linux, BSD, etc.), in the file explorer, in the address bar, you can go to `smb://raspberrypi/HDD100G` replace `raspberrypi` with the hostname of your raspberry pi OR the IP address of the Raspberry Pi and `HDD100G` with the name entered as the header for your smb.conf entry.

And that's it! That's how I configured my own Raspberry Pi samba share server.

> Thank you for reading!

---
