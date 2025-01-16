## Objective

___
```ad-info
title:Objective 

- Keeping track of exploitation progress and other useful information. 
```
___

## Exploitation Progress

> List of all Forests, Domains and Hosts. Add a check once the object is fully compromised.

- [ ] Forest
	- [ ] Domain
		- [ ] Host
		- [ ] Host
- [ ] Forest
	- [ ] Domain
		- [ ] Host
		- [ ] Host
- [ ] Forest
	- [ ] Domain
		- [ ] Host
		- [ ] Host

> Keep track of the exploitation path you followed for reporting purposes. Also keep a list of backdoors/persistence established throughout the network.

```ad-note
title: Exploitation Progress (Attack Path and Backdoors)

**Attack Path** 
___
1. XXX to XXX
2. XXX to XXX
3. XXX to XXX

**Backdoors**
___
1. Backdoor on XXX via - 
2. Backdoor on XXX via -
3. Backdoor on XXX via -
```

## Stolen Credentials

> Harvested credential material.

```ad-note
title:Stolen Credentials

*Domain\User:Password/Hash*
*Domain\User:Password/Hash*
*Domain\User:Password/Hash*
___
> For local users use Hostname\User instead of Domain\User
```

```ad-note
title: Kerberos Tickets (B64)
collapse:true

___
*Domain\User:Ticket*
*Ticket info screenshot:*
___
*Domain\User:Ticket*
*Ticket info screenshot*
___
> Include a screenshot of the ticket's information to keep track of expiracy info and SPNs etc.
```

**To-do**: Add admonition block for other alternative credential material, such as ADCS.

## Trusts

> Domain and forest trusts information for quick reference. 

```ad-note
title:All Trusts

Tool: PowerView
___
~~~powershell
Get-DomainTrustMapping
~~~
___
*Screenshot*:
___

___
> Can also use Bloodhound (4.3.1 recommended) or AD-Module to retrieve this info.
```

## MSSQL DB Links

> MSSQL DB Links for quick reference.

```ad-note
title: MSSQL DB Links

Tool: PowerUpSQL
___
~~~powershell
Get-SQLInstanceDomain | Get-SQLServerLinkCrawl -verbose
~~~
*Screenshot*
___

___
```

## Password Cracking

> Track stolen hashes that need cracking.

```ad-note
title:Stolen Hashes for Cracking

*Hash Type*
*Domain\User:HASH*
___
*Hash Type*
*Domain\User:HASH*
___
*Hash Type*
*Domain\User:HASH*
___
*Hash Type*
*Domain\User:HASH*
___
```

