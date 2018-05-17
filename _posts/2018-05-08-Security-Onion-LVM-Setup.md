---
layout: post
title: Security Onion LVM Setup
tags: [intrusion-detection, security-onion, nsm, LVM]
comments: true
---

Recently I built and deployed a multi-node Security Onion cluster to take advantage of full packet capture, Bro, Snort, ELK, and the assortment of fantastic open source forensic tools included with the distro. I leaned heavily on the [Security Onion Wiki](https://github.com/Security-Onion-Solutions/security-onion/wiki/) throughout the process, and although the guys over at [Security Onion Solutions](https://securityonionsolutions.com/) have done a fantastic job with the documentation, there wasn't much there regarding LVM and ideal partitioning. To help the community I put together this quick guide to basic LVM administration and repartitioning Security Onion from the default LVM setup. Be aware that both servers are being built as heavy nodes. You can read more about node types and deployment architecture at the [Architecture Wiki Page](https://github.com/Security-Onion-Solutions/security-onion/wiki/Elastic-Architecture).

# Hardware

Test Server
8 x 2TB drives in RAID 50
128GB RAM
2 x 12 core CPUs (24 total cores)

Production Server
2 x 120GB drives in RAID 1
8 x 10TB drives in RAID 5
4 x 10Tb drives in RAID 10
128GB RAM
2 x 24 core CPUs (48 total cores)

# Bootable USB

All LVM changes need to be performed with the disks unmounted, so you will need to create a bootable linux USB (I used Kali) and boot from that.

# Test Server

This was an extra server I had laying around so I'm currently using it to test updates before pushing them to production, and experiment with new configurations/tools. 8 x 2TB drives in RAID 50 left me with ~12TB usable storage. DESCRIBE DEFAULT PARTITION SCHEME + PVS + VGS + LVS

By default the Security Onion LVM setup will create two LVs:
- /dev/securityonion-vg/root
- /dev/securityonion-vg/swap_1.

Swap should be unnecessary (with the proper hardware) so the first step is to remove the swap_1 LV.

`lvremove /dev/securityonion-vg/swap_1
e2fsck -f /dev/securityonion-vg/root
resize2fs /dev/securityonion-vg/root`

lvextend /dev/securityonion-vg/root /dev/sda3

# Production Server

This process was slightly different because we had multiple RAID arrays. The 2 x 120GB drives in RAID 1 give me ~120GB usable storage to install the OS, the 8 x 10TB drives in RAID 5 give me ~70TB usable storage for /nsm/sensor_data (pcaps), and the 4 x 10TB drives in RAID 10 give me ~20TB storage for the ELK components (/nsm/elasticsearch and /nsm/logstash). 

I selected the 120GB disk for system install:

By default the Security Onion LVM installer created one VG and three LVs:
- /dev/securityonion-vg (~110GB)
- /dev/securityonion-vg/root (~9GB)
- /dev/securityonion-vg/home (~97GB)
- /dev/securityonion-vg/swap_1. (~4GB)

Swap is still unnecessary (with the proper hardware) so first we remove the swap_1 LV. After removing swap we need to reduce the home LV and expand the root LV. In this scenario I gave my home LV 40G and my root LV 50G, but you can allocate whatever you feel comfortable with here.

lvremove /dev/securityonion-vg/swap_1
e2fsck -f /dev/securityonion-vg/home
resize2fs -f /dev/securityonion-vg/home 40G
e2fsck -f /dev/securityonion-vg/home
lvreduce -L 40G /dev/securityonion-vg/home

lvextend -L 50G /dev/securityonion-vg/root
e2fsck -f /dev/securityonion-vg/root
resize2fs -f /dev/securityonion-vg/root 50G
e2fsck -f /dev/securityonion-vg/root

# First option creates 20G LV, second option creates LV using all remaining VG space
lvcreate -n var -L 20G /dev/securityonion-vg
lvcreate -n var -l 100%FREE /dev/securityonion-vg 
mk

# Make filesystem for /var LV, create mount point for new /var LV, mount new /var LV, transfer old /var contents to new /var LV
mkfs -t ext4 /dev/securityonion-vg/var
e2fsck -f /dev/securityonion-vg/var
mkdir /mnt/var.new
mkdir /mnt/onionroot
mount -t ext4 /dev/securityonion-vg/var /mnt/var.new
mount -t ext4 /dev/securityonion-vg/root /mnt/onionroot
rsync -raX /var/ /var.new/

# Edit fstab file - add new LVs and remove swap entry
vim /mnt/onionroot/etc/fstab





## Deleting swap volume
lvremove /dev/securityonion-vg/swap_1
lvextend /dev/securityonion-vg/root /dev/sda3
e2fsck -f /dev/securityonion-vg/root
resize2fs /dev/securityonion-vg/root

## Shrinking LVM Volumes - https://www.rootusers.com/lvm-resize-how-to-decrease-an-lvm-partition/
e2fsck -f /dev/securityonion-vg/root
resize2fs /dev/securityonion-vg/root 30G
lvreduce -L 30G /dev/securityonion-vg/root
resize2fs /dev/securityonion-vg/root

## Expanding LVM Volumes - https://www.rootusers.com/lvm-resize-how-to-increase-an-lvm-partition/

## Creating new LVM Volumes
lvcreate -n nsm -L 10G /dev/securityonion-vg # Created new 10G volume
lvcreate -n nsm -l 100%FREE /dev/securityonion-vg # Creates new volume and uses 100% of free space in Volume Group

## Format and Mount the Logical Volume - https://www.howtogeek.com/howto/40702/how-to-manage-and-use-lvm-logical-volume-management-in-ubuntu/
# Should be done from bootable usb linux OS
mkfs -t xfs /dev/securityonion-vg/nsm
mkdir /nsm.new
mount -t xfs /dev/securityonion-vg/nsm /nsm.new
rsync -raX /nsm/ /nsm.new/
mkdir /mnt/onionroot
mount -t ext4 /dev/securityonion-vg/root /mnt/onionroot
## for permanent mount must edit /etc/fstab

# References
fstab - https://help.ubuntu.com/community/Fstab
Disabling Swap - https://askubuntu.com/questions/532121/problem-removing-swap-partition
Administering LVM - https://www.tecmint.com/manage-and-create-lvm-parition-using-vgcreate-lvcreate-and-lvextend/
Creating Volumes & Changing Mount Points - https://serverfault.com/questions/429937/how-to-move-var-to-another-existing-partition
Shrinking LVM Volumes - https://www.rootusers.com/lvm-resize-how-to-decrease-an-lvm-partition/
Expanding LVM Volumes - https://www.rootusers.com/lvm-resize-how-to-increase-an-lvm-partition/
Format and Mount Logical Volumes - https://www.howtogeek.com/howto/40702/how-to-manage-and-use-lvm-logical-volume-management-in-ubuntu/
7 Ways to Check Filesystem Type - https://www.tecmint.com/find-linux-filesystem-type/
