Content-Type: text/x-zim-wiki
Wiki-Format: zim 0.4
Creation-Date: 2018-10-30T18:01:20+03:00

# InstallingArch
Created вторник 30 Октябрь 2018

Скачать арч
Постаивть арч на флешку

## Pre-installation

### Поставить раскладку клавиатуры 

список раскладок:

`# ls /usr/share/kbd/keymaps/**/*.map.gz`

**По умолчанию стоит английская, мне дальше с ней норм**

### Verify the boot mode 
If UEFI mode is enabled on an UEFI motherboard, Archiso will boot Arch Linux accordingly via systemd-boot. To verify this, list the efivars directory:
`# ls /sys/firmware/efi/efivars`
If the directory does not exist, the system may be booted in BIOS or CSM mode. Refer to your motherboard's manual for details. 

**У меня UEFI, эта папка должна быть**

### Connect to the Internet 
The installation image enables the **dhcpcd** daemon for wired network devices on boot. The connection may be verified with ping:
`# ping archlinux.org`
If no connection is available, stop the dhcpcd service with 'systemctl stop dhcpcd@interface where the `interface` name can be tab-completed. Proceed to configure the network as described in **Network configuration**.

**Попробовать подсоединиться проводом в первую очередь**

### Update the system clock 
Use timedatectl(1) to ensure the system clock is accurate:
`# timedatectl set-ntp true`
To check the service status, use `timedatectl status`. 

**Ничего дополнительного. Если сеть работает, должно работать**

### Partition the disks 
When recognized by the live system, disks are assigned to a block device such as `/dev/sda` or `/dev/nvme0n1`. To identify these devices, use `lsblk` or `fdisk`.
`# fdisk -l`
Results ending in **rom**, **loop** or **airoot** may be ignored.
The following partitions are **required** for a chosen device:
	One partition for the root directory /.
	If UEFI is enabled, an EFI system partition.

Note: Swap space can be set on a separate partition or a swap file.
**Я поставлю swap файлом**

To modify partition tables, use **fdisk** or **parted**.
`# fdisk /dev/sda`

2 варианта:
Всё в /, либо отдельно home.
Я хочу отдельно home
/ на 50гб, остальное home.

`# fdisk -l /dev/sda`
**- список разделов. Проверить, что тут всё ок**
/dev/sda - моё устройство, наверное имя такое же будет

Можно это сделать переустановкой манджаро или прямо из винды сейчас, наверное. Я сделал через винду

### Format the partitions 

Once the partitions have been created, each must be formatted with an appropriate file system. For example, to format the root partition on /dev/sda1 with ext4, run:
`# mkfs.ext4 /dev/sda1`

### Mount the file systems 

Mount the file system on the root partition to /mnt, for example:

`# mount /dev/sda1 /mnt`

Create mount points for any remaining partitions and mount them accordingly:

```
# mkdir /mnt/boot
# mount /dev/sda2 /mnt/boot
```


**В моём случае:**

```
# mkdir /mnt/home
# mount /dev/sda2 /mnt/home
```


**genfstab** will later detect mounted file systems and swap space. 


## Installation
### Select the mirrors

Packages to be installed must be downloaded from mirror servers, which are defined in `/etc/pacman.d/mirrorlist`. On the live system, all mirrors are enabled, and sorted by their synchronization status and speed at the time the installation image was created.

The higher a mirror is placed in the list, the more priority it is given when downloading a package. You may want to edit the file accordingly, and move the geographically closest mirrors to the top of the list, although other criteria should be taken into account.

This file will later be copied to the new system by pacstrap, so it is worth getting right.

### Install the base packages

Use the `pacstrap` script to install the _base_ package group:

`# pacstrap /mnt base`

This group does not include all tools from the live installation, such as `btrfs-progs` or specific wireless firmware

To install packages and other groups such as `base-devel`, append the names to pacstrap (space separated) or to individual pacman commands after the **#Chroot** step. 

## Configure the system
### Fstab

Generate an fstab file (use -U or -L to define by UUID or labels, respectively):

`# genfstab -U /mnt >> /mnt/etc/fstab`

Check the resulting file in /mnt/etc/fstab afterwards, and edit it in case of errors.

### Chroot

Change root into the new system:

`# arch-chroot /mnt`

### Time zone

Set the time zone:

`# ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime`

Run hwclock(8) to generate /etc/adjtime:

`# hwclock --systohc`

This command assumes the hardware clock is set to UTC. See System time#Time standard for details.

### Localization

Uncomment en_US.UTF-8 UTF-8 and other needed locales in /etc/locale.gen, and generate them with:

`# locale-gen`

Set the LANG variable in locale.conf(5) accordingly, for example:

```
/etc/locale.conf

LANG=en_US.UTF-8
```

If you set the keyboard layout, make the changes persistent in vconsole.conf(5):

```
/etc/vconsole.conf

KEYMAP=de-latin1
```

### Network configuration

Create the hostname file:

```
/etc/hostname

myhostname
```

Add matching entries to hosts(5):

```
/etc/hosts

127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

If the system has a permanent IP address, it should be used instead of 127.0.1.1.

Complete the network configuration for the newly installed environment.

### Initramfs

Creating a new initramfs is usually not required, because mkinitcpio was run on installation of the linux package with pacstrap.

For special configurations, modify the mkinitcpio.conf(5) file and recreate the initramfs image:

`# mkinitcpio -p linux`

### Root password

Set the root password:

`# passwd`

### Boot loader

A Linux-capable boot loader must be installed in order to boot Arch Linux. See Category:Boot loaders for available choices.

If you have an Intel or AMD CPU, enable microcode updates. 

Install the `refind-efi` package.

Install `efibootmgr`


```
pacman -S refind-efi efibootmgr
```

After installing rEFInd's files to the ESP, verify that rEFInd has created `refind_linux.conf` containing the required kernel parameters (e.g. root=) in the same directory as your kernel. If it has not created this file, you will need to set up #Passing kernel parameters manually or you will most likely get a kernel panic on your next boot. 

By default, rEFInd will scan all of your drives (that it has drivers for) and add a boot entry for each EFI bootloader it finds, which should include your kernel (since Arch enables EFISTUB by default). So you may have a bootable system at this point. 

## Reboot


Exit the chroot environment by typing `exit` or pressing `Ctrl+D`.

Optionally manually unmount all the partitions with `umount -R /mnt`: this allows noticing any "busy" partitions, and finding the cause with fuser(1).

Finally, restart the machine by typing `reboot`: any partitions still mounted will be automatically unmounted by systemd. Remember to remove the installation media and then login into the new system with the root account. 

## Post installation

### Creating a user

```
# useradd --create-home l8doku
# passwd l8doku
```

### Installing a graphical environment

```
pacman -S xorg-server 
```

I will worry about NVIDIA drivers later

#### Plasma

Before installing Plasma, make sure you have a working Xorg installation on your system.

```
# pacman -S plasma
```

To enable support for Wayland in Plasma, also install the plasma-wayland-session package. 
