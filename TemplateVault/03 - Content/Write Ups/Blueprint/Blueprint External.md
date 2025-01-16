---
Topics:
  - "[[01 - Pentesting]]"
  - "[[01 - Red Team]]"
Types:
  - "[[02 - Write Ups]]"
tags:
  - writeup
date created: Thursday, July 18th 2024
date modified: Friday, July 19th 2024
---

# Blueprint External

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
10.10.15.198 Blueprint
```

> *Add this info to /etc/hosts* once determined. (Could use netexec or similar tools for this)
~~~

### Port Scans

> [Nmap Cheat Sheet 2024: All the Commands & Flags](https://www.stationx.net/nmap-cheat-sheet/)

~~~ad-check 
title:TCP Port Scan

```bash
sudo nmap 10.10.15.198 -A -p- -sC -sV -Pn -oN 10.10.15.198_nmap
```
*Screenshot*

___
```less
Nmap scan report for 10.10.15.198
Host is up (0.26s latency).
Not shown: 57435 closed tcp ports (reset), 8087 filtered tcp ports (no-response)
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
|_http-title: 404 - File or directory not found.
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
| http-methods:
|   Supported Methods: GET HEAD POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-04-11 22:52  oscommerce-2.3.4/
| -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
| -     2019-04-11 22:52  oscommerce-2.3.4/docs/
|_
|_http-title: Index of /
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2009-11-10T23:48:47
| Not valid after:  2019-11-08T23:48:47
| MD5:   a0a4:4cc9:9e84:b26f:9e63:9f9e:d229:dee0
|_SHA-1: b023:8c54:7a90:5bfa:119c:4e8b:acca:eacf:3649:1ff6
| tls-alpn:
|_  http/1.1
445/tcp   open  microsoft-ds Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql        MariaDB (unauthorized)
8080/tcp  open  http         Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
| http-methods:
|   Supported Methods: GET HEAD POST OPTIONS TRACE
|_  Potentially risky methods: TRACE
| http-ls: Volume /
| SIZE  TIME              FILENAME
| -     2019-04-11 22:52  oscommerce-2.3.4/
| -     2019-04-11 22:52  oscommerce-2.3.4/catalog/
| -     2019-04-11 22:52  oscommerce-2.3.4/docs/
|_
|_http-title: Index of /
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  msrpc        Microsoft Windows RPC
49160/tcp open  msrpc        Microsoft Windows RPC
```
___

___
![[Pasted image 20240718220747.png]]
![[Pasted image 20240718221333.png]]
___
~~~

~~~ad-check
title:UDP Port Scan
collapse:yes
```bash
sudo nmap 10.10.15.198 -sU -A --top-ports 25 --min-rate 5000 -oN 10.10.15.198_nmap_udp
```
*Screenshot*
___

___
~~~

~~~ad-check
title:Vulnerability Scan
collapse:yes
```bash
sudo nmap 10.10.15.198 -p- --script vuln --min-rate 800 -Pn -oN 10.10.15.198_nmap_vuln
```
*Screenshot*
___

___
~~~

### RPC (135, 593 TCP)

> [135, 593 - Pentesting MSRPC | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/135-pentesting-msrpc)

~~~ad-check
title:Logging into RPC

```bash
rpcclient 10.10.15.198 -U 'user%pass' 

rpcclient 10.10.15.198 -U 'guest%'
```

*Screenshot*
___
![[Pasted image 20240718220104.png]]
![[Pasted image 20240718220427.png]]

**Guest User Works!**

![[Pasted image 20240718221853.png]]
![[Pasted image 20240718222026.png]]
___
~~~

**Notes**: RPC is accessible as guest, was able to enumerate users.

### SMB (445 TCP)

> [139,445 - Pentesting SMB | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-smb)

- [ ] Write Access in XXX

~~~ad-check
title:Listing Available Shares

```bash
smbclient -L 10.10.15.198 -U '%'
```

```bash
smbclient -L 10.10.15.198 -U 'guest%'
```

```bash
netexec smb 10.10.15.198 --shares -u '' -p ''
```

*Screenshot*
___

___
![[Pasted image 20240718221739.png]]
___

```bash
netexec smb 10.10.15.198 -u 'guest' -p '' -M spider_plus
```

___
![[Pasted image 20240718225718.png]]
___
~~~

**Notes**: Got read access to Users, used spider module and reviewed json file. Nothing of interest here.

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

### Web Application - 443

#### Other Notes

**Notes**: Found a vulnerable program on /.

![[Pasted image 20240718234353.png]]

Got an error when running script against port 80. This port is not vulnerable.

Port 443 shows a certificate error, I proceeded to edit the script to ignore the certificate.

![[Pasted image 20240718235235.png]]

Modified exploit HTTP requests to ignore certificates. (lines 40, 46, 65)

![[Pasted image 20240719000802.png]]

Executing the exploit grants me a privileged shell.

![[Pasted image 20240719000904.png]]

Ran `tasklist`, no AV was found.

![[Pasted image 20240719002155.png]]

Ran `systeminfo` to determine correct architecture.

![[Pasted image 20240719002429.png]]

Generated a meterpreter binary, transferred using `certutil`, started a listener and executed it on the target machine.

![[Pasted image 20240719003641.png]]

![[Pasted image 20240719004051.png]]

Moving onto [[Blueprint Internal]].
