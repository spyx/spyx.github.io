---
layout: post
title: TBBT:FunWithFlags Walkthrough
categories: [CTF, Vulnhub]
---

Hi all. I have one more walkthrough for you today. I tried TBBT: FunWithFlags. According to manual our victim machine should be on 192.168.1.105. We decide to ping it to make sure we have direct connection.

![](/images/vulnhub_tbbt/check_ip.png)

And we have connection so lets do some recon. I run nmap...

![](/images/vulnhub_tbbt/nmap.png)

and 2 ports caught my attention. Port 1337 (waste?) and port 21 (ftp). I tried connect to port 1337 via netcat.

![](/images/vulnhub_tbbt/sheldon.png)

And i got first flag. I did not expect Sheldon one....

Them i decide to look through port 21.

![](/images/vulnhub_tbbt/ftp.png)

Few notes from Sheldon and them public folders from all actors. On Howard was zip file which
i downloaded for later. Penny shared her password with everyone and Bernadette mention something about their pharmacy website. Let have a look.

Howard zip file was password protected. I used frackzip to crack this zip again rockyou.txt file.

![](/images/vulnhub_tbbt/zip_crack.png)

We got hit. Them tried to retrieve some file this that picture but it was password protected. Decide to run stegcracker to crack this password too... And Howard's flag is mine. 

![](/images/vulnhub_tbbt/howard.png)

After them i run my dirb command

![](/images/vulnhub_tbbt/dirb.png)

I cat see robots.txt and private directory to look for. Robots.txt was blind spot. I inspected private and land on pharmacy website. Lets try Penny's login and her password from FTP. It worked.

![](/images/vulnhub_tbbt/penny_login.png)

As you can see there is form. After playing aroung i figure out that form is vulnerable to SQL injection. Lunch my sqlmap dump all users from database. 

![](/images/vulnhub_tbbt/bernadette.png)

With Bernadette flag... I shifted my attention to music directory. I enumerate website via wpscan and found old plugin and some users.

![](/images/vulnhub_tbbt/wpscan.png)


After some googling i found metasploit exploit for that module. Let try..

![](/images/vulnhub_tbbt/remote_access.png)


I am in. I also did not end up on wordpress. Start breaking passwords for all 3 users. Got 2 hits and log in 
THem after looking into some posts i found raj flag too....

![](/images/vulnhub_tbbt/raj.png)


THem i shiffted to my meterpreter. I checked Penny's home folder and there was flag encrypted in base64. Quick decode and I have a flag

![](/images/vulnhub_tbbt/penny.png)


Amy's folder contain diary and secret file. You can quickly run strings commands against this program and you will get all strings... ons of them is our flag.

![](/images/vulnhub_tbbt/amy.png)


On Lenards page there was shell script. After inspecting crontab i noticed that this script run every minute with root privileges. Lets make netcat reverse shell ...

![](/images/vulnhub_tbbt/cron.png)

![](/images/vulnhub_tbbt/exploit_root.png)

and start our listener and wait to get root

![](/images/vulnhub_tbbt/root.png)


THank you all for reading... 