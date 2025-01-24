# arch
For this project I am using a vm ( VirtualBox ).
Installed the iso and booted it in the vm and installed it in the vm.

I followed the archwiki(https://wiki.archlinux.org/title/Installation_guide) installation guide to install and setup. 

### Partitioning disks and disk encryption
Partitioning the disks and enabling disk encryption. Partitioning of disks can be done using <a href=https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/5/html/deployment_guide/ch-lvm#s1-lvm-intro-whatis>LVM</a> or the traditions method, I chose to partition using the LVMs.
For initial partitioning I followed the guide here https://wiki.archlinux.org/title/Installation_guide. <img src=https://github.com/rghdrizzle/arch/blob/main/Screenshot%202025-01-18%20192554.png>

The partition is done on the sda disk with 2 partitions, 1: for boot as EFI system type and 2: as Linux LVM type. [To change the type during the partition before writing, enter t]. <img src=https://github.com/rghdrizzle/arch/blob/main/Screenshot%202025-01-18%20192613.png>. 
After writing this partition I followed the LVM on LUKs guide in arch wiki [https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS] to encrypt the disks using cryptsetup. After encrypting LVM container <img src=https://github.com/rghdrizzle/arch/blob/main/Screenshot%202025-01-18%20193812.png>. 
After we create an encrypted disk we then open the disk to have a decrypted container of the disk, so now the LVM container is encrypted with a passphrase. Further more we create logical volumes by first creating a physical volume on top of the opened decrypted container, then we create a volume group to the decrypted container. Then we create all logical volumes in the volume group i.e swap <img src=https://github.com/rghdrizzle/arch/blob/main/Screenshot%202025-01-18%20194302.png>
,root <img src=https://github.com/rghdrizzle/arch/blob/main/Screenshot%202025-01-18%20194600.png>
and home ( You can create more volumes as you wish and how to do it is in the wiki ), for the swap volume size it depends on RAM size, since I allocated 4GB for the vm, I allocated 6GB as swap memory since the actual RAM size is small. After that we create a file system for those volumes and we used Ext4 for root and home volumes and mkswap for the swap volume. Now swap volume is a special allocated volume where if the device uses up all the RAM in the system and if there is a need to use more ram, the OS will use the swap volume as memory i.e using a part of the disk as memory. Of course using disk as memory isnt efficient but it is good to have it in cases where we need that extra memory. After formatting the file system we have to mount these volumes to a specific locatio  (/mnt). For swap instead of mounting we simply use swapon. Since we already created a boot partition initially we can directly format the filesystem intended for boot system using mkfs.fat -F32 since I used an UEFI system and then we mount this to /mnt/boot. 
Image after disk partition and encryption
<img src=https://github.com/rghdrizzle/arch/blob/main/Screenshot%202025-01-18%20203909.png>


### Installation of the kernel and configuring the system

I used pacstrap to install the linux base kernal into the system ```# pacstrap -K /mnt base linux linux-firmware```. After installing this, I generated a fstab file which is a file which defines how the disk partitions are mounted and this file is read by the systemd during the bootup so the system knows about the mounts. Next is an important part where we change the root to the new system we just installed by using ```# arch-chroot /mnt```. So now we actually entered into the root system ( remember how we mounted root to /mnt ? that is where the whole file system of the system and exists, so we move into the system instead of using the live installation shell). After changing the root we can install packages. Before that we set up the timezone and then generated a locale.conf file.
### Network configuration ,initramfs and boot loader

For network manager I am using systemd-networkd since I want this system to be systemd centric which makes things easier to manage and setup. Since arch comes with a systemd package it already contains networkd, all we got to do is to enable it by entering ```systemctl enable systemd-networkd.service ```. After this we also have to setup resolved which is a systemd service for dns resolution. This can be done in the same as we have enabled networkd. Now for configuring wired and wireless network you can refer to the <a href = https://wiki.archlinux.org/title/Systemd-networkd>guide</a>. Since I am using virtualbox as the device I can only setup a wired network connection, so I created a conf file for wired networks(refer to guide to know more on how to do that). Now if we have to setup a wireless connection, we create a configuration for it and also install iwd which is a wireless daemon and we can install it using pacman and after installation we have to enable the service using $*systemctl enable iwd.service* . After this configuring the network is complete and we can follow the installation guide.

Since I created Luks on the lvm, the guide ( the dm-crypt guide ) said to come back to crypt guide to configure initramfs and the boot loader: https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS
To re generate a mkinitcpio.conf file make sure to install lvm2 package and then add the following to the conf file and then regenerate using the *mkinitcpio -P* command
```
HOOKS=(base **systemd** autodetect microcode modconf kms **keyboard** **sd-vconsole** block **sd-encrypt** **lvm2** filesystems fsck)
```
You might encounter an error regarding vconsole.conf, to fix it just edit the vconsole.conf file and add a FONT ( I used latarcyrheb-sun32 ).
After this install a bootloader of your choice and in my case I chose the <a href=https://wiki.archlinux.org/title/Systemd-boot>systemd-boot</a> as my choice and to install it we simply enter ```# bootctl install```. This will install the necessary files. Next we go to /boot/loader/entries directory and create a configuration file named arch.conf. This is where the loader will know which os to boot up and in this case there is only one so it will boot up arch. Inside the file I inserted the following <img src=https://github.com/rghdrizzle/arch/blob/main/Screenshot%202025-01-23%20210911.png></img>. 
The UUID here is the uuid of the disk partition where we have the lvm so in my case it is sda2 which can be fetched by using the blkid command. After that we save the file and hopefully the bootloader will load arch for us when we reboot. Before we reboot we create a new user and give it root access by adding it to the wheel group and also editing the visudo file to uncomment the configuration to give root access to wheel group. Then we also create a password for the root and the new user. After that we can reboot the system and enter into the arch system we just configured and installed.



#### Resources and links used
- https://wiki.archlinux.org/title/Installation_guide
- https://www.youtube.com/watch?v=Y8-KKMdZn5Q
- [Error-kernel panic exception]- https://www.reddit.com/r/archlinux/comments/njnfc4/arch_linux_lts_5104_kernel_panic_exception/
- [How-To: Resize LVM]- https://www.redhat.com/en/blog/resize-lvm-simple
- [Error- black screen while booting arch]- https://www.reddit.com/r/virtualbox/comments/oqnmx6/virtualbox_with_arch_results_in_black_screen/
- [Error- https://bbs.archlinux.org/viewtopic.php?pid=1969918#p1969918 (tl;dr just choose a boot manager in the firmware interface and in my case it was the vbox harddrive)]
