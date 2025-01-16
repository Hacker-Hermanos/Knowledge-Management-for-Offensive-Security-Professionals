<%*
let machineIp = await tp.system.prompt("Introduce machine's IP");
-%>
## Objective

___
```ad-info
title:Objective 

- Keep track of the external discovery/enumeration process
```
___

## Discovery

~~~ad-info
title:Machine Information

```txt
<% machineIp %> {HostName}
```

> *Add this info to /etc/hosts* once determined. (Could use netexec or similar tools for this)
~~~

### Port Scans

> [Nmap Cheat Sheet 2024: All the Commands & Flags](https://www.stationx.net/nmap-cheat-sheet/)

~~~ad-check 
title:TCP Port Scan

```bash
sudo nmap <% machineIp %> -A -p- -sC -sV -Pn -oN <% machineIp %>_nmap
```
*Screenshot*
___

___
~~~

~~~ad-check
title:UDP Port Scan
collapse:yes
```bash
sudo nmap <% machineIp %> -sU -A --top-ports 25 --min-rate 5000 -oN <% machineIp %>_nmap_udp
```
*Screenshot*
___

___
~~~

~~~ad-check
title:Vulnerability Scan
collapse:yes
```bash
sudo nmap <% machineIp %> -p- --script vuln --min-rate 800 -Pn -oN <% machineIp %>_nmap_vuln
```
*Screenshot*
___

___
~~~

### DNS (53 TCP)

> [53 - Pentesting DNS | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-dns)

~~~ad-check
title: Query Domain Name

```bash
nslookup                    
> server <% machineIp %>
> <% machineIp %> OR {HostName}
```
*Screenshot*
___


___
~~~

~~~ad-check
title:Query DNS Records

```bash
nslookup -type=ANY FQDN <% machineIp %>

```

*Screenshot*
___

___
~~~

~~~ad-check
title: Zone Transfer

```bash
dig axfr <% machineIp %> {HostName}

```

*Screenshot*
___


~~~

### SNMP (161, 162, 10161, 10162 UDP)

> [161,162,10161,10162/udp - Pentesting SNMP | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp)
> [SNMP RCE | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-snmp/snmp-rce)

~~~ad-check
title:Enumerate Valid Strings

```bash
onesixtyone -c /usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt <% machineIp %>
```
*Screenshot*
___

___
~~~

~~~ad-check
title:Enumerate SNMP Configuration

Get all SNMP info using found string (check write access enabled):

```bash
snmp-check <% machineIp %> -w -c {string}
```

*Screenshot*
___

___
~~~

~~~ad-check
title:Executing Extended Queries

Thanks to extended queries (download-mibs), it is possible to enumerate even more about the system with the following command:

```bash
snmpwalk -v2c -c {string} <% machineIp %> NET-SNMP-EXTEND-MIB::nsExtendOutputFull
```

*Screenshot*
___

___
~~~

~~~ad-check
title:Execute ExtendedObjects

```bash
snmpwalk -v2c -c {string} <% machineIp %> NET-SNMP-EXTEND-MIB::nsExtendObjects

```

*Screenshot*
___

___
~~~

### FTP (21 TCP)

> [21 - Pentesting FTP | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp)

- [ ] Write Permissions?

~~~ad-check
```bash
ftp -nv anonymous@<% machineIp %>
```

*Screenshot*
___

___
~~~

~~~ad-check
Brute-force FTP for default credentials:

```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ftp-betterdefaultpasslist.txt <% machineIp %> ftp
```

*Screenshot*
___

___
~~~

### SSH Brute Force (22)

~~~ad-check
title:Brute-Force SSH for default credentials:

```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/ssh-betterdefaultpasslist.txt <% machineIp %> ssh
```

*Screenshot*
___

___
~~~

### SMTP (24, 465, 587 TCP)

> [25,465,587 - Pentesting SMTP/s | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smtp)

Once you have a domain name to work with and a list of possible usernames (& departments) you can identify department mailboxes that exist on the server using smtp-user-enum:

~~~ad-check
title:SMTP Brute Force

```bash
smtp-user-enum -M RCPT -U wordlist.txt -t <% machineIp %>
```
*Screenshot*
___

___
~~~

### POP and IMAP (110, 995; 143, 993)

> [110,995 - Pentesting POP | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-pop)
> [143,993 - Pentesting IMAP | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-imap)

If any valid users were identified and POP3/IMAP services are running we might want to guess for simple passwords:

*Note*: In order to perform this bruteforce attack, you will need to strip off the '@domain' part and only use the username.

~~~ad-check
title:IMAP Brute Force
```bash
hydra -L users.txt -P users.txt <% machineIp %> -e nsr imap(s)
```

```bash
hydra -L users.txt -P users.txt <% machineIp %> -e nsr pop3(s)
```

*Screenshot*
___

___
~~~

If this attack returns valid credentials, we can connect to POP3/IMAP using the following command to access mailboxes to read messages:

~~~ad-check
title:Logging into POP3/IMAP

```bash
nc -nv <% machineIp %> 110
USER {user}
PASS {pass}
LIST # show messages
RETR {number} # read message
```

*Screenshot*
___



~~~

~~~ad-note
title:Steps to Reproduce: Phishing Attack
collapse:yes

If there is any indicative of users expecting a link, we can try to launch a phishing attack by accessing SMTP once again and sending a link that points to a listener:

```bash
nc -nv <% machineIp %> 25
helo hacker
MAIL FROM: (impersonated user)@domain
RCPT TO: (target user)@domain

```

We start a listener:

```bash
rlwrap nc -nlvp 80
```

We write the body, containing a link and send it:

```bash
DATA
Subject: Phishing

Hello,

This is Andy from IT, unfortunately we had a problem with our database. We need you to please register at http://ATTKIP/. Please let me know if you encounter any problems.

Regards,
IT
.
```

Once we add that final dot the message is sent, keep checking the listener to see if any callbacks are received.
~~~

### RPC (135, 593 TCP)

> [135, 593 - Pentesting MSRPC | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/135-pentesting-msrpc)

~~~ad-check
title:Logging into RPC

```bash
rpcclient <% machineIp %> -U 'user%pass' 
```

```bash
rpcclient <% machineIp %> -U 'guest%'
```

*Screenshot*
___


~~~

### SMB (445 TCP)

> [139,445 - Pentesting SMB | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb)

- [ ] Write Access in XXX

~~~ad-check
title:Listing Available Shares

```bash
smbclient -L <% machineIp %> -U '%'
```

```bash
smbclient -L <% machineIp %> -U 'guest%'
```
*Screenshot*
___


```bash
netexec smb <% machineIp %> --shares -u '' -p ''
```
*Screenshot*
___


~~~

~~~ad-info
title:Reminder

To download files recursively in smbclient, use:

```bash
smb: \> mask ""
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```
~~~

### LDAP (389, 636, 3268, 3269)

> [389, 636, 3268, 3269 - Pentesting LDAP | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ldap)
> You will need the machine's fully qualified domain name (FQDN) for this. 

~~~ad-check
title:Get Distinguished Name

```bash
ldapsearch -H ldap://EXAMPLE.htb.local/ -x namingcontexts 
```
OR
```bash
ldapsearch -H ldap://EXAMPLE.htb.local/ -x namingcontexts -s base
```
*Screenshot*
___



~~~

~~~ad-check
title:Query all objects

```bash
ldapsearch -H ldap://EXAMPLE.htb.local/ -x -b 'DC=htb,DC=local' '(objectClass=*)'
```

*Screenshot*
___


~~~

~~~ad-check
title:Query users

```bash
ldapsearch -H ldap://EXAMPLE.htb.local/ -x -b 'DC=htb,DC=local' '(objectClass=person)'
```

*Screenshot*
___


~~~

~~~ad-check
title:Make a user wordlist using LDAP (for bruteforcing later)

```bash
ldapsearch -H ldap://FOREST.htb.local/ -x -b 'DC=htb,DC=local' '(objectClass=person)' | grep -i samaccountname | cut -d ' ' -f 2 > users_LDAP.txt
```

*Screenshot*
___


~~~

### Kerberos (88)

> [GitHub - ropnop/kerbrute: A tool to perform Kerberos pre-auth bruteforcing](https://github.com/ropnop/kerbrute)
> If we have a list of names (from a website maybe) we can convert it to a possible users wordlist, and then we can enumerate valid usernames if kerberos is open using kerbrute:

~~~ad-info
title:Username Wordlist creation

```bash
namemash.py names.txt > possible_users.txt
```
~~~

~~~ad-check
title:Brute Forcing Kerberos
Now we can feed this wordlist into kerbrute:

```bash
kerbrute userenum --dc <% machineIp %> -d DOMAIN -t 20 possible_users.txt
```

*Screenshot*
___

___
~~~

### SQL Brute Force (3306, 1433, 5432 ...)

~~~ad-check
title:Brute-force MySQL, MSSQL, PostgreSQL for default credentials:

```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/mysql-betterdefaultpasslist.txt <% machineIp %> mysql
```

```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/mssql-betterdefaultpasslist.txt <% machineIp %> mssql
```

```bash
hydra -C /usr/share/seclists/Passwords/Default-Credentials/postgres-betterdefaultpasslist.txt <% machineIp %> postgres
```

*Screenshot*
___

___
~~~

### Other Ports (PORTS)

~~~ad-check
```txt
nc -nv <% machineIp %> {PORT}
```
*Screenshot*
___

___
~~~


### Web Application - {Port}

> Duplicate this section for each HTTP/S open port present in the machine. Not all checks are applicable always.

> URLs are using "http", remember to change if applicable. You can use Command Palette > Search and Replace http://<% machineIp %>/

#### CheckList

- [ ] SSL Certificate
- [ ] Subdomains
- [ ] Fingerprints
- [ ] Directory Brute Force
- [ ] Directory Brute Force 403s
- [ ] Thorough Brute Force Wordlists
- [ ] Thorough Brute Force Files
- [ ] Spider
- [ ] Screenshots
- [ ] Interesting Locations
- [ ] Source code Examination
- [ ] Media Metadata Analysis

#### SSL Certificate 

~~~ad-check
title:SSL Certificate Screenshot

Paste screenshot here, take note of subdomains
~~~

#### Subdomain Brute Force

> Needs Hostname

~~~ad-check
title:Brute Force Subdomains

```bash
gobuster vhost -u http://{hostname} -t 100 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt --append-domain -q 
```

```bash
wfuzz -u http://{hostname} -H "Host: FUZZ.{hostname}" -w /usr/share/secLists/Discovery/DNS/subdomains-top1million-5000.txt --hh 33560
```
*Screenshot*
___

___
~~~

#### Fingerprinting

~~~ad-check
title:Fingerprinting

*Technologies Enumeration*
___
```bash
whatweb http://<% machineIp %>/ | tr -s ',' '\n'
```
Screenshot:
___

___

*Request Headers*
___
```bash
curl -iX GET http://<% machineIp %>/
```
Screenshot:
___

___

*Web App Firewalls*
___
```bash
wafw00f http://<% machineIp %>/ -a
```
Screenshot:
___

___
~~~

#### Directory Brute Forcing + Spidering

> [GitHub - gustanini/dirspider](https://github.com/gustanini/dirspider)
> -r for recursion

~~~ad-check
title:Existing files/directories (Directory Bruteforcing + Spidering)

```bash
dirspider.sh http://<% machineIp %>/ [-r]
```
*Screenshot*
___

___
~~~

~~~ad-check
title:403's (forbidden sites)

```bash
dirsearch -u http://<% machineIp %>/ -r --deep-recursive -F -t 100 -x 404,400,500
```
*Screenshot*
___

___
~~~

#### API

~~~ad-check
title:API Enumeration

- Create a wordlist for API enumeration using patterns:

```bash
for i in $(seq 1 15); print {GOBUSTER}/$i >> api.list
```

- Check for valid locations:

```bash
gobuster dir -u http://<% machineIp %>/ -t 100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -p api.list -t 100 --exclude-length 0
```
*Screenshot*
___

___
~~~

#### Thorough Directory Brute Forcing

> Optional Check

~~~ad-check

- Try with these wordlists

```txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt # 220k+ words
/usr/share/seclists/Discovery/Web-Content/raft-medium-words.txt # 60k+ words
/usr/share/seclists/Discovery/Web-Content/raft-large-words.txt # 100k+ words
```

```bash
dirsearch -u http://<% machineIp %>/ -r --deep-recursive -F -t 100 -x 404,400,500 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
*Screenshot*
___

___
~~~

#### Files

~~~ad-check
title:File Enumeration 

Remember to look for files specifying their extensions if applicable:

```bash
dirsearch -u http://<% machineIp %>/ -r --deep-recursive -F -t 100 -x 404,400,500 -e php, pdf, aspx, html, htm
```
*Screenshot*
___

___
~~~

#### Spider

> Check spider report for URLs that might have been missed by directory brute force

~~~ad-check
title:Spidering 

```bash
gospider -S list -d 5 -t 50 --sitemap --robots > spider.out
```
*Screenshot*
___

___
~~~

#### Screenshots

> If the web application is large, use `gowitness` to take screenshots to visually go through all locations

~~~ad-check
title:Screenshots

```bash
gowitness file -f spider.out -t 20
```

```bash
xdg-open .
```
*Screenshots*
___

___
~~~

#### Source Code Analysis

> Take note here of any findings embedded in the source code here + screenshots:

~~~ad-info
Location: http://<% machineIp %>/


~~~

#### Media Metadata Analysis 

~~~ad-check
title:Metadata Analysis

Remember to download any interesting pieces of media for examination with exiftool. Look for usernames, programs, dates:

```bash
wget http://<% machineIp %>/file
```

```bash
exiftool -a -u file
```
*Screenshot*
___

___
~~~

#### Other Notes

**Notes for HTTP Discovery Here**

