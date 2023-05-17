## HTB Optimum Writeup
---

### Box Overview

This box is quite old by the time I'm doing it, some of the tags that give it away are HFS, Python, and Outdated Software. Hopefully it's not another Metasploit but there's always a more fun alternate solution in that case.


### Reconnaissance
---

#### Nmap Scan

The nmap scan surpisingly only found 1 port open, I won't bother with a UDP scan until a little later if I really can't find anything. But I doubt it'll be necessary, especially on an old easy box. 


![](/docs/images/optimum/20230517160709.png)

It does report that it's an old HttpFileServer running on httpd 2.3.


#### Gobuster Scan

I didn't find anything so I won't even include a screenshot for this part.


#### Walking the Application

We already know the application is running this HttpFileServer (which was probably what HFS in HTB meant), so my expectations are already set for an old-looking application with little to no JavaScript.

![](/docs/images/optimum/20230517162058.png)

Pretty standard stuff, some basic info like Server time and uptime is present and there is an option to login.

![](/docs/images/optimum/20230517162234.png)

The login option gives this web pop-up login screen. 


#### Searchsploit Search

I know the that that the server is running a really old HttpFileServer so there might be some nice exploits that can be exploited.

![](/docs/images/optimum/20230517165522.png)

I'm feeling a bit tired today so the "Remote Command Execution (Metasploit)" one looks interesting. I know this isn't OSCP valid but some meterpreter practice can't hurt.


### Exploitation

We can jump right into meterpreter and select the relevant exploit.

![](/docs/images/optimum/20230517165755.png)

That was pretty quick. This shell gives me access to the standard user account, so I'm going to start by trying the linux_exploit_suggester. 

![](/docs/images/optimum/20230517170953.png)

Some basic setup, the exploit used creates multiple sessions automatically so I needed to use SESSION 3.

![](/docs/images/optimum/20230517171028.png)

The modules gives me back two possible options, a `bypassuac_eventvw` exploit and a `ms16_032_secondary_logon_handle_privesc`.

![](/docs/images/optimum/20230517171527.png)

Okay that one didn't work, time to try the other one.

![](/docs/images/optimum/20230517171641.png)

Okay cool that one did work. This was kind of a cheesy way to get root but it worked to get the flags. 

