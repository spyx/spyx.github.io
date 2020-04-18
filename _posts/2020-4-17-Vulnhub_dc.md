---
layout: post
title: Vulnhub DC:1
categories: [CTF, Vulnhub]
---

Hi All. Another walkthrough for you. Now i decide to scan to DC:1 machine. After quick nmap scan we noticed that this server is running probably older version of drupal. 

![](/images/vulnhub_dc/01_nmap.png)

As i know that this is older machine. I turn my hope to metasploit. Quickly lunch msfconsole and search for drupal. This exploit was the newest one with excellent rank. 

![](/images/vulnhub_dc/02_msfconsole.png)

I know there is some Form API thingy but lets hope. Set up victim ip address and run exploit. 

![](/images/vulnhub_dc/03_meterpreter.png)

I got shell. :grinning: Them shift my focus to get root account. sudo command did not work but i was able to find some sticky bit on find command. Find command enable you to run shell from it. 

![](/images/vulnhub_dc/04_root.png)

Thats it I hope you enjoy it

Cheers 
Spyx