
# Remotely reset expired passwords

Original post [here](https://www.n00py.io/2021/09/resetting-expired-passwords-remotely/). Thanks to n00py. I have rewritten most of this here to be useful to me, plus typed up the commands/code for easier use. n00py's individual images, as opposed to any screenshots by me, will be credited individually. 

## Overview

Suppose we have valid creds, but the password is expired. The following techniques allow us to remotely reset them.

This works in two cases:

1. Where the account's password has simply expired (`STATUS_PASSWORD_EXPIRED`)
2. Where the account has "user must change password at next logon" set (`STATUS_PASSWORD_MUST_CHANGE`)

Does *not* work for:

3. Account disabled (`STATUS_ACCOUNT_DISABLED`)
4. *Account* expired (`STATUS_ACCOUNT_EXPIRED`)

## Outlook Web Access (OWA) / Active Directory Federation Services (ADFS)

Requires exposed webapp. Typical patterns:

**For OWA**

* `mail.<domain>.<tld>`
* `<HOSTNAME>/OWA/`

![[Pasted image 20240403114924.png]]

*Image credit: n00py*

![[Pasted image 20240403114940.png]]

*Image credit: n00py*

**For ADFS:**

* `adfs.<domain>.<tld>`
* `<HOSTNAME>/ADFS/`

## RDP

NOTE: Requires that Network Level Authentication (NLA) not be enforced

NOTE 2: User doesn't actually need RDP permissions

### Step 1

Detect that NLA is not enforced.

**nmap**

```
nmap <target> -p 3389 --script rdp-enum-encryption
```

`SSL: SUCCESS` indicates NLA isn't enforced.

**Metasploit**

```
auxiliary(scanner/rdp/rdp_scanner)
```

### Step 2

Connect to system. Then simply reset the password in the normal Windows way.

## Manual Powershell

Good if we have access to Windows, can access the IPC$ share, and don't want to install anything or drop binaries to disk:


```powershell
function Set-PasswordRemotely {
    [CmdletBinding(DefaultParameterSetName = 'Secure')]
    param(
        [Parameter(ParameterSetName = 'Secure', Mandatory)][string] $UserName,
        [Parameter(ParameterSetName = 'Secure', Mandatory)][securestring] $OldPassword,
        [Parameter(ParameterSetName = 'Secure', Mandatory)][securestring] $NewPassword,
        [Parameter(ParameterSetName = 'Secure')][alias('DC', 'Server', 'ComputerName')][string] $DomainController
    )
    Begin {
        $DllImport = @'
[DllImport("netapi32.dll", CharSet = CharSet.Unicode)]
public static extern bool NetUserChangePassword(string domain, string username, string oldpassword, string newpassword);
'@
        $NetApi32 = Add-Type -MemberDefinition $DllImport -Name 'NetApi32' -Namespace 'Win32' -PassThru

        if (-not $DomainController) {
            if ($env:computername -eq $env:userdomain) {
                # not joined to domain, lets prompt for DC
                $DomainController = Read-Host -Prompt 'Domain Controller DNS name or IP Address'
            } else {
                $Domain = $Env:USERDNSDOMAIN
                $Context = [System.DirectoryServices.ActiveDirectory.DirectoryContext]::new([System.DirectoryServices.ActiveDirectory.DirectoryContextType]::Domain, $Domain)
                $DomainController = ([System.DirectoryServices.ActiveDirectory.DomainController]::FindOne($Context)).Name
            }
        }
    }
    Process {
        if ($DomainController -and $OldPassword -and $NewPassword -and $UserName) {
            $OldPasswordPlain = [System.Net.NetworkCredential]::new([string]::Empty, $OldPassword).Password
            $NewPasswordPlain = [System.Net.NetworkCredential]::new([string]::Empty, $NewPassword).Password

            $result = $NetApi32::NetUserChangePassword($DomainController, $UserName, $OldPasswordPlain, $NewPasswordPlain)
            if ($result) {
                Write-Host -Object "Set-PasswordRemotely - Password change for account $UserName failed on $DomainController. Please try again." -ForegroundColor Red
            } else {
                Write-Host -Object "Set-PasswordRemotely - Password change for account $UserName succeeded on $DomainController." -ForegroundColor Cyan
            }
        } else {
            Write-Warning "Set-PasswordRemotely - Password change for account failed. All parameters are required. "
        }
    }
}
```

Interactive usage:

```
Set-PasswordRemotely
```


Noninteractive usage:

```
# Domain in hewwo.local format

Set-PasswordRemotely <user> <old password> <new password> <domain>
```
## smbpasswd

This and its Impacket noninteractive counterpart are probably the easiest/laziest, and the first thing to try on a standard pentest.

NOTE: Anonymous access to the IPC$ share must be enabled

```
smbpasswd -r <target> -U '<user>'
```

### Impacket, noninteractive version

`smbpasswd.py` in Impacket. Usage:

```
# Target in hewwo.local format

python3 smbpasswd.py <user>:<password>@<target> -newpass <new_password>
```

NOTE: To get this to work without needing a NULL SMB session, unlike the interactive version, if you have valid creds, modify the code to have your creds at the following lines:

```python
if anonymous:
	rpctransport.set credentials(username='<valid_user>', password='valud_password')...
```

## ChangePw

Old third-party Windows utility. Can be downloaded [here](https://www.joeware.net/freetools/tools/changepw/index.htm).

NOTE: Requires access to IPC$ share

NOTE 2: Do not need admin to run

NOTE 3: Even if an anonymous connection isn't performed, ChangePw connects with the current logged-in user. Therefore, will work even with NULL session limitation as long as you have at least one valid user.

NOTE 4: Traffic generated is similar to smbpasswd

Usage:

```cmd
.\changepw.exe /d:<domain> /u:<user> /o:<oldpass> /p:<newpass>
```

## SetADAccountPwd

Part of MS Remote Server Administration Tools (RSAT). Download [here](https://www.microsoft.com/en-gb/download/details.aspx?id=45520), Can also download via powershell (see below)

NOTE: Requires local admin for RSAT installation if not present

### Step 1

Check if RSAT is already installed. If not but you have local admin, you can install from the internet.

```powershell
Add-WindowsCapability -Online -Name RSAT*
```

### Step 2

Usage:

```powershell
Set-ADAccountPassword -Identity <user> -Server <target IP>
```

## Rubeus

Works even if we don't have a plaintext password provided we have either a valid hash or a Kerberos ticket for the user.

### Step 1 

Get a ticket w/ old PW:

**Raw Rubeus.exe**

With password.

```
# Domain just hewwo for example, not hewwo.local

.\Rubeus.exe /user:'<domain>\<user>' /password:<oldpass> /changepw /outfile:<user>.kirbi
```

**With Sliver Rubeus module**

```
# -i to run reflectively (in current Sliver process) rather than fork&exec, usually better opsec

rubeus -i /user:'<domain>\<user>' /password:<oldpass> /changepw /outfile:<user>.kirbi /nowrap
```
### Step 2

Change using ticket

**Raw Rubeus.exe**

```
.\Rubeus.exe changepw /new:<newpass> /ticket:<user>.kirbi
```

**With Sliver Rubeus module**

```
rubeus -i /changepw
```






