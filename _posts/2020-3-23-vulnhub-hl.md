---
layout: post
title: haclabs:no_name Walkthrough
categories: [CTF,Vulnhub]
---

Hi everyone. Another day another walkthrough. As usually i tried to figure out IP address of victim machine with masscan

![](/images/vuln_hl/masscan.png)

We get IP address so let's do some basic reconnaissance with nmap and dirb

![](/images/vuln_hl/nmap_full.png)

```
dirb http://<ip_address>
```

This dirb found index.php. After inspecting index page and source code there was comment there. 

![](/images/vuln_hl/comment.png)

4 pictures and comment with passphrase? Lest download steghide and see if there is any secrets in pictures. You can download steghide in kali linux pretty quickly. If you are not root type **sudo** in front fo this command

```
apt-get install steghide
```

I found picture which asked my for passphrase

![](/images/vuln_hl/stego.png)

After place passphrase from comment i received txt file. Quickly look on content and it was base 64 encodes. After decoding this i get name for another page - superadmin.php :smirk:

On this page you can place ip address and will received ping back. Lets try add **& id** or **| id** to see if we can have some command injections. First command failed on is and resulted on empty page. Send on other side give us output of id. From that point there has to be some whitelistening on page. Lets try this command | cat superadmin.php. Bingo!! :relieved: that work after inspecting source code we see code for that page

![](/images/vuln_hl/injection_comment.png)

From first line I can see exact commands and characters there are not allowed. One of them is nc which i want to use for getting shell. After little googling how to bypass this whitelisting i got on article which use 

```bash
`echo "<base64 encoded payload>" | based4 -d`
```

For payload i create simple nc reverse shell command. Sorry all i forgot to create screenshot so hopefully you can figure out your payload. Btw nc on attacker machine is not supporting -e option but you can use nc.traditional :bowtie:

After placed my command i got shell 

![](/images/vuln_hl/flag1.png)

Look into home directories for users. Look into yash and got first flag. Not sure why i was getting double characters but i got shell and shell worked. If you know why let me know. Also there was clue for haclabs password. Lets use **find** command to find all files belong to yash

![](/images/vuln_hl/haclabs_pass.png)

Bingo. Look into hidden file and look for password. After that, just log in as haclabs

![](/images/vuln_hl/haclabs_user.png) 

i was eager to get root access and you can see from screenshot. :grin: Also go to home directory and retrieve second flag. As you can see user can use find command with sudo. 

![](/images/vuln_hl/root.png)

Find command has option to spawn shell. As we have access to root level via sudo we can get root admin easily

I hope you like this walkthrough.... See you in next one 




