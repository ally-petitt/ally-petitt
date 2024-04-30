+++
title = 'How to Encrypt a Drive in Linux'
date = 2023-05-05
draft = false
showtoc=true
tags= ["Linux", "Drive Encryption", "Tutorial"]
+++

Introduction
------------

Hey everyone, this is a pretty quick article on LUKS drive encryption on Linux with the `cryptsetup` library. By following the steps outlined here, you will be able to encrypt a drive, decrypt it, and mount it. This was done in a Kali Linux VM and commands may vary for other distributions.

Disclaimer: This is not an area that I have much experience in so if details are inaccurate, I apologize in advance.

Creating a new partition
------------------------

I’m using a virtual machine with 2 virtual hard disks.

![](https://cdn-images-1.medium.com/max/800/1*ao7i4G4xZFiEXNb2nX2-bQ.png)
*Demonstrating my disks with “lsblk -e7”*

I’ll be using a tool called `parted` to create a partition on `/dev/sdb`. Historically, when a system uses the Master Boot Record (MBR) partition table, `fdisk` is used to manage the partition. In this article, I’ll use `parted` because of its usability in scripting and automation.

I’ll start by listing the information of the drives.


```
parted -s /dev/sdb print all
```
![](https://cdn-images-1.medium.com/max/800/1*5w3JBHztlFmtojmfpSwqAg.png)
The output `msdos` under `/dev/sda`, indicates that `sda` is using the MBR partition table. Learn more about partition table types [here](https://en.wikipedia.org/wiki/Disk_partitioning) and [here](https://wiki.archlinux.org/title/Partitioning).

I’ll now create the GPT partition table on `/dev/sdb`. You’ll notice that after running the command, `/dev/sdb` now appears as a `gpt` partition table.

![](https://cdn-images-1.medium.com/max/800/1*tZM4UvgCMQXekmpmoDJumg.png)
I then create a partition on the disk with the ext4 file system.

![](https://cdn-images-1.medium.com/max/800/1*dbuTKtvJcQZI4cFNANRylQ.png)
The commands I used for this are below. The values for the start and ending offsets when creating the partitions can be expressed in both percentages and exact byte values. Reference the [man page](https://linux.die.net/man/8/parted) for more details.


```
parted -s /dev/sdb mklabel gpt  
parted -s /dev/sdb mkpart primary ext4 0% 50MiB
```
Encrypting the Drive
--------------------

This demonstrates how to encrypt file system using LUKS. It is important to remember the passphrase that you enter while encrypting the partition because it is a key piece of information when decrypting the drive.


```
cryptsetup luksFormat /dev/sdb # encrypt /dev/sdb with luks
```
Decrypting the Drive
--------------------

You’ll need to do this before you’re able to mount and use the partition on the drive.


```
cryptsetup open /dev/sdb encrypted # open the encrypted drive as /dev/mapper/encrypted  
mkfs.ext4 /dev/mapper/encrypted # create a filesystem on the device (only needed the first time you open the encrypted drive)
```
Mounting the Partition
----------------------

In order to actually use the partition and the file system on it, we must mount it.


```
mkdir -p /mnt/encrypted # prepare the mount point  
mount /dev/mapper/encrypted /mnt/encrypted # mount the decrypted filesystem on /mnt/encrypted
```
Unmounting the Partition
------------------------

When you’re done using the drive, you can unmount it.


```
umount /mnt/encrypted 
```
Closing and Re-encrypting the Partition
---------------------------------------

In its unmounted state, it is still decrypted. To re-encrypt and close the drive, you can run the following command:


```
cryptsetup close /dev/mapper/encrypted
```
Digging Deeper
==============

Because I’m a curious person, I’ll share with you some commands that you can use to get more information on the file system that you just created and other findings that I thought were interesting.

File System Metadata
--------------------

After decrypting the drive, you can view the metadata of your filesystem.


```
cryptsetup --type luks open /dev/sdb encrypted
```
These are some commands that will give you infromation.


```
df -hT /dev/mapper/encrypted  
tune2fs -l /dev/sdb
```
As shown in the screenshots, you’re able to view the number of inodes, the block count, block size, filesystem magic number, and much more.

![](https://cdn-images-1.medium.com/max/800/1*k6QJaeh0pBMsxnfbGk6Rog.png)
![](https://cdn-images-1.medium.com/max/800/1*BXNCkBoJaesaE0AfBZlOsQ.png)
Inodes
------

[Inodes](https://en.wikipedia.org/wiki/Inode) are a data structure that contain information about files in the Linux filesystem. It contains metadata such as the block number that the file is located in on the hard drive, permissions, and file owner. In an ext4 filesystem, the number of inodes in is fixed, whereas in XFS and JFS, the number of inodes is dynamic. The result is that in ext4 filesystems where many inodes are used, such as in situations where many directories, symbolic links, and/or small files are made, an error message that the system is out of space may occur when there is plenty of space left. The reason for this is simply that the filesystem has no more available inodes to assign to new files. This is a relatively common occurence for mail servers that often hold many small files.

The number of inodes on your system’s filesystems can be viewed with `df -hi`.

![](https://cdn-images-1.medium.com/max/800/1*_TgsDD7BK_fPitV0J31ywg.png)
You can query the inode information on a specific file with the following command:


```
ls -il file
```
![](https://cdn-images-1.medium.com/max/800/1*nBvEce088HNvxUbcz1cF7w.png)
The inode number is on the leftmost column of output. In this case, it is `131079`. Additionally, you can see the read, write, and execute permissions on the file with the owner and group associated with it.

As an alternative, you can search for the file that is associated with a specific inode with this command:


```
find / -inum 1234567 -ls
```
![](https://cdn-images-1.medium.com/max/800/1*15DlndsZGVZ0Q9A3lAtXeg.png)
I was able to find the file `/tmp/test.txt` that was associated with `131079`.

Duplicating the Encrypted Drive
-------------------------------

This command can be used to create an exact duplicate of `/dev/sdb` in `/media/sdb.img`.


```
dd if=/dev/sdb of=/media/sdb.img
```
![](https://cdn-images-1.medium.com/max/800/1*hzoKv4rgFMHTAQtFFIcaYQ.png)
Next Steps
==========

Moving forward, you can expand upon what was done in this article by implementing an added layer of abstraction and flexibility with [LVM](https://linuxconfig.org/linux-lvm-logical-volume-manager). You can also continue to experiment with different ways of encrypting partitions such as those outlined in [this article](https://www.baeldung.com/linux/encrypt-partition). There are many different ways to configure your system and I would encourage you to continue learning. Thank you for reading!