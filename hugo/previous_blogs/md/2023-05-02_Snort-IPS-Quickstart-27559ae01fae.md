Snort IPS Quickstart
====================

Introduction
============

Snort is an open source Intrusion Prevention System (IPS) that detects malicious network traffic by comparing the network packets to a set of rules, often created by Snort and the community. Snort can be used as a packet sniffer, packet logger, and intrusion prevention system.

In this article, I’ll go over some of the first steps of installing, configuring, and running Snort so that new users have a place to branch off of.

Quick Install
=============

You can install this on Ubuntu easily with the following command:


```
$ sudo apt-get install snort
```
Kali Linux
----------

For my Kali friends, you might get the message that you’re unable to locate the `snort` package when trying to install it. This happens because the repositories that your distribution looks into when searching for apt packages doesn’t contain `snort`. To fix this, you can try to append the following repos to your `/etc/apt/sources.list`.


```
deb http://http.kali.org/kali kali-rolling main non-free contrib  
deb http://http.kali.org/kali sana main non-free contrib  
deb http://security.kali.org/kali-security sana/updates main contrib non-free  
deb http://old.kali.org/kali moto main non-free contrib
```
Let the changes take effect and install Snort.


```
$ sudo apt-get update  
$ sudo apt-get install snort
```
Building From Source
====================

If you’re crazy enough to build Snort from the source code, this section is for you.

Installing Dependencies
-----------------------

