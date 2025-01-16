## Objective

___
```ad-info
title:Objective 

- This note is used to keep track of the domain's discovery/enumeration, privilege escalation and lateral movement processes (once a foothold is established)
```
___

## Discovery

```ad-info

Commands starting with verb-ad use the AD Module. [GitHub - samratashok/ADModule: Microsoft signed ActiveDirectory PowerShell module](https://github.com/samratashok/ADModule)
Commands starting with verb-domain use Powerview. [PowerSploit/Recon/PowerView.ps1 at master · PowerShellMafia/PowerSploit · GitHub](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)
```


### Users

*All users*

~~~powershell
get-aduser -filter * | select SamAccountName, enabled, sid
~~~

*Screenshot*

___

___

*Users with SPNs*


~~~powershell
get-adserviceaccount -filter * | select samaccountname, SID, objectclass, ObjectGuid, distinguishedname, enabled | format-list
~~~

*Screenshot*

___

___

*Users with Administrative Privileges*

~~~powershell
get-aduser -filter * | ?{$_.samaccountname -match "admin"} | Select-Object samaccountname 
~~~

*Screenshot*

___

___

*Domain Admins Members*

~~~powershell
get-adgroupmember "Domain Admins" -Recursive
~~~

*Screenshot*

___

___

*Enterprise Admins* (run this on the root forest using -server)

~~~powershell
get-adgroupmember "Enterprise Admins" -Recursive
~~~

*Screenshot*

___

___

### Groups

*Names + SID List* (only copy non default ones here. RID >= 1000)

~~~powershell
get-adgroup -filter * | select name, sid
~~~

*Screenshot*

___

___

*Admin Groups*


~~~powershell
get-adgroup -filter 'Name -like "*admin*"' | select Name,sid
~~~

*Screenshot*

___

___

*Get User Memberships* (always check new compromised users).

