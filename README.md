# Hackintosh-Dell-15-G5587
## The complete guide to hackintoshing Dell G5 5587 laptop (Dual boot w/ Windows 10 Pro 1903)
### The Setup
#### My hardware:
##### CPU: Intel Core i5-8300H @ 8x 2.3GHz Coffee Lake
##### GPU: GeForce GTX 1050 TI
##### Motherboard: Dell Inc. 03TW4P
##### Chipset: Intel HM370
##### Audio: Realtek ALC3246-CG with Waves MaxxAudio Pro
##### BIOS: Dell Inc. 1.2.1
##### Wifi: Intel AC9560
##### SSD: 500 GB WD Original (WDS500G2B0B)
##### Three USB flash drives: 16 GB for Catalina, 8 GB for Windows 10, 4 GB for Try Ubuntu (fixing partitions if broken)
(It's most likely possible to do it w/o third flash drive because I was doing everything for scratch and bumped into diskutil issues)
### Installation Process
#### Prequisites
##### Important info: Install macOS first then Windows (NOT VICE-VERSA because it won't work!)
If you don't have a Mac it's possible to download an archive with gibMacOS or from the internet
My archive: https://mega.nz/#!wZsWiaZI!lJwcllKcE8MviuI-ztsOY7K6i0yCoVl9ewERdDGJAK8
(unpack it where you like)
Download the EFI folder from this repository and unpack it too
##### Warning! Following steps are form insanelymac's guide to install High Sierra
###### Note: The process is described for 8GB drive. For 16/32/64/etc it's the same because you have 200 MB EFI first partition and 1.9 GB last partition, the difference is in the numbers between partitions, but they are very easy to figure out
1. First of all you need to cd to the SharedSupport folder withing 'Install macOS Catalina.app' folder
2. Run "7z l BaseSystem.dmg > temp.txt"
3. Open temp.txt with any text editor and find the following text:
```
----
Path = 4.hfs
Size = 2008604672
Packed Size = 498940512
Comment = disk image (Apple_HFS : 4)
Method = Copy Zero2 ZLIB CRC
--
```
4. Run bytes to megabytes count with the size of 4.hfs 
###### 2008604672 Bytes = 1915.5546875 MB (in binary)
5. Count the sectors
###### 2008604672 / 512 = 3923056 Sectors ( 512 Bytes per sector)
6. Open GParted. In this example, I have an 8GB installer disk, /dev/sdb, initialized as GPT from Device Menu 
7. Create the following 2 new partitions 
Part 1 200MB FAT32 labelled EFI
Part 2 5817MB HFS+ labelled Installer_App (Install hfsplus and hfsprogs via apt-get if HFS+ isn't available in your GParted)
Leave 2174MB free for OS X Base System restore (=1916+129+129 MB of loader space before and after to keep Apple's Disk Utility happy) 
8. Use gdisk to set correct partition type, name and attributes for the EFI System Partition and create new sdb3 for OS X Base System
```
fusion71au@fusion71au-VirtualBox ~ $ sudo gdisk /dev/sdb
[sudo] password for fusion71au:
GPT fdisk (gdisk) version 0.8.8

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): t
Partition number (1-3): 1
Current type is 'Microsoft basic data'
Hex code or GUID (L to show codes, Enter = 8300): EF00
Changed type of partition to 'EFI System'
 
Command (? for help): c
Partition number (1-3): 1
Enter name: EFI
 
Command (? for help): x
 
Expert command (? for help): a
Partition number (1-3): 1
Known attributes are:
0: system partition
1: hide from EFI
2: legacy BIOS bootable
60: read-only
62: hidden
63: do not automount
 
Attribute value is 0000000000000000. Set fields are:
  No fields set
 
Toggle which attribute field (0-63, 64 or <Enter> to exit): 0
Have enabled the 'system partition' attribute.
Attribute value is 0000000000000001. Set fields are:
0 (system partition)
 
Toggle which attribute field (0-63, 64 or <Enter> to exit):  
 
Expert command (? for help): m

Command (? for help): n
Partition number (3-128, default 3): 3
First sector (34-16777182, default = 12324864) or {+-}size{KMGTP}: +129M <--- For Disk Utility loader space
Last sector (12589056-16777182, default = 16777182) or {+-}size{KMGTP}: 16512112 <--- Equals 12589056+3923056 sectors for Base System
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): AF00
Changed type of partition to 'Apple HFS/HFS+'

Command (? for help): p
Disk /dev/sdb: 16777216 sectors, 8.0 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): 10D5C66B-C765-5247-9DF4-358C0FEB2208
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 16777182
Partitions will be aligned on 2048-sector boundaries
Total free space is 531468 sectors (259.5 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          411647   200.0 MiB   EF00  
   2          411648        12324863   5.7 GiB     AF00  
   3        12589056        16512112   1.9 GiB     AF00  Apple HFS/HFS+

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has completed successfully. 
```
9. Open Terminal, type lsblk to show the system's attached disks, partitions and their mount points
```
fusion71au@fusion71au-VirtualBox ~ $ lsblk
 
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0    50G  0 disk  
├─sda1   8:1    0   200M  0 part  
├─sda2   8:2    0   350M  0 part /boot
├─sda3   8:3    0    15G  0 part /
├─sda4   8:4    0  32.5G  0 part /home
└─sda5   8:5    0     2G  0 part [SWAP]
sdb      8:16   0     8G  0 disk  
├─sdb1   8:17   0   200M  0 part   
├─sdb2   8:18   0   5.7G  0 part
├─sdb3   8:19   0   1.9G  0 part
sr0     11:0    1  1024M  0 rom   
```
10. Type dmg2img -l BaseSystem.dmg to list the "partitions" in the compressed disk image file "BaseSystem.dmg"
```
fusion71au@fusion71au-VirtualBox ~/Downloads/SharedSupport $ dmg2img -l BaseSystem.dmg
 
dmg2img v1.6.5 (c) vu1tur (to@vu1tur.eu.org)
 
BaseSystem.dmg --> (partition list)
 
partition 0: Protective Master Boot Record (MBR : 0)
partition 1: GPT Header (Primary GPT Header : 1)
partition 2: GPT Partition Data (Primary GPT Table : 2)
partition 3:  (Apple_Free : 3)
partition 4: disk image (Apple_HFS : 4)
partition 5:  (Apple_Free : 5)
partition 6: GPT Partition Data (Backup GPT Table : 6)
partition 7: GPT Header (Backup GPT Header : 7)
```
11. Use the command sudo dmg2img -v -i BaseSystem.dmg -p 4 -o /dev/sdb3 (the 1.9GB partition) to write the 4.hfs image to your "OS X Base System" volume i.e. sdb3 partition. 
```
fusion71au@fusion71au-VirtualBox ~/Downloads/SharedSupport $ sudo dmg2img -v -i BaseSystem.dmg -p 4 -o /dev/sdb3
[sudo] password for fusion71au:  
 
dmg2img v1.6.5 (c) vu1tur (to@vu1tur.eu.org)
 
BaseSystem.dmg --> /dev/sdb3
 
reading property list, 52391 bytes from address 491582553 ...
partition 0: begin=203, size=430, decoded=284
partition 1: begin=948, size=430, decoded=284
partition 2: begin=1695, size=430, decoded=284
partition 3: begin=2424, size=430, decoded=284
partition 4: begin=3137, size=42778, decoded=28804
partition 5: begin=46198, size=430, decoded=284
partition 6: begin=46926, size=430, decoded=284
partition 7: begin=47671, size=430, decoded=284
 
decompressing:
opening partition 4 ...       [715] 100.00%  ok
 
Archive successfully decompressed as /dev/sdb3
 
You should be able to mount the image [as root] by:
 
modprobe hfsplus
mount -t hfsplus -o loop /dev/sdb3 /mnt
```
12. Create mounting folders in /media/your_username  
```
sudo mkdir /media/fusion71au/EFI
sudo mkdir /media/fusion71au/Installer_App  
```
13. Mount the Installer_App volume (corresponding to sdb2) and copy the SharedSupport folder to its root 
```
sudo mount /dev/sdb2 /media/fusion71au/Installer_App  
sudo cp -R ~/Downloads/SharedSupport /media/fusion71au/Installer_App/
```
14. Mount the EFI partition, sdb1, and copy the EFI folder containing Clover into it
```
sudo mount /dev/sdb1 /media/fusion71au/EFI
sudo cp -R ~/Downloads/EFI /media/fusion71au/EFI/
```
##### Time to boot to OS X Base System and install macOS
1. Unmount sdb1 & sdb2 via sudo unmount
2. Reboot into your USB Clover (you can add this option manually in your BIOS finding /boot/EFI/bootx64.EFI file)
3. If you see Install macOS option, you've been successful, press Enter and wait until BaseSystem loads
4. On the Utilites tab open terminal (DiskUtility GUI didn't work for me)
5. Type ```diskutil list``` and determine your disk to install
6. Type ```diskutil eraseDisk``` APFS MacintoshHD disk# (where # - is your number)
###### NOTE: You need to nuke a whole disk, eraseVolume doesn't work
7. Then type these following commands
```
-bash-3.2# cd /
-bash-3.2# cp -R Install\ macOS\ Catalina.app /Volumes/Installer_App/
-bash-3.2# mv /Volumes/Installer_App/SharedSupport /Volumes/Installer_App/Install\ macOS\ Catalina.app/Contents/
```
8. Start installation with the startosinstall utility in Terminal
```
-bash-3.2# /Volumes/Installer_App/Install\ macOS\ Catalina.app/Contents/Resources/startosinstall --volume /Volumes/MacintoshHD
```
9. After installation reboot into Clover and you'll see two macOS options, pick one that is not Install macOS and start configuring your macOS
##### Installing Windows
##### NOTE: Windows Install Utility can't install on a formatted partition with Disk Utility (even after deleting and creating partition by hand)
0. Make a bootable USB with Ubuntu to use "Try Ubuntu"
1. Open Apple Disk Utility and create a separate partition for Windows
2. Boot into Ubuntu USB and install hfsprogs and hfsutils
3. Open GParted and find an empty partition and format it into FAT32 (NTFS didn't work for me)
4. Boot into Windows bootable USB and format that FAT32 partition and begin installing (the process is straightforward afterwards)
## Credits
### fusion71au from insanelymac - https://www.insanelymac.com/forum/topic/329828-making-a-bootable-high-sierra-usb-installer-entirely-from-scratch-in-windows-or-linux-mint-without-access-to-mac-or-app-store-installerapp/?do=findComment&comment=2538638
### Richhhhhhhh from tonymacx86 for EFI archive - https://www.tonymacx86.com/threads/dell-g5-5587-working-with-catalina-10-15-2.290636/
