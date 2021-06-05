---
layout: post
title: Volatility 101 or Root-Me C&C Challenge
categories: [forensics, volatility, root-me]
---

# Intro

Volatility is an open-source application for analyzing RAM. As I was tried to learn more about this tool and more memory analysis I end up on  [root-me challenges](https://www.root-me.org/en/Challenges/Forensic/)

Root-me provide Command & Control challenges for memory analysis. Lets dive into it.

# Install

As first I need to install volatility. Last stable version is from 2016 and it's build on python. You can download executable from [here](https://www.volatilityfoundation.org/releases). I decided to not install it this way, but I used [docker](https://hub.docker.com/r/phocean/volatility) image instead. For set up volatility you simple create function and them you call this function which parse all arguments as will. 

Set up function
```bash
function volatility() {
  docker run --rm --user=$(id -u):$(id -g) -v "$(pwd)":/dumps:Z -ti phocean/volatility $@
}
```

Example of running
```bash
volatility -f /dumps/dump.vmem imageinfo
```

# Command and Control 2 Challenge

Our task is to find computer name. First we need to figure you profiles. We can do it with **imageinfo** plugin. 

```bash
root@tiny:/# volatility -f /dumps/ch2.dmp imageinfo
Volatility Foundation Volatility Framework 2.6.1
INFO    : volatility.debug    : Determining profile based on KDBG search...
          Suggested Profile(s) : Win7SP1x86_23418, Win7SP0x86, Win7SP1x86_24000, Win7SP1x86
                     AS Layer1 : IA32PagedMemoryPae (Kernel AS)
                     AS Layer2 : FileAddressSpace (/dumps/ch2.dmp)
                      PAE type : PAE
                           DTB : 0x185000L
                          KDBG : 0x82929be8L
          Number of Processors : 1
     Image Type (Service Pack) : 0
                KPCR for CPU 0 : 0x8292ac00L
             KUSER_SHARED_DATA : 0xffdf0000L
           Image date and time : 2013-01-12 16:59:18 UTC+0000
     Image local date and time : 2013-01-12 17:59:18 +0100
```

In forth line I see suggested profile. I can assume this RAM image was captured on Windows 7 machine. Computer name is stored in registry key. I can pull them from 2 locations. 


```bash
root@tiny:/# volatility -f /dumps/ch2.dmp --profile=Win7SP1x86_23418 printkey -K 'ControlSet001\Control\ComputerName\ActiveComputerName'
Volatility Foundation Volatility Framework 2.6.1
Legend: (S) = Stable   (V) = Volatile

----------------------------
Registry: \REGISTRY\MACHINE\SYSTEM
Key name: ActiveComputerName (V)
Last updated: 2013-01-12 16:38:14 UTC+0000

Subkeys:

Values:
REG_SZ        ComputerName    : (V) WIN-ET**redacted**

```

or 

```bash
root@tiny:/# volatility -f /dumps/ch2.dmp --profile=Win7SP1x86_23418 printkey -K 'ControlSet001\Control\ComputerName\ComputerName'
Volatility Foundation Volatility Framework 2.6.1
Legend: (S) = Stable   (V) = Volatile

----------------------------
Registry: \REGISTRY\MACHINE\SYSTEM
Key name: ComputerName (S)
Last updated: 2013-01-12 00:58:30 UTC+0000

Subkeys:

Values:
REG_SZ                        : (S) mnmsrvc
REG_SZ        ComputerName    : (S) WIN-ET**redacted**
```

I decide to redacted answer so it will ont be easy to just copy and paste.


# Command and Control 3 Challenge

In this challenge I  have to find some file that seems suspicious. I look for **pslist** which list all processes. We did not see anything unusual. Attacker usually try to kept similar name to avoid detection. After trying few plugins **cmdline** give me some interesting stuff

```bash
root@tiny:/# volatility -f /dumps/ch2.dmp --profile=Win7SP1x86_23418 cmdline
```

```bash
************************************************************************
iexplore.exe pid:   2772
Command line : "C:\Users\John Doe\AppData\Roaming\Microsoft\Internet Explorer\Quick Launch\iexplore.exe"  
************************************************************************
```
 I can see that iexplorer.exe was lunched from John Doe user. Try to get MD5 from path of this process and get another flag. 


# Command and Control 4 Challenge

Now I have to find IP address and port.

i decide to run plugin **pstree** which print presses list as tree

```
root@tiny:/dumps# volatility -f /dumps/ch2.dmp --profile=Win7SP1x86_23418 pstree
Volatility Foundation Volatility Framework 2.6.1
Name                                                  Pid   PPid   Thds   Hnds Time
-------------------------------------------------- ------ ------ ------ ------ ----
** clear **
 0x87ac6030:explorer.exe                             2548   2484     24    766 2013-01-12 16:40:27 UTC+0000
. 0x87b6b030:iexplore.exe                            2772   2548      2     74 2013-01-12 16:40:34 UTC+0000
.. 0x89898030:cmd.exe                                1616   2772      2    101 2013-01-12 16:55:49 UTC+0000
. 0x95495c18:taskmgr.exe                             1232   2548      6    116 2013-01-12 16:42:29 UTC+0000
. 0x87bf7030:cmd.exe                                 3152   2548      1     23 2013-01-12 16:44:50 UTC+0000
.. 0x87cbfd40:winpmem-1.3.1.                         3144   3152      1     23 2013-01-12 16:59:17 UTC+0000
. 0x898fe8c0:StikyNot.exe                            2744   2548      8    135 2013-01-12 16:40:32 UTC+0000
. 0x87b784b0:AvastUI.exe                             2720   2548     14    220 2013-01-12 16:40:31 UTC+0000
. 0x87b82438:VMwareTray.exe                          2660   2548      5     80 2013-01-12 16:40:29 UTC+0000
. 0x87c6a2a0:swriter.exe                             3452   2548      1     19 2013-01-12 16:41:01 UTC+0000
.. 0x87ba4030:soffice.exe                            3512   3452      1     28 2013-01-12 16:41:03 UTC+0000
... 0x87b8ca58:soffice.bin                           3564   3512     12    400 2013-01-12 16:41:05 UTC+0000
. 0x9549f678:iexplore.exe                            1136   2548     18    454 2013-01-12 16:57:44 UTC+0000
.. 0x87d4d338:iexplore.exe                           3044   1136     37    937 2013-01-12 16:57:46 UTC+0000
. 0x87aa9220:VMwareUser.exe                          2676   2548      8    190 2013-01-12 16:40:30 UTC+0000
 0x95483d18:soffice.bin                              3556   3544      0 ------ 2013-01-12 16:41:05 UTC+0000
```
I had to clear output to make to more visible. As I can see our iexplore.exe open another process cmd.exe with PID 1616. Now lets look into network connections for these processes. I can use **netstan** plugin for that. 

```bash
root@tiny:/# volatility -f /dumps/ch2.dmp --profile=Win7SP1x86_23418 netscan | grep 2772
0x1dedb4f8         TCPv4    127.0.0.1:49178                127.0.0.1:12080      ESTABLISHED      2772     iexplore.exe   
root@tiny:/# volatility -f /dumps/ch2.dmp --profile=Win7SP1x86_23418 netscan | grep 1616
```

First process give me localhost ip. This look more like tunnel for me. I tried **consoles** plugin to extract command line history.

```bash
root@tiny:/# volatility -f /dumps/ch2.dmp --profile=Win7SP1x86_23418 consoles
Volatility Foundation Volatility Framework 2.6.1
**************************************************
**clear**
**************************************************
ConsoleProcess: conhost.exe Pid: 2168
Console: 0x1081c0 CommandHistorySize: 50
HistoryBufferCount: 3 HistoryBufferMax: 4
OriginalTitle: %SystemRoot%\system32\cmd.exe
Title: C:\Windows\system32\cmd.exe
AttachedProcess: cmd.exe Pid: 1616 Handle: 0x64
----
CommandHistory: 0x427a60 Application: tcprelay.exe Flags: 
CommandCount: 0 LastAdded: -1 LastDisplayed: -1
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x0
----
CommandHistory: 0x427890 Application: whoami.exe Flags: 
CommandCount: 0 LastAdded: -1 LastDisplayed: -1
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x0
----
CommandHistory: 0x427700 Application: cmd.exe Flags: Allocated
CommandCount: 0 LastAdded: -1 LastDisplayed: -1
FirstCommand: 0 CommandCountMax: 50
ProcessHandle: 0x64
----
Screen 0x416348 X:80 Y:300
Dump:
```

I can see mine cmd.exe with PID 1616. Application it was lunched is tcprelay.exe. This executable is probably used to forward traffic. Lets dump address memory of process for our conhost and do some static analysis on it. I used **memdump** plugin.

```bash
root@tiny:/# volatility -f /dumps/ch2.dmp --profile=Win7SP1x86_23418 memdump -p 2168 --dump-dir /dumps
Volatility Foundation Volatility Framework 2.6.1
************************************************************************
Writing conhost.exe [  2168] to 2168.dmp
```

After downloading I run strings command on that memory dump.


```bash
root@tiny:/# strings 2168.dmp | grep tcprelay
tcprelay.exe 192.168.0.** 33** yourcsecret.co.tv 443 
tcprelay.c
C:\Users\John Doe\AppData\Local\Temp\TEMP23\tcprelay.exeJ"
C:\Users\John Doe\AppData\Local\Temp\TEMP23\tcprelay.exeN_
C:\Users\JOHNDO~1\AppData\Local\Temp\TEMP23\tcprelay.exeg[j
C:\Users\JOHNDO~1\AppData\Local\Temp\TEMP23\tcprelay.exe
C:\Users\JOHNDO~1\AppData\Local\Temp\TEMP23\tcprelay.exe
5C:\Users\JOHNDO~1\AppData\Local\Temp\TEMP23\tcprelay.exeg[j
```

As we can see there is IP address and Port - well redacted :) 

