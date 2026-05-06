### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - The Concept of Attacks <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### The Concept of Attacks



Every attack on a service follows the same 4-stage cycle: `Source` -> `Process` -> `Privileges` -> `Destination`. The cycle is linear (destination is not always reused as a new source).

| Stage | Description |
|---|---|
| `Source` | Origin of info passed to the process: code, libraries, config, APIs, user input |
| `Process` | The component that processes the input: PID, input, data processing, variables, logging |
| `Privileges` | Permission context: System/root, user, groups, policies, rules |
| `Destination` | Where the result lands: local (files/processes) or network (interface, addr, route) |

<br>

---

<br>

### Source



Inputs an attacker can poison:

| Source | Description |
|---|---|
| `Code` | Output of program code reused as input |
| `Libraries` | Shared collection of routines / classes / configs (e.g. Log4j) |
| `Config` | Static or prescribed values that drive how info is processed |
| `APIs` | Interface for retrieving / providing data |
| `User Input` | Manual entry by a person |

Log4j ([CVE-2021-44228](https://cve.mitre.org/cgi-bin/cvename.cgi?name=cve-2021-44228)): attacker manipulates the HTTP `User-Agent` header and inserts a JNDI lookup; library treats it as a command, not data.

<br>

---

<br>

### Processes



Process components vulnerable to abuse:

| Component | Description |
|---|---|
| `PID` | Identifies the running process and its inherited privileges |
| `Input` | Info from a user or a programmed function |
| `Data processing` | The hard-coded logic dictating how info is handled |
| `Variables` | Placeholders moved between functions during processing |
| `Logging` | Recorded events; persisted in files or registry |

Log4j case: process logs the User-Agent string via a function. Vulnerability is misinterpretation of the string as code.

<br>

---

<br>

### Privileges



Permission context the process runs under:

| Privilege | Description |
|---|---|
| `System` | Highest: `SYSTEM` (Windows) / `root` (Linux) |
| `User` | Per-user permissions; service-specific accounts often used |
| `Groups` | Permission sets shared by multiple users |
| `Policies` | App-specific command execution rules |
| `Rules` | App-internal action permissions |

Log4j was dangerous because logging often runs with admin privileges to access protected log dirs. Exploited library + admin context = full RCE.

<br>

---

<br>

### Destination



Where the process result lands:

| Destination | Description |
|---|---|
| `Local` | System's own environment: local files / records modified or forwarded to local services |
| `Network` | Result forwarded to remote interface (IP / port / network) |

Log4j: misinterpreted UA -> JNDI lookup -> attacker-controlled remote LDAP server -> Java class fetched & executed -> RCE.

<br>

---

<br>

### Log4j - Mapping to Concept of Attacks

#### Initiation:

| # | Step | Category |
|---|---|---|
| 1 | Attacker manipulates user agent with JNDI lookup. | `Source` |
| 2 | Process misinterprets UA, executes the command. | `Process` |
| 3 | Command runs with admin privileges (logging perms). | `Privileges` |
| 4 | JNDI points to attacker server (malicious Java class). | `Destination` |

#### Trigger RCE (cycle restart):

| # | Step | Category |
|---|---|---|
| 5 | Malicious Java class retrieved -> reused as source. | `Source` |
| 6 | Process reads / executes the Java class code. | `Process` |
| 7 | Code runs with admin privileges. | `Privileges` |
| 8 | Reverse shell back over network to attacker. | `Destination` |
