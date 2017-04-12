***About SimpleOSbackup***

SimpleOSbackup or simply - SOSbackup is a bash shell script used for backup of Linux OS system. 
You can run backup to a partition or whole disk. In case you are running backup to a partition, 
the backup will exclude  boot  fs (/boot, /boot/efi etc.). If you are running backup to a disk, 
SOSbackup will erease current partition table on it (if there is any) and create a new one. 
More info in the section of specific functions.

Backup is fully operational and can choosen from GRUB or LILO menu. The boot loader on the original 
disk is still needed. So far the boot loader on the backup device does not work. In the situation 
where your original disk will break up you will need to install the GRUB or LILO manually to the backup 
device. One day maybe i will implement that into the script too :). However, the boot loader config 
file will be configured by SOSbackup so you will not need to edit it manually. 
You must be a root to be able to run SOSbackup for OS backup.
This script was tested on several Linux distributions in various configurations. 
	    
I am not responsible for any damage done using this tool.


**Functions**

The main function of SOSbackup is to create a bootable copy og your system. SOSbackup will automatically edit fstab on the
backup disk and add backup device to thr boot loader config file (Grub Legacy, Grub 2.0, LILO). Tt will chek the size of the
destination device, check if all components are present, generate a log file and send a report via email.

**Create backup to the partition (option -p):**
You can backup the system to a separate disk partition. This option will exclude your current boot partition (for example /boot)
so this fs will be used also by OS boot from the backup device. However SOSbackup will create a copy of current kernel and
initramfs on it (the copy will have "SOSbackup" instead of kernel version in the file name). In /etc/sosbkp.conf you can exclude
another folders or files which you do not want to backup. before you start the backup, your destination backup partition needs
to be formatted (ideally to ext4). 

**Create backup to the disk (option -d):**
You can backup the system to a separate disk. You need one empty disk drive. SOSbackup will automatically create 
a 500MB boot partition and format it to vfat for EFI+GPT or to ext2 BIOS+MBR. Then it will create second partition for 
the rest of the data (size will depend of the size of backed up the data) and format it to ext4 or xfs (depends on what
filesystem is using the original root partition). If are using LILO, SOSbackup will install aslo the boot loader into 
the backup disk. If you are using Grub, you will need to boot the backup OS and install it manually to the backup disk (just 
run grub-install or grub2-install). This will help you to boot your backup system in case your primary boot disk will fail.


**How to use sosbkp:**
```
sosbkp [OPTIONS]...
```

**Arguments:** 
```
-p <partition>, --to-partition=<partition>  Start SOSbackup to the partition 
                              THIS OPTION NEEDS TO BE ON THE END OF THE COMMAND 
                              Partition needs to be formated before you start SOSbackup. 
                              If you are using LVM partition as the destination, use path /dev/mapper/...
                              SOSbackup will erease all data on the partition by default.
                              This argument needs to be the last one.
	    
-d <disk>, --to-disk=<disk>   Start SOSbackup to the disk.  
                              THIS OPTION NEEDS TO BE ON THE END OF THE COMMAND 
                              SOSbackup will erease existing partitions on the backup device any and create
                              two new partitions. The first one is for boot fs and the second for the rest
                              of the data. SOSbackup will edit boot loader menu and add the new boot device.
                              The backup device will be not bootable (even the configuration 
                              files are ready and do not need to be edited). In case your 
                              original boot device will fail or damage, you will need to 
                              install the boot loader to the new disk (backup disk).
                              This argument needs to be the last one.
    
-S, --shutdown                Shutdown after backup 
	    
-x, --no-email                Report will be not sent via email 
	    
-X, --no-selinux              Do not copy SELinux labels 
	    
-c, --clear                   Can be used after interrupted backup 
	    
-n, --no-delete               Do NOT delete previous backup data. Will update current backup. 
                              Keep in mind that rsync needs some free space for temporarry files. 
	    
-s, --show                    Show last backup info 
	    
-f, --force                   Force backup and suppress warnings. (Missing rsync can not be suppresed) 
	    
-v, --version                 Show SOSbackup version 
	    
    --debug                   Debug mode 
	    
-h, --help                    This Help page
```  
**Examples:**

```
sosbkp --to-partition=/dev/sdb1 
sosbkp -Sxp /dev/sdb1 
sosbkp -S -f -p /dev/mapper/backup_vg-backup_lv 
sosbkp --no-selinux -d /dev/sdc 
sosbkp -d sdc
```