[From PowerTools.ps1](https://github.com/gustanini/PowershellTools/blob/main/PowerTools.ps1)

~~~powershell
Get-NestedGroupMembership USER
~~~

*Screenshot*

___

___

### Computers

*All Computers, names, machine account names and SIDs*

~~~powershell
Get-ADComputer -filter * | select dnshostname, SamAccountName, SID
~~~

*Screenshot*

___

___

*Accessible Computers (ICMP)*

~~~powershell
get-adcomputer -filter * | %{Test-Connection -Count 1 -ComputerName $_.DNSHostName -erroraction silentlycontinue} | select -expandproperty Address
~~~

*Screenshot*

___

___

*Domain Controllers*

~~~powershell
Get-ADDomainController -filter * | select-object ComputerObjectDN, domain, enabled, forest, hostname, ipv4address, isglobalcatalog, ldapport, name, operatingsystem, serverobjectdn
~~~

*Screenshot*

___

___

### GPO

> These GPO Commands use powerview. You can import GPOModule as an alternative.

*List of GPOs*

~~~powershell
Get-DomainGPO | select displayname, objectguid
~~~

*Screenshot*

___

___

*GPOs + Location*

~~~powershell
Get-DomainGPO | select displayname, gpcfilesyspath, distinguishedname, objectguid | fl
~~~

*Screenshot*

___

___

**High Value Targets**

*Restricted Group GPO*

> Identify users or groups granted admin access due to a GPO and display computers under their administrative control.

___

~~~powershell
Get-DomainGPOLocalGroup
~~~

*Screenshot*

___

___

*Admin Access Through GPO 1*

> Take domain computers, determine users and groups with admin access on them through GPO (hit or miss)

~~~powershell
get-domaincomputer | Get-DomainGPOComputerLocalGroupMapping -ErrorAction SilentlyContinue
~~~

*Screenshot*

___

___

*Admin Access Through GPO 2*


> Take domain users, determine where they have admin access through GPO

~~~powershell
Get-Domainuser | Get-DomainGPOUserLocalGroupMapping -ErrorAction SilentlyContinue
~~~

*Screenshot*
___

___

### OU

*OU List + GUIDs*

~~~powershell
Get-ADOrganizationalUnit -Filter * | Select-Object name, DistinguishedName, ObjectGUID, LinkedGroupPolicyObjects | fl
~~~

*Screenshot*

___

___

*GPOs applied on an OU*

> This one can be a pain to enumerate individually, rather get all GPOs in one terminal and all OUs in another and correlate.

*first terminal*

~~~powershell
Get-ADOrganizationalUnit -Filter * | select Name,DistinguishedName,LinkedGroupPolicyObjects | fl
~~~

*second terminal*

~~~powershell
get-domaingpo | select displayname,name | fl
~~~

*Screenshots*

___

___

*List of Computers in a OU* (try without last where-object filter, then filter if needed)

> Look at the distinguished name, first CN is computer name, first OU is the organizational unit.

~~~powershell
Get-ADOrganizationalUnit -filter * | %{Get-ADComputer -filter * -SearchBase $_} | ?{$_.SamAccountName -notmatch "student"}
~~~

*Screenshot*

___

___

**High Value Targets**

*Users/Groups with admin access using all OUs*

~~~powershell
(Get-DomainOU).distinguishedname | %{Get-DomainComputer -SearchBase $_} | Get-DomainGPOComputerLocalGroupMapping -ErrorAction SilentlyContinue
~~~

*Screenshot*

___

___

*List users/groups with admin access using a specific OU*

> Use OU Distinguished Name.

~~~powershell
(Get-DomainOU -Identity 'OU=Mgmt,DC=us,DC=techcorp,DC=local').distinguishedname | %{Get-DomainComputer -SearchBase $_} | Get-DomainGPOComputerLocalGroupMapping -ErrorAction SilentlyContinue
~~~

*Screenshot*

___

___

### ACLs 

```ad-info

- *ObjectDN* = target object
- *IdentityReference* = Object that has permissions over target
- *ActiveDirectoryRights* = Rights over target
```

*Compromised Users ACLs*

> Run against every new user you compromise and their group memberships (enumerate their nested group memberships).

[The following command uses PowerTools.ps1](https://github.com/gustanini/PowershellTools)

~~~powershell
Find-ADInterestingACL -Identity "user1|group1"
~~~

*Screenshot*

___

___

*Get Domain Admins' ACLs*

> This command will generate a ton of output and is not really that useful unless used for a specific attack. (Use Distinguished Name)

~~~powershell
(get-acl "AD:\CN=Domain Admins,CN=Users,DC=us,DC=techcorp,DC=local").access | ?{$_.ActiveDirectoryRights -match 'write'} | select ActiveDirectoryRights, AccessControlType, IdentityReference | ?{$_.IdentityReference -notmatch 'BUILTIN|NT|Exchange'} | fl
~~~

*Screenshot*

___

___

*Get High Value Groups/Users ACLs*

> This command will generate a ton of output and is not really that useful unless used for a specific attack. (Use Distinguished Name)

~~~powershell
(get-acl "AD:\CN=MachineAdmins,OU=Mgmt,DC=us,DC=techcorp,DC=local").access | ?{$_.ActiveDirectoryRights -match 'write'} | select ActiveDirectoryRights, AccessControlType, IdentityReference | ?{$_.IdentityReference -notmatch 'BUILTIN|NT|Exchange'} |fl
~~~

*Screenshot*

___

___

### Domains

*Current Domain Information*

~~~powershell
get-addomain | select Name, InfrastructureMaster, DistinguishedName, domainsid, LinkedGroupPolicyObjects, ChildDomains, ComputersContainer, DomainControllersContainer, Forest, ParentDomain, DNSRoot
~~~

*Screenshot*

___

___

*Passwords Policy* 

```powershell
(Get-DomainPolicyData).systemaccess
```

*Screenshot*

___

___

*Kerberos Policy*

~~~powershell
(Get-DomainPolicy).KerberosPolicy
~~~

*Screenshot*

___

___

### Trusts

*Forest Information*

~~~powershell
Get-ADForest
~~~

*Screenshot*

___

___

*Domains in Current Forest*

~~~powershell
(Get-ADForest).domains
~~~

*Screenshot*

___

___

*Global Catalogs*

~~~powershell
(Get-ADForest).globalcatalogs
~~~

*Screenshot*

___

___

*Current Domain Trusts*

~~~powershell
Get-ADTrust -Filter *
~~~

*Screenshot*

___

___

*Current Forest Trusts*

~~~powershell
Get-ADTrust -Filter 'intraForest -ne $True' -Server (Get-ADForest).Name
~~~

*Screenshot*

___

___
### User Hunting

> First create a *list of computers for the current domain*

~~~powershell
mkdir c:\temp
(Get-ADComputer -Filter *).dnshostname > C:\Temp\Computers.txt
~~~

*Domain Admins Sessions*

Check access flag checks if your user has admin access there

~~~powershell
Find-DomainUserLocation [-CheckAccess]
~~~

*Screenshot*

___

___

*Sessions on Commonly Used Servers*

~~~powershell
Find-DomainUserLocation -Stealth
~~~

*Screenshot*

___

___

*Computers where custom Group users have sessions*

~~~powershell
Find-DomainUserLocation -UserGroupIdentity "StudentUsers"
~~~

*Screenshot*

___

___

*Find Computers where current user has admin access*

> PSRemotingLocalAdminAccess

~~~powershell
. .\Find-PSRemotingLocalAdminAccess.ps1
~~~

~~~powershell
Find-PSRemotingLocalAdminAccess -ComputerFile C:\Temp\Computers.txt
~~~

> WMILocalAdminAccess

~~~powershell
. .\Find-WMILocalAdminAccess
~~~

~~~powershell
Find-WMILocalAdminAccess -ComputerFile C:\Temp\Computers.txt
~~~

*Screenshot*

___

___

### Shares

*Computer Shares*

> If -CheckShareAccess is passed, then only shares the current user has read access to are returned

~~~powershell
# powerview
Find-DomainShare [-CheckShareAccess]
~~~

~~~powershell
# ad-module
Get-ADComputer -filter * -properties Name | Select -ExpandProperty Name | % {Get-CIMInstance -Class win32_share -ComputerName $_ -ErrorAction SilentlyContinue}
~~~

*Screenshot*

___

___

### Port Scan

> It will be faster to perform a port scan from the compromised machine than from out attacker machine since we are connected to the other machines directly.

Use `Invoke-Portscan.ps1` from PowerSploit to perform this discovery.

```powershell
Invoke-Portscan -Hosts web06 -TopPorts 50
```

*Screenshot*

___

## Privilege Escalation / Lateral Movement

### Checklist

- [ ] Kerberoasting Attacks
	- [ ] [[AD Privilege Escalation - Kerberoasting Attack]]
	- [ ] Targeted Kerberoasting
	 - [ ] [[AD Privilege Escalation - AS-REP Roasting Attack]]
	- [ ] Targeted AS-REP Roasting
- [ ] [[AD Privilege Escalation - LAPS]]
- [ ] [[AD Privilege Escalation - gMSA]]
- [ ] Kerberos Delegation Attacks 
	- [ ] [[AD Privilege Escalation - Kerberos Unconstrained Delegation Attack]], [[Unconstrained Delegation Abuse (AD)]], [[Unconstrained Delegation + Printer Bug + DCSync (AD)]]
	- [ ] [[AD Privilege Escalation - Kerberos Constrained Delegation Attack]], [[Constrained Delegation Abuse (AD)]]
	- [ ] [[AD Privilege Escalation - Kerberos Resource Based Constrained Delegation Attack]], [[Resource Based Constrained Delegation + Fake Machine Account (AD)]]
- [ ] [[AD Privilege Escalation - DNSAdmins Group]]
- [ ] MSSQL Attacks 
	- [ ] [[MSSQL Discovery (AD)]]
	- [ ] [[MSSQL Execution (AD)]]
	- [ ] [[MSSQL Techniques (AD)]]
	- [ ] [[MSSQL NTLM Relay Attack (AD)]]
	- [ ] [[MSSQL Privilege Escalation (AD)]]
	- [ ] [[MSSQL Links Lateral Movement (AD)]]
	- [ ] [[MSSQL Custom Assemblies Execution (AD)]]
	- [ ] [[MSSQL Links Local Privilege Escalation (AD)]]
	- [ ] [[MSSQL Credential Acces via UNC Path Injection (AD)]]
- [ ] Cross Domain Attacks
	- [ ] [[AD Privilege Escalation - XDomain ADCS Enrolee Supplies Subject Abuse]]
	- [ ] [[AD Privilege Escalation - XDomain Azure AD Integration Attacks]]
	- [ ] [[AD Privilege Escalation - XDomain Trust Attacks]]
- [ ] [[AD Privilege Escalation - ADCS Shadow Credentials]]
- [ ] Cross Forest Attacks
	- [ ] [[AD Privilege Escalation - XForest Kerberoasting Attack]]
	- [ ] [[AD Privilege Escalation - XForest Kerberos Unconstrained Delegation]]
	- [ ] [[AD Privilege Escalation - XForest Kerberos Constrained Delegation]]
	- [ ] [[AD Privilege Escalation - XForest FSP Abuse]]
	- [ ] [[AD Privilege Escalation - XForest PAM Trust Abuse]]

> *Note*: Every time you compromise a new user/group/computer, remember to go back and enumerate *interesting ACLs* and *local admin access* using them and also impersonate them and run `Find-WMILocalAdminAccess.ps1` `Find-PSRemotingLocalAdminAccess.ps1` to check for local admin access.

*Screenshots*

___

___

## Persistence

### Checklist

- [ ] [[Persistence via Kerberos Tickets (AD)]] (To-Do, split this note in each category)
	- [ ] Silver Tickets
	- [ ] Golden Tickets
	- [ ] Diamond Tickets
- [ ] [[Persistence via msDS-AllowedToDelegateTo ACE (AD)]]
- [ ] [[Persistence via Scheduled Tasks (AD)]]
- [ ] [[Persistence via Skeleton Key (AD)]]
- [ ] [[Persistence via DSRM Password (AD)]]
- [ ] [[Persistence via AdminSDHolder ACE (AD)]]
- [ ] [[Persistence via ACE Rights Abuse (AD)]]
- [ ] [[Persistence via Security Descriptor ACE (AD)]]
- [ ] [[Persistence via Custom SSP (AD)]]
- [ ] [[Persistence via DCShadow (AD)]]
- [ ] [[Persistence via Golden gMSA (AD)]]

> *Note*: Establish persistence, write down how it was performed and its output below. Add backdoors to `Credentials` file.

*Screenshots*

___

___