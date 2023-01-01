When utilized correctly, Arch is a fairly minimal-bloat install, meaning that resource consumption on your system will be similarly minimal. Personally, my old Thinkpad X230 can comfortably run Arch using less than 1 GB of RAM. This softens the hardware limitations that are required. Nevertheless, the Arch Linux Wiki highlights the following basic requirements for installation and use of the OS:

"Arch Linux should run on any x86_64-compatible machine with a minimum of 512 MiB RAM, though more memory is needed to boot the live system for installation.[1] A basic installation should take less than 2 GiB of disk space. As the installation process needs to retrieve packages from a remote repository, this guide assumes a working internet connection is available."

On the subject, the Arch Linux Wiki is one of the most comprehensive repos for documentation, best practices, and relevant troubleshooting as a whole, let alone for (Arch) Linux. The contributors spend countless hours to ensure the information is up-to-date and complete. You can visit it at https://wiki.archlinux.org/

Proceeding with this guide, I will assume you already have a LiveUSB set up with the Arch Linux installation ISO. If not, this can easily be set up using a tool like UNetbootin.

You should see a blank terminal on the screen.

## Step 1 - Connect to internet

**`root@archiso ~ # iwctl`**  

This will bring up the IWD utility. We now want to list the wireless interfaces available to us.

**`[iwd]# device list`**  

For this example, we will use "wlan0". Let's now scan for networks to connect to.

**`[iwd]# station wlan0 scan`**  
**`[iwd]# station wlan0 get-networks`**

We should now have a list of SSID's to connect to. Let's use an example SSID "MyWiFi".

**`[iwd]# station wlan0 connect MyWiFi`**

You will now be prompted to enter the password for the SSID in question; once this is done, verify the connection. 

**`[iwd]# station wlan0 show`**  
**`[iwd]# exit`**  
**`root@archiso ~ # ping google.com`**  

If all is well, you will see constant ping requests sent out. Ctrl+C to end the ping. 0% packet loss is optimal. 


## Step 2 - Set clock on system

**`root@archiso ~ # timedatectl set-ntp true`**


## Step 3 - Delete/Create/Format partitions

Let's start by listing off the storage devices at our disposal. 

**`root@archiso ~ # fdisk -l`**  

Select the disk you want to write the Arch install to. In this instance, we'll use /dev/sda as the example. NOTE: if your disk does not have a recognized partition table, or is using MBR, let's set up GPT.

**`root@archiso ~ # gdisk /dev/sda`**  

Type in "w" + Enter to write, and "y" + Enter to confirm.

**`fdisk /dev/sda`**  

If you have existing partitions, delete them all using the "d" command. We will want to create new partitions for our install. You will be prompted to enter a command:

**`Command (m for help):`** n

We have 3 basic partitions to make: EFI filesystem, SWAP, and root. The rules for such are as follows: 512 MB for the EFI filesystem, SWAP should be equal to the amount of RAM you have, and the rest will be allocated to root. Give these partitions numbers, and be sure to remember which is which. In this instance, we'll have the EFI, SWAP, and root be partitions 1, 2, and 3, respectively. 

**`Partition number (1-128, default 1):`** 1  
**`First sector (##-#####, default 2048):`** 2048  
**`Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-#######, default #######):`** +512M  
*`Created a new partition 1 of type 'Linux filesystem' and size of 512 MiB`*  
**`Command (m for help):`** n  
**`Partition number (2-128, default 2):`** 2  
**`First sector (##-#####, default ##########):`** \[just hit enter]  

The instance I'm using in this example has 4 GB of RAM; therefore, I will allocate the same for my SWAP.

**`Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-#######, default #######):`** +4G  
*`Created a new partition 2 of type 'Linux filesystem' and size of 4 GiB`*  
**`Command (m for help):`** n  
**`Partition number (3-128, default 3):`** 3  
**`First sector (##-#####, default ##########):`** \[just hit enter]  

Since we are allocating the rest of the memory to the root partition, just hit enter to automatically assign the last available value to the last sector:

**`Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-#######, default #######):`** \[just hit enter]  
*`Created a new partition 3 of type 'Linux filesystem' and size of 27.5 GiB`*  

The instance I'm using has a disk size of 32 GB; 4 GB went to the swap, and 512 MB went to the EFI partition, leaving us with the 27.5 GB that you see in the output. Next, let's change these partitions to the appropriate type. 

**`Command (m for help):`** t  
**`Partition number (1-3, default 3):`** 1

This next portion will have you input a numerical value to define the partition type. Type in "L" and hit enter to view a list of all the available types. You'll see that the values for the EFI filesystem and SWAP are 1 and 19, respectively. We will leave the root partition as a Linux filesystem type. This is the default type that was assigned, so we will not need to change the partition type for Partition 3, in this case. 

**`Partition type or alias (type L to list all):`** 1  
**`Changed type of partition 'Linux filesystem' to 'EFI System'.`**  
**`Command (m for help):`** t  
**`Partition number (1-3, default 3):`** 2  
**`Partition type or alias (type L to list all):`** 19  
**`Changed type of partition 'Linux filesystem' to 'Linux swap'.`**  

Now let's verify that all our partitions are looking correct.

**`Command (m for help):`** p  

