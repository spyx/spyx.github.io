---
layout: post
title: Reset Windows 10 local password with Kali
categories: [windows, kali]
---

If you ever need to reset windows password and you have physical access to HW/Vm you can do it with Kali live image.

First step si to boot Live Kali image. If disk/partition is not mounted automatically you can do it with *mount* command

Them move to Windows/system32/config folder

![](/images/reset-pass/01-config-folder.png)

Right click in that folder and open in terminal

![](/images/reset-pass/02-open-terminal.png)

List all available local users 

```bash 
chntpw -l SAM
```

![](/images/reset-pass/03-list-users.png)

Choose user you can to clear password

```bash 
chntpw -u <username> SAM
```

![](/images/reset-pass/04-clear-pass.png)

There are multiple options and i want to clear administrator password as i forgot it :) 

Last option is to quit and save changes 

![](/images/reset-pass/05-save-sam.png)

