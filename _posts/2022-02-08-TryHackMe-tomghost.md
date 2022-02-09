## tomghost
---

#### Starting the Machine
<p></p>

![](/docs/images/tomghost/tomghost1.png)

This is just a quick overview of the machine, the first hint that can be seen is the term called "GhostCat". This could possibly be the name of a tool or exploit that is part of the intended route. There is also an offensive word in a username. This could be used for initial access or hold some important privileges later on. 

#### Nmap Scan
<p></p>

![](/docs/images/tomghost/tomghost2.png)

This -sC -sV nmap scan shows a couple open ports and some information about their services. Firstly, there is an ssh port open. The SSH ports are an excellent sign as obtaining an ssh shell through a key or password is one of the more reliable methods of persistence, as opposed to a meterpreter or nc shell. 

Another important port is the :8009 and :8080 ports. These are both ports that Apache uses to interface with the Tomcat application. Maybe the exploitation of these could lead to some level of access. 

#### Gobuster Scan
```
# I didn't think this was necessary since its just exploiting a Tomcat service
```

I just included the Gobuster field since all my writeup have then. But since this is a relatively simply box about exploiting Tomcat, I didn't choose to use it. 


#### Tomcat Service (:8080)

![](/docs/images/tomghost/tomghost3.png)

This is the initial page that is shown when visiting the Tomcat web portal on port 8080. Upon some initial exploration, the page can only be accessed from the localhost machine. This could mean some potential port forwarding later on. 


Using the version number, 9.0.30, an exploit can be found named 'Ghostcat'. This is a file read vulnerability that allows files on the system to be read by unauthenticated users. There is also a  Metasploit module that can be used. 

![](/docs/images/tomghost/tomghost4.png)


This can be found in Metasploit as shown below: 

![](/docs/images/tomghost/tomghost5.png)
![](/docs/images/tomghost/tomghost6.png)

The text at the bottom sounds interesting:
```
-- SNIP --
 version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
        skyfuck:8730281lkjlkjdqlksalks
  </description>

</web-app>
```

This reveals a username and password, lets try using these to login to the SSH port mentioned earlier. 


I can try SSHing into this and find the first user flag.

![](/docs/images/tomghost/tomghost7.png)

Since the Tomcat installation doesn't allow access to the application without being the host, I can use SSH port forwarding to forward the port locally. This actualy didn't lead anywhere but it was still part of the process that I thought would be useful to include. 

![](/docs/images/tomghost/tomghost8.png)


Now linPEAS tells me that there's a possible private SSH key in the tryhackme.asc file in the home directory. (I forgot to show linPEAS output).

![](/docs/images/tomghost/tomghost9.png)

Using the gpg command, the credentials can attempt to be decrypted from the credentials.pgp file.

Just for reference, PGP (or Pretty Good Privacy), is an encryption program that allows for the encryption of files, disks, emails, etc. 

And the $ gpg command is the main program for the GnuPG system. It can be used to encrypt, decrypt, verify signatures, and more. 

![](/docs/images/tomghost/tomghost10.png)

This doesn't work right away, so I'm going to extract the file in order to take them offline and try to brute force the key for them. 

![](/docs/images/tomghost/tomghost11.png)


Using John, the private key can be cracked. For this a part of John the Ripper called gpg2john is extract the PGP private keys (tryhackme.asc) hash. From this a hash can be obtained, and using normal john, the hash can be cracked against the rockyou.txt wordlist. 

![](/docs/images/tomghost/tomghost12.png)

![](/docs/images/tomghost/tomghost13.png)

The resulting password is 'alexandru'. This is the password needed to access the credentials.gpg file. 

Going back to the machine, first import the tryhackme.asc key into the GPG keychain. 

![](/docs/images/tomghost/tomghost14.png)

Then attempt to decrypt the key again.

![](/docs/images/tomghost/tomghost15.png)

This gives merlins username and password. We can try SSHing into this from another terminal session. 

![](/docs/images/tomghost/tomghost16.png)

Now we have access to the merlin user. and upon inspecting the 'sudo -l' result, the zip command can be seen as accessible by root. I have no idea why a computer would allow this, but referring ot GTFObins gives a pretty promising result. 

Reference: https://gtfobins.github.io/gtfobins/zip/

Then from the root user, using the zip exploit, a root shell can be obtained as well as the root flag. 

![](/docs/images/tomghost/tomghost17.png)