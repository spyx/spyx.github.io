---
layout: post
title: Powershell Empire with windows 10 (http/ launcher_bat)
categories: [powershell, pentest]
---

Disclaimer! I dont want to have bad opinion on Empire just want to save some nice afternoon some person who spend half day troubleshoot this. I only try to use http listener with launcher_bat stager.


Hi everyone. After some time I decide to create some attack persistence on windows machines. After successful implementation of meterpreter i decide to use tool powershell empire. Original version was not longer supported on [github](https://github.com/EmpireProject/Empire). As kali still has python 2.7 i decide to fork it. I end up getting some error so i move for some googling. After certain time i found that BC security is released empire v3 and they have kali linux page. I simply typed

```
sudo apt-get install powershell-empire
```

Installation went smooth and I have Empire console in front of me. 
I run my first http listener and start working on launcher_bat stager. When i moved stager to new machine and start to run ...nothing happen. Them i decide to turn off Windows Defender and Firewall too. Same results. Them i decide to start looking for packets in network. As both machines runs on virtualbox .After some google foo i have [solution](https://www.virtualbox.org/wiki/Network_tips) for my problem. I can capture all pakets on certain machine...

```
VBoxManage modifyvm [your-vm] --nictrace[adapter-number] on --nictracefi [adapter-number] file.pcap
```

when i check packets i never see stager to make any connections. Them start research why this is happening and end up on this github [issue](https://github.com/EmpireProject/Empire/issues/1232). From i can understand... after 1803 released of Windows 10 there is feature which block default stager created within app. 

Please let me know if you have solution will be happy to  create tutorial on it.
If you have any questions feel free to reach me on twitter. 

Cheers
Spyx.

