+++
title = 'Restoring Files in /usr/include'
date = 2023-04-13
draft = false
showtoc=true
tags= ["Troubleshooting", "Tutorial"]
+++

Hi guys, I made a mistake. In my frustration trying to debug my C program, I inadvertently deleted all the files within my `/usr/include` folder. I didn’t realize at the time that this was a very important folder! As explained [here](https://www.kernel.org/doc/Documentation/kbuild/headers_install.txt), it stores the Linux kernel’s libc header files! Rookie mistake, but luckily for us, there’s ways to fix it.

If your `/usr/include` folder is also looking more empty than the shelves during COVID, I come bearing the solution.

Getting Kernel Headers
======================

You’ll want to download the kernel install from here: <https://www.kernel.org/>. Choose the one that matches your Linux kernel version as close as possible. I’ll be using 6.2.10. From there, we’ll decompress the file and copy the contents of the include folder to `/usr/include`.


```
# download the linux kernel files form kernel.org  
wget https://cdn.kernel.org/pub/linux/kernel/v6.x/linux-6.2.10.tar.xz  
tar -xf linux-6.2.10.tar.xz # decompress the tarball  
# copy the contents of the include folder into /usr/include/  
sudo cp -r ./linux-6.2.10.tar.xz/include/ /usr/include/
```
Restoring Additional Missing Files
==================================

If you find that while compiling something, you still get errors, try this:


```
apt-file search /path/to/<MISSING\_HEADER>.h
```
In my case, I was missing `string.h`, so I used `apt-file search /usr/include/string.h` to see that I was needing the `libc6-dev` library.

![](https://cdn-images-1.medium.com/max/800/1*fAw86hUT7orMXK8ZVFfZHQ.png)
I had this installed previously, but since I deleted the files, I need to remove the installation completely and reinstall it.


```
sudo apt remove --purge libc6-dev  
sudo apt install libc6-dev
```
I also had to do the same with `linux-libc-dev`:


```
sudo apt remove --purge linux-libc-dev  
sudo apt install linux-libc-dev
```
Continue on with this methodology until you’ve installed all your missing dependencies. With these two steps combined, you should be able to restore your `/usr/include` file. This worked for me. Best of luck!