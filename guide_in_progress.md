#
# DO NOT USE THIS AS A GUIDE, THIS IS AN IN PROGRESS PAGE ABOUT **PONTENTIALLY** USING NTFS3 DRIVER. VISIT THE WIKI OF THIS REPO INSTEAD!!
# CURRENTLY IT LACKS ACLS, THEREFORE ITS INAPPROPRIATE.
#


Welcome to the NTFS-ROOTFS wiki!
--------------------------------

In this wiki we will guide you through the steps required to have a fully functioning Linux installation on NTFS.

# Overview
* [FAQ](#faq)
* [Getting started](#getting-started)
  * [Pre-install setup](#preinstall-setup)
  * [ArchLinux](#archlinux)
* [Contribution](#contribution)
* [Thanks](#thanks)
# FAQ

## Why?

Because there are moments where people just don't like partitioning or the chance of ruining their preexisting Windows installation and like the convenience of installing a Linux distro in their existing NTFS partition.

## How?

You know that fuse exists and ZFS exists and that ZFS can be used as the rootfs. So since NTFS-3G runs on fuse, why couldn't NTFS be a root filesystem. A new *read-write* **kernel** NTFS driver is in development and it supports all the necessary POSIX-y bits to get a bootable Linux system.

### What about the bootloader?

GRUB4DOS ftw! And on EFI any bootloader since EFI partitions exist.

## Can I put my installation in a directory (like C:\Linux or /mnt/windows/Linux)?

Absolutely, just follow this guide.

## How stable?

Apart from not being able to shutdown the machine, it's pretty stable.

## Performance

Apart from mount times, the driver is pretty snappy.

## Do I have to disable Hibernation and Fast boot on Windows.

YES, DO IT NOW as you'd have trouble mounting the drive from within Linux.

## How do I verify if I have done it properly
 * It should boot.
 * You should be able to log in.
 * You should be able to modify files in your /home directory
 * You should test if sudo or SUID flags work.
 * You should check if chown, chmod..etc. work.
 * You should check if the changes are persistent across reboots/restarts.



# Getting started

First, you should check the compatibility list.

For now, currently these distros work:

 * [ArchLinux](#archlinux)
 * Or any Linux distro with DKMS and initram capabilities
 * OR the even more extreme route of statically including the kernel (please don't do this)

# Preinstall setup

So first, what will you do is create the ntfs partition or have your current Windows installation.

If you have a Windows installation I suggest you create a directory that will contain the Linux install like ```C:\Linux```.

Don't forget to disable Secure Boot, Fast boot and Hibernation. This will prevent you from Linux not being able to boot because the partition is unbootable.

Once you're done, you need find a compatible distro or try to install on your own and possibly contribute to this guide ;)

# ArchLinux

## Base system installation

Download ArchLinux ISO and boot into it. Once booted, please install ``ntfs-3g`` by running ``pacman -Sy ntfs-3g`` and letting it finish.

Once done, you have two ways of installing the system:

   1. The boring usual way. Format the drive as NTFS and just use the partition itself as a rootfs.
   2. Install Linux as a sub-directory on a existing NTFS filesystem.

This guide will teach you both ways just in case you're interested in any of the two.

### The boring way

#### Getting the NTFS3 driver

Even though ```ntfs-3g``` already exists in the Arch repo, it becomes unusable since it's issue is that 

#### Formatting and partition setup

Find an partition (pre-formatted or otherwise (format via ```mkfs -t ntfs```)) and mount it to /mnt WITH THE FOLLOWING FLAGS ```-o permissions,suid -t ntfs-3g```. This will use the usual FUSE ntfs-3g driver for now as the new ntfs3 driver is unavailable for now.

DO NOT FORGET THE EFI PARTITON IF ON UEFI. BE SURE THAT YOU'VE SET THIS UP BY THIS POINT.

#### Install the actual base system

Install the system via usual means (don't do the bootloader now, save that for later)

Once you're done ``pacstrap``-ing, use ``arch-chroot``, ``manjaro-chroot``, ``artix-chroot`` or WHATEVER to chroot into the newly installed system.

#### Setting up an admin user account

Set up the user account now, that includes:
   * Creating the user - ```useradd $username```
   * Giving the user a password - ```passwd $username```
   * Installing sudo and adding that user to the sudoers or wheel group (whichever you prefer) - ```usermod -aG wheel $username``` (Don't forget to edit /etc/sudoers accordingly)
   * Creating the home directory and fixing permissions - ```mkdir /home/$username && chown -R $username:$username /home/$username```

#### Installing the AUR

   * Install ```wget``` and ```base-devel``` packages - ```pacman -Sy wget base-devel```
   * ```su $username``` - This will drop you in the user's shell
   * Go to ```https://aur.archlinux.org``` on a separate device (or use ```elinks``` like a omegachad).
   * Search for an "AUR helper" or "AUR package manager" that you like - I personally prefer ```yay```
   * Click on it's AUR package page, then on the RIGHT side of the page you'll find a "Download snapshot" button. Copy it's URL.
   * Type that url into a wget command in this fashion: ```wget "$url"``` (```wget https://aur.archlinux.org/cgit/aur.git/snapshot/yay-bin.tar.gz``` for yay fxp.)
   * ```tar xvf $downloaded.file```
   * ```cd $my_preffered_aur_helper```
   * ```makepkg -sri```
   * Let it download packages, build the application then install the AUR helper itself

#### Installing NTFS3 and configuring the rest of the system

   * Once you're done, you'll have to install your preferred Linux kernel (any will do) AS WELL AS IT'S HEADERS
   * ```aurhelper -S ntfs3-dkms```
   * Let it build and install the new NTFS driver
   * Exit the user shell via ```exit```
   * Now with your favorite text editor open ```/etc/makepkg.conf``` and within ```MODULES=()``` add ntfs3 like so ```MODULES=(ntfs3)```
   * Rebuild the initramfs by reinstalling the kernel package (I don't want to explain to you how to figure out the ```mkinitcpio -p``` command)

Once that's done, there are only two last steps:
   1. Installing the bootloader
   2. Generating the fstab file

The first one is kinda complicated, but essentially do the same thing as you'd usually do when setting up the bootloader, but make sure that you include 

### The (sub-directory) fun way

#### Pre-cursor

First read/skim through the "boring way" to get familiar with the process.

#### Partitioning part amendments

Just like the "boring way" find or create a NTFS partition to use, keeping in mind the EFI partition if your system has one.

Install ```ntfs-3g``` just like the in the way above and mount the root filesystem with the options ```-t ntfs-3g -o permissions,suid```

But unlike the "boring way" you must now create a new mountpoint INSIDE of the partition, so lets say that you have a NTFS partition at ```/mnt```, you must now create a directory WITHIN that mountpoint (fxp. ```mkdir /mnt/linux```).

#### A new mountpoint

But this mountpoint you won't be ```pacstrap```-ing and ```chroot```-ing into. Instead you'll have to create a new mountpoint within the Arch ISO's system to act as a "real mountpoint" instead of just an directory.

You can achieve this via ```mkdir /new_mnt``` fxp.

Now in order to trick the system into thinking that there's a "real" partition in there instead of just a subdirectory, you have to ```mount -o bind``` the two locations together. For example, given our real NTFS partition is at ```/mnt``` and the root filesystem is at ```/mnt/linux``` and our new mountpoint at ```/new_mnt```, the command will look like this:

```mount -o bind,suid,permissions /mnt/linux /new_mnt```

#### Continuing with the "boring" guide and it's further amendments

Now follow the rest of the instructions from the "boring guide" and return here after installing the new NTFS driver (```ntfs3-dkms```)

From here also install ```mkinitcpio-dir``` using the AUR helper as well.

And when editing the ```/etc/mkinitcpio.conf``` file add ```dir``` to ```HOOKS``` in the same fashion as with the ```MODULES``` and regenerating the initramfs by reinstalling the kernel or similar methods.

Also, when you generate the fstab file, be ABSOLUTELY sure that you exclude the root filesystem (aka. "the ```/``` entry") from the file by just removing the line with that mountpoint entirely.

Now you can return to the usual bootloader installation as with the "boring" way, but make sure that you ALSO ADD ``dir=/$subdir_where_you_installed_archlinux`` (```dir=/linux``` in our case) to the Linux boot flags and regenerate the bootloader config accordingly. 

## Final notes

You will not be able to properly shutdown, reboot the system.

Instead, when shutting down you will have to after some time manually power off the system.

# Contribution

If you somehow fixed the issues that appeared during the install or made a guide on how to install it on a different distro, create a issue or a pull request.

# Thanks

As you may or may not have noticed I cloned [vadmium's](https://github.com/vadmium) [mkinitcpio-dir](https://github.com/vadmium/mkinitcpio-dir), it had some issues while building so that's why I cloned it. And he made the initcpio script, so huge thanks to him.
