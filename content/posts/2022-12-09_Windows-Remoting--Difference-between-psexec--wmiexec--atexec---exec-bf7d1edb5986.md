+++
title = 'Windows Remoting: Difference between psexec, wmiexec, atexec, *exec'
date = 2022-12-09
draft = false
showtoc=true
+++

![](https://cdn-images-1.medium.com/max/800/0*2pekupqLLWtBkgpl)
*https://images.pexels.com/photos/3760778/pexels-photo-3760778.jpeg?auto=compress&amp;cs=tinysrgb&amp;w=1260&amp;h=750&amp;dpr=1*

If you’re anything like me, you discovered [Impacket](https://github.com/SecureAuthCorp/impacket), either through a course, Ippsec, or your own research, and you look at the scripts. Your grin turns into horror as you realize the sheer amount of scripts that end with “exec”. They all give you remote access but when do you use which one!? Don’t worry, I have your back. Let’s break them down.

PsExec
======

PsExec works by writing a randomly-named binary to the `ADMIN$` SMB share (hence why you require write access to that share in order to use it). The binary establishes a named pipe that is used by the SVCManager to create a new service. This named pipe can be used by the user to execute commands remotely. You can imagine the binary as executing the following command:


```
sc create [serviceName] binPath= "C:\Windows\[uploaded-binary].exe"
```
All of your command input and output occurs over the named pipe via SMB (445/TCP).

As pointed out by [Jeremy Dupuis](https://jb05s.github.io/Attacking-Windows-Lateral-Movement-with-Impacket/), PsExec leaves artifacts behind that require manual cleaning as the binary that is uploaded is not automatically removed. In fact, this is what the error logs look like after he ran a single command on PsExec before exiting.

![Picture of an error log from this article: https://jb05s.github.io/Attacking-Windows-Lateral-Movement-with-Impacket/](https://cdn-images-1.medium.com/max/800/0*Pj8MeQykWNNX4Kr_.png)
*https://jb05s.github.io/images/attacking-windows-impacket/psexec-eventlog-sys.png*

![](https://cdn-images-1.medium.com/max/800/0*8jYiFbXMIpkEf-r1.png)
*https://jb05s.github.io/images/attacking-windows-impacket/psexec-eventlog-sec.png*

As you can see, the logs showed:

* 1 System Event IDs: 7045 (Service Started)
* 12 Security Event IDs: 4672 (Special Privilege Logon), 4624 (Logon), 4634 (Logoff)

SmbExec- the next logical step
==============================

SmbExec works similarly to PsExec. The main difference is that PsExec will upload a `.exe` file to the `ADMIN$` share while SmbExec uploads a `.bin` file along with a temporary file.

If you’re interested in learning how to replicate this manually, [HackTricks](https://book.hacktricks.xyz/windows-hardening/lateral-movement/smbexec#manual-smbexec) has a section demonstrating how to do so.

Referencing the images from [Jeremy Dupuis](https://jb05s.github.io/Attacking-Windows-Lateral-Movement-with-Impacket/), we can see the log output for establishing a connection via SmbExec, executing one command, and exiting.

![](https://cdn-images-1.medium.com/max/800/0*-KV26ONRuN8dlFWy.png)
*https://jb05s.github.io/images/attacking-windows-impacket/smbexec-eventlogs.png*

![](https://cdn-images-1.medium.com/max/800/0*MsVaNUTQpbyjLkci.png)
*https://jb05s.github.io/images/attacking-windows-impacket/smbexec-eventlogs-sec.png*

The resulting logs are:

* 4 System Event IDs: 7045 (Service Started), 7009 (Service Error — Timeout)
* 3 Security Event IDs: 4672 (Special Privilege Logon), 4624 (Logon), 4634 (Logoff)

Wmiexec > Psexec?
=================

WMIexec works via Windows Management Instrumentation (WMI). WMI works by negotiating a random port (>1024) with the client over an initial connection to RCP (135/TCP). WMI and RPC are commonly used for network administration, so it is common for the ports to be open and unfiltered on an internal network.

The user sends input to the remote host over the random port. The input is executed with `cmd.exe` and the output is written to a file in the `ADMIN$` SMB share. The filename starts with `__`, followed by the timestamp.

The advantage to this method is that it allows us to execute code without writing on the disk or creating a new system. The result is a lowered chance of detection by Windows Security Essentials and Bit9, for instance.

In addition, you can utilize WMI for remote access via the program `pth-wmis` which comes preinstalled with Kali Linux.

Again, we can view the log output of a connection, executing a single command, and exiting as demonstrated by [Jeremy Dupuis](https://jb05s.github.io/Attacking-Windows-Lateral-Movement-with-Impacket/).

![](https://cdn-images-1.medium.com/max/800/0*TzXqdM8qKIHbTsMN.png)
*https://jb05s.github.io/images/attacking-windows-impacket/wmiexec-eventlogs.png*

The result is:

* 14 Security Event IDs: 4672 (Special Privilege Logon), 4624 (Logon), 4634 (Logoff)

If you’re interested in learning more about WmiExec, [this article](https://www.crowdstrike.com/blog/how-to-detect-and-prevent-impackets-wmiexec/) goes into detail about how it works on a low level and how it is detected.

AtExec
======

This program works by remotely executing scheduled tasks on a remote target through RCP. It creates a scheduled task via the Task Schedule Service. The task is executed with `cmd.exe` and the output of the command (`STDERR`and `STDERR`) is written in a temporary file in the `ADMIN$` SMB share. AtExec retrieves the value of this file before deleting it.

DcomExec
========

This program uses the [Distributed Component Object Model (DCOM)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/4a893f3d-bd29-48cd-9f43-d9777a4415b0) protocol. DCOM is a protocol that relies heavily on RPC to help software components communicate on networked computers. It has the same user interface as PsExec, and works as explained [here](https://kylemistele.medium.com/impacket-deep-dives-vol-1-command-execution-abb0144a351d):


> Dcomexec uses the MMC20 Application (which is accessible over the network with authentication) and its `ExecuteShellCommand` method to execute arbitrary commands. It also supports using the ShellWindows application and the ShellBrowserWindow applications.

TLDR;
=====

* PsExec works over SMB by uploading a `.exe` file that creates a named pipe between you and the remote host
* SmbExec works similarly, except instead of a `.exe` file, it uses a `.bin` file.
* WmiExec uses the Windows Management Instrumentation service to sent input to the host and output is written to a file in SMB.
* AtExec works through executing scheduled tasks in SMB
* DcomExec uses the DCOM protocol with RPC to execute commands
* They all look different in logs

I hope you learned something because I certainly did.

**References**
==============

* [Attacking Windows: Performing Lateral Movement with Impacket](https://jb05s.github.io/Attacking-Windows-Lateral-Movement-with-Impacket/)
* <https://www.trustedsec.com/blog/no_psexec_needed/>
* [https://book.hacktricks.xyz/windows-hardening/lateral-movement/smbexec](https://book.hacktricks.xyz/windows-hardening/lateral-movement/smbexec#manual-smbexec)
* <https://www.crowdstrike.com/blog/how-to-detect-and-prevent-impackets-wmiexec/>
* <https://kylemistele.medium.com/impacket-deep-dives-vol-1-command-execution-abb0144a351d>

Other “exec” tools that you can learn more about
------------------------------------------------

* [ScExec](https://github.com/skorov/scexec)
* [MsiExec](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/msiexec)