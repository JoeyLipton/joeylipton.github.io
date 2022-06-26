## HTB Brainfuck Writeup
---


#### Reconnaissance

##### Nmap Scan

![](/docs/images/brainfuck/brainfuck1.png)

- A basic nmap scan shows SSH, Nginx, SMTP, POP3, Dovecot, and an HTTPS nginx endpoint.


##### Gobuster Scan

![](/docs/images/brainfuck/brainfuck2.png)

- This doesn't get anything fruitful, but the website does contain some certificate so this might be of interest.


##### Certificate Analysis

![](/docs/images/brainfuck/brainfuck3.png)

- The certificate does contain an email,  `orestis@brainfuck.htb` , this might be useful for brute-forcing credentials.


![](/docs/images/brainfuck/brainfuck4.png)

- The certificate does contain 2 domains for our /etc/hosts file that can be used to reach 2 hidden domains.


##### `brainfuck.htb` endpoint

![](/docs/images/brainfuck/brainfuck5.png)

- So the first website is based on Wordpress which is a good starting point for something like `$ wpscan`.

```

         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.20
                         
[+] Headers
 | Interesting Entry: Server: nginx/1.10.0 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%
[+] XML-RPC seems to be enabled: https://brainfuck.htb/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
[+] The external WP-Cron seems to be enabled: https://brainfuck.htb/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 
----- SNIP -----

[+] wp-support-plus-responsive-ticket-system
 | Location: https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/
 | Last Updated: 2019-09-03T07:57:00.000Z
 | [!] The version is out of date, the latest version is 9.1.2
 | Found By: Urls In Homepage (Passive Detection)
 |
 | Version: 7.1.3 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - https://brainfuck.htb/wp-content/plugins/wp-support-plus-responsive-ticket-system/readme.txt
```

- The most interesting part was just some outdated ticketing system stuff, but it does scan with good results, so I'm gonna go ahead and enumerate some users since there is a devpost on the site indicating users.


```
[i] User(s) Identified:

[+] admin
 | Found By: Author Posts - Display Name (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] administrator
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

- So there is an administrator user account. But back to the oudated ticketing system.

```
$ searchsploit wordpress ticket
--------------------------------------------------------------------------------
 Exploit Title   
 
----- SNIP -----

WordPress Plugin WP Support Plus Responsive Ticket System 7.1.3 - Privilege Escalation
WordPress Plugin WP Support Plus Responsive Ticket System 7.1.3 - SQL Injection
```

- So there is a Privilege Escalation and SQL Injection

I'm gonna start with the privilege escalation one since it's sounds more promising.

![](/docs/images/brainfuck/brainfuck7.png)

- So this one comes with a simple request to get access to the admin panel, let's save that in its own file called `exploit.html` for later.

![](/docs/images/brainfuck/brainfuck9.png)


- We can run this by launching a Python web server and accessing the html file form there, triggering the payload. 


![](/docs/images/brainfuck/brainfuck10.png)

- Once that's done, we are logged in


The DevPost on the main page did mention SMTP being enabled to let's check that out first.

![](/docs/images/brainfuck/brainfuck11.png)


The password is masked to we can just switch over to inspect element and see that says.


![](/docs/images/brainfuck/brainfuck12.png)

We can the credentials of `orestis:kHGuERB29DNiNE`, these are SMTP creds. So they're used to access a mail server. I'm just gonna use Thunderbird to do this (if you're getting issues, try thunderbird-nightly).


##### Email Server Access

![](/docs/images/brainfuck/brainfuck13.png)

Some credentials are listed in the email server, this is probably pointing to the secret forum.



##### `sup3rs3cr3t.brainfuck.htb` endpoint

![](/docs/images/brainfuck/brainfuck6.png)

- The main page doesn't really have anything intersting so we can go ahead and apply the credentials. 
    - `orestis:kIEnnfEKJ#9UmdO`

After logging in, the page redirects to the main page with some new posts.

![](/docs/images/brainfuck/brainfuck14.png)

The SSH access looks promising, so we can go ahead and check that out.


![](/docs/images/brainfuck/brainfuck15.png)

So we can see that they switched over to SSH key based authentication. Maybe the 'key' blog post will have better results.


![](/docs/images/brainfuck/brainfuck17.png)


So this one looks like its encrypted using some cipher or such. 

Also 'orestis' signs all of his messages with "Oresis - hacking for fun and profit", so we can use that plaintext to decrypt the rest of the text.

This is probably a one-time pad, so we can use a website like `rumkin.com` to decrypt this. 

![](/docs/images/brainfuck/brainfuck18.png)


The key turns out to be 'brainfuckmy' repeated.

![](/docs/images/brainfuck/brainfuck19.png)

So we can reproduce this.


![](/docs/images/brainfuck/brainfuck20.png)

Now there's an SSH key file location in the description, we can crack the encryption and get the key location.


![](/docs/images/brainfuck/brainfuck21.png)

Since the 'mybrainfuck' part starts at a random point, some rotation is needed to get the file location.


![](/docs/images/brainfuck/brainfuck22.png)

This one actually needs a password so we need to crack that using something like john (hashcat doesn't work in my VM feelsbad).


![](/docs/images/brainfuck/brainfuck23.png)

The password is found to be `3poulakia!`.

![](/docs/images/brainfuck/brainfuck24.png)

And we're in!


##### User Access

USER FLAG: 

![](/docs/images/brainfuck/brainfuck25.png)

We get the user flag upon access.

Looking at the files in the home/orestis directory is a little more intersting. 

![](/docs/images/brainfuck/brainfuck26.png)

There's a couple files of note:
    - debug.txt
    - encrypt.sage
    - output.txt
    - mail/

mail/ probably doesn't have anything in it.

Looking at encrypt.sage, it looks like an RSA encryption.

![](/docs/images/brainfuck/brainfuck27.png)

So given the script, we need to decryt the RSA.

I'm just using a script I found on GitHub (credit: https://gist.github.com/intrd/3f6e8f02e16faa54729b9288a8f59582) and replacing the values with the debug and output.txt values.

![](/docs/images/brainfuck/brainfuck28.png)

This then gives the flag.txt output without needing root access. I can submit this and be done with the box.

ROOT FLAG: 

![](/docs/images/brainfuck/brainfuck29.png)
