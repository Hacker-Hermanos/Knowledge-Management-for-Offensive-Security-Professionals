## Objective

___
```ad-info
title:Objective 

- This note is used to keep track of the target Linux machine's discovery, privilege escalation, persistence and credential access processes
- This note also contains different cheatsheets and resources to aid in those methodologies
- Feel free to remove any sections that are non-applicable to keep this note compact.
```

> You can execute discovery scripts like LinEnum or LinPeas to retrieve this information or do it manually.
> Try using simpler scripts to retrieve this info before running overwhelming scripts like `linpeas.sh`.

```ad-seealso
title: Useful Tools
collapse:yes
*Scripts*:
___
[[Payloads Canvas.canvas|QuickEnum - Payloads Canvas]]
[LinEnum.sh](https://github.com/rebootuser/LinEnum)
[linpeas.sh](https://linpeas.sh/)
___

```
___

## Internal Discovery Notes

> Run the commands yourself or let the script do it for you. Either way paste relevant screenshots here for future reference.

*Current User Information*

~~~bash
id
~~~

*Screenshot*:
___

___

*Local Users*

~~~bash
cat /etc/passwd | egrep -v 'nologin|false'
~~~

___

___

*Host Name*

~~~bash
hostname
~~~

*Screenshot*:
___

___

*Kernel Information*

~~~bash
uname -a && cat /etc/issue && cat /etc/os-release
~~~

*Screenshot*:
___

___

*Process Information*

~~~bash
ps aux
~~~

*Screenshot*:
___

___

*Network Information*

~~~bash
ip a || ifconfig && route
~~~

~~~bash
ss -plunt || sockstat -4l
~~~

*Screenshot*:
___

___

*Firewall Rules *

~~~bash
cat /etc/iptables/rules.v4
~~~

*Screenshot*:
___

___

*Scheduled Tasks*

~~~bash
cat /etc/crontab; ls -lah /etc/cron*
~~~

> **Careful with this one**

~~~bash
crontab -l 
~~~

*Screenshot*:
___

___

*Installed Software*

~~~bash
ls /opt/ && ls /etc/
~~~

*Screenshot*:
___

___

*Files with Insecure Permissions*

~~~bash
ls -la /etc/shadow && ls -la /etc/passwd
~~~

~~~bash
ls -la /etc/shadow && ls -la /etc/passwd
~~~

~~~bash
find / -writable -type d 2>/dev/null
~~~

~~~bash
find / -perm -4000 2>/dev/null
~~~

*Screenshot*:
___

___

*Drives Information*

~~~bash
cat /etc/fstab
~~~

*Screenshot*:
___

___

*Credentials/Passwords* (from LinPeas or LinEnum)

*Screenshot*:

___

___

## Credential Access

> Because this section is extensive, just screenshot important findings and paste them in the findings block below. All blocks are collapsed by default to make navigation less painful.

### Password Hunting

~~~ad-note
title:Useful Password Hunting Credential Access Commands
collapse:yes
Look for files containing interesting strings in current directory recursively

```bash
echo "\nFiles containing \"pass\" string in $(pwd)\n" && grep -rli pass .
```

```bash
echo "Files containing \"user\" string in $(pwd)\n" && grep -rli user .
```

Discover passwords using LinEnum (-t thorough, -k keyword)

```bash
./LinEnum.sh -t -k password
```

