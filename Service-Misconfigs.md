### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - Service Misconfigurations <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Service Misconfigurations



Misconfigurations occur when admins / devs / support don't lock down the service security framework, leaving open paths for unauthorized users.

<br>

---

<br>

### Authentication



Default creds are still common, especially on older / vendor appliances. After grabbing the service banner, try:

	admin:admin
	admin:password
	admin:<blank>
	root:12345678
	administrator:Password

Misconfig types worth testing:

- `Default credentials` - vendors that ship with built-in admin accounts
- `Weak / blank passwords` - admin "I'll change it later" scenario
- `Anonymous authentication` - no creds required (FTP/SMB null sessions)
- `Misconfigured access rights` - users granted more permissions than their role needs

Mitigations: RBAC ([Role-Based Access Control](https://en.wikipedia.org/wiki/Role-based_access_control)) or ACLs ([Access Control Lists](https://en.wikipedia.org/wiki/Access-control_list)) per least-privilege.

<br>

---

<br>

### Unnecessary Defaults



Default settings prioritize usability over security. Per [OWASP Top 10 - Security Misconfiguration](https://owasp.org/Top10/A05_2021-Security_Misconfiguration/), watch for:

- Unnecessary features enabled (ports, services, pages, accounts, privileges)
- Default accounts / passwords still active
- Verbose error handling exposing stack traces
- Latest security features disabled or unconfigured on upgraded systems

<br>

---

<br>

### Preventing Misconfiguration



Lock down most-critical infra; disable communications not required by the program.

- Disable admin interfaces
- Turn off debugging
- Disable default credentials
- Block unauthorized access / directory listing
- Run regular scans + audits
- Use a repeatable hardening process across dev/QA/prod (different creds per env)
- Minimal platform - remove unused features, components, sample apps
- Patch management tied to config review
- Segmented architecture (segmentation, containers, cloud security groups)
- Send security headers to clients
- Automate config-effectiveness verification