# Command and Control 5 Challenge

Find password.... Volatility offer few plugins to acquire passwords hashes.. lsadump, cachedump and hashdump. Tried them all and... 

```bash
root@tiny:/# volatility -f /dumps/ch2.dmp --profile=Win7SP1x86_23418 lsadump
Volatility Foundation Volatility Framework 2.6.1
ERROR   : volatility.debug    : Unable to read LSA secrets from registry
root@tiny:/# volatility -f /dumps/ch2.dmp --profile=Win7SP1x86_23418 cachedump
Volatility Foundation Volatility Framework 2.6.1
ERROR   : volatility.debug    : Unable to read hashes from registry
root@tiny:/# volatility -f /dumps/ch2.dmp --profile=Win7SP1x86_23418 hashdump
Volatility Foundation Volatility Framework 2.6.1
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
John Doe:1000:aad3b435b51404eeaad3b435b51404ee:b9f917853e3dbf6e6831ecce60725930:::
```

THe last plugin was successful and give us all hashes on machine. I quickly lookup all hashes and John Doe's give as clear text. I could also try to crack this passwords with hashcat or john the ripper if I need it. Also as these are NTLM hashes I can use them to authenticate to PC without without knowing password.

# Command and Control 6 Challenge

This challenge require some more set up. Is I will be running sample on real system :) (just VM) First of I need to get download our executable . i used **procdump** plugin for that. I could upload this file to hybrid analysis or maybe virustotal as well but for my exercise I decide to build quickly my lab.

```bash
root@tiny:/# volatility -f /dumps/ch2.dmp --profile=Win7SP1x86_23418 procdump -p 2772 --dump-dir /dumps                                                                                                       
Volatility Foundation Volatility Framework 2.6.1                                                                                                                                                                   
Process(V) ImageBase  Name                 Result
---------- ---------- -------------------- ------                                                        
0x87b6b030 0x00400000 iexplore.exe         OK: executable.2772.exe   
```

### ! ALERT
Please make sure you run example isolated environment. For this challenge I have VM windows 10 with ip 10.10.10.129 and Remnux VM with ip 10.10.10.128. In windows I set up defualt gateway and DNS to 10.10.10.128.

On remnux terminal i typed

```bash
fakedns
```

this will intercept all DNS request and response to them with ip address 10.10.10.128.

I double click on my executable and confirm it is running on Process Explorer

![](/images/volatility/proc.png)

As we can see it is running. Now lets have a look into our linux VM

![](/images/volatility/fakedns.png)

As we can see we get few dns request. One of them is flag :) 


I hope you enjoy it. :) 

Spyx

