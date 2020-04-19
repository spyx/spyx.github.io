---
layout: post
title: InfoSecWarrior CTF 2020:01
categories: [CTF, Vulnhub]
---

Hi all. Lets quickly dive in. First we indentify our target ip address.
Us usually i use masscan

![](/images/vulnhub_infosec1/01_masscan.png)

As we can see our target in on 192.168.1.67. Yours will be probably different. 

After looking for website and use dirb. It found this sitemap.xml address. When we inspected it. There was index.htnl address. Did you noticed typo? 

![](/images/vulnhub_infosec1/02_xml)

We found some pictures. Look into source code and found hidden form. 

![](/images/vulnhub_infosec1/03_hiddenCmd.png)

I try to connect to this cmd.php but i got another hint. I lunched my burp and intercept traffic. Send it to repeater and change GET to POST and assing proper value on body of request. 

![](/images/vulnhub_infosec1/05_cmdPost.png)

Them i tried to upload shell but that did not work. I tried to read content of cmd.php and found first user.

![](/images/vulnhub_infosec1/06_user.png)

Tried SSH and i was in. And have my flag. Them i start looking to get root access. Run sudo -l command and get some result. I noticed that rpm can execute shell with sudo priviledges. Give it try and ...

![](/images/vulnhub_infosec1/08_root.png)

Got root access

I hope you like it and enjoy it
See you soon
Marek



