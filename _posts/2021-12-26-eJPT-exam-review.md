---
title: "eLearnSecurity eJPT Exam Review"
date: 2021-12-26T00:18:30+01:00
categories:
  - Reviews
tags:
  - elearnsecurity
  - ejpt
share: false
#layout: single
#classes: wide
excerpt_separator: <!--more-->
---

For a long time I've been thinking about getting my first cybersecurity certification. Today, I've passed the eJPT at my first attempt, and wanted to leave a bit of feedback through this post.

<!--more-->

## The exam
The exam is pretty straight forward. Upon begining it, you receive an *.ovpn* file, a user dictionary, a password dictionary and a *.pcap* file. Those will come in handy for the exam later.

After being in THM and HTB for a bit more than a year, I got used to getting footholds, user, privesc and root. Well, the eJPT is definitely not like that. It's based on **enumeration**. And I can't stress this enough. Simple enumeration is enough to get what's required to pass.

One thing that I would like to note is that OWASP-ZAP came very handy thanks to it's Automated Scan tool. For some reason, BurpSuite is very widely used in cybersecurity, even though the Community Edition's intruder is latched, and OWASP-ZAP doesn't get the credit it deserves. For entry-level certifications like this one, automation is expected and allowed and OWASP-ZAP will help you enumerate a website.

I highly sugest using a FOSS to take notes. I have been using Obsidian for a while now and I'm very happy with it. You are not required to submit any notes but staying organized is critical when engaging several nodes.

And remember, if you have usernames and passwords, bruteforce everything you can. And if you have some hashes, crack, add the passwords to your dictionary and bruteforce again. Always be cracking (or bruteforcing)


## INE
All that you need to know is covered in INE's free PTS path. They teach you all the tools you need to know to gather the information you need to pass the exam. If, during the exam, you happen to get stuck, go back to INE's platform and see if you have followed their steps on something.


The only thing that I found a bit misleading is the networking / routing part. The labs are slightly different from the eJPT exam. I got stuck there for a couple hours, until I thought about what kind of devices do you need in a network to be able to route to different networks. But, apart from that, the experience with INE is very positive and I'm even consiering purchasing a month or two before attempting eCPPTv2.

*Update*: I've just realized that, in order to access labs, you need a Premium (not a monthly) suscripton to INE. Keep that in mind if you are thinking about doing the same!
## Useful tools and commands
- Smbmap, smbclient, enum4linux, crackmapexec
- Hydra 
- John the Ripper and Hashcat
- Nmap, including enumeration scripts. 
- Sqlmap, burpsuite, OWASP-ZAP
- *sudo ip route add (subnet) via (gateway)*