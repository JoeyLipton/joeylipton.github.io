## HTB Nibbles Writeup
---

### Reconnaissance

#### Nmap Scan

![](/docs/images/nibbles/nibbles1.png)

- Basic HTTP page with an SSH port
    - OpenSSH usually doesn't have any vulnerabilities so its usually not worth looking into it
    - HTTP page is running Apache so maybe an outdated-related vulnerability



#### Walking the Application

The base page doesn't have anything interesting but the page source does point to a `/nibbleblog/` directory.

![](/docs/images/nibbles/nibbles2.png)

Following this endpoint gets us to a new blog page.


![](/docs/images/nibbles/nibbles3.png)

So we get this cool new endpoint to scan for stuff, let's start with a quick gobuster search.

![](/docs/images/nibbles/nibbles4.png)

God bless automated tools. So we can see an `/admin` endpoint under this which is worth exploring.


![](/docs/images/nibbles/nibbles5.png)

This leads to a set of folders with a bunch of stuff in them.

I didn't see a version number anywhere so we're going to look for that first, there was a `/README` in the gobuster results, maybe some info is over there.

![](/docs/images/nibbles/nibbles6.png)

### Exploitation

So this one is Nibbleblog version 4.0.3, there's probably an exploit for that somewhere. 

After a quick search, there is a Metasploit module that supports a file upload vulnerability.

![](/docs/images/nibbles/nibbles7.png)

This can just be found inside the Metasploit framework.


![](/docs/images/nibbles/nibbles8.png)

Now we need a username and password for the exploit to run, so I guess this is an authenticated attack.

I honestly just guess that the username was admin and the password was nibbles. The box gives you five tries before getting locked out for a couple minutes.


![](/docs/images/nibbles/nibbles9.png)

I guess we're doing this manually then, metasploit was never my cup of tea anyways.

![](/docs/images/nibbles/nibbles10.png)

This is the exploit just copied over using Searchsploit, its still a msf ruby script but it might have some instructions inside it. 

![](/docs/images/nibbles/nibbles11.png)

It does contain a blog post that could be interesting.

For this one, since the app just run .php files, we can upload a PHP reverse shell and get a user shell on the system.

![](/docs/images/nibbles/nibbles12.png)

We get a couple error messages but it shouldn't affect the shell.


![](/docs/images/nibbles/nibbles13.png)

Going to the directory mentioned in the blog post, we can see that the 'image.php' file is uploaded to the box.

![](/docs/images/nibbles/nibbles14.png)

And clicking on it gives us a shell. 

![](/docs/images/nibbles/nibbles15.png)

Easy peasy so far.


### Privilege Escalation

Okay I just got lucky and entered these commands right after each other.

![](/docs/images/nibbles/nibbles16.png)


So the monitor.sh script can be run as the root user.

For this one you can literally overwrite the `monitor.sh` script with bash and get a root shell.


![](/docs/images/nibbles/nibbles17.png)

And then you're done! That wraps up this box.








