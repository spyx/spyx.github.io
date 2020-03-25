---
layout: post
title: Recon Walkthrough
categories: [CTF, Vulnhub]
---

Hi all. I have another walkthrough for you. Let do some recon :bowtie:

![](/images/recon/masscan -p 80 192.168.1.0/24)

Same thing all over. Identify your target and start scanning and enumeration.

![](/images/recon/nmap.png)

We have open port 22(ssh) and 80(http). Lets run dirb to figure find some secrets. Second output tell me that this server is probably wordpress installed. We will lunch wpscan with enumarations options...

```
wpscan <address> -e at -e ap -e u
```
Option **-e** means enumerate ***at*** for all themes, ***ap*** for all plugins and ***u*** is for users. And we find 2 users/targets

![](/images/recon/wp_users.png)

Lets use rocky list to see if we can get any hit. 

```
wpscan <address> -U recon,reconauthor -P /use/share/wordlist/rockyou.txt
```

If you assign more memory for your virtual machines less free time you have. In my case i had some time to make myself of cups of coffee. :coffee::coffee::coffee:

![](/images/recon/wp_user.png)

And we got hit. I tried some try with metaspploit wp_admin_upload module but that did not work. I return to check all recon i did. 
In dirb ... wp-content/uploads was listable. Quick look there and notice folder name articulate_upload. Look into some exploits and find this [one](https://www.exploit-db.com/exploits/46981). Basically you can make zip file,upload it to server and get reverse shell. I had to change script as cmd trick didn't work for me. Also I tried some oneliner php shells but with same results.  So i download pentestmonkey's php [shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell). We changed only IP address. And navigate to directory where your php shell was uploaded. And lunch it. Also do not forget run netcat on your machine

![](/images/recon/shell.png)

And we have shell. Quickly run my sudo command 

![](/images/recon/gdb.png)

Well as we can see we potentionally can get access as offensivehack

```
sudo -u offensivehack /usb/bin/gdb -nx -ex '!sh' -ex quit
```

And we are offensivehack now. :sunglasses:

![](/images/recon/user.png)

After looking for some clues and i tried ***groups*** command. And i noticed that this user is in docker group? Is there is any docker image that I can use?  [Let's see](https://hub.docker.com/r/chrisfosterelli/rootplease/)...

![](/images/recon/root.png)

That's nice :sunglasses:

I hope you like this walkthrough. Hit me on twitter if you have any questions

