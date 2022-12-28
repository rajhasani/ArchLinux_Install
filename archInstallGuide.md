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
**`root@archiso ~ # pacstrap /mnt base linux linux-firmware vim

This part will take some time. I also threw Vim in there, so I can have a functional text editor up and running once the install is complete. 