You will see a table with your three partitions. Pay attention to the types, and the associated sizes. In this example, EFI should be 512M, swap should be 4G, and Linux filesystem should be 27.5G. If all looks good, write the changes to the disk. 

**`Command (m for help):`** w  
**`root@archiso ~ # fdisk -l`**  

Running fdisk -l this time should show your new partitions under the appropriate disk. In my instance, since I was altering the partitions on /dev/sda, my partitions are /dev/sda1, /dev/sda2, and /dev/sda3 for EFI, swap, and root, respectively. 

We will now want to format the EFI partition to FAT32:

**`root@archiso ~ # mkfs.fat -F32 /dev/sda1`**  
*`mkfs.fat 4.2 (2021-01-31)`*

Next, we will format the swap partition as such, and enable it:

**`root@archiso ~ # mkswap /dev/sda2`**  
*`Setting up swapspace version 1, size = 4 GiB (4294963200 bytes)`*  
*`no label, UUID= [unique identifier for partition]`*  
**`root@archiso ~ # swapon /dev/sda2`**  

Finally, we will format the root partition to EXT4:

**`root@archiso ~ # mkfs.ext4 /dev/sda3`**  

## Step 4 - Install Arch

We will now be installing the Linux kernel. First, we must mount our root partition to a mount point, and then run the kernel install: 

**`root@archiso ~ # mount /dev/sda3 /mnt`**  
**`root@archiso ~ # pacstrap /mnt base linux linux-firmware vim`**

This part may take some time. I also threw Vim in there, so I can have a functional text editor up and running once the install is complete. 


### Troubleshooting

There are a number of reasons pacstrap can fail. Some useful tips:

* Ensure that you set your system time correctly. As a refresher, **`root@archiso ~ # timedatectl set-ntp true`**  
* Update your mirrors to ensure you are downloading the most accurate, up-to-date packages: **`root@archiso ~ # reflector --verbose --latest 10 --country "United States" --protocol https --sort rate --save /etc/pacman.d/mirrorlist`**  
* Packages are signed with PGP keys to ensure authenticity, and those keys may be out of date. Run the following:  
  * **`root@archiso ~ # dirmngr </dev/null`**  
  * **`root@archiso ~ # pacman-key --populate archlinux`**  
  * **`root@archiso ~ # pacman-key --refresh-keys`**  


### Step 5 - Configure installation

Your installation currently resides at the mount point. Navigate to that:

**`root@archiso ~ # arch-chroot /mnt`**  

Next, we want to configure the timezone, sync the hardware clock, and set the locale on the installation.

**`[root@archiso /]# timedatectl list-timezones`**  
**`[root@archiso /]# timedatectl set-timezone <TIMEZONE>`**  
**`[root@archiso /]# hwclock --systohc`**  
**`[root@archiso /]# vim /etc/locale.gen`**  

Remove the "#" in front of "en_US.UTF-8 UTF-8"

**`[root@archiso /]# locale-gen`**  
**`[root@archiso /]# echo LANG=en_US.UTF-8 > /etc/locale.conf`**  
**`[root@archiso /]# export LANG=en_US.UTF-8`**  

Now let's name our host:

**`[root@archiso /]# vim /etc/hostname`**  

Insert the hostname into the text file; I used "arch" for this example. Next, we'll create a hosts file:

**`[root@archiso /]# vim /etc/hosts`**  

Add the following lines to the end of the file (I used my hostname in the third line, replace "arch" with your specified hostname from earlier):
* 127.0.0.1      localhost
* ::1      localhost
* 127.0.1.1      arch.localdomain arch

Next, we'll set the root password, and create a primary user (I'll use "raj" as my username):

**`[root@archiso /]# passwd`**  
**`[root@archiso /]# useradd -g users -G power,storage,wheel -m raj`**  
**`[root@archiso /]# passwd raj`**  

Next, we'll install our bootloader, which will be GRUB in this instance:

**`[root@archiso /]# pacman -S grub efibootmgr`**  
**`[root@archiso /]# mkdir /boot/efi`**  
**`[root@archiso /]# genfstab -U /mnt >> /mnt/etc/fstab`**

Mount the EFI filesystem from earlier to /boot/:

**`[root@archiso /]# mount /dev/sda1 /boot/efi`**  

Set up GRUB: 

**`[root@archiso /]# grub-install --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi`**  
**`[root@archiso /]# grub-mkconfig -o /boot/grub/grub.cfg`**  

Now let's install/enable our network drivers and other goodies:

**`[root@archiso /]# pacman -S networkmanager network-manager-applet dialog wpa_supplicant base-devel linux-headers`**  
**`[root@archiso /]# systemctl enable NetworkManager`**  

Finally, let's set up our desktop environment:

**`[root@archiso /]# pacman -S xorg`**  
**`[root@archiso /]# pacman -S sddm`**  
**`[root@archiso /]# systemctl enable sddm`**  
**`[root@archiso /]# pacman -S plasma konsole dolphin screenfetch`**  

In this next step, remove the hashes before "%wheel ALL=(ALL) ALL"

**`[root@archiso /]# EDITOR=vim visudo`**  

The base installation is complete. Exit, unmount, and reboot:

**`[root@archiso /]# exit`**  
**`root@archiso ~ # umount -a`**  
**`root@archiso ~ # shutdown now`**  
