---
layout: post
title: Escalate My Privilege Walkthrough
categories: [CTF, Vulnhub]
---

HI all. I have another walkthrough fro you. When i lunch this VM they provide me with IP address.

![](/images/vulnhub_secalateMe/victim.png)

I quickly inspected webpage but nothing interesting was there. I run dirb busted to looking for some clues. 

![](/images/vulnhub_secalateMe/02_dirb.png)

Dirb find robots.txt file on server. After inspecting it there was link to one php. I inspected it and I had shell access. As i rather prefer install terminal i quickly upload php shell to root of webserver and start listener on my machine.

![](/images/vulnhub_secalateMe/03_uploadShell.png)

Quickly move to folder to look for users. I found armour user there. He has txt file in his home folder named Credentials.txt. THere was clue to his passcode.


![](/images/vulnhub_secalateMe/04_userPass.png)

Re-create his password witch echo and md5sum commands. And give it try. And We are in.

![](/images/vulnhub_secalateMe/05_armour.png)

After running ***sudo -l*** command. And we have lots of options to get root access.

![](/images/vulnhub_secalateMe/06_sudo.png)

I tried nmap and that worked. 

![](/images/vulnhub_secalateMe/07_root.png)

THank you all for reading....