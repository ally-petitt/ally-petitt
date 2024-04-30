How to Create and Deploy Your Own Cloud Server with NextCloud
=============================================================

<img class="graf-image" data-height="851" data-image-id="0*-WVjY5h4EpphjjX7" data-is-featured="true" data-width="1200" src="https://cdn-images-1.medium.com/max/800/0*-WVjY5h4EpphjjX7"/>

Why Create a Cloud Server?
==========================

As many security-conscious people are aware, saving something in the cloud really means saving it on somebody else’s computer. When using cloud services, you don’t own the data that you upload, nor do you own the program that you’re using. Additionally, it is within the cloud service provider’s rights to delete your data or remove your access to it if they had technical issues, went bankrupt, or you missed a bill. Not all of them will do that, but there is no law protecting the customer from something like this happening. There aren’t any measures in place to prevent the government or corporations from looking through your data and using it for their own objectives (in the US).

Aside from data security, creating your cloud server provides the opportunity to learn more about how cloud storage works, get practice with hosting a docker container and connecting to it on your LAN, and challenge yourself to do something new.

Why NextCloud
=============

NextCloud is free and open source. They do not collect or share any of your data. For the paranoid, you can audit the [source code](https://github.com/nextcloud) yourself to verify this. The cost of a NextCloud server will be limited to the of the infrastructure that you’re using to host it.

My Setup
========

In this demonstration, I’ll be hosting the NextCloud server in a Kali Linux virtual machine. Although I’m using Kali, you can do this on any operating system that allows you to download Docker. I’ll be accessing it through other devices on my LAN.

Creating Your Docker Container
==============================

To start, if you don’t already have docker installed, you can find the release for your operating system here: <https://docs.docker.com/desktop/release-notes/>

Pull and run the NextCloud container
------------------------------------

To find the versions of NextCloud that are available to you, visit their docker website: <https://hub.docker.com/_/nextcloud>.

<img class="graf-image" data-height="638" data-image-id="1*yVUBpaz7cPnubERvyi-o7w.png" data-width="784" src="https://cdn-images-1.medium.com/max/800/1*yVUBpaz7cPnubERvyi-o7w.png"/>

In this example, I’ll be using version 24.0.11, but you can substitute with whichever version you’re using.


```
sudo docker pull nextcloud:24.0.11
```
The output of that command will look like this once it is completed:

<img class="graf-image" data-height="372" data-image-id="1*GN-cYnfs54AiOogx8-NAQw.png" data-width="644" src="https://cdn-images-1.medium.com/max/800/1*GN-cYnfs54AiOogx8-NAQw.png"/>

Next, I’ll start the container and bind it to port 80 on my Kali VM.


```
sudo docker run --name my\_cloud -d -p 80:80 nextcloud:24.0.11 
```
**Explanation:**

* `--name` will set the container name to be `my_cloud`
* `-d` will run the container in detached mode. This means that the container will run in the background of my terminal instead of displaying all the output
* `-p` allows for port-binding from port 80 of the docker container to port 80 of the machine that the container is running on so that we can access it in the browser

Now, visiting `<http://localhost:80>` in the browser of my Kali VM leads to the NextCloud web interface.

<img class="graf-image" data-height="796" data-image-id="1*09ClzbW6QlZFUKeE8VCFqw.png" data-width="1155" src="https://cdn-images-1.medium.com/max/800/1*09ClzbW6QlZFUKeE8VCFqw.png"/>

Setting up NextCloud for use
============================

Create Admin Account
--------------------

Like the initial homepage says, you can create an admin account by entering your desired credentials into the login form. Then click the install button below.

<img class="graf-image" data-height="754" data-image-id="1*NCg8BFzJsn7Q8DSYJc22gw.png" data-width="1153" src="https://cdn-images-1.medium.com/max/800/1*NCg8BFzJsn7Q8DSYJc22gw.png"/>

You will be prompted with a screen that optionally allows you to install reccomended apps.

Create User Account
-------------------

Each user of NextCloud will only be able to view their respective files. To create a user account, click on your profile picture and click `Users`.

<img class="graf-image" data-height="452" data-image-id="1*rpUr6mASFndr9leQFgxjSw.png" data-width="297" src="https://cdn-images-1.medium.com/max/800/1*rpUr6mASFndr9leQFgxjSw.png"/>

Then, click the `New user` button in the upper-righthand corner.

<img class="graf-image" data-height="758" data-image-id="1*XZ6nK22WFx20KFCEcIweNg.png" data-width="1155" src="https://cdn-images-1.medium.com/max/800/1*XZ6nK22WFx20KFCEcIweNg.png"/>

Fill in the form that appears with your desired information. and click `Add a new user`. You can verify that the user was added by logging out of your administrator account and logging in with the user credentials.

Whitelisting Domains
--------------------

Now, we’ll need to configure the NextCloud server so that we can access it from other devices.

You’ll need to know your IP address. I use the `ifconfig` command on my Kali VM to determine. Windows users can use `ipconfig`.

<img class="graf-image" data-height="86" data-image-id="1*EPZQQ90fBx0es5jESHsJQg.png" data-width="594" src="https://cdn-images-1.medium.com/max/800/1*EPZQQ90fBx0es5jESHsJQg.png"/>

I’ll install vim as a text editor so that I can edit the config files easier.


```
┌──(kali㉿kali)-[~]  
└─$ sudo docker exec -it my\_cloud bash   
root@f6d476ccd902:/var/www/html# apt update  
-- snip --  
root@f6d476ccd902:/var/www/html# apt install -y vim  
-- snip --  
root@f6d476ccd902:/var/www/html# vim config/config.php
```
In your `config/config.php` file, add the option of `trusted_domains` at the bottom, starting on line 21 (type `:set number` in vim to show line numbers). This option will allow you to connect to the NextCloud server through different domain names. Without this option, you would not be able to use your server through a domain name besides localhost, which is not ideal if you’re trying to access it remotely.


```
 'trusted\_domains' =>  
 [  
 'localhost',  
 '127.0.0.1',  
 '<YOUR IP ADDRESS>',  
 '<YOUR DOMAIN NAME (optional)>',  
 ],
```
<img class="graf-image" data-height="446" data-image-id="1*RFzQIvqVYLkE7HLgJ2eciA.png" data-width="526" src="https://cdn-images-1.medium.com/max/800/1*RFzQIvqVYLkE7HLgJ2eciA.png"/>

Now, I can use my NextCloud server from 192.168.1.53, 127.0.0.1, and localhost.

**Note:** You can optionally choose to configure your own database. The documentation for doing that can be found [here](https://docs.nextcloud.com/server/latest/admin_manual/configuration_database/linux_database_configuration.html).

Connecting From Other Devices
=============================

PC
--

To connect from you PC, just type in the IP address of your NextCloud server into your browser and type in your login credentials.

<img class="graf-image" data-height="904" data-image-id="1*YSMzF6AAy_s1ddUyTn1fRA.png" data-width="1578" src="https://cdn-images-1.medium.com/max/800/1*YSMzF6AAy_s1ddUyTn1fRA.png"/>

Mobile
------

You can also connect to the website from your phone. Alternatively, you can download the NextCloud app, available in the [App Store](https://apps.apple.com/us/app/nextcloud/id1125420102) and [Google Play Store](https://apps.apple.com/us/app/nextcloud/id1125420102). Just type the same URL into the app that you would type in the browser.

<img class="graf-image" data-height="947" data-image-id="0*rXIb7WPys_Tqgbh_" data-width="462" src="https://cdn-images-1.medium.com/max/800/0*rXIb7WPys_Tqgbh_"/>

Conclusion
==========

NextCloud allows for a tremendous amount of flexibility. There are many configurations that you can set including brute force protection, antivirus scanning, OAuth2, and more as shown in [their documentation](https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/index.html). I hope that you got value out of this article and were inspired to take control of your data.