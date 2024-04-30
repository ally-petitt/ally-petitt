VPN Troubleshooting: How to fix “Inactivity Timeout ( — ping-restart)”
======================================================================

If your VPN log looks something like this:

<img class="graf-image" data-height="162" data-image-id="1*1SN24_SvT2cN2X0GeX-DUA.png" data-width="809" src="https://cdn-images-1.medium.com/max/800/1*1SN24_SvT2cN2X0GeX-DUA.png"/>

I’m here to help. During my time working through the PEN-200 labs, I’ve faced the constant struggle of losing connection to the host every few minutes to seconds. I tried to troubleshoot this “Inactivity Timeout” error with an Offsec employee for 3 hours to no avail. Finally, I figured out the solution and I am here to share it with those of you who have the same struggle.

Cause of This Error
===================

In a successful OpenVPN connection, the VPN server will send pings to the client to ensure that the VPN connection is still active. If the ping is not received by the client, the server knows that the VPN is disconnected and attempts to reset the connection. This is when the message “Inactivity timeout ( — ping-restart)” appears in the VPN log.

<img alt="VPN server with arrow titled “ping” pointing towards VPN client" class="graf-image" data-height="423" data-image-id="1*GCaxAU5pr5MvU-aF5ujO7w.png" data-is-featured="true" data-width="837" src="https://cdn-images-1.medium.com/max/800/1*GCaxAU5pr5MvU-aF5ujO7w.png"/>

Reason #1: Firewalls
====================

A potential cause for this error, which is often overlooked, is a running **firewall**. This was the primary reason that I was seeing the errors as I was unaware that my firewall was interfering with the VPN connection, and I had another firewall that I didn’t know was running. Linux users, you can check the firewall status with the following commands:


```
sudo systemctl status ufw  
sudo systemctl status firewalld
```
If the firewall is running, it is possible that this is the reason you are having VPN issues. To fix this, either **disable the firewall or change its configuration to not interfere with your VPN connection.**

Reason #2: Multiple VPN connections at once
===========================================

Another possible cause of this error is when running two `openvpn` clients with the same profile from different computers. Make sure that you’re not running the VPN in your virtual machine and host OS at the same time. If your setup looks something like this:

<img alt="Host computer with openvpn running with virtual machine that also have openvpn running" class="graf-image" data-height="1080" data-image-id="1*c25ACGc0dibyMqgCdBy9Xw.png" data-width="1624" src="https://cdn-images-1.medium.com/max/800/1*c25ACGc0dibyMqgCdBy9Xw.png"/>

There’s a chance that this is the reason for your troubles. To solve this, **pick one VM to work in and kill any of your other VPN connections**. You can use this command in the terminal to ensure that `openvpn` is no longer running.


```
killall -e openvpn
```
Reason #3: DNS Name Resolution
==============================

Additionally, if running `ping [www.google.com](http://www.google.como)` results in the message:

<img alt="Picture showing error message “ping: www.google.com: Name or service not known”" class="graf-image" data-height="39" data-image-id="1*fGnBltujryQ_p2CLOjUAiw.png" data-width="341" src="https://cdn-images-1.medium.com/max/800/1*fGnBltujryQ_p2CLOjUAiw.png"/>

You probably need to **add a working nameserver to your** `**/etc/resolv.conf**` **file**. For example, your `/etc/resolve.con` may look like this once you add the working nameservers:

<img class="graf-image" data-height="108" data-image-id="1*QygUgBksC3lNe-XBgKPG9A.png" data-width="557" src="https://cdn-images-1.medium.com/max/800/1*QygUgBksC3lNe-XBgKPG9A.png"/>

Note: `1.1.1.1` is the public DNS resolver from Cloudflare and `8.8.8.8` is from Google. For more details regarding configuration, look at [this article](https://www.shellhacks.com/setup-dns-resolution-resolvconf-example/).

Reasons #4–12
=============

These are additional reasons that I found in [this article](https://www.sparklabs.com/support/kb/article/error-inactivity-timeout-ping-restart/).

* Address or port of VPN server is incorrect
* VPN server is offline
* Network between your computer and the server dropped out
* The ping and ping-restart values are invalid or don’t match
* Configuration problems
* Your country may attempt to take down your VPN connection for censorship purposes
* NAT router that blocks your connection
* Firewall on your router

Additional Reading
==================

* <https://www.sparklabs.com/support/kb/article/error-inactivity-timeout-ping-restart/>