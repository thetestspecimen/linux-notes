# Introduction

This guide provides information on how to setup an Arch Linux installation from scratch with the following features:

1. Encrypted root, swapfile and boot (only the EFI partition is unencrypted)
2. Btrfs file system and subvolumes setup
3. Encrytped swapfile (rather than swap partition), including the necessary settings for hibernation
4. Gnome desktop installation
5. NVIDIA proprietary driver setup
7. Wayland/Gnome(gdm) configuration for NVIDIA
8. Snapper snapshot installation and configuration, including btrfs-assistant GUI

To put this guide together I have referenced various blog posts, and of course the [Arch Wiki](https://wiki.archlinux.org/). Specific sources are detailed in the references section at the end.

This guide should produce a complete and functional system, with GUI and snapshot capability.

If you have any suggestions for improvements to any of the methods used in this guide, then please feel free to put in a bug report, or pull request. 

I am aware that preferred / recommended methods change regularly, so I have tried to stick closely to the Arch Wiki where possible.

**Note: this guide at some point may become outdated, the [arch wiki](https://wiki.archlinux.org/) should always be considered the main and trusted source for any information.**

# Pre-requisites
It is assumed that you have already downloaded Arch Linux, burned it to a USB drive, and booted into the live environment.

If you are now sitting looking at the Arch live environment commandline then you are ready to go, otherwise please get setup first.

You may also need buckets of patience...best of luck!

# Set the console keyboard layout and font
The default console keymap is "US". Available layouts can be listed with: 
```bash
localectl list-keymaps
```
To set the keyboard layout, pass the keyboard layout name to loadkeys. For example, to set a UK keyboard layout:
```bash
loadkeys uk
```
Console fonts are located in ```/usr/share/kbd/consolefonts/``` and can be set with ```setfont```, omitting the path and file extension. For example, to use one of the largest fonts suitable for HiDPI screens, run: 
```bash
setfont ter-132b
```

# Check if the system is UEFI

If the following command returns 64, then the system is booted in UEFI mode, and has a 64-bit x64 UEFI. If the command returns 32, then system is booted in UEFI mode and has a 32-bit IA32 UEFI. Either of which is fine to proceed with this guide, as we will be using GRUB.

If the file doesn't exist then the system isn't UEFI, so you won't be able to proceed.

```bash
cat /sys/firmware/efi/fw_platform_size
```

# Connect to Wifi

If your device is plugged in via Ethernet cable then you should be good to go. Otherwise, connect to a Wi-Fi network using ```iwctl```:

```bash
# Enter the iwctl interface
iwctl
# Find the name of your wireless device:
device list
# Scan for networks:
station <device name> scan
# List network SSID:
station <device name> get-networks
# Connect to network:
station <device-name> connect <SSID>
```
Leave iwctl by pressing **Ctrl+C**.

Test the connection:
```bash
ping archlinux.org
```
If the ping responds, then stop the process using **Ctrl+C**.

# Update the system clock

Enable and start network time synchronisation:
```bash
# check current settings
timedatectl
# list timezones (change "Europe/" as appropriate)
timedatectl list-timezones | grep "Europe/"
# or if you want to list everything omit the 'grep'
timedatectl list-timezones
# set timezone (change "Europe/London" to your timezone)
timedatectl set-timezone "Europe/London"
# turn on ntp
timedatectl set-ntp true
# check the settings have updated
timedatectl
```

# Prepare the drive

Disks are assigned to a block device such as ```/dev/sda```, ```dev/nvme0n1``` or ```/dev/mmcblk0```. Let's list out the devices:
```bash
fdisk -l
```

## Wipe the disk

It is a good idea to wipe the disk before proceeding any further. 

Create a container called "wipe_me".

**Note:** the "block-device" should be the main device not one of the device partitions. For example, it could be ```/dev/sdf``` , but not ```/dev/sdf1``` , or ```/dev/nvme0n1``` but not ```/dev/nvmen0n1p1```.

**NOTE: YOU ARE ABOUT TO WIPE THE DISK! BE SURE YOU DON'T NEED THE DATA ON THE DISK AS IT IS NOT RECOVERABLE!**
```bash
cryptsetup open --type plain -d /dev/urandom /dev/<block-device> wipe_me
```
Zero out the container. This may take a while depending on the size and type of drive:
```bash
dd bs=1M if=/dev/zero of=/dev/mapper/wipe_me status=progress
```
Then close the container:
```bash
cryptsetup close wipe_me
```

## Partition the disk

With the drive erased, use fdisk to partition the disk. 

Using fdisk is an interactive process. Start the interactive process by telling fdisk the drive to setup.

**Note:** until you give the write command 'w' (write table to disk and exit), nothing will be changed on the disk. So if you make a mistake, just type 'q' (quit without saving changes), and start again.

```bash
# As per the pervious section the "block-device" should be the main device not one of the device partitions.
fdisk /dev/<block-device> 
```
This is now the interactive part of the process with fdisk. 

Enter 'm' to see the available commands:
```bash
m 
```
Output:
```text
  GPT
   M   enter protective/hybrid MBR

  Generic
   d   delete a partition
   F   list free unpartitioned space
   l   list known partition types
   n   add a new partition
   p   print the partition table
   t   change a partition type
   v   verify the partition table
   i   print information about a partition

  Misc
   m   print this menu
   x   extra functionality (experts only)

  Script
   I   load disk layout from sfdisk script file
   O   dump disk layout to sfdisk script file

  Save & Exit
   w   write table to disk and exit
   q   quit without saving changes

  Create a new label
   g   create a new empty GPT partition table
   G   create a new empty SGI (IRIX) partition table
   o   create a new empty MBR (DOS) partition table
   s   create a new empty Sun partition table
```
### Start the partitioning

Two partitions will be created:

1. The EFI partition - this will be ONLY the EFI, and will not contain boot.
2. The root partition, which will contain everything else, including boot and swap (in our case a swapfile), and will be encrypted.

```bash
# CREATE A NEW PARTITION TABLE
g # create a new empty GPT partition table

# CREATE THE EFI PARTITION
n # add a new partition
*enter* # Partition number (1-128, default 1)
*enter* # First sector (2048-250069646, default 2048)
+512M # Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-250069646, default 250068991)

# Created a new partition 1 of type 'Linux filesystem' and of size 512 MiB.

t # change the partition type, as we need an EFI partition, not 'Linux filesystem'
uefi # 'uefi' is an alias for '1', so you can use either '1' or 'uefi' here

# Changed type of partition 'Linux filesystem' to 'EFI System'.

# CREATE THE ROOT PARTITION
n # add a new partition
*enter* # Partition number (2-128, default 2)
*enter* # First sector (1050624-250069646, default 3147776)
*enter* # Last sector, +/-sectors or +/-size{K,M,G,T,P} (1050624-250069646, default 250068991) 

# Created a new partition 2 of type 'Linux filesystem' and of size 118.7 GiB.

# PRINT THE PARTITION TABLE
p # print the partition table

# Disk /dev/sdf: 119.24 GiB, 128035676160 bytes, 250069680 sectors
# Disk model: SSD 850 PRO 128G
# Units: sectors of 1 * 512 = 512 bytes
# Sector size (logical/physical): 512 bytes / 512 bytes
# I/O size (minimum/optimal): 512 bytes / 33553920 bytes
# Disklabel type: gpt
# Disk identifier: 63A0B976-C21F-47B8-B9FB-DD16BA279098

# Device       Start       End   Sectors   Size Type
# /dev/sdf1     2048   1050623   1048576   512M EFI System
# /dev/sdf2  1050624 250068991 249018368 118.7G Linux filesystem

# WRITE THE TABLE TO DISK
w # write table to disk and exit
```

**Note:** From now on I will use the partition names above as examples ( i.e.  ```/dev/sdf1``` and ```/dev/sdf2```) , but they need to be changed as appropriate for your actual setup. For example with an nvme device the partition names will likely be in the format ```/dev/nvme0n1p1``` and ```/dev/nvme0n1p2```. 

Use whatever is in the "Device" column when you printed the partition table above.

# Create the filesystems

There are two partitions, and each will have it's own filesystem. The root partition will use btrfs, but the EFI partition cannot use btrfs, and so will be set as FAT.

## Create the EFI filesystems

```bash
mkfs.vfat -F 32 /dev/sdf1
```

## Encrypt and open the root partition

**You must use luks1 here** - Currently, the latest grub does support opening a luks2 partition, **but** it does not support the argon2id encryption algorithm yet. So to have an encrypted boot requires luks1 for the moment.

**Note:** I have upped the iteration parameter to 5000 from the default 3000. This will result in the unlocking of encrypted partition taking a little longer at boot (nothing excessive, but it depends on hardware). If this is a problem (i.e. you have a slow processor), please change the 5000 in the command below back to 3000. I would not recommend going any lower than the default of 3000.

```bash
# You will be asked to create a password. Make sure it is a strong password
cryptsetup --type luks1 -c aes-xts-plain64 -h sha512 -i 5000 -s 512 luksFormat /dev/sdf2
# Now open the new encrypted partition. You will be prompted to enter the password you created in the previous step.
cryptsetup luksOpen /dev/sdf2 cryptroot
```

## Create the root filesystem
```bash
mkfs.btrfs -L archlinux /dev/mapper/cryptroot
```

## Mount the root device
```bash
mount /dev/mapper/cryptroot /mnt
```

# Configuring btrfs
As btrfs is being used, then subvolumes should be created.

## Create btrfs subvolumes

Various subvolumes will be created, this is mainly to help with snapshots later in the process. 

When taking a snapshot of root, the other subvolumes will not be included in the snapshot. (This is useful, for example, if you want to access logs after restoring a previous snapshot, as @log stops the logs from being rolled back along with root).

If you don't want so many, then you must at least keep the first four below (@cache, @log and @tmp are not strictly necessary).
```bash
btrfs subvolume create /mnt/@
btrfs subvolume create /mnt/@swap
btrfs subvolume create /mnt/@home
btrfs subvolume create /mnt/@snapshots
btrfs subvolume create /mnt/@cache
btrfs subvolume create /mnt/@log
btrfs subvolume create /mnt/@tmp
```

## Unmount the root partition

```bash
umount /mnt
```

## Set the options for subvolume mounting

```bash
# set subvolume options for main btrfs subvolumes
export opts="ssd,noatime,compress=zstd:1,space_cache=v2,discard=async"
```
Options:
* **ssd** - specification of the allocation scheme suitable for SSD drives (rather than rotational drives).
* **noatime** - significantly improves read intensive workload performance, and also reduces writes in some circumstances.
* **compress=zstd:1** - zstd level 1 compression is optimal for NVME devices, while gaining some compression. If compression is increased too much with rapid NVME drives, the compression can become a bottleneck. Omit the ```:1``` to use the default compression level of 3. zstd accepts a value range of 1-15, with higher levels trading speed and memory for higher compression ratios.
* **space_cache=v2** - creates cache in memory for greatly improved performance. Be sure to use v2 and not v1.
* **discard=async** - keeps the disk tidy by enabling the discarding of freed file blocks. Specifically, the asynchronous mode (async) gathers extents in larger chunks before sending them to the devices for TRIM.

Please take a look at the [official documentation](https://btrfs.readthedocs.io/en/latest/ch-mount-options.html) for further details.

## Mount the root BTRFS subvolume

```bash
mount -o ${opts},subvol=@ /dev/mapper/cryptroot /mnt
```

## Create mountpoints for other BTRFS subvolumes
Currently the root partition doesn't contain any subfolders to mount the other partitions to, so they need to be created.

```bash
mkdir -p /mnt/{swap,home,.snapshots,var/cache,var/log,var/tmp}
```

## Mount the other subvolumes (except swap)
Mount the previously created btrfs subvolumes to the folders created in the previous section.

```bash
mount -o ${opts},subvol=@home /dev/mapper/cryptroot /mnt/home
mount -o ${opts},subvol=@snapshots /dev/mapper/cryptroot /mnt/.snapshots
mount -o ${opts},subvol=@cache /dev/mapper/cryptroot /mnt/var/cache
mount -o ${opts},subvol=@log /dev/mapper/cryptroot /mnt/var/log
mount -o ${opts},subvol=@tmp /dev/mapper/cryptroot /mnt/var/tmp
```

## Mount swap
The swap subvolume needs different mounting options as it cannot use COW (Copy-On-Write), and hence cannot use compression:

```bash
mount -o noatime,ssd,subvol=@swap /dev/mapper/cryptroot /mnt/swap
```

# Setup swap
The two lines below are sufficient to create a swapfile with all the necessary correct settings (such as nocow). See the [official guidance](https://btrfs.readthedocs.io/en/latest/Swapfile.html) for more details.

**Note:** If you want to use hybernation you must assign a swap file that is at least as large as your RAM.

```bash
# create swapfile
btrfs filesystem mkswapfile --size 62g --uuid clear /mnt/swap/swapfile
# activate swapfile
swapon /mnt/swap/swapfile
```

# Mount EFI and boot

The EFI is mounted on the first partition, which is formated with FAT. This guide uses ```/efi``` rather than the quite common ```/boot/efi```. This follows the recommendation of the [Arch wiki](https://wiki.archlinux.org/title/EFI_system_partition#Typical_mount_points), and also makes sense since in this instance the UEFI info is on a completely separate partition to boot. 

The boot directory just needs creating, not mounting, as the root directory is already mounted, which is where boot will reside.
```bash
mkdir /mnt/efi
mkdir /mnt/boot
mount /dev/sdf1 /mnt/efi
```

# Syncronise the Package Database

```bash
pacman -Syy
```

# Update the Mirror List

Below, swap out the countries for those of your choice.
```bash
reflector --verbose --protocol https --latest 5 --sort rate --country 'Sweden,Monaco,Switzerland,Germany' --save /etc/pacman.d/mirrorlist
```

# Install the Arch Base System
Add to, or change, the listed programs as you wish. However, this is a good starting point.

Be sure that if you remove anything you know what you are doing, as some packages are required later on in the install process. 

For example, you may wish to switch out ```intel-ucode``` for ```amd-ucode``` if you have an AMD CPU. Also ```neovim``` could be swapped out for ```nano``` , or removed completely. However, ```vim``` is required later for using ```visudo```, so please leave it in place.

You can install other packages later, so you don't need to go crazy here.

**Note:**  an ```lts``` kernel will be installed as an option later, so there is no need to do it here.

```bash
pacstrap /mnt base base-devel linux linux-headers linux-firmware intel-ucode btrfs-progs grub efibootmgr vim neovim networkmanager gvfs exfatprogs dosfstools e2fsprogs man-db man-pages texinfo openssh git reflector wget cryptsetup wpa_supplicant terminus-font    
```

# Generate fstab

This will autogenerate the fstab based on the subvolumes that have been created so far.
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```
Check result with:
```bash
cat /mnt/etc/fstab
```

## Remove subvolid
As per the [Arch wiki](https://wiki.archlinux.org/title/Btrfs#Mounting_subvolumes) for btrfs:

"One can mimic traditional file system partitions by creating various subvolumes under the top level of the file system and then mounting them at the appropriate mount points. It is preferable to mount using ```subvol=/path/to/subvolume```, rather than the ```subvolid```, as the ```subvolid``` may change when restoring #Snapshots, requiring a change of mount configuration."

Therefore if you want to use something like [timeshift](https://github.com/linuxmint/timeshift) or [snapper](https://github.com/openSUSE/snapper) later, then you should make this adjustment.

```bash
# completely remove the "subvolid" entries from fstab
vim  /mnt/etc/fstab
```

# Enter the new system
We will now 'chroot' into the newely installed system.
```bash
arch-chroot /mnt /bin/bash
```

# Basic Settings

Now we are operating in the new install, so let's start by setting some basics.

## Set system clock
Change the 'Europe/London' as appropriate for your timezone.
```bash
ln -s /usr/share/zoneinfo/Europe/London /etc/localtime
hwclock --systohc
```

## Set hostname
Change 'your_hostname' to your actual hostname. (This can be anything you want, it is essentially what you want your computer to be named.)
```bash
echo your_hostname > /etc/hostname
```

## Set hosts
Again, change 'your_hostname' to be the same as the hostname you set in the previous step.
```bash
cat > /etc/hosts <<EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   your_hostname.localdomain your_hostname
EOF
```
## Set Locale

Remember to change the relevant parts to your specific locale.

### Option 1

```bash
export locale="en_GB.UTF-8"
sed -i "s/^#\(${locale}\)/\1/" /etc/locale.gen
echo "LANG=${locale}" > /etc/locale.conf
```

### Option 2

Alternatively, go in and edit the files directly.

In ```/etc/locale.gen``` uncomment  ```en_GB.UTF-8 UTF-8```: 
```bash
en_GB.UTF-8 UTF-8
```
Add the following line to ```/etc/locale.conf```:
```bash
LANG=en_GB.UTF-8
```

### Generate the locales
The locales you just created need to be generated. To do this run the below:
```bash
locale-gen
```

## Set keyboard layout and terminal font

```bash
echo "KEYMAP=uk" >> /etc/vconsole.conf
echo "FONT=ter-v28n" >> /etc/vconsole.conf 
```

Other font sizes if 28 is too small or large for you:

ter-v20n
ter-v24n
ter-v32n

(also change the 'n' to 'b' if you want 'bold' rather than 'normal')

## Set default editor
Swap in your favourite editor here (e.g. ```nano``` or ```vim``` rather than ```nvim```)
```bash
echo "EDITOR=nvim" >> /etc/environment
echo "VISUAL=nvim" >> /etc/environment
```

# Set accounts and passwords
Some accounts and associated passwords should now be created

## Set the root password
The root account will already exist, but it doesn't have a password yet.
```bash
passwd
```

## Create a user and user password
This will be your user account, so swap in your username where it says ```user_name``` and set your account password.

The below will also add your user to the ```wheel``` group, which will allow the user to use sudo commands in conjunction with their password.
```bash
# add user
useradd -m -G wheel -s /bin/bash user_name
# set password
passwd user_name
```

## Activate wheel
It is recommended to never adust ```/etc/sudoers``` directly, as any mistakes can permanently bork your system. ```visudo``` is a command designed specifically to edit this file, and has checks in place to ensure that any mistakes are caught.

```bash
# issuing this command will open /etc/sudoers with vim
visudo
```
Now uncomment this line:
```bash
## Uncomment to allow members of group wheel to execute any command
# %wheel ALL=(ALL:ALL) ALL
```
so it becomes:
```bash
## Uncomment to allow members of group wheel to execute any command
%wheel ALL=(ALL:ALL) ALL
```

# Create the crypto keyfile
This section will create an encrypted keyfile. The reason this is necessary is to remove the requirement to supply the encryption unlock password twice at boot. 

With the keyfile, the password is supplied once, and the system will be completely decrypted.
```bash
cd /
dd bs=512 count=4 if=/dev/random of=crypto_keyfile.bin iflag=fullblock
chmod 000 /crypto_keyfile.bin
chmod 600 /boot/initramfs-linux*
cryptsetup luksAddKey /dev/sdf2 /crypto_keyfile.bin
# enter your encryption password when prompted
cryptsetup luksDump /dev/sdf2  
# You should now see that LUKS Key Slots 0 and 1 are both occupied
```

# Configure mkinitcpio
[mkinitcpio](https://wiki.archlinux.org/title/Mkinitcpio) is a script that creates the initial ramdisk. It basically sets up various kernel modules, and performs initialisation steps.

Open the conf file:
```bash
nvim /etc/mkinitcpio.conf
```
Edit the following:
```bash
# load the keyfile we created
FILES=(/crypto_keyfile.bin)

# Remove "kms" from hooks if you use a NVIDIA card so that you can use proprietary drivers later
# Add "resume" if you want to use hibernation later
# Add "encrypt" before "filesystem"
# the below is my suggested ordering, but it could be adjusted if you know what you are doing
HOOKS=(base udev keyboard autodetect keymap modconf microcode consolefont block encrypt resume filesystems fsck)

# Add "crc32c-intel" if you have an intel CPU which supports SSE4.2 otherwise use "crc32c"
MODULES=(btrfs crc32c-intel)
```
Recompile initcpio:
```bash
mkinitcpio -p linux
```
# Get UUID of encrypted partition
```bash
blkid -s UUID -o value /dev/sdf2
# output:
# 180901b5-151a-45e3-ba87-28f02b124666
```
# Get UUID of root
```bash
blkid -s UUID -o value /dev/mapper/cryptroot
# output:
# aba2d556-2891-4f79-a78e-4f9a6e439b02
```
# Get swapfile offset for resume
This [must be done](https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate#Acquire_swap_file_offset) as per the below for btrfs (using the 'filefrag' method will be inaccurate):
```bash
btrfs inspect-internal map-swapfile -r /swap/swapfile
# output:
# 533760
```

Open the grub setting file:
```bash
nvim /etc/default/grub
```

Add the following to what already exists. Note that the UUID for the encrypted partition is used for the "cryptdevice" and the UUID for root is used for "resume".

If you don't need hibernation you can remove "resume" and "resume_offset"
```bash
GRUB_CMDLINE_LINUX_DEFAULT="cryptdevice=UUID=180901b5-151a-45e3-ba87-28f02b124666:cryptroot resume=UUID=aba2d556-2891-4f79-a78e-4f9a6e439b02 resume_offset=533760"
GRUB_PRELOAD_MODULES="luks"
# This exists, but needs turning on (i.e. remove the initial #)
GRUB_ENABLE_CRYPTODISK=y
```
Install grub in ESP:
**Note:** the ```--bootloader-id``` can be whatever you want. This is the name that will be shown in bios/grub so you can identify the install.
```bash
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=Arch
```
Verify that a GRUB entry has been added to the UEFI bootloader by running ...
```bash
efibootmgr
```
Generate the GRUB configuration file ...
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```
Verify that grub.cfg has entries for ```insmod cryptodisk``` and ```insmod luks``` by running:
```bash
grep 'cryptodisk\|luks' /boot/grub/grub.cfg
```

# Exit and reboot
```bash
exit
swapoff -a
umount -R /mnt
reboot
```
After reboot you should end up at the arch commandline login screen (fingers crossed!).
# After reboot checks

```bash
# Failed systemd services
sudo systemctl --failed

# High priority errors in the systemd journal
# There may be items here, it doesn't mean the install failed
# For example I had broadcom bluetooth fails, which is normal for my system
sudo journalctl -p 3 -xb

# Check swap is active (various methods):
free -m
swapon --show
cat /proc/swaps
```

# Connect to wifi (if required)
```bash
# Start NetworkManager
sudo systemctl enable NetworkManager.service
sudo systemctl start NetworkManager.service

# Connect to WIFI
nmcli device wifi list
nmcli device wifi connect SSID_or_BSSID password your_password

# Check WIFI Connection
nmcli connection show
```

# Pacman config (optional)
```bash
sudo nvim /etc/pacman.conf
```
Uncomment (or add) the following if you wish. "Color" will provide color in the terminal, and "ILoveCandy" will make the loading bar into a "pacman" like animation.
```bash
# Misc options
Color
ILoveCandy
```
Uncomment the following to allow parallel downloads:
```bash
ParallelDownloads = 5
```
You can change the number 5 to whatever you like depending on your connection speed.

# Update system
```bash
sudo pacman -Syyu
```

# Install LTS kernal (recommended, but optional)
Install the Long-Term Support (LTS) Linux kernel as a fallback option to Arch's default kernel.

As arch is generally at the bleeding edge, it can mean that there is more chance of conflicts arising with a specific new kernel. One way to mitigate this is to have an ```lts``` kernel to fall back to. Hence, the recommendation.

**Note:** this will not replace the 'normal' kernel, you will have a choice between 'normal' and 'lts' at boot.
```bash
sudo pacman -S linux-lts linux-lts-headers
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
Reboot and select LTS kernel to test.

After reboot using the LTS kernel, confirm that the running kernel is indeed ```lts```
```bash
uname -r
```

## Change the kernel menu order in grub

You may find that the LTS kernel is now the first in the boot order. This will result in the LTS kernel being automatically loaded every time you boot, rather than the normal kernel, unless you intervene at the boot menu.

This is inconvenient and can get quite annoying. There are basically two options to fix this.

My recommndation is to do the following:

###  Option 1: Change the code that generates the list

As [per this discussion thread](https://bbs.archlinux.org/viewtopic.php?id=266583) , do the following:
```bash
# open the following file for editing
sudo nvim /etc/grub.d/10_linux

# Change:
reverse_sorted_list=$(echo $list | tr ' ' '\n' | sed -e 's/\.old$/ 1/; / 1$/! s/$/ 2/' | version_sort -r | sed -e 's/ 1$/.old/; s/ 2$//')
# To:
reverse_sorted_list=$(echo $list | tr ' ' '\n' | sed -e 's/\.old$/ 1/; / 1$/! s/$/ 2/' | version_sort -V | sed -e 's/ 1$/.old/; s/ 2$//')
```
Essentially, change in the above the following part:
```bash
version_sort -r
# -r, --reverse
# reverse the result of comparisons
```
to:
```bash
version_sort -V
# -V, --version-sort
# natural sort of (version) numbers within text
```
Then regenerate grub:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

### Option 2: Change the default menu entry [Alternative]

As an alternative to the above, this is the official version as described in the [Arch wiki](https://wiki.archlinux.org/title/GRUB/Tips_and_tricks#Changing_the_default_menu_entry):
```bash
sudo nvim /etc/default/grub
```
The following change will auto pick the correct menu options for you on boot (i.e. the second menu item 'Advanced options for Arch Linux', and then the third menu item 'Arch Linux, with Linux linux').

The menus are zero indexed, so 0=first menu item, 1=second menu item etc. Hence, 1>2.
```bash
# Change:
GRUB_DEFAULT=0
# To:
GRUB_DEFAULT="1>2"
```

I'm not fond of this solution as it feels a bit 'hacky', but it is up to you.

# Install pipewire for sound
```bash
sudo pacman -S pipewire pipewire-alsa pipewire-pulse pipewire-jack wireplumber alsa-utils
```
Then reboot.
 
# Install yay for access to the AUR
```bash
git clone https://aur.archlinux.org/yay-git.git
cd yay-git
makepkg -si
```

# Install GNOME

I'm aware that a desktop environment is a personal thing, so you can of course install whatever you want here if you don't like GNOME. However, just bear in mind that I have not tested this guide with any other desktop environment, so if you hit problems you will need to get creative!
```bash
sudo pacman -S gnome gnome-extra
sudo systemctl enable gdm.service
```

Then reboot.

**NOTE: on initial reboot I was met with a black screen immediately after providing my user password at the Gnome login screen, and was unable to proceed to the Gnome desktop. A simple reboot aleviated this. Whether you will experience the same I am not sure.**

After reboot you should have a fully funcitonal Gnome desktop environment using wayland. However, there are various other items that can be setup as well, which the next few sections will go through:

1. Setup a NVIDIA graphics card with proprietary drivers from NVIDIA
2. Disable root account for additional security
3. Setup snapper to allow snapshots to be taken (including GUI)

# NVIDIA Setup (Optional)

If you have a NVIDIA graphics card then this section is recommended. 

## Enable the multilib repository

Open pacman conf:
```bash
sudo nvim /etc/pacman.conf
```
Uncomment the following lines by removing the # character at the start of them
```bash
[multilib]
Include = /etc/pacman.d/mirrorlist
```
Update the package list, and install packages with yay:
```bash
yay -Syu
yay -S nvidia nvidia-lts nvidia-utils lib32-nvidia-utils
yay -S nvidia-settings
```

## Set enviromental variables
Force the GBM (Generic Buffer Management) backend.
```bash
sudo nvim /etc/environment
# Append
GBM_BACKEND=nvidia-drm
__GLX_VENDOR_LIBRARY_NAME=nvidia
```

## DRM Settings
Enable DRM (Direct Rendering Manager)

From the [Arch Wiki](https://wiki.archlinux.org/title/NVIDIA#DRM_kernel_mode_setting):
"Since NVIDIA does not support automatic KMS late loading, enabling DRM (Direct Rendering Manager) kernel mode setting is required to make Wayland compositors function properly." 

### Option 1 - Add parameters using modprobe (recommended method)

This is the preferred method, as per the [Arch Wiki](https://wiki.archlinux.org/title/NVIDIA#DRM_kernel_mode_setting):

"To enable DRM (Direct Rendering Manager), set the ```modeset=1``` kernel module parameter for the ```nvidia_drm``` module."

"Additionally, with driver version 545 and above, you can also set the experimental ```nvidia_drm.fbdev=1``` parameter, which is required to tell the NVIDIA driver to provide its own framebuffer device instead of relying on efifb or vesafb, which do not work under simpledrm."

**Note:** ```nvidia_drm.fbdev=1``` has known issues that are only possibly fixed in the driver version 550 and above.

```bash
echo -e 'options nvidia_drm modeset=1 fbdev=1' | sudo tee -a /etc/modprobe.d/nvidia.conf 
```

### Other parameters

There are also other (non-essential) parameters that can be set at the same time. There is a great rundown of potential parameter settings on the [Gentoo Wiki](https://wiki.gentoo.org/wiki/NVIDIA/nvidia-drivers), but I have selected a relevant few to apply here:

* **NVreg_UsePageAttributeTable --> Default:0** - This is one of the latest and newest additions to the NVIDIA driver modules option. It allows the driver to take full advantage of the PAT technology - a newer way of allocating memory, replacing the older Memory Type Range Register (MTRR) method. The PAT method creates a partition type table at a specific address mapped inside the register and utilizes the memory architecture and instruction set more efficiently and faster. If the computer supports PAT and the feature is enabled in the kernel then this flag can be enabled. Without PAT support, users may experience unstable performance and even crashes if this is enabled. So be careful with these options. 

* **NVreg_InitializeSystemMemoryAllocations --> Default:1** - Tell the NVIDIA driver to clear system memory allocations prior to using it for the GPUs. Disabling can give a slight performance boost but at the cost of increased security risks. By default the driver will wipe the allocated by zeroing out its content. 

* **NVreg_PreserveVideoMemoryAllocations --> Default:0** - By default the NVIDIA Linux drivers save and restore only essential video memory allocations on system suspend and resume. Quoting NVIDIA "The resulting loss of video memory contents is partially compensated for by the user-space NVIDIA drivers, and by some applications, but can lead to failures such as rendering corruption and application crashes upon exit from power management cycles.". The "still experimental" interface enables saving all video memory (given enough space on disk or RAM).  

If you want to apply these parameters, do the following:

```bash
echo -e 'options nvidia NVreg_UsePageAttributeTable=1 NVreg_InitializeSystemMemoryAllocations=0 NVreg_PreserveVideoMemoryAllocations=1' | sudo tee -a /etc/modprobe.d/nvidia.conf 
```

### Option 2 - update grub (alternate method - NOT recommended)

As an alternative to the previous modprobe method parameters can also be passed to the Linux kernel during its initial boot, through the GRUB bootloader.

Open the grub settngs file:
```bash
sudo nvim /etc/default/grub
```
Add the ```nvidia-drm.modeset=1```' to ```GRUB_CMDLINE_LINUX_DEFAULT```:
```bash
GRUB_CMDLINE_LINUX_DEFAULT="nvidia_drm.fbdev=1 nvidia_drm.modeset=1 loglevel=3 quiet..."
```
Regenerate the grub config:
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## Update mkinitcpio
```bash
sudo nvim /etc/mkinitcpio.conf
```
Append the following to what is already there:
```bash
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm)

# Add this if you used the recommended method (option 1) from the previous section
FILES=(/etc/modprobe.d/nvidia.conf)

# If you haven't already, please also remove "kms" from "HOOKS=()"
```
Removing ```kms```  from ```HOOKS=()``` ensures that the initramfs will avoid including the open-source “nouveau” driver, which may conflict with the proprietary NVIDIA drivers.

Recompile:
```bash
sudo mkinitcpio -P
```

## Preserve video memory after suspend

By default the NVIDIA Linux drivers save and restore only essential video memory allocations on system suspend and resume. Quoting NVIDIA:

"The resulting loss of video memory contents is partially compensated for by the user-space NVIDIA drivers, and by some applications, but can lead to failures such as rendering corruption and application crashes upon exit from power management cycles."

The "still experimental" interface enables saving all video memory (given enough space on disk or RAM).

```bash
sudo systemctl enable nvidia-suspend.service nvidia-hibernate.service nvidia-resume.service
```

## Add Pacman Hook
This will automatically update initramfs after a NVIDIA upgrade.
[Arch wiki source](https://wiki.archlinux.org/title/NVIDIA#pacman_hook)
```bash
sudo mkdir /etc/pacman.d/hooks
sudo nvim /etc/pacman.d/hooks/nvidia.hook
```
Add the following to the file:
```bash
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
Target=nvidia
Target=nvidia-lts
Target=linux
Target=linux-lts

[Action]
Description=Updating NVIDIA module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux*) exit 0; esac; done; /usr/bin/mkinitcpio -P'
```

## Force enable wayland

The udev rules contained in ```61-gdm.rules``` need to be turned off to force wayland to work with NVIDIA graphics.

You will likely find that ```61-gdm.rules``` does not exist in ```/etc/udev/rules.d/```, which is normal, and what you want.

The following command creates a "symbolic link" to "no information" (i.e. ```/dev/null```). This effectively overrides ```/usr/lib/udev/rules.d/61-gdm.rules``` (which does exist), as ```/etc/udev``` takes precedence over ```/usr/lib/```. 

**NOTE:** it is not a good idea to adjust the file in ```/usr/lib``` as it will be overwritten automatically on updates.

```bash
sudo ln -s /dev/null /etc/udev/rules.d/61-gdm.rules
```

## Reboot and check
```bash
# Regenerate initramfs:
sudo mkinitcpio -P
# Reboot
reboot
```
**NOTE: as with the reboot after installing Gnome, on initial reboot I was met with a black screen immediately after providing my user password at the Gnome login screen, and was unable to proceed to the Gnome desktop. A simple reboot aleviated this. Whether you will experience the same I am not sure.**

After reboot you can check if NVIDIA drm settings were applied with:
```bash
sudo cat /sys/module/nvidia_drm/parameters/modeset
```
It should return ```Y```

# Disable the root account (Optional)
As per the [Arch Wiki](https://wiki.archlinux.org/title/Sudo#Disable_root_login):
"Users may wish to disable the root login. Without root, attackers must first guess a user name configured as a sudoer as well as the user password." 
```bash
sudo passwd -l root
```
**Note:** be careful here, as you could lock yourself out! Make sure you added your user to the wheel group, and activated wheel using visudo (all of this was covered earlier in this install guide).

# Enable snapshots with snapper

There are basically two options when it comes to making snapshots with a Btrfs filesystem:

* Timeshift
* Snapper

I have used timeshift for many years, and it has saved my skin quite a few times. It just works. However, it is not as flexible / configurable as snapper, so in this instance snapper will be installed.

## Basic setup
This section comes across as a mess of deleting and recreating things, but unfortunately this is necessary because of the way the snapper 'create_config' currently works.
```bash
# install snapper
sudo pacman -S snapper snap-pac

# remove the original snapshots folder as the snapper script will want to create it's own
sudo umount /.snapshots
sudo rm -rf /.snapshots

# run config creation for the root filesystem
sudo snapper -c root create-config /

# list the btrfs subvolumes after config
sudo btrfs subvolume list /

# in the subvolumes list you will note that snapper has created ".snapshots", but also that our original "@snapshots" still exists. We don't need both.
# remove the subvolume created by snapper, so we can use "@snapshots"
sudo btrfs subvolume delete /.snapshots
# re-make the directory we deleted earlier and mount it
sudo mkdir /.snapshots
sudo mount -a
# update the permissions for the new folder
sudo chmod 750 /.snapshots
```
## Manual Snapshot
Snapper is essentially setup, so an initial snapshot can be taken here if you wish.
```bash
# example syntax for creating a manual snapshot
sudo snapper -c root create -d "First snapshot"
```
## Automatic Timeline Snapshots
Open the config for "root":
```bash
sudo nvim /etc/snapper/configs/root
```
Set the following:
```bash
# required
ALLOW_USERS="your_username"
ALLOW_GROUPS="wheel"
# the below can be changed as you please
TIMELINE_MIN_AGE="3600"
TIMELINE_LIMIT_HOURLY="5"
TIMELINE_LIMIT_DAILY="7"
TIMELINE_LIMIT_WEEKLY="2"
TIMELINE_LIMIT_MONTHLY="1"
TIMELINE_LIMIT_QUARTERLY="0"
TIMELINE_LIMIT_YEARLY="0"
```
Enable services:
```bash
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
```
## How to list configs and snapshots
```bash
snapper list-configs
snapper -c root list
```
## Skip indexing on .snapshots
It doesn't make sense to have indexing on the snapshots, so it may as well be disabled.
```bash
sudo pacman -S plocate
sudo nvim /etc/updatedb.conf
# add the following
PRUNENAMES = ".snapshots"
```
## Auto update Grub
This will allow the snapshots to appear, and be accessible from the GRUB menu on boot.

Install the grub-btrfs package:
```bash
sudo pacman -S grub-btrfs
```
If you have followed this guide then you don't need to change anything in the grub-btrfs config. However, if you have a different location for boot/grub then you may need to edit the following parameters in the config file:
```GRUB_BTRFS_GRUB_DIRNAME="/boot/grub"```
```GRUB_BTRFS_BOOT_DIRNAME="/boot"```

```bash
sudo nvim /etc/default/grub-btrfs/config
```
## Automatically update grub upon snapshot creation or deletion
```bash
sudo pacman -S inotify-tools
sudo systemctl enable grub-btrfsd
sudo systemctl start grub-btrfsd
```
## Read-only snapshots and overlayfs
Typically, if you boot into a snapshot it will be readonly. This can cause problems (crashing/freezing) as some parts of the root directory need write access to funciton properly. 

To get around this, 'overlayfs' can be used. Overlayfs allows any changes made to be temporarily saved in RAM. 

This effectively gets around the write access issue, so no crashing/freezing. However, because it doesn't change the snapshot at all (the changes are in RAM), it also retains the integrity of the snapshot. **Note:** all changes made are lost on reboot.
```bash
sudo nvim /etc/mkinitcpio.conf
# add 'grub-btrfs-overlayfs' as the last item in HOOKS
HOOKS=(base ... fsck grub-btrfs-overlayfs)
# Regenerate initramfs
sudo mkinitcpio -P
```
## Install btrfs-assistant (GUI)
btrfs-assistant provides an intuitive GUI interface for managing snapshots and settings for snapper. It makes restoring, creating and monitoring snapshots a breeze.

```bash
yay -S btrfs-assistant
```

# The End
There are many more things that could be added to this guide, but I think here is a good place to stop.

You should now have a fully working Arch Linux install with:

1. Encrypted root, swapfile and boot
2. A btrfs filesystem with compression and snapshot capabilities 
4. Gnome desktop using Wayland
5. NVIDIA proprietary drivers
8. Snapper snapshots including GUI
9. The license to say "I use Arch BTW!"

# References
Below are links to some of the general references and  blog post used to create this guide.
## Arch install guidance
These two were absolutely essential for this guide:
[The official Arch guide](https://wiki.archlinux.org/title/Installation_guide)
[HardenedArray -  Arch install guide](https://gist.github.com/HardenedArray/ee3041c04165926fca02deca675effe1) 
There are also plenty of other guides that helped fill in the blanks here and there, but be careful as things can be outdated. **If you are unsure, the Arch wiki should always be your main reference**:
[Daniel Wayne Armstrong - Arch install guide](https://www.dwarmstrong.org/archlinux-install/)
[dev.to (Tim Assavarat) - Arch install guide](https://dev.to/tassavarat/installing-arch-linux-with-btrfs-and-encryption-48na)
[Cody Hou - Arch install guide](https://www.codyhou.com/arch-encrypt-swap/)
[Zach Hilman - Arch install guide](https://wiki.archlinux.org/title/User:ZachHilman/Installation_-_Btrfs_%2B_LUKS2_%2B_Secure_Boot)
[Raven2cz - Arch install](https://github.com/raven2cz/geek-room/tree/main/arch-install-luks-btrfs)
## NVIDIA settings guidance
[Arch wiki - Kernel module settings](https://wiki.archlinux.org/title/Kernel_module#Setting_module_options)
[Gentoo wiki - NVIDIA driver guidance](https://wiki.gentoo.org/wiki/NVIDIA/nvidia-drivers)
[Korvahannu Github - driver installation guide](https://github.com/korvahannu/arch-nvidia-drivers-installation-guide)
[linuxiac.com - nvidia-wayland-arch](https://linuxiac.com/nvidia-with-wayland-on-arch-setup-guide/)
## Btrfs
Some good guidance and comparisons are made here to help with a decision on what is appropriate for you in terms of Btrfs compression:
[thelinuxcode - btrfs compression](https://thelinuxcode.com/enable-btrfs-filesystem-compression/)
Swapfile info:
[Btrfs official docs -  swapfile info](https://btrfs.readthedocs.io/en/latest/Swapfile.html)

## Snapper setup
[Daniel Wayne Armstrong - Snapper guide](https://www.dwarmstrong.org/btrfs-snapshots-rollbacks/)
[Lorenzo Bettini - Snapper guide](https://www.lorenzobettini.it/2023/03/snapper-and-grub-btrfs-in-arch-linux/)