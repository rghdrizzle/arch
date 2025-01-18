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


#### Resources and links used
- https://wiki.archlinux.org/title/Installation_guide
- https://www.youtube.com/watch?v=Y8-KKMdZn5Q
- [Error-kernel panic exception]- https://www.reddit.com/r/archlinux/comments/njnfc4/arch_linux_lts_5104_kernel_panic_exception/
- [How-To: Resize LVM]- https://www.redhat.com/en/blog/resize-lvm-simple
- [Error- black screen while booting arch]- https://www.reddit.com/r/virtualbox/comments/oqnmx6/virtualbox_with_arch_results_in_black_screen/
