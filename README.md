# How to create an Alpine Linux VM with a Golang compiler

Recipes and instructions on how to build an Alpine Linux VM with Golang compiler support.

## Set up the virtual machine

In order to install a new Alpine Linux VM, you need to:

1. go to the Alpine Linux download site and download the latest Alpine Linux ISO image; under the "VIRTUAL" label there is one that is specifically optimised to be run as a guest VM (for example, see [here](http://dl-cdn.alpinelinux.org/alpine/v3.9/releases/x86_64/alpine-virt-3.9.0-x86_64.iso))
2. run Oracle VirtualBox, create a new VM and make sure that is has:
   1. some RAM (not very mus: actually a bare VM only seems to take as much as 64 MB)
   2. a 8 GB dynamically-allocated hard disk
   3. the Alpine Linux ISO image mounted into the CD-ROM
   4. NAT networking with port-forwarding to grant both:
      1. access to the internet from the guest, which is necessary to download additional software packages once the VM is up and running, and 
      2. access to the SSH server from the host: the single port forwarding rule to add via the GUI, which can be named "SSH", must allow TCP connections from "empty" host's (literally leave the field empty in the GUI) port 2222 to "empty" guest on port 22; 
   this will make sure that when you run `ssh -p 2222 root@localhost` on the host you'll end up logging into the guest
3. fire the VM and, once the live distribution has started up, log in as `root` (no password) and at the prompt run `setup-alpine`, then follow the instructions; the only things worth being noted are:
   1. for Italy, choose `it`, then again `it` as keyboard layout and sub-layout; then choose `CET` for the timezone
   2. choose `sda` as the drive to install to, and `sys` (meaning persistent, full system) as the install type.

During the install process you will be requested to change `root`'s password; any password (including `password`) will do, even if it's too weak.

Once the machine is up and running, go the VirtualBox GUI and remove the ISO image from the CD-ROM, then on VirtualBox's GUI, menu **Machine**, send an `ACPI Shutdown` to halt it.

Take a snapshot before you start messing around with it.

In order to restart the machine from the persistent disk, run `reboot` at the guest's prompt.

## Enable `root` access via SSH

In order to enable SSH access to the `root` user:

1. edit the `/etc/ssh/sshd_config` file as `root`
2. add a line in the `Authentication` section of the file that says `PermitRootLogin yes`
3. restart the sshd service:

```bash
$> /etc/init.d/sshd restart
```
 and then check its status with

 ```bash
 $> rc-status
 ```

 Then on the host you can connect to the guest by running

 ```bash
 $> ssh -p 2222 root@localhost
 ```

## Install guest additions

This step is required to create a Hashicorp Vagrant base image; for details, see [here](https://www.vagrantup.com/docs/virtualbox/boxes.html).

In order to setup VirtualBox Guest Additions, you must:

TODO: check how to installl VBoxGuestAdditions from APK repo.

1. log on to the guest and run the following script:

```bash
#!/bin/sh
wget -O VBoxGuestAdditions.iso https://download.virtualbox.org/virtualbox/6.0.4/VBoxGuestAdditions_6.0.4.iso
mkdir /media/VBoxGuestAdditions
mount -t iso9660 -o ro,loop VBoxGuestAdditions.iso /media/VBoxGuestAdditions
sh /media/VBoxGuestAdditions/VBoxLinuxAdditions.run
rm VBoxGuestAdditions.iso
umount /media/VBoxGuestAdditions
rmdir /media/VBoxGuestAdditions
```

