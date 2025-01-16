---
Topics:
  - "[[01 - Pentesting]]"
  - "[[01 - Red Team]]"
Types:
  - "[[02 - Cheatsheets]]"
tags:
  - cheatsheet
  - activedirectory
  - discovery
date created: Tuesday, July 2nd 2024
date modified: Tuesday, July 2nd 2024
---

# ADPT Discovery

## Objective

___

```ad-info
title: Objective

To showcase various common Active Directory discovery techniques.
```

___

## Abstract

___

```ad-abstract
title: Overview

This note contains commands used for enumerating Active Directory objects within a set using Powerview, the AD-Module, Bloodhound and Adalanche. 
```

___

## MITRE Tactics

___

| ID                                                | Name                                                           | Description                                                                         |
| ------------------------------------------------- | -------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| [TA0007](https://attack.mitre.org/tactics/TA0007) | [Discovery](https://attack.mitre.org/tactics/TA0007)           | The adversary is trying to figure out your environment.                             |

___

## Examples

___

### Using PowerView

~~~ad-example 
title:AD Discovery Using Powerview

[Powerview v.3.0](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)

[Powerview Wiki](https://powersploit.readthedocs.io/en/latest/)

- **Get Current Domain:** Get-Domain
- **Enumerate Other Domains:** `Get-Domain -Domain <DomainName>`
- **Get Domain SID:** `Get-DomainSID`
- **Get Domain Policy:**

```powershell
Get-DomainPolicy

#Will show us the policy configurations of the Domain about system access or kerberos
Get-DomainPolicy | Select-Object -ExpandProperty SystemAccess
Get-DomainPolicy | Select-Object -ExpandProperty KerberosPolicy
```

- **Get Domain Controllers:**

```powershell
Get-DomainController
Get-DomainController -Domain <DomainName>
```

- **Enumerate Domain Users:**

```powershell
# Save all Domain Users to a file
Get-DomainUser | Out-File -FilePath .\DomainUsers.txt

# Will return specific properties of a specific user
Get-DomainUser -Identity [username] -Properties DisplayName, MemberOf | Format-List

# enumeration user logged on a machine
Get-NetLoggedon -ComputerName <ComputerName>

# enumeration Session Information for a machine
Get-NetSession -ComputerName <ComputerName>

# enumeration domain machines of the current/specified domain where specific users are logged into
Find-DomainUserLocation -Domain <DomainName> | Select-Object UserName, SessionFromName
```

- **Enum Domain Computers:**

```powershell


Get-DomainComputer -Properties OperatingSystem, Name, DnsHostName | Sort-Object -Property DnsHostName

#Enumerate Live machines
Get-DomainComputer -Ping -Properties OperatingSystem, Name, DnsHostName | Sort-Object -Property DnsHostName

```txt

- **Enum Groups and Group Members:**

```powershell
#Save all Domain Groups to a file:
Get-DomainGroup | Out-File -FilePath .\DomainGroup.txt

#Return members of Specific Group (eg. Domain Admins & Enterprise Admins)
Get-DomainGroup -Identity '<GroupName>' | Select-Object -ExpandProperty Member
Get-DomainGroupMember -Identity '<GroupName>' | Select-Object MemberDistinguishedName

#enumeration the local groups on the local (or remote) machine. Requires local admin rights on the remote machine
Get-NetLocalGroup | Select-Object GroupName

#Enumerates members of a specific local group on the local (or remote) machine. Also requires local admin rights on the remote machine
Get-NetLocalGroupMember -GroupName Administrators | Select-Object MemberName, IsGroup, IsDomain

#Return all GPOs in a domain that modify local group memberships through Restricted Groups or Group Policy Preferences
Get-DomainGPOLocalGroup | Select-Object GPODisplayName, GroupName
```

- **Enumerate Shares:**

```powershell


#Enumerate Domain Shares
Find-DomainShare

#Enumerate Domain Shares the current user has access
Find-DomainShare -CheckShareAccess

#Enumerate "Interesting" Files on accessible shares
Find-InterestingDomainShareFile -Include *passwords*

```txt

- **Enum Group Policies:**

```powershell
Get-DomainGPO -Properties DisplayName | Sort-Object -Property DisplayName

#enumeration all GPOs to a specific computer
Get-DomainGPO -ComputerIdentity <ComputerName> -Properties DisplayName | Sort-Object -Property DisplayName

#Get users that are part of a Machine's local Admin group
Get-DomainGPOComputerLocalGroupMapping -ComputerName <ComputerName>
```

- **Enum OUs:**

```powershell


Get-DomainOU -Properties Name | Sort-Object -Property Name

```txt

- **Enum ACLs:**

```powershell
# Returns the ACLs associated with the specified account
Get-DomaiObjectAcl -Identity <AccountName> -ResolveGUIDs

#Search for interesting ACEs
Find-InterestingDomainAcl -ResolveGUIDs

#Check the ACLs associated with a specified path (e.g smb share)
Get-PathAcl -Path "\\Path\Of\A\Share"
```

- **Enum Domain Trust:**

```powershell


Get-DomainTrust
Get-DomainTrust -Domain <DomainName>

#Enumerate all trusts for the current domain and then enumerates all trusts for each domain it finds
Get-DomainTrustMapping

```txt

- **Enum Forest Trust:**

```powershell
Get-ForestDomain
Get-ForestDomain -Forest <ForestName>

#Map the Trust of the Forest
Get-ForestTrust
Get-ForestTrust -Forest <ForestName>
```

- **User Hunting:**

```powershell
#Finds all machines on the current domain where the current user has local admin access
Find-LocalAdminAccess -Verbose

#Find local admins on all machines of the domain
Find-DomainLocalGroupMember -Verbose

#Find computers were a Domain Admin OR a spesified user has a session
Find-DomainUserLocation | Select-Object UserName, SessionFromName

#Confirming admin access
Test-AdminAccess
```

**Priv Esc to Domain Admin with User Hunting:** \
I have local admin access on a machine -> A Domain Admin has a session on that machine -> I steal his token and impersonate him -> Profit!
~~~

### Using AD Module

~~~ad-example
title:AD Discovery Using AD-Module

- **Get Current Domain:** `Get-ADDomain`
- **Enum Other Domains:** `Get-ADDomain -Identity <Domain>`
- **Get Domain SID:** `Get-DomainSID`
- **Get Domain Controlers:**

  ```powershell
  Get-ADDomainController
  Get-ADDomainController -Identity <DomainName>
  ```

- **Enumerate Domain Users:**

  ```powershell
  Get-ADUser -Filter * -Identity <user> -Properties *

  #Get a spesific "string" on a user's attribute
  Get-ADUser -Filter 'Description -like "*wtver*"' -Properties Description | select Name, Description
  ```

- **Enum Domain Computers:**

  ```powershell
  Get-ADComputer -Filter * -Properties *
  Get-ADGroup -Filter *
  ```

- **Enum Domain Trust:**

  ```powershell
  Get-ADTrust -Filter *
  Get-ADTrust -Identity <DomainName>
  ```

- **Enum Forest Trust:**

  ```powershell
  Get-ADForest
  Get-ADForest -Identity <ForestName>

  #Domains of Forest Enumeration
  (Get-ADForest).Domains
  ```

- **Enum Local AppLocker Effective Policy:**

  ```powershell
  Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
  ```
~~~

### Using BloodHound

~~~ad-example
title:AD Discovery Using BloodHound
#### Remote BloodHound

[Python BloodHound Repository](https://github.com/fox-it/BloodHound.py) or install it with `pip3 install bloodhound`

```powershell
bloodhound-python -u <UserName> -p <Password> -ns <Domain Controller's Ip> -d <Domain> -c All
```

#### On Site BloodHound

```powershell
#Using exe ingestor
.\SharpHound.exe --CollectionMethod All --LdapUsername <UserName> --LdapPassword <Password> --domain <Domain> --domaincontroller <Domain Controller's Ip> --OutputDirectory <PathToFile>

#Using PowerShell module ingestor
. .\SharpHound.ps1
Invoke-BloodHound -CollectionMethod All --LdapUsername <UserName> --LdapPassword <Password> --OutputDirectory <PathToFile>
```
~~~

### Using Adalanche

~~~ad-example
title:AD Discovery Using Adalanche

#### Remote Adalanche

```bash
# kali linux:
./adalanche collect activedirectory --domain <Domain> \
--username <Username@Domain> --password <Password> \
--server <DC>

# Example:
./adalanche collect activedirectory --domain windcorp.local \
--username spoNge369@windcorp.local --password 'password123!' \
--server dc.windcorp.htb
## -> Terminating successfully

## Any error?:

# LDAP Result Code 200 "Network Error": x509: certificate signed by unknown authority ?

./adalanche collect activedirectory --domain windcorp.local \
--username spoNge369@windcorp.local --password 'password123!' \
--server dc.windcorp.htb --tlsmode NoTLS --port 389

# Invalid Credentials ?
./adalanche collect activedirectory --domain windcorp.local \
--username spoNge369@windcorp.local --password 'password123!' \
--server dc.windcorp.htb --tlsmode NoTLS --port 389 \
--authmode basic

# Analyze data 
# go to web browser -> 127.0.0.1:8080
./adalanche analyze
```
~~~

___

## Resources

___

| Link                                                         | Description                                       |
| ------------------------------------------------------------ | ------------------------------------------------- |
| [ldapdomaindump](https://github.com/dirkjanm/ldapdomaindump) | Information dumper via LDAP.                      |
| [adidnsdump](https://github.com/dirkjanm/adidnsdump)         | Integrated DNS dumping by any authenticated user. |
| [ACLight](https://github.com/cyberark/ACLight)               | Advanced Discovery of Privileged Accounts.        |
| [ADRecon](https://github.com/sense-of-security/ADRecon)      | Detailed Active Directory Recon Tool.             |

```ad-seealso
title: **Related Vault Links**

~~~dataview
LIST FROM #activedirectory 
and #discovery
~~~
```

___

