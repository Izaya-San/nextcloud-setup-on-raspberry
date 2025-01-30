# Table of content
- [Table of content](#table-of-content)
- [General notes](#general-notes)
- [Checks and dependencies](#checks-and-dependencies)
- [Create the RAID](#create-the-raid)
  - [New RAID 5](#new-raid-5)
    - [Disks formating](#disks-formating)
    - [RAID creation](#raid-creation)
  - [Existing RAID 5](#existing-raid-5)
    - [Check RAID state](#check-raid-state)
    - [Re-assemble](#re-assemble)
- [Save the configuration](#save-the-configuration)
- [Mount](#mount)

# General notes
This section is about storing the nextcloud data folder, hence it's a critical part as you don't want to lose all your data if something happen. You could use your SD Card (don't do that), or your SSD, or even an external harddrive plugged to your Raspberry Pi; but in any case, if your disk break you'll lose everything. I've opted for a RAID 5 to tacle this issue, but you can imagine something else that best suits you.

What's a RAID 5 ? It's a "redundant array of independent disks" and Wikipedia tells us it's a:

    Configurations that employ the techniques of striping, mirroring, or parity to create large reliable data stores from multiple general-purpose computer hard disk drives (HDDs). The most common types are RAID 0 (striping), RAID 1 (mirroring) and its variants, RAID 5 (distributed parity), and RAID 6 (dual parity).

In this example we'll use a RAID 5 with 3 disks meaning one disk can break, and we'll be able to rebuild this disk from the two others. But if more than one disk breaks, it'll mean data loss. I consider it sufficient for my use case, but you can opt for the RAID type of your choice.

# Checks and dependencies
Plug your 3 disks to the Raspberry Pi and don't forget to power them.

```sh
# Verify you see all your disks
sudo fdisk -l
# Verify that they are not mounted (you should see your SSD here)
findmnt /
# verify that there is no older RAID setted up in your fstab (which should not be the case on a fresh install)
cat /etc/fstab

# Install mdadm that will handle the RAID for us
sudo apt install mdadm
```

# Create the RAID
I'll explain here how to create a RAID 5 in two cases:
- From new disks (an installation from scratch): [see](#new-raid-5)
- Reinstall an existing RAID 5 on a new OS (a reinstallation with existing data): [see](#existing-raid-5)

## New RAID 5
We'll treat the use case where you're installing a fresh new RAID with new disks. If you already have RAID, jump to the next section.

### Disks formating
This step will erase your disks content.

For simplicity, we'll delete all the disks partitions and recreate one partition on the full disk. If you have disks of variable sizes, the partition size should be the one of the smallest disk. If you have more complex setups like you want to only use a part of each disk, check `fdisk` and `mdadm` commands in more details.

Disks naming: your partitions are named like this `/dev/sdxy`. `x` is your disk letter, so when we refer to the full disk we'll use `/dev/sdx`. `y` is your partition number, so when we refer to a specific partition we'll use `/dev/sdxy`. In the following commands, change the disk name by yours.

At the end, we should have each disk with 1 partition, each being the same size and type: `Linux raid autodetect` and format `ext4`. Note: if needed change the disklabel for `DOS` because with other types, it's possible that you can't get `Linux raid autodetect`.

Here we use `fdisk` which set up our partitions on the disks, and `mkfs` which format partitions.

```sh
# Delete all partitions
sudo fdisk /dev/sdx
  # to delete the partition
  d
  # to save the change
  w

# Create a new partition
sudo fdisk /dev/sdx
  # to create a new partition
  n
  # to set it to primary
  p
  # Then you can leave it empty. If you need a more precise setup, you can choose the number of partition (for example 4 will create sda4), the first sector and the last sector
  # to save
  w

# Set partition type to Linux raid autodetect
sudo fdisk /dev/sdx
  # to change the type
  t
  # set it to "Linux raid autodetect"
  fd
  # to save
  w

# Format the partitions we've created to ext4
sudo mkfs -t ext4 /dev/sdxy
```
Do this on each disk.

### RAID creation
```sh
# Create a RAID 5 on your disks partitions and wait until it finishs. In this example the RAID is named /dev/md127 and the partitions are /dev/sda1 /dev/sdb1 /dev/sdc1, change them with yours
sudo mdadm --create /dev/md127 --level=5 --raid-devices=3 /dev/sda1 /dev/sdb1 /dev/sdc1
  # To check the creation progress use
  cat /proc/mdstat

# You can also format the full RAID with
sudo mkfs -t ext4 /dev/md127
```

## Existing RAID 5
We'll treat here the case where you already have RAID disks with data on them, and you just want to plug them back on a new machine.
```sh
# Plug the disks to the Raspberry Pi and reboot should suffice to rediscover the existing RAID structure on the disks.
sudo reboot
```

### Check RAID state
```sh
# List visible partitions
sudo blkid
# You should have 3 partitions, one for each disk, that you can identify with thanks to `TYPE="linux_raid_member"`
# For each of them, check the partition state
sudo mdadm --examine /dev/sdxy
```
Example of an output:
```
/dev/sda1:
          Magic : zse654f65
        Version : 1.2
    Feature Map : 0x1
     Array UUID : qsd5gv4dz:qdv465z4v:q6s5v4es:qs6v5d4
           Name : raidname:127
  Creation Time : Sun May 22 17:36:04 2022
     Raid Level : raid5
   Raid Devices : 3

 Avail Dev Size : 3906762928 sectors (1862.89 GiB 2000.26 GB)
     Array Size : 3906762752 KiB (3.64 TiB 4.00 TB)
  Used Dev Size : 3906762752 sectors (1862.89 GiB 2000.26 GB)
    Data Offset : 264192 sectors
   Super Offset : 8 sectors
   Unused Space : before=264112 sectors, after=176 sectors
          State : clean
    Device UUID : 54sfg45:dsv6s546z:s6edv54:sdv654z

Internal Bitmap : 8 sectors from superblock
    Update Time : Thu Jan 30 20:58:01 2025
  Bad Block Log : 512 entries available at offset 16 sectors
       Checksum : d8ab3fd3 - correct
         Events : 50072

         Layout : left-symmetric
     Chunk Size : 512K

   Device Role : Active device 0
   Array State : AAA ('A' == active, '.' == missing, 'R' == replacing)

```
Verify that the partition have the same `Array UUID`, `Name`. You should check that they have close `Events` numbers has having an important difference can lead to data loss during re-assembling. Check the `State` which should be clean, and the `Array State` which should show them all `Active`.

If anything feels wrong, please refer to mdadm documentation to take the right decision.

```sh
# Examin what RAID is visible. This should print something like INACTIVE-ARRAY /dev/md127 metadata=1.2 name=raidname:127 UUID=qsd5gv4dz:qdv465z4v:q6s5v4es:qs6v5d4
sudo mdadm --detail --scan
# Now we know our RAID is /dev/md127, check its state
sudo mdadm --detail /dev/md127
```
It'll display something like this
```
/dev/md127:
           Version : 1.2
     Creation Time : Sun May 22 17:36:04 2022
        Raid Level : raid5
        Array Size : 3906762752 (3.64 TiB 4.00 TB)
     Used Dev Size : 1953381376 (1862.89 GiB 2000.26 GB)
      Raid Devices : 3
     Total Devices : 3
       Persistence : Superblock is persistent

     Intent Bitmap : Internal

       Update Time : Thu Jan 30 20:58:01 2025
             State : clean
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : bitmap

              Name : raidname:127
              UUID : qsd5gv4dz:qdv465z4v:q6s5v4es:qs6v5d4
            Events : 50072

    Number   Major   Minor   RaidDevice State
       4       8        1        0      active sync   /dev/sda1
       1       8       17        1      active sync   /dev/sdb1
       3       8       33        2      active sync   /dev/sdc1
```
What's important here is that you find your 3 partitions `active sync`, the `UUID` and `Name` matches and the `State` is `clean`. If you have something wrong here, check the official documentation. For example, you could have one partition missing, that's not an issue as you can recreate a missing partition from the two others. For now, we'll suppose everything is correct and working.

### Re-assemble
This is the critical part and there are a lot of ways to do it depending on your situation. I'll give one way I've used for my special use case, but you really should deep dive your proper use case as every variation can make a big difference. So please, be sure to understand everything before trying to do something as it could cost you your data loss.

```sh
# If running, stop it
sudo mdadm --stop /dev/md127
# If mounted, umount it
sudo umount /dev/md127

# This is the critical part: try to assemble it
sudo mdadm --assemble --verbose /dev/md127 --uuid qsd5gv4dz:qdv465z4v:q6s5v4es:qs6v5d4
# Check the current state of the RAID, it should show it's in a build process
sudo mdadm --detail /dev/md127
# Check the progression of the build
cat /proc/mdstat
```

# Save the configuration
Once your RAID is fully ready
```sh
# Copy your RAID configuration to save it. It should display something like `ARRAY /dev/md127 metadata=1.2 name=raidname:127 UUID=qsd5gv4dz:qdv465z4v:q6s5v4es:qs6v5d4`
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
update-initramfs -u
# Reboot and your RAID should still be accessible after the reboot
sudo reboot
```

# Mount
Now we have a RAID available, we'll mount it, so we can access its content.
```sh
# Create the folder on which we'll mount the raid. This folder is in /var/www where all websites are usually stored and which is created by Apache
sudo mkdir /var/www/raid5

# Update your fstab which is responsible of mounting disks automatically at startup
sudo nano /etc/fstab
    # Add this line
    /dev/md127  /var/www/raid5               ext4    defaults  0       2

# Reload fstab
sudo mount -av
    # Typical answer
    /                        : ignored
    /boot/firmware           : already mounted
    /var/www/raid5           : successfully mounted

# Check you have access to the RAID content. If you are resintalling an existing RAID, you should see all your data
ls /var/www/raid5

# Rebooting and check it's still mounted after it
sudo reboot
ls /var/www/raid5
```
