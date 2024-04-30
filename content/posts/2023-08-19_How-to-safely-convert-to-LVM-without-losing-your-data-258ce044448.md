+++
title = 'How to safely convert to LVM without losing your data'
date = 2023-08-19
draft = false
showtoc=true
+++

![](https://cdn-images-1.medium.com/max/800/0*es1LJrnEBsSNgXZM.png)
Introduction
============

This article is a walkthrough that demonstrates the solution to a particular situation that computer owners may encounter when updating their system. For readers who do not fit into the scenario listed below, this is also a great article for familiarizing yourself with the practical application of logical volume manager (LVM). Otherwise, feel free to modify your approach as works best with your scenario.

**Scenario:** You have a hard drive with all your files on it that uses physical partitions. You just bought a new hard drive and would like to use both drives together to manage logical partitions rather than physical ones. You also don’t want to lose the data that was on your original physical partitions.

As implied from above, the only requirements for this approach are that you have a system that can use LVM and you have a storage device with equal or greater size than the amount of data stored on the primary storage device. Additionally, it is assumed that you are using Linux.

**Warning:** This has the potential to cause damage and potentially brick your system. Your implementation may also vary depending on your setup. Continue at your own risk.

Benefits of LVM
---------------

The full scope of LVMs capabilities can be seen [here](https://man7.org/linux/man-pages/man8/lvm.8.html). It is a flexible utility that offers unique advantages over traditional physical partitions:

* Grouping multiple drives into a single volume group
* Support for [thin provisioning](https://en.wikipedia.org/wiki/Thin_provisioning)
* Snapshot capabilities that can be used for backups
* Easy to resize partitions and add/remove drives

More detail on the benefits can be found [here](https://linuxhint.com/whatis_logical_volume_management/). We are now ready to discuss the steps to convert to LVM.

Steps to Safely Convert to LVM
==============================

For the purpose of this article, the primary storage device used will be referred to as `sda` and the new storage device will be called `sdb`. An overview of the plan is the following:

1. Create LVM group on `sdb`
2. Copy data from `sda`partitions to `sdb` logical partitions
3. Expand LVM group on `sdb` to include unused `sda` partitions

Prerequisite
------------

Install the LVM tools and `rsync` with the command that is appropriate with your Linux distribution.


```
sudo apt-get install lvm2 rsync # For Ubuntu/Debian  
sudo pacman -Sy lvm2 rsync # Arch Linux
```
1. Create LVM group on `sdb`
----------------------------

Create a volume group called `myvg` on `sdb`.


```
$ sudo pvcreate /dev/sdb  
$ sudo vgcreate myvg /dev/sdb
```
2. Create a logical volume
--------------------------

Create a logical volume on the `myvg` volume group to store the new data in. This one is named `backup`.


```
$ sudo lvcreate -n backup -L 300G myvg
```
To verify your progress, you can run the commands `sudo vgs` and `sudo lvs` to list the recently created volume groups and logical volumes.

3. Mount the new logical volume
-------------------------------

Create a filesystem. For quick creation, just press the Enter key for each of the prompts.


```
sudo mkfs.ext4 /dev/myvg/backup
```
Then, create a mount point and mount the `backup` logical volume to it.


```
sudo mkdir /mnt/backup  
sudo mount /dev/myvg/backup /mnt/backup
```
4. Copy data to the logical volume
----------------------------------

Then, mount the partition that you would like to transfer onto a new mount point. For instance, I would like to transfer `sda3` onto `backup` so I will use the following command:


```
sudo mount /dev/sda3 /mnt/root
```
In this case, `rsync` will be used to transfer the files from `sda` to `sdb`. In this case, the 500GB portion of `sdb` that was used to create the `backup` logical volume. After, I will unmount the logical volume and physical partition.


```
sudo rsync -av /mnt/root /mnt/backup  
sudo umount /mnt/backup  
sudo umount /dev/root
```
**Repeat steps 2–4 with all of the partitions that you with to preserve.**

5. Expand the LVM volumes
-------------------------

Now, we will add `sda2` and `sda3` into the volume group of LVM. In order to do this, we must initialize them as physical volumes and extend them to the volume group.


```
sudo pvcreate /dev/sdY  
sudo vgextend myvg /dev/sdY
```
5. Add new partitions to fstab
------------------------------

Finally, find the UUIDs of the new logical volumes with `lsblk -f` and add those to your `/etc/fstab` with the following syntax:


```
UUID=<your-home-uuid> /home ext4 rw,relatime 0 2  
UUID=<your-root-uuid> / ext4 rw,relatime 0 1
```
As always, make changes to the above configuration option if necessary.

Conclusion
==========

These are the steps that worked for me with my setup. I hope that you found this article helpful and informative. Feel free to reach out if you have any questions.