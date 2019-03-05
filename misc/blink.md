# blink
slopey112 - 3/4/19

We are given an .apk file and asked to find the flag inside it. apk files are essentially software packages, analagous to .exe files on Windows. We can, however, reverse engineer these guys by first converting them to a jar file by using the `dex2jar` command line utility.
```
root@kali:~/Downloads# d2j-dex2jar blink.apk
dex2jar blink.apk -> ./blink-dex2jar.jar
```
After we get that jar file, we can then compile it back to java. At this point, I only had 10 minutes left in the competition, so I just used a website, namely [Java Decompilers](www.javadecompilers.com). Per Java nomenclature, the actual programs related to the app is stored under /com, so we'll go there first. The following folder hierarchy is one way, so we'll navigate down until we find some actual files. Finally, we find four files, R.java, r2d2.java, MainActivity.java, BuildConfig.java. At this point, I just went through every file, looking for anything that may seem interesting. When I went to r2d2.java, however, I found something very interesting.
![r2d2.java](img/misc/r2d2.png)
It was the base64 representation of a jpg image.

Going on [Online JPG Tools](onlinejpgtools.com), I quickly converted the base64 into a jpg image, and sure enough, we find the flag.
![puckman](img/misc/puckman.png)