[Password Hunting – Linux Privilege Escalation](https://juggernaut-sec.com/password-hunting-lpe/)
~~~

~~~ad-note
title:Hunting for Passwords
collapse:yes
###### History Files
___
Check history files under `/home` for passwords
```bash
find /home/ -type f -name "*_history*" 2>/dev/null
```
Read history files quickly
```bash
find /home/ -type f -name "*_history*" -exec cat {} \; 2>/dev/null
```

###### Config Files (/home/)
___
Check config files in `/home` for passwords.
```bash
find /home/ -type f -name "*config*" 2>/dev/null
```
Read them quickly 
```bash
find /home/ -type f -name "*config*" -exec cat {} \; 2>/dev/null
```

###### Config Files (/var/www/[html])
___
Check config files in webroot used by webserver to connect to database (`config.php`, `info.php` or similar)
```bash
find /var/www/ -type f -name "*config*" 2>/dev/null && find /var/www/ -type f -name "*php*" 2>/dev/null
```
> cat interesting files manually as there will potentially be many useless documents here.

###### Directory (/var/spool/mail/)
___
```bash
find /var/spool/mail -type f 2>/dev/null
```

###### Directory (/opt/)

> Check this directory thoroughly for config files or credentials.

```bash
ls -la /opt
```

```
find /var/www/ -type f -name "*config*" 2>/dev/null
```

```
find /var/www/ -type f -name "*config*" -exec cat {} \; 2>/dev/null
```
~~~

*Screenshots*:

___

___

### Hunting for SSH Keys + SSH Abuse

~~~ad-note
title:Hunting for SSH Keys
collapse:yes

Finding .ssh directories 

```bash
find /home/ -type d -name ".ssh" 2>/dev/null
```
___

```dataview
list from #ssh 
and #linux
sort file.name asc
```
~~~

*Screenshots*:

___

___

### Kerberos on Linux

> If kerberos is installed, proceed with the list of ansible notes to exploit it.

~~~ad-note
title:Kerberos on Linux Discovery
collapse:true 

Enumerate credential cache file path using environment variable

```bash
env | grep KRB5CCNAME
```

Enumerate krb5.conf file. Probably needs root access

```bash
cat /etc/krb5.conf
```

Enumerate /tmp/ for Kerberos tickets.

```bash
ls /tmp/
```

___

```dataview
list from #kerberos
and #linux
sort file.name asc
```
~~~

*Screenshots*:

___

___

### Ansible

> If ansible is installed, proceed with the list of ansible notes to exploit it.

~~~ad-note
title:Ansible Discovery
collapse:true 

Enumerate installation

```bash
ansible
```
___

```dataview
list from #ansible 
and #linux
sort file.name asc
```
~~~

*Screenshots*:

___

___

### Artifactory

```ad-note
title: Artifactory Discovery
collapse:yes

An Artifactory installation can be discovered using ps or on nmap scans on port 8081. 

~~~bash
ps aux | grep artifactory
~~~
___
~~~dataview
list from #artifactory 
sort file.name asc
~~~
```

*Screenshots*:

___

___

### Databases

~~~ad-note
title:Database Discovery 
collapse:yes
Connecting to a database (mydb) using mysql

- If you use a [`--password`](https://dev.mysql.com/doc/refman/8.0/en/connection-options.html#option_general_password) or `-p` option and specify a password value, there must be _no space_ between [`--password=`](https://dev.mysql.com/doc/refman/8.0/en/connection-options.html#option_general_password) or `-p` and the password following it.

```bash
mysql -h localhost -u myname -ppassword mydb
```

MYSQL Commands Cheatsheet: [3306 - Pentesting Mysql | HackTricks | HackTricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-mysql)
~~~

*Screenshots*:

___

___

## Privilege Escalation

> *Note*: Now that you found viable escalation vector, perform privilege escalation, write down how it was performed and screenshots below.

I performed privilege escalation via XXX.

## Persistence 

> You can create a backdoor user, or add your own ssh key to authorized_keys or more intricate attacks.
> Keep track of this backdoor on your tracker note.

~~~ad-example
title:Creating a Backdoor User in Linux

- Create your backdoor user.

```bash
adduser backdoor
```
- Paste the following password in the prompt.
```txt
Password123!
```
- Add "backdoor" user to `sudo` group
```bash
usermod -aG sudo backdoor
```
~~~

*Screenshots*:

___

___

## Port Redirection

> Establish a socks proxy session after performing credential access *if necessary* to gain access to the next victim on the network.

```ad-seealso
title: Pivoting Notes
collapse:yes

[Port Forwarding – Linux Privilege Escalation](https://juggernaut-sec.com/port-forwarding-lpe/)
[[Chisel HTTP Tunneling Cheatsheet]]
[[Command and Control - Port Forwarding (Socat)]]
```

> Source: [How to Create SSH Tunneling or Port Forwarding in Linux](https://www.tecmint.com/create-ssh-tunneling-port-forwarding-in-linux/)

~~~ad-example
title:Local SSH Port Forwarding (Linux)
collapse:yes 

This type of port forwarding lets you connect from your local computer to a remote server. Assuming you are behind a restrictive firewall or blocked by an outgoing firewall from accessing an application running on port **3000** on your remote server.

You can forward a local port (e.g **8080**) which you can then use to access the application locally as follows. The `-L` flag defines the port forwarded to the remote host and remote port.

```bash
ssh admin@server1.example.com -L 8080:server1.example.com:3000
```

Adding the `-N` flag means do not execute a remote command, you will not get a shell in this case.

```bash
ssh -N admin@server1.example.com -L 8080:server1.example.com:3000
```

The `-f` switch instructs ssh to run in the background.

```bash
ssh -f -N admin@server1.example.com -L 8080:server1.example.com:3000
```

Now, on your local machine, open a browser, instead of accessing the remote application using the address **server1.example.com:3000**, you can simply use `localhost:8080` or `192.168.43.31:8080`, as shown in the screenshot below.

~~~

~~~ad-example
title: Remote SSH Port Forwarding (Linux)
collapse:yes 

Remote port forwarding allows you to connect to a port from a remote machine.

```bash
sudo service sshd start 
```

Next run the following command to forward port **5000** on the remote machine to port **3000** on the local machine.

```bash
ssh -f -N admin@TARGET_IP -R 5000:localhost:3000
```
~~~

~~~ad-example
title:Dynamic SSH Port Forwarding (SOCKS) (Linux)
collapse:yes

Dynamic port forwarding sets up your machine as a **SOCKS proxy server** that listens on port **1080**, by default.

You can enable dynamic port forwarding using the **-D** option. The following command will start a SOCKS proxy on port **1080** allowing you to connect to the remote host.

```bash
ssh -f -N -D 1080 admin@server1.example.com
```

From now on, you can make applications on your machine use this SSH proxy server by editing their settings and configuring them to use it, or with proxychains. Note that the **SOCKS** proxy will stop working after you close your SSH session.
~~~

~~~ad-example
title:Transparent Socks Proxy Session using SSHuttle (Linux)
collapse:yes

- Establish a socks proxy session using SSHuttle

```bash
sshuttle -r backdoor@192.168.1.1 172.16.1.0/24
```
~~~

```ad-example
title:Port Forwarding with NetSh (Windows)
collapse:yes

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

netsh interface portproxy delete v4tov4 listenport=9090 listenaddress=localhost
```

*Screenshots*:

___

___

## Lateral Movement 

```ad-note
title:Internal Links List of Lateral Movement Techniques
collapse:true 

~~~dataview
list from #lateralmovement 
sort file.name asc
~~~
```

> Enumerate access to the other machines in your target list using your new credentials.

```bash
crackmapexec PROTOCOL -k ips.lst -u "USER" -p "PASSWORD"
```

*Screenshots*:

___

___

*Create a link here to the next machine's note, when you click the link, a new note will be created for that machine.* 

