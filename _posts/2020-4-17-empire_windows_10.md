---
layout: post
title: Powershell Empire with windows 10 
categories: [powershell, pentest]
---

Disclaimer! If your tool is not working try harder :) 


Hi everyone. After some time I decide to create some attack persistence on windows machines. After successful implementation of meterpreter i decide to use tool powershell empire. Original version was not longer supported on [github](https://github.com/EmpireProject/Empire). As kali still has python 2.7 i decide to fork it. I end up getting some error so i move for some googling advice. BC security is released empire v3 and they have kali linux page package. I simply typed...

```
sudo apt-get install powershell-empire
```

Them just simply run 
```
powershell-empire
```

![](/images/empire/01_empireScreen.png)

First thing we will set up listener. There are multiple options but i went with http. 

![](/images/empire/02_listeners.png)

Them you type ***execute***. You will see that your listeners is now listening. 

![](/images/empire/03_execute.png)


Make sure port, name is set up. You can also change DefaultDelay from 5s to 1s. When I was looking for some tutorial they start to create stager. Problem with stager was not working for me. Listeners have option to create launcher which will generate code for you. 

![](/images/empire/04_launcher.png)

I copied this powershell into powershell terminal and get shell back

![](/images/empire/05_connection.png)

Move to this agent and test connection to my victim machine

![](/images/empire/06_shell.png)

Also as you can see I have win10 machine. I turned off Defender for learning purposes. 

![](/images/empire/07_sysinfo.png)

I hope you enjoy this reading...

Cheers
Spyx.

