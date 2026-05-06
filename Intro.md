### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - Interacting with Common Services <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Interacting with Common Services



To attack a service, know its purpose, how to interact with it, what tools to use, and what can be done with it. Common targets: file shares (SMB/NFS/FTP), email (SMTP/POP3/IMAP), and databases (MySQL/MSSQL).

<br>

---

<br>

### File Share Services - SMB



SMB serves file/printer/IPC over `TCP/445` (modern) or `139` (NetBIOS). Samba is the Linux SMB implementation.

Windows GUI - open Run dialog (`WIN+R`) and type a UNC path:

```diff
+ \\<server-ip>\<share>
```

<br>

Windows CMD - list directory contents over SMB:

```diff
+ C:\> dir \\192.168.220.129\Finance\
```

<br>

Map the share to drive letter `n`:

```diff
+ C:\> net use n: \\192.168.220.129\Finance
+ C:\> net use n: \\192.168.220.129\Finance /user:plaintext Password123
```

<br>

Recursively count files in mapped share:

```diff
+ C:\> dir n: /a-d /s /b | find /c ":\"
```

| Flag | Description |
|---|---|
| `/a-d` | files only (no dirs) |
| `/s` | recursive |
| `/b` | bare format |

<br>

Find filenames matching a pattern:

```diff
+ C:\> dir n:\*cred* /s /b
+ C:\> dir n:\*secret* /s /b
```

<br>

Search inside files for a string:

```diff
+ C:\> findstr /s /i cred n:\*.*
```

<br>

PowerShell equivalents:

```diff
+ PS C:\> Get-ChildItem \\192.168.220.129\Finance\
+ PS C:\> New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem"
+ PS C:\> (Get-ChildItem -File -Recurse | Measure-Object).Count
+ PS C:\> Get-ChildItem -Recurse -Path N:\ -Include *cred* -File
+ PS C:\> Get-ChildItem -Recurse -Path N:\ | Select-String "cred" -List
```

<br>

PowerShell with credentials via `PSCredential` object:

```diff
+ $username = 'plaintext'
+ $password = 'Password123'
+ $secpassword = ConvertTo-SecureString $password -AsPlainText -Force
+ $cred = New-Object System.Management.Automation.PSCredential $username, $secpassword
+ New-PSDrive -Name "N" -Root "\\192.168.220.129\Finance" -PSProvider "FileSystem" -Credential $cred
```

<br>

Linux - mount an SMB share (requires `cifs-utils`):

```diff
+ $ sudo mkdir /mnt/Finance
+ $ sudo mount -t cifs -o username=plaintext,password=Password123,domain=. //192.168.220.129/Finance /mnt/Finance
```

<br>

Mount with credentials file:

```diff
+ $ mount -t cifs //192.168.220.129/Finance /mnt/Finance -o credentials=/path/credentialfile
```

`credentialfile` format:

	username=plaintext
	password=Password123
	domain=.

<br>

Search mounted share:

```diff
+ $ find /mnt/Finance/ -name *cred*
+ $ grep -rn /mnt/Finance/ -ie cred
```

<br>

---

<br>

### Email Services



SMTP for sending, POP3/IMAP for retrieving. Linux GUI client `Evolution`:

```diff
+ $ sudo apt-get install evolution
```

<br>

If sandbox error, start with:

```diff
+ $ export WEBKIT_FORCE_SANDBOX=0 && evolution
```

Use TLS (dedicated port) or STARTTLS as appropriate.

<br>

---

<br>

### Database Interaction



Common targets: MySQL (`tcp/3306`) and MSSQL (`tcp/1433`).

MSSQL CLI - Linux (`sqsh`) and Windows (`sqlcmd`):

```diff
+ $ sqsh -S 10.129.20.13 -U username -P Password123
+ C:\> sqlcmd -S 10.129.20.13 -U username -P Password123
```

<br>

MySQL CLI:

```diff
+ $ mysql -u username -pPassword123 -h 10.129.20.13
+ C:\> mysql.exe -u username -pPassword123 -h 10.129.20.13
```

<br>

GUI clients: MySQL Workbench, SSMS (Windows-only), DBeaver (cross-platform).

Install/run DBeaver from .deb:

```diff
+ $ sudo dpkg -i dbeaver-<version>.deb
+ $ dbeaver &
```

<br>

---

<br>

### Useful Tools - Common Services

| Service | Tools |
|---|---|
| SMB | `smbclient`, `CrackMapExec`, `SMBMap`, `Impacket` (`smbexec.py`, `psexec.py`) |
| FTP | `ftp`, `lftp`, `ncftp`, `crossftp`, `filezilla` |
| Email | `Thunderbird`, `Claws`, `Geary`, `mutt`, `sendmail`, `swaks`, `sendEmail`, `MailSpring` |
| Databases | `mycli`, `mssql-cli`, `dbeaver`, `MySQL Workbench`, `SSMS`, `Impacket` (`mssqlclient.py`) |

<br>

---

<br>

### Troubleshooting Connections



Common access failure causes:

- Authentication / privilege issues
- Network connectivity / firewall blocking traffic
- Protocol version mismatch
