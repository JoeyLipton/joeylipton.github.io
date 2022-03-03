## TryHackMe Mustacchio Writeup
---

[Link to TryHackMe Room](https://tryhackme.com/room/mustacchio)


#### Mustacchio Room

![](/docs/images//mustacchio/mustacchio1.png)

The room is a standard TryHackMe ctf room with a user flag and root flag entries. 

#### Nmap Scan

```
$ nmap -sC -sV -p- 10.10.247.201 -oA nmap/nmap-mustacchio
```

![](/docs/images//mustacchio/mustacchio2.png)

#### Gobuster Scan

```
gobuster dir -u http://10.10.247.201 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobuster/gob-mustacchio
```

![](/docs/images//mustacchio/mustacchio12.png)

#### Visiting the Website :80

![](/docs/images//mustacchio/mustacchio11.png)

Upon some looking around, most of the endpoint were just static pages. However some file pages were active, the one that was found to be interesting was the /custom/ endpoint. 

![](/docs/images//mustacchio/mustacchio13.png)

Inside were 2 directories, the css/ and js/. Since JS is the most exploitable part of a website, checking there first was obvious. 

![](/docs/images//mustacchio/mustacchio14.png)


There are 2 files in the /custom/js directory, I was definitely going to check out users.bak.


#### users.bak file

```
wget http://10.10.247.201/custom/js/users.bak
```

![](/docs/images//mustacchio/mustacchio15.png)

Honestly I thought it was going to be a bit obfuscated so I used strings. But it gives me a direct admin:password type of output. So using some easy-cheesy web tools I'm just going to see if I can get the password. 

Sources:
[hash-identifier](https://hashes.com/en/tools/hash_identifier)
[CrackStation](https://crackstation.net/)

![](/docs/images//mustacchio/mustacchio16.png)

<p></p>

![](/docs/images//mustacchio/mustacchio17.png)

Nice, the resulting password is bulldog19. Now to figure out where to use it. 


#### Visiting Website :8765

After a username and password was extracted from the main page. The next visit is the second http page that was found in the nmap scan. The nginx page on port 8765.

![](/docs/images//mustacchio/mustacchio18.png)

This page is found to be an admin page for the webserver. Using the admin credentials obtained earlier, it can be accessed.


![](/docs/images//mustacchio/mustacchio19.png)

There is a single comment block on the page to enter information into. The page is also a PHP portal, so there is a chance a php reverse shell could do it. 


![](/docs/images//mustacchio/mustacchio3.png)

This turns out to be an XML space. So there is potential for and XXE exploit. Let's take a look at the request in Burp Suite. 


![](/docs/images//mustacchio/mustacchio4.png)


Following the dontfoget.bak link downloads another file. 

![](/docs/images//mustacchio/mustacchio5.png)

cat dontforget.bak

![](/docs/images//mustacchio/mustacchio6.png)

XXE exploit

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE replace [<!ENTITY ent SYSTEM "file:///etc/passwd"> ]>
<comment>
  <name>&ent;</name>
  <author>Barry Clad</author>
  <com>testing</com>
</comment>
```

![](/docs/images//mustacchio/mustacchio7.png)

.ssh/id_rsa

![](/docs/images//mustacchio/mustacchio8.png)

Private SSH Key


![](/docs/images//mustacchio/mustacchio9.png)


Now we need to decrypt this key.

This can be done using ssh2john.

![](/docs/images//mustacchio/mustacchio10.png)

NOTE: The base64.decodestring() function has been deprecated in Python 3.8, so change it to decodebytes(). Like actually go and change the file.



![](/docs/images//mustacchio/mustacchio22.png)

And there we go! The password for barry is "urieljames"

Also before the key can work, it wasn't formatted right out of the XXE output. So it needs to be spaced out properly manually or with a script. I recommend just going into BurpSuite and copying the request from repeater. 

![](/docs/images//mustacchio/mustacchio23.png)


This is how the file should look like and then $ chmod 600 the file as well. 


Next I can SSH into the machine using the RSA private key and the passphrase from John.

```
ssh barry@10.10.188.151 -i ssh-creds/barry.key
```


![](/docs/images//mustacchio/mustacchio20.png)


#### Privilege Escalation - SUID

Now onto the privilege escalation. Normally I'd use linPeas but the hint tells me that it uses an SUID binary so I can use a find command and search for them.

```
find / -perm -u=s -type f 2>/dev/null
```

![](/docs/images//mustacchio/mustacchio24.png)

And this does give a response that can be investigated. The binary /home/joe/live_log.


![](/docs/images//mustacchio/mustacchio25.png)

So it is an executable file that allows has root privileges.

Lets see what it actually does since this does seem to be a custom application.

![](/docs/images//mustacchio/mustacchio26.png)

After executing the live_log binary, it outputs apache/nginx logs from the home page of the admin website. So potentially there could be some log poisoning to execute commands. Or just manipulation of the SUID commands. 


![](/docs/images//mustacchio/mustacchio27.png)

After a $ strings check, I can see that the "tail" command is not defined from its absolute pathname. This makes it so the user that is running the application needs to use their terminal paths to 

```
PATH=/home/barry:$PATH
```

First step is to create the /home/barry a path before the current path so it takes priority over the /usr/bin and /usr/sbin folders.

![](/docs/images//mustacchio/mustacchio30.png)


Now to create the script. For this I'm just using the /bin/bash string just to be simple. Since this is an easy box it should just work right out of the gate. Hopefully, sometimes there need to be some script parameters if the root user is restricted to no shell or /bin/sh.


![](/docs/images//mustacchio/mustacchio29.png)

And now all that needs to be done is run the live_log script and it should work. ****


![](/docs/images//mustacchio/mustacchio31.png)