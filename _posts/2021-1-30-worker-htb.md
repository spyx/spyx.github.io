---
layout: post
title: HTB Worker Walkthrough
categories: [CTF, HTB]
---

This is walkthroug of retired machine on HTB - Worker

![](/images/worker-htb/logo.png)

As first I ran some default nmap scan and them I scanned for all TCP ports

![](/images/worker-htb/01_namp.png)

Only 3 ports open:
* HTTP on port 80
* SVN service on port 3690
* WinRM on port 5985

When i checked web port it was jsut default IIS page.

![](/images/worker-htb/02_default-website.png)

I enumerate snv with nmap scrips 

```bash
nmap -p 3690 --script svn-brute --script-args svn-brute.repo=/svn/ 10.10.10.203
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-30 01:39 EST
Nmap scan report for devops.worker.htb (10.10.10.203)
Host is up (0.16s latency).

PORT     STATE SERVICE
3690/tcp open  svn
| svn-brute:   
|_  Anonymous SVN detected, no authentication needed

Nmap done: 1 IP address (1 host up) scanned in 1.04 seconds
```

Bingo. I can get repository from svn as no authnetication is needed

```bash
svn checkout svn://10.10.10.203
```

There is a website for dimension.worker.htb and one file called moved.txt

![](/images/worker-htb/03_moved.png)

Repository was moved to devops.worker.htb. I update my /etc/hosts file for this subdomain. But this repository require authentication

![](/images/worker-htb/04_login-req.png)

As i did not have any credential i decide to return to svn as there will be posibility that commints before contains any credentials.Svn checkout has option -r to pull commit before

![](/images/worker-htb/05_creds-devops.png)

Bingo i got some credentials. As WinRM is enabled I tried to log into system but no luck. THem i tried this creds on devops site and I was in!

Site contain some websites i picked twenty.worker.htb. On this repository I created new branch as it not allow my pull request in master one

![](/images/worker-htb/06_branch.png)

I copies shell from local system. As system use windows i decide to go with base aspx shell. Under new branch i uploaded my shell.aspx

![](/images/worker-htb/08_upload-shell.png)

THem pull request and approve it. I did nt get any error message so i edited my /etc/hosts file and visit twenty.worker.htb/cmd.aspx

![](/images/worker-htb/11_webshell.png)

Quickly run whoami and systeminfo. I was running as web service and system was windows server 2019. As its new system i decide to run nishang powershell script to get proper reverse shell

![](/images/worker-htb/12_foothold.png)

I was in! Move to public folder and run winpeas.exe. Winpeas tell me there is W drive there. Start looking for some clues and find passwd file

![](/images/worker-htb/13_svnusers.png)

File contain password and usernames in clear text. One user match one in C:\users folder. 

![](/images/worker-htb/14_user.png)

Tried evil-winrm and I was in as user robisl. I start looking for some options and re-run winpeas maybe i missed something. Them i tried to log as robisl to devops.worker.htb and there was another repo. THis repo allow run pipeline for application. As I was able to run anytihing why not add rohisl as administrator.

![](/images/worker-htb/16_pipeline.png)

It worked and after relunch my evil-winrm sesison i was administrator on box

![](/images/worker-htb/17_admin.png)

THank you reading 