Before you can build Snort, you must first install its dependencies. These are listed in their [README.md](https://github.com/snort3/snort3#dependencies) on GitHub, but for the sake of brevity, I’ll put some of them here. Keep in mind that some of the dependencies have more dependencies which is why some appear below and not in the documentation.


```
sudo apt update && apt install -y gcc libpcre3-dev zlib1g-dev libluajit-5.1-dev   
libpcap-dev openssl libssl-dev libnghttp2-dev libdumbnet-dev   
bison flex libdnet autoconf libtool cmake
```
DAQ
---

Snort has another depenency called DAQ that needs to be installed. I’ll be downloading their latest release form GitHub and extracting it in a folder called `daq`.


```
$ wget https://github.com/snort3/libdaq/archive/refs/tags/v3.0.11.zip  
$ unzip v3.0.11.zip -d daq && cd daq/libdaq-3.0.11
```
After downloading and extracting it, I’ll run `bootstrap` to generate the configuration script and then proceed to install it.


```
$ ./bootstrap  
$ ./configure && make && sudo make install
```
hwloc
-----

Another dependency is `hwloc`. You can find additional methods of installation on their [GitHub](https://github.com/open-mpi/hwloc) and [website](https://www.open-mpi.org/software/hwloc/v2.9/). This is the way that I did it:


```
$ git clone https://github.com/open-mpi/hwloc.git  
$ cd hwloc && ./autogen.sh  
$ ./configure && make && sudo make install
```
OpenSSL
-------

If you don’t already have `openssl` installed on your system, you could install it from source:


```
$ git clone https://github.com/openssl/openssl.git && cd openssl  
$ ./Configure && make && make test
```
It is also possible to install it through `apt`. You may still need to install some additional headers for Snort to work. For this, you can try


```
$ sudo apt install libssl-dev openssl
```
Snort Install
-------------

Finally, we can start building Snort. I’ll be building directly from their source code on GitHub. Following the instructions on their README.md in [GitHub](https://github.com/snort3/snort3#readme), I clone their GitHub repository and run these commands:


```
$ git clone https://github.com/snort3/snort3.git  
$ cd snort3
```
Then, I’ll build the program. You’ll need `cmake` among other packages in order to do this, so I’ve included the install command for those packages.


```
$ sudo apt install -y gcc cmake libpcre3-dev zlib1g-dev libluajit-5.1-dev libpcap-dev openssl libssl-dev libnghttp2-dev libdumbnet-dev bison flex autoconf libtool  
$ ./configure\_cmake.sh --prefix=$(pwd) --with-daq-libraries=/path/to/libdaq-3.0.11  
$ cd build  
$ make -j $(nproc) install
```
Configuration
=============

Capturing all Network Traffic
-----------------------------

To start, we’ll set our network adapter to run in promiscuous mode. This means that it will capture all packets on the network rather than only the ones that were assigned to be captured by it. This can be done through WiFi settings or through the command line.


```
$ sudo ip link set wlan0 promisc on
```
Modifying the Configuration File
--------------------------------

Most configurations will go within `/etc/snort/snort.conf`.


```
$ sudo vim /etc/snort/snort.conf
```
There’s many configuration options within this file that are broken up into 9 sections. Most of our changes will be in section 1.

<img class="graf-image" data-height="271" data-image-id="1*pngRmEDDXcscoXuasWQFdQ.png" data-is-featured="true" data-width="723" src="https://cdn-images-1.medium.com/max/800/1*pngRmEDDXcscoXuasWQFdQ.png"/>

On line 45 of the configuration file, we’ll change the value of `HOME_NET` from `any` to be the network that you would like to monitor. In my case, it is `192.168.1.0/24`.

<img class="graf-image" data-height="43" data-image-id="1*uPIpmb3Q2AUJj3v3GkL2lQ.png" data-width="514" src="https://cdn-images-1.medium.com/max/800/1*uPIpmb3Q2AUJj3v3GkL2lQ.png"/>

I would encourage you to go through the other variables in the configuration file to include the ports and hosts that are running various services so that Snort can detect them and apply the rules to them.

Rules are included in step #7. The syntax for rule files is `include /path/to/rule.rules`. In this case, `$RULE_PATH` refers to `/etc/snort`. You can include multiple rule files and add your own under `/etc/snort/local.rules` or whichever file name that you configure for you own rules. This structure allows you to compartmentalize various rule sets and keep them organized.

<img class="graf-image" data-height="118" data-image-id="1*Pz6IDFwxqQwrMWmZS83kQg.png" data-width="432" src="https://cdn-images-1.medium.com/max/800/1*Pz6IDFwxqQwrMWmZS83kQg.png"/>

As a side note, if you would like to download the latest community rules, you can find them at the [official website](https://www.snort.org/downloads#rules). You would extract the tarball and add the rule files to your `/etc/snort/snort.conf`.


```
$ wget https://www.snort.org/downloads/community/snort3-community-rules.tar.gz  
$ tar -xf snort3-community-rules.tar.gz
```
Running Snort
=============

Once you have your configuration file created, you can test that everything works with the following command:


```
$ sudo snort -T -i wlan0 -c /etc/snort/snort.conf
```
You will get a lot of output. The most important pieces of information to be aware of in this output are the Snort rules. Here, you will be able to see how many were loaded in.

<img class="graf-image" data-height="310" data-image-id="1*4STD2kRrAx3xnRfCA7ALqA.png" data-width="718" src="https://cdn-images-1.medium.com/max/800/1*4STD2kRrAx3xnRfCA7ALqA.png"/>

To actually run Snort as a daemon, you would change the `-T` option to `-D`.


```
$ sudo snort -D -i eth0 -c /etc/snort/snort.conf  
Spawning daemon child...  
My daemon child 197993 lives...  
Daemon parent exiting (0)
```
To verify that it is working, you can use `ps aux`.


```
$ ps aux | grep snort  
root 196973 0.0 0.3 450316 118644 ? Ssl 10:25 0:00 snort -D -i wlan0 -c /etc/snort/snort.conf
```
All alerts from Snort will be sent to `/var/log/snort/alert` unless otherwise specified in a command line argument.

Moving Forward
==============

For those who are interested in learning Snort on a deeper level, one recommendation is learning how to create your own rules and learning what the other configuration options are. It may be worth it to read their `man` page as well to be aware of options available on the CLI. You can also combine Snort with a SIEM such as Splunk and other solutions like pfSense.