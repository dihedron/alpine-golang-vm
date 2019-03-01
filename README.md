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

During the install process you will be requested to change `root`'s password; any password (including `password`) will do, but in order for the machine to be a valid Vagrant Base Box, `root`'s password must be `vagrant`.

Once the machine is up and running, go the VirtualBox GUI and remove the ISO image from the CD-ROM, then on VirtualBox's GUI, menu `Machine`, send an `ACPI Shutdown` to halt it, or at the command prompt type

```bash
$> poweroff
```

Take a snapshot before you start installing stuff, just in case...

`reboot` restarts the machine; `poweroff` shuts it down by sending an ACPI signal, `halt` stops the CPU (without powering down).

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

## Install VirtualBox guest additions

This step is required to create a Hashicorp Vagrant base image; for details, see [here](https://www.vagrantup.com/docs/virtualbox/boxes.html).

In order to setup VirtualBox Guest Additions, you must edit the Alpine Linux's package manager's configuration file, `/etc/apk/repositories`, to include the `community` packages. To do so, simply edit the file and make sure that the `community` is uncommented:

```
  #/media/cdrom/apks
  http://mirror1.hs-esslingen.de/pub/Mirrors/alpine/v3.9/main
- #http://mirror1.hs-esslingen.de/pub/Mirrors/alpine/v3.9/community
+ http://mirror1.hs-esslingen.de/pub/Mirrors/alpine/v3.9/community
  #http://mirror1.hs-esslingen.de/pub/Mirrors/alpine/edge/main
  #http://mirror1.hs-esslingen.de/pub/Mirrors/alpine/edge/community
  #http://mirror1.hs-esslingen.de/pub/Mirrors/alpine/edge/testing
```

then update the packages and proceed to installation:

```bash
$> apk update
$> apk add virtualbox-guest-additions virtualbox-guest-modules-virt
```

then reboot the system:

```bash
$> reboot
```

an when it starts up again, install the module:

```bash
$> modprobe -a vboxsf
```

Now you can open the `Devices > Shared Folders > Shared olders Settings` menu on the VirtualBox GUI and define one or more shared folders; suppose you want to mount your host's `/path/to/my/data` folder as the guest's `/mnt/outside`, you select the host's path (`/path/to/my/data`), give a symbolic name to the shared folder (e.g. `Data`), and specify that you want it mounted as `permanent`, `auto-mount`ed at boot in `read/write` mode, then provide the `/mnt/outside` path as the mountpoint.

Then at the guest's CLI you type:

```bash
$> mount -t vboxsf <Shared Folder Name> <Guest Mountpoint>
```

e.g.

```bash
$> mount -t vboxsf Data /mnt/outside
```

and the host's `Data` folder, which is `path/to/my/data`, will be available in read/write mode under the guest's `mnt/outside`.

That's it!

## Fulfill the other Vagrant Base Box requirements

In order for the machine to be a valid Vagrant Base Box (see [here](https://www.vagrantup.com/docs/boxes/base.html) for details), it needs to:

1. install package `sudo` and enable the `wheel` group (the sudoers)

```bash
$> apk install sudo
$> sed -e 's/# %wheel ALL=(ALL) NOPASSWD: ALL/%wheel ALL=(ALL) NOPASSWD: ALL/g' -i /etc/sudoers
```

2. have a valid `vagrant` user, with `vagrant` password

```bash
$> adduser -S vagrant -s /bin/ash -G wheel
$> passwd vagrant
Type password: *******
Re-type password: *******
```

2. have a set of insecure pre-defined keys for automatic logon of the `vagrant` user, which can be downloaded from [here](https://raw.githubusercontent.com/hashicorp/vagrant/master/keys/vagrant.pub); the public key should be downloaded to the Alpine machine and copied into `vagrant`'s home directory:

```bash
$> mkdir -p /home/vagrant/.ssh/
$> cat /mn/outside/vagrant.pub >> /home/vagrant/.ssh/authorized_keys
$> chmod 700 /home/vagrant/.ssh
$> chmod 600 /home/vagrant/.ssh/authorized_keys
```

## Package the Alpine Linux Base Box

In order to package the VM as a Vagrant Base Box, run

```bash
$> vagrant package --base <my-virtual-machine>
```

where `<my-virtual-machine>` is the name of the VirtualBox image.

## Install the golang toolchain

TODO