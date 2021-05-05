---
layout: post
title: How to correctly expand volume size on Xpenology
date: '2017-09-12 02:39:24'
tags:
- esxi
- xpenology
- dsm
- synology
---

## tl;dr

You should always **add another new disk** in ESXi if you want to expand your existing volume, and using `Manage` under `Storage Manager - Volume`, you should be able to see the expand option. From this way, you won't deal with partition table conversion, LVM resizing and all other kind of **geek** stuff. 

------

I've been using Xpenology on ESXi since my N54L upgraded with P410 RAID card. Recently, I bought two extra hard drives and would like to expand my Xpenology storage. The process wasn't so easy and took me some days to finally figure it out, so I'd like to share with you my experiences. Wish it may help you.

Mostly the steps are from [Xpenology on VMware – Disk Extend](https://www.gadjev.com/2017/06/11/xpenology-on-vmware-disk-extend/), my DSM version is `DSM 5.2-5644 Update 5`, should be the latest official stable one. Since I did RAID 1+0 from the hardware level, I don't need Xpenology to do any data protection for me, so I created my volume with RAID type `SHR (without data protection)`

After inserting and initializing these two new disks with HP RAID software, expanding datastore on ESXi, I opened my Xpenology VM settings and undoubtedly expand the disk size to a larger figure. That's where I made my mistakes. When Xpenology creates a volume from a disk, **it's using all the spaces and can not be modified later without losing your data**. In another word, if you resized your existing disk, from `Storage Manager - HDD/SSD`, you should be able to see a larger figure expected, but `Manage` button under `Storage Manager - Volume` is greyed out. Even though, if your disk created earlier is smaller than 2.0TB, you got a MBR partition table, and you should convert to GPT if you want to expand it over 2.0TB. 

**Backup your data before doing this!**

In my situation, I got an 1.8TB MBR-based disk and want to expand to 3.0TB GPT-based disk. There's the steps:

1. GPT Conversion, skip this if you already have an GPT disk. In ESXi, mount the data disk to another ubuntu machine and use `sudo gdisk /dev/sdX` (replace `sdX` with your own device name) to launch gdisk tool. Type `v` to verify first, then type `w` to convert your MBR partition table into GPT without data losses. 
2. Resize partitions. Type `sudo parted` to launch the partition editor, Type `select /dev/sdX`, then `unit s` for changing metric to sectors. After typing `print free`, you should get a brief view about your partition structure, which should be something like this:

  <script src="https://gist.github.com/EvianZhow/01779800d2d8463ff0027f5ad1ea76f9.js"></script>

  The last sector of our free space (which just extended by ESXi) is `6442450910s` in this situation. That's what we want to change into later. **Every disk varies so do not just copy and paste**. Type `resizepart`, correctly select your disk partition and enter the last sector of the free space. Resizing your disk should be done at this moment. Please take care that newer version of `parted` has removed `resizepart` function so rollback to an older version if your `parted` doesn't come with this. 
3. Power off the ubuntu machine, and power on your Xpenology VM, you should be able to see `Manage` button under `Storage Manager - Volume` isn't greyed out anymore, you can try the expand option but it will do nothing, although the system log tells you expand successfully. 
4. Prepare to resize MD devices and LVG. If you've read the references above, you should know we are going to unmount the volume before doing any resizing jobs next. Since I've installed GitLab, Plex and so many applications, it was pretty hard to kill all the process and be successfully unmounted. But I found a better way to do this. Connect to Xpenology VM using ESXi Remote Console, login with `root` account, the password should be [the same](https://www.synology.com/en-global/knowledgebase/DSM/tutorial/General/How_to_login_to_DSM_with_root_permission_via_SSH_Telnet) as `admin`. Type `syno_poweroff_task -d` and it will leaves you with a shell and everything else is closed. Data volumes should already unmounted during shutdown process. 
5. Type `vgdisplay -v` or `vgs` to identify the LVM on your system. In my situation, I've got `vg1000` and the path is `/dev/vg1000/lv`.
6. Stop and re-assemble the software RAID array by typing `mdadm -S /dev/mdX` (replace /dev/mdX with your MD device, should be able to figure out by typing `print devices` under `parted` tool) and type `mdadm -A /dev/mdX -U devicesize /dev/sdXy` (replace /dev/mdX with your MD device and /dev/sdXy with your disk partition)
7. Extend the size of the MD array by typing `mdadm –-grow /dev/mdX -z max`
7. Extend the LVM physical volume and you’ll see the free space in the physical by typing `pvresize /dev/mdX `
8. Type `vgs` to get a brief overview about your LVM volume group. There should be a lot of `VFree` left in your LVM volume if previous steps expanded successfully.
9. Activate the LVM volume group by typing `vgchange -a y vg1000`.
10. Extend the VLM Logical Volume to the last full TB or GB `lvextend -L +1.8TB /dev/vg1000/lv`.
11. Extend it with the remaining MBs as well (use `vgs` to see how much exact MBs are outstanding) by typing `lvextend -L +100MB /dev/vg1000/lv`.
12. Type `vgs` to confirm there is 0 in `VFree`, which means all free spaces have been allocated.
13. Reboot and click `Manage` button under `Storage Manager - Volume`, select the `Expand the volume with unallocated disk space`, and click `Next`. This time the expand operation should works as expected. 

### References
- https://www.gadjev.com/2017/06/11/xpenology-on-vmware-disk-extend/
- https://forum.synology.com/enu/viewtopic.php?t=83186