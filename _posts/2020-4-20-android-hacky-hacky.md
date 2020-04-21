---
layout: post
title: Android Hacking 101
categories: [Android, Hacking]
---

Some quickly tips and commands to start with Android hacking

Decompiling apk
```
apktool d -s <your.app> -o <your output folder>
```

Look for API keys and clues on 
AndroidManifest.xml
res/values/strings.xml

Look for databases

```
find . -name *.sqlite
find . -name *.db
```

After unpacking if there is classess.dex file you can use this tool to convert from dex2jar. Just download project and unzip it. If you are on Mac or Linux make sure all .sh files can be executed :) chmod is your friend... 
```
./d2j-dex2jar.sh <your dex>
```

jd-gui is small app that convert java binary to readable format

That's it. Simple tool to start android hacking :) 



