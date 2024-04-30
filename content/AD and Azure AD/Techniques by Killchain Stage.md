
Thanks to infosecn1nja for the basic concept/overall organization.

ATT&CK mappings are very much a work in progress, as ATT&CK to Killchain mappings are often a pain in general.

Big TODO: Integrate defense techniques from AD Hardening and Defense
# Discovery

## Active Directory Federation Services (ADFS)

## AppLocker Config

## Data Hunting

### Domain SQL Servers

## Email

### Sensitive Files

## LAPS

## LDAP

Enumeration via LDAP

https://github.com/dirkjanm/ldapdomaindump
## SPN Scanning

## User Hunting

### BloodHound

### Group Scoping

### PowerUpSQL

## Password Hunting



# Privilege Escalation

## AD ACLs
## Active Directory Certificate Services (ADCS) Abuse

## DCShadow

## DFSCoerce

https://github.com/Wh04m1001/DFSCoerce

## Domain Trust Abuse
## DNSAdmins Privilege Abuse

## Exchange 
## GPO Permission Abuse

## Kerberos stuff

Some general info on delegation:

https://shenaniganslabs.io/media/Constructing%20Kerberos%20Attacks%20with%20Delegation%20Primitives.pdf

https://blog.redxorblue.com/2019/12/no-shells-required-using-impacket-to.html

### Unconstrained Delegation

### Constrained Delegation

### Resource-Based Constrained Delegation
### CVE-2020-17049 (Bronze Bit)

* Overview: https://www.netspi.com/blog/technical/network-penetration-testing/cve-2020-17049-kerberos-bronze-bit-overview/
* Theory deep dive: https://www.netspi.com/blog/technical/network-penetration-testing/cve-2020-17049-kerberos-bronze-bit-theory/
* Exploitation: https://www.netspi.com/blog/technical/network-penetration-testing/cve-2020-17049-kerberos-bronze-bit-attack/
### MS14-068

## MSSQL

## NTLM Relay and LLMNR/NBNS Spoofing
## PetitPotam
## sAMAccountName Spoofing

## Password Hunting

### SYSVOL and GP Preferences

## Red Forest (MS ESAE Attacks)

https://learn.microsoft.com/en-us/security/privileged-access-workstations/esae-retirement

https://infocondb.org/con/troopers/troopers18/attack-and-defend-microsoft-enhanced-security-administrative-environment

https://blog.netwrix.com/2023/02/06/active-directory-red-forest/
## RID Hijacking
## Zerologon

Using CS BOF in Sliver (TODO)


# Lateral Movement

## Automation

* GoFetch
* DeathStar
* ANGRYPUPPY (CS)

TODO (among others): ANGRYPUPPY with Sliver
## MSSQL 

## Pass-the-hash (PTH)

## Password Spraying

## System Center Configuration Manager (SCCM)

## WSUS


# Evasion and OPSEC

## AMSI Bypass

## AppLocker Bypass

### .NET Assemblies

g_amsiContext corruption

## Device Guard Bypass
## EDR Evasion

![[content/assets/silence_EDR_crowdstrike.jpg]]

Hand - *EDR Evasion*

Ideas of sections:

* General
* Direct Syscalls and Indirect Syscalls
* sRDI
* Process argument spoofing
* Unhooking Windows APIs 
* EDR sensor detection
* .NET Assembly obfuscation (e.g. https://www.r-tec.net/r-tec-blog-net-assembly-obfuscation-for-memory-scanner-evasion.html)
* etc.
## Event Logs
## HoneyToken evasion
## MS ATA and ATP Evasion

https://www.blackhat.com/docs/eu-17/materials/eu-17-Thompson-Red-Team-Techniques-For-Evading-Bypassing-And-Disabling-MS-Advanced-Threat-Protection-And-Advanced-Threat-Analytics.pdf

* DLL hijacking
* DiagTrack or firewall service comm blocking 
* Relevant PPL bypasses
* Being trusted installer
* WMI local namespace abuse
* DC exclusion
* etc.
## OPSEC

### C2 OPSEC 

https://www.cobaltstrike.com/blog/opsec-considerations-for-beacon-commands

**Sliver stuff**

* Go binaries and you
* Customization
* Extensions in C and Rust
* basics
* Sliver's best-in-business COMSEC

## Powershell ScriptBlock Logging Bypass
## Sysmon evasion


# Credential Dumping

## DCSync

## Kerberos Stuff

### Kerberoasting

### AS-REP Roasting

## LLMNR/NBT-NS poisoning

## LSASS 

Requires local admin/SeDebugPrivilege

### MultiDump

(Somewhat) stealthy dumping via `ProcDump` or `comsvc.dll`:

https://github.com/Xre0uS/MultiDump

NOTE: MultiDump is (sometimes) detected these days, requires modfication.
## NTDS.DIT

## Misc

* Plaintext creds in AD
## Security Accounts Manager (SAM)

## Windows Credential Manager/Vault

* Via DPAPI Abuse etc.

# Persistence

## AD ACL abuse (again)
## AdminSDHolder and SDProp Abuse
## DCShadow (again)

## Directory Services Restore Mode (DSRM) abuse

## GPOs
## Kerberos

### Diamond Ticket

### Golden Ticket

## Security Service Provider (SSP) abuse

### Silver Ticket

## SeEnableDelegatioPrivilege

## SID History

## Skeleton Keys