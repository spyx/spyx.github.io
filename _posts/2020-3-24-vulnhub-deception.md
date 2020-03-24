---
layout: post
title: Deception Walkthrough
categories: [CTF,Vulnhub]
---

Hi all. Another day another walkthrough. I hope you enjoy learning as I do :smirk: As always i do masscan, nmap and dirb to look for some clues.

![](/images/deception/masscan.png)

Now i know ip address let do full nmap scan

![](/images/deception/nmap.png)

as usually we have port 22 and 80 open. Lets lunch dirb

![](/images/deception/dirb.png)

At the end I can see machine is running wordpress. Another clue is there is robot.txt file too. Let's have a look what we supposed to not see 

![](/images/deception/robots_txt.png)

Okay have a look at robots.html.

![](/images/deception/robots_html.png)

Some pop up with decreasing numbers. You can try to go zero or inspect source of this page which you will discover source of this javascript. After study code I know that when you reach 0 you will be redirect to admindelete.html page. 

![](/images/deception/admindelete.png)

As you can see there was nothing useful and I start loosing my mind because i could not figure out. After some time i discover hint.html :satisfied:

![](/images/deception/hint.png)

Really there is some source code on main page? And sure thing there was.

![](/images/deception/API.png)

After this challenge I have open element tag to look for all clues.. I collect all API and try to log in. For my first time I failed as APIs are not sorted from top to down. After correcting it i was in

![](/images/deception/yash_user.png)

You may asked how i know username. Well i was enumerate user through wpscan and get this user. I liked last 3 rows from previous picture. But looking in .bashrc explained to me :smirk:

![](/images/deception/yash_bashrc.png)

Hehe pretty funny ale in his folder was one file. It look pretty much as mess until i was username there. Used some grep kung-fu i was able to remove all noise from there

![](/images/deception/haclabs_pass)

Hmmm first line is username. Second is variable stored in A and last one is interesting. It is code for reverse this variable A. So haclabs passcode is **haclabs987654321**? Lets try it

![](/images/deception/flag2.png)

Boom second flag already mine. Now to get root access. I usually look for sudo -l command to see what exactly user can run as root. 

![](/images/deception/root.png)

And haclabs can run everything 

Hope you like this walkthrough :simple_smile:

