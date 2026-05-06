### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - Finding Sensitive Information <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Finding Sensitive Information



When attacking a service, play detective - every detail matters. Tiny data points (a username, an empty file with a meaningful name) chain into RCE on adjacent services.

Sensitive info to hunt for:

- Usernames
- Email addresses
- Passwords
- DNS records
- IP addresses
- Source code
- Configuration files
- PII

Common services where it leaks:

- File shares
- Email
- Databases

Two prerequisites for finding it:

1. Understand the service and how it works
2. Know what you're looking for
