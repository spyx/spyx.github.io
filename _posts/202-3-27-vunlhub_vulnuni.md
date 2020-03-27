---
layout: post
title: Vulnuni Walkthrough
categories: [CTF, Vulnhub]
---

HI all. Another day with another walkthrough. I started with masscan to identify my target.

![](/images/vulnhub_vulnuni/masscan.png)

My target is on 192.168.1.72 address. Your will be slightly different. Right after that I lunched nmap and dirb to do same basic reconnaissance ...

Until scans are done. I start look into port 80 to see if I can find any clues. 
After little poking around I found hidden comment.

![](/images/vulnhub_vulnuni/comment.png)

There is some hidden platform. ***"Till new version is installed"*** hmmm maybe another clue.

I landed on login page dome some greek education portal. 

![](/images/vulnhub_vulnuni/login_page.png)

In comments was mention until new version is installed? Lets ask Google.... I found 2 exploits... [exploit-1](https://www.exploit-db.com/exploits/48163) or [exlpoit_2](https://www.exploit-db.com/exploits/48106). First one require authenticated user but second one is more interesting. There is SQL injection for login page. Tried second exploit it worked...

![](/images/vulnhub_vulnuni/sqlmap_db.png)

After poking with sqlmap i downloaded all usernames and passwords. Log in and noticed that I can upload files. Lets upload some php :bowtie: My first attempt did not work as system added ***s*** to end of my php. I quickly look into second exploit as now I am authenticate user now :grin: In second exploit there was option to bypass my upload problem. Just remove .php to .php3. 

![](/images/vulnhub_vulnuni/upload.png)

as my shell is uploaded. Spin netcat listener and visit file where is uploaded.

![](/images/vulnhub_vulnuni/url.png)

Quickly look into my listener....

![](/images/vulnhub_vulnuni/shell.png)

I have shell. I tried lots of different tricks to acquire root access. Them i tried some exploits from [exploit-db](https://www.exploit-db.com/exploits/40616
) and one of this was right hit. 

![](/images/vulnhub_vulnuni/root.png)

I hope you enjoyed this walkthrough