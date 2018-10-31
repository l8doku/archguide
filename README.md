Content-Type: text/x-zim-wiki
Wiki-Format: zim 0.4
Creation-Date: 2018-10-30T18:01:20+03:00

# InstallingArch
Created вторник 30 Октябрь 2018

Скачать арч
Постаивть арч на флешку

## Гайд 

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
``# ping archlinux.org``
If no connection is available, stop the dhcpcd service with 'systemctl stop dhcpcd@interface` where the `interface` name can be tab-completed. Proceed to configure the network as described in **Network configuration**.

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

`'
# mkdir /mnt/boot
# mount /dev/sda2 /mnt/boot
`'


**В моём случае:**

`'
# mkdir /mnt/home
# mount /dev/sda2 /mnt/home
`'


**genfstab** will later detect mounted file systems and swap space. 

