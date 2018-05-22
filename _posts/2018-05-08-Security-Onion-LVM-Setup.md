---
layout: post
title: Security Onion LVM Setup
tags: [intrusion-detection, security-onion, nsm, lvm, linux]
comments: true
---

Recently I've been building a Security Onion cluster to take advantage of full packet capture, Bro, Snort, ELK, and the assortment of fantastic open source forensic tools included with the distro. I leaned heavily on the [Security Onion Wiki](https://github.com/Security-Onion-Solutions/security-onion/wiki/) throughout the process, and although the squad over at [Security Onion Solutions](https://securityonionsolutions.com/) have done a fantastic job with documentation, there wasn't much guidance regarding LVM and ideal partitioning schemes. I took this lack of guidance as an opportunity to learn LVM. Throughout the process I put together this basic guide to repartitioning the default Security Onion LVM scheme.  

Originally the plan was to build heavy nodes because I didn't have enough servers for a storage node, but managed to acquire another server and decided to give the distributed deployment a whirl. Because I didn't go through with the heavy node deployment I did not provide complete notes for it here, but I do provide a recommended heavy node RAID configuration. You can read more about node types and deployment architecture at the [Architecture](https://github.com/Security-Onion-Solutions/security-onion/wiki/Elastic-Architecture) Wiki Page. Lastly, this may not be the most correct method for reallocating default LVM space in Security Onion, but this is how I did it. The deployment is still young, but I have had no problems so far. Hope it helps!

# Hardware

### Master Node
- Virtualized 
- 600GB Disk Space 
- 8 CPUs 
- 32GB RAM 

### Sensor (Forward Node)
- 2 x 120GB drives in RAID 1 (/, /var) - Usually I prefer more storage here, but this is what I have to work with
- 12 x 10TB drives in RAID 5 (/nsm/sensor_data)
- 2 x 24 core CPUs (48 total cores)
- 128GB RAM

### Sensor (Heavy Node)
- 2 x 120GB drives in RAID 1 (/, /var)
- 8 x 10TB drives in RAID 5 (/nsm/sensor_data)
- 4 x 10TB drives in RAID 10 (/nsm/elasticsearch, /nsm/logstash)
- 2 x 24 core CPUs (48 total cores)
- 128GB RAM

### Storage Node
- 12 x 4TB Drives in RAID 10
- 2 x 20 core CPUs (40 total cores)
- 128GB RAM

# Bootable USB & Live ISO

All LVM changes need to be performed on unmounted disks, so you will need to create a bootable linux USB (I used Kali) to administer LVM on sensors and storage nodes.

The master node is virtualized, so after installing the OS from ISO you will need to mount a Linux ISO that allows live boot. The Security Onion ISO does allow live boot, but the boot menu doesn't explicitly show a "Live Boot" option.

Power on the VM and launch a web console through VCenter (assuming you use VMware in production). You may need to CTRL+ALT+DEL a few times followed by a quick ESC to catch the boot menu because it fades quickly. Once booted into the ISO, select "Install". Quit the installer after it loads, it should drop you to a login screen. Login with the username "securityonion" and password field blank, it will grant you a session as the "securityonion" user who has sudo privileges. Open a terminal and type `sudo su -` to elevate privileges to root.

# Master Node
This was the first server I built. After installing the Security Onion ISO I mounted a Kali ISO and booted into a live Kali environment.

By default the Security Onion LVM installer created one VG and two LVs:
- /dev/securityonion-vg (~600GB)
- /dev/securityonion-vg/root (~588GB)
- /dev/securityonion-vg/swap_1 (~10GB)

There are two LVM tweaks we need to make on this server. One is to reclaim swap space for the root partition; swap is unnecessary, assuming you have allocated sufficient RAM for your server. The second is to carve out dedicated space for /var so if the need for verbose logging ever arrises you don't need to worry about logs filling the entire filesystem.

```
lvremove /dev/securityonion-vg/swap_1
e2fsck -f /dev/securityonion-vg/root 
resize2fs -f /dev/securityonion-vg/root 500G
e2fsck -f /dev/securityonion-vg/root
lvreduce -L 500G /dev/securityonion-vg/root
lvcreate -n var -l 100%FREE /dev/securityonion-vg
mkfs -t ext4 /dev/securityonion-vg/var
e2fsck -f /dev/securityonion-vg/var
mkdir /mnt/var.new
mkdir /mnt/onionroot
mount -t ext4 /dev/securityonion-vg/var /mnt/var.new
mount -t ext4 /dev/securityonion-vg/root /mnt/onionroot
rsync -ravHPSAX /mnt/onionroot/var/ /mnt/var.new/
mv /mnt/onionroot/var /mnt/onionroot/var.bak
mkdir /mnt/onionroot/var
```

Now we need to edit fstab to make sure the new /var partition is mounting during boot, and ensure the system is no longer trying to mount swap space.

```
vim /mnt/onionroot/etc/fstab
```

Comment out:
```
/dev/mapper/securityonion--vg-swap_1 none    swap    sw    0    0
```

Add:
```
/dev/mapper/securityonion--vg-var /var    ext4    defaults    0    0
```

`:wq` to save and quit vim.

```
reboot
```

NOTE: The last field in an fstab entry tells the system what order to check that filesystem during boot. 0 tells the system to skip the check, 1 tells the system which entry to check first, and 2 is for other entries (checks in listed order). I originally set the /var entry to 2, but the system kept hanging during boot. I changed it to 0 and now there are no problems.

# Sensor (Forward Node)

On this sensor we have two RAID arrays, the 2 x 120GB drives in RAID 1 give me ~120GB usable space to install the OS, and the 12 x 10TB drives in RAID 5 give me ~110TB usable space for /nsm/sensor_data (pcaps).

I selected the 120GB disk for system install:

By default the Security Onion LVM installer created one VG and three LVs:
- /dev/securityonion-vg (~110GB)
- /dev/securityonion-vg/root (~9GB)
- /dev/securityonion-vg/home (~97GB)
- /dev/securityonion-vg/swap_1 (~4GB)

Swap is still unnecessary (with the proper hardware) so first we remove the swap_1 LV. After removing swap we need to reduce the home LV and expand the root LV. We will also want to carve out dedicated room for /var. In this scenario I gave my home LV 40G, root LV 50G, and var LV 20G, but you can allocate whatever you feel comfortable with here.

```
lvremove /dev/securityonion-vg/swap_1
e2fsck -f /dev/securityonion-vg/home
resize2fs -f /dev/securityonion-vg/home 40G
e2fsck -f /dev/securityonion-vg/home
lvreduce -L 40G /dev/securityonion-vg/home
lvextend -L 50G /dev/securityonion-vg/root
e2fsck -f /dev/securityonion-vg/root
resize2fs -f /dev/securityonion-vg/root 50G
e2fsck -f /dev/securityonion-vg/root
lvcreate -n var -l 100%FREE /dev/securityonion-vg
mkfs -t ext4 /dev/securityonion-vg/var
e2fsck -f /dev/securityonion-vg/var
mkdir /mnt/var.new
mkdir /mnt/onionroot
mount -t ext4 /dev/securityonion-vg/var /mnt/var.new
mount -t ext4 /dev/securityonion-vg/root /mnt/onionroot
rsync -avrHPSAX /mnt/onionroot/var/ /mnt/var.new/
mv /mnt/onionroot/var /mnt/onionroot/var.bak
mkdir /mnt/onionroot/var
```

Next we will create a new ~100TB VG, PV, and LV for /nsm/sensor_data, in this example I use /dev/sdb1.

```
pvcreate /dev/sdb1
vgcreate sensor_data-vg /dev/sdb1
lvcreate -n sensor_data -l 100%FREE /dev/sensor_data-vg
mkfs -t xfs /dev/sensor_data-vg/sensor_data
e2fsck -f /dev/sensor_data-vg/sensor_data
```

Now we need to edit fstab again to make sure the new /var and /nsm/sensor_data partitions are mounting during boot, and ensure the system is no longer trying to mount swap space.

```
vim /mnt/onionroot/etc/fstab
```

Comment out:
```
/dev/mapper/securityonion--vg-swap_1 none    swap    sw    0    0
```

Add:
```
/dev/mapper/securityonion--vg-var /var    ext4    defaults    0    0
/dev/mapper/sensor_data--vg-sensor_data /nsm/sensor_data    xfs    defaults    0    0
```

`:wq` to save and quit vim.

```
reboot
```

# Storage Node
On this sensor we have ~22TB usable storage in RAID 10. After installation we boot from our USB again and configure partitions like such...

By default the Security Onion LVM installer created one VG and two LVs:
- /dev/securityonion-vg (~22GB)
- /dev/securityonion-vg/root (~21.7GB)
- /dev/securityonion-vg/swap_1 (~128GB)

Swap is still unnecessary (with the proper hardware) so first we remove the swap_1 LV. After removing swap we need to reduce the root LV, and carve out dedicated space for /var and /nsm. In this scenario I gave my root LV 1TB, var LV 100GB, and nsm LV all remaining space, but you can allocate whatever you feel comfortable with here.

```
lvremove /dev/securityonion-vg/swap_1
e2fsck -f /dev/securityonion-vg/root
resize2fs -f /dev/securityonion-vg/root 1T
e2fsck -f /dev/securityonion-vg/root
lvreduce -L 1T /dev/securityonion-vg/root
lvcreate -n var -L 100G /dev/securityonion-vg
lvcreate -n nsm -l 100%FREE /dev/securityonion-vg
mkfs -t ext4 /dev/securityonion-vg/var
e2fsck -f /dev/securityonion-vg/var
mkfs -t ext4 /dev/securityonion-vg/nsm
e2fsck -f /dev/securityonion-vg/nsm
mkdir /mnt/var.new
mkdir /mnt/nsm.new
mkdir /mnt/onionroot
mount -t ext4 /dev/securityonion-vg/var /mnt/var.new
mount -t ext4 /dev/securityonion-vg/nsm /mnt/nsm.new
mount -t ext4 /dev/securityonion-vg/root /mnt/onionroot
rsync -avrHPSAX /mnt/onionroot/var/ /mnt/var.new/
rsync -avrHPSAX /mnt/onionroot/nsm/ /mnt/nsm.new/
mv /mnt/onionroot/var /mnt/onionroot/var.bak
mv /mnt/onionroot/nsm /mnt/onionroot/nsm.bak
mkdir /mnt/onionroot/var
mkdir /mnt/onionroot/nsm
```

Now we need to edit fstab again to make sure the new /var and /nsm partitions are mounting during boot, and ensure the system is no longer trying to mount swap space.

```
vim /mnt/onionroot/etc/fstab
```

Comment out:
```
/dev/mapper/securityonion--vg-swap_1 none    swap    sw    0    0
```

Add:
```
/dev/mapper/securityonion--vg-var /var    ext4    defaults    0    0
/dev/mapper/securityonion--vg-nsm /nsm    ext4    defaults    0    0
```

`:wq` to save and quit vim.

```
reboot
```

That's all folks! I'll update these notes if any pertinent issues come to light.
