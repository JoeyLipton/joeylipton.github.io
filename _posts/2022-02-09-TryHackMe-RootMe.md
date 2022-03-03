## TryHackMe RootMe Writeup
---

#### Starting the Box

![](/docs/images/rootme/rootme1.png)


#### Nmap Scan

```
$ nmap -sC -sV 10.10.40.145 -oA recon/recon-rootme
```

![](/docs/images/rootme/rootme2.png)

#### Gobuster Scan

```
gobuster dir -u http://10.10.40.145 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobuster/gob-rootme
```

This is just a standard gobuster scan on the box to look for endpoints, the command itself usually doesn't finish but its a good way to find initial access.


![](/docs/images/rootme/rootme3.png)


#### /panel/ endpoint

The /panel/ endpoint found in gobuster looks interesting as this isn't a normal endpoint found in Apache so its worth checking out. 

![](/docs/images/rootme/rootme4.png)

This endpoint allows for file uploads, the first easiest thing to test is a PHP reverse shell. 


![](/docs/images/rootme/rootme5.png)

So this actually doesn't work, after some testing it specifically blocks the .php file extension. The .png, .jpg, etc file extensions work fine. Now the trick is to still get a file that will execute on the server. The simplest one for checking validation is .php5 extensions. 



Copy over PHP reverse shell from /usr/share/webshells-> change IP and PORT -> Open NC port

![](/docs/images/rootme/rootme6.png)

SUCCESS!! Now that this file is uploaded, the /uploads directory can be visited to activate the payload. 

![](/docs/images/rootme/rootme7.png)

Clicking on the payload activates and gives a reverse shell.


![](/docs/images/rootme/rootme8.png)

Now to find the user flag. Checking the user folders doesn't work out this time, so it's most likely somewhere in the www-data folder since its a web-user. 

![](/docs/images/rootme/rootme9.png)

To find flag:

```
find / -type f -name user.txt 2> /dev/null
```

![](/docs/images/rootme/rootme10.png)

The user flag has been found. Now onto privilege escalation.

#### Privilege Escalation

The TryHackMe site says that the priv esc method is a SUID binary. So I can start there. The easiest, OSCP-approved way to send over linpeas.sh onto the system and run it. This will give a big overview of the system.

Use linpeas to find SUID binary

![](/docs/images/rootme/rootme11.png)

The /usr/bin/python file is allowed to run with root privileges. First place to check is GTFObins. 


![](/docs/images/rootme/rootme12.png)


From this, the command that was used was:

```
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

The resulting output:

![](/docs/images/rootme/rootme13.png)

The box has now been completed with root access. 