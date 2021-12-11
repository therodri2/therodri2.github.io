---
title: "Attacking Kerberos - ASRepRoast"
date: 2021-12-11T00:20:30+01:00
categories:
  - Guides
tags:
  - Active Directory
  - Windows
  - OSCP
share: false
#layout: single
#classes: wide
excerpt_separator: <!--more-->
---

# ASRepRoast

### Preface
This is the first vulnerabilty we will cover in Active Directory series. The reason is because you only need a list of valid usernames to check if this vulnerability exists, and, if it does, you are very likely going to be able to enumerate the whole domain after getting credentials.

A valid list of users can be obtained using several methods. There's a good tool called [Kerbrute](https://github.com/ropnop/kerbrute) which allows for user enumeration and even password bruteforcing. We will be using this technique today.

This vulnerabilty has to do with Kerberos. Kerberos is probably the main authentication  (not authorization) service used by Active Directory. Although there are alternatives, this is probably the service used the most.

This means that Kerberos is there to identify users that are willing to use another service, but it's up to those services to make sure that the user has got enough privileges. The only thing Kerberos does is identify the username.

There's a lot of information online about how Kerberos works and  technical aspects of this attack. This is really out of the scope of this guide. We want to keep things simple and, from a pentester point of view, you don't really need to fully understand the underlying attack. However, I really suggest investigating a bit more. [This](https://ethicalhackingguru.com/how-asreproasting-works-and-how-to-defend-against-it/) is a good resource to start.
<br/>
<br/>

### Requirements on the Domain Controller
For an ASRepRoast attack to work, users must have a flag enabled on Active Directory's Server Manager. This flag is **disabled** by default, so, in real life, not all domains are vulnerable to this attack. However, you will be pleased to know that it's, still, a very common vulnerability.

###### How can we do that on our local lab? 
Well, spawn your Domain Controller and open the Windows Server Manager. 

Click on **Tools -> Active Directory Users and Computers** and rick-click on the user that we want to make vulnerable. Open the properties, click the *Account* tab and scroll all the way down. You will find **"Do not require Kerberos preauthentication"** option there, which you have to flag.

In my case, I will make `m.verstappen` vulnerable.

![[10preauth.png]]
<br/>
<br/>

### Exploitation
We will be simulating the whole enumeration and exploitation process. This is how you would do it in a real engagement.
###### NMap
We always start with an NMap scan:

![[1nmap.png]]

We can see that our Domain Controller has kerberos listeninig on port 88. This means we can use kerbrute to check for valid usernames. However, we don't have any information on the domain yet, and we need something to get at least a potential userlist.

###### SMB Enumeration
Probably 99% of the times you find SMB in a machine you want to start by enumerating it. 

Enumerating SMB can be a bit tricky. Some machines use SMBv1, whereas others use SMBv2. Furthermore, some of them allow a **null session** to login (which means logging in with no username and no password), some others allow **guest** usernames with any random password, similar to the anonymous user of FTP services, and some others only allow proper identification.

This means that you really have to spend a bit of time enumerating SMB with different tools. Good tools are SMBMap, enum4linux, crackmapexec and a few metasploit modules. 

We are not really going to get into SMB enumeration here. If you ever have to enumerate SMB, have a look at [HackTricks.xyz's guide](https://book.hacktricks.xyz/pentesting/pentesting-smb). This is probably one of the best websites you can have in your bookmarks as a pentester.

Without further more, let's see what we can find:

![[2smbenumdomain.png]]

We've managed to obtain the domain name, which, in this case, is *fia.local*. However, if we try to ping fia.local we won't have any success because our machine doesn't know how to resolve it. Let's add it to our */etc/hosts* file to enable proper communication!

![[3addtohost.png]]

Now, if we ping the domain, it works!

![[4pingworks.png]]

###### Generating a potential userlist
We've just seen that the domain name is *fia.local*. If you have read my [Guide on Setting up Active Directory](https://shroudri.github.io/guides/setting-up-active-directory/), you will know that this is a formula-1 based domain.

Fia stands for "Fédération Internationale de l'Automobile". From an attacker's point of view, if we are attacking something to do with motorsport, we'd rather get a good wordlist of drivers.

![[5gettingdrivers.png]]

I've found a website from which we can parse the names of all drivers. 

![[6parsingdrivers.png]]

> curl -s https://www.skysports.com/f1/standings | grep -i data-long-name | awk -F "=" '{print $2}' | tr -d \\"

![[7driverlist.png]]

This gives us a list with all the drivers involved in this year's F1 championship. Now we need to generate usernames based on this list. I wrote a simple python script for that called [username_generator](https://github.com/shroudri/username_generator). You can download it and use it to generate a potential list of usernames. It covers the most typical combinations used in enterprises, and you can even set it to generate uppercase permutations. It's worth having a look at it!

![[8generateusernames.png]]

Nice! That's a good list to use. Other ways to get potential usernames include:
- Using RPC service if enabled (there's a tool called RPCclient for that)
- Getting a list of names+surnames from "About Us" sections on websites. 
- Sometimes, users can be extracted from the metadata of images and documents
- There are many other ways to get a userlist. Be creative!
	
Let' move on!
<br/>

###### Using Kerbrute to find the actual login usernames
As I said before, we will be using [Kerbrute](https://github.com/ropnop/kerbrute) userenum option. This allows us to check for valid usernames from a wordlist. Let's use the wordlist we've just generated!

![[9validusernames.png]]

Kerbrute is showing us that 2 usernames are valid and actually exist on the domain:
```
╰─ ./kerbrute userenum ~/ctf/fia/usernames -d fia.local --dc 192.168.0.26

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: v1.0.3 (9dad6e1) - 12/11/21 - Ronnie Flathers @ropnop

2021/12/11 23:16:42 >  Using KDC(s):
2021/12/11 23:16:42 >   192.168.0.26:88

2021/12/11 23:16:42 >  [+] VALID USERNAME:       m.verstappen@fia.local
2021/12/11 23:16:42 >  [+] VALID USERNAME:       l.hamilton@fia.local
2021/12/11 23:16:42 >  Done! Tested 168 usernames (2 valid) in 0.025 seconds

```

Nice! Time to check if any of those usernames is ASRepRoasteable!

###### Exploiting ASRepRoasteable users
[Impacket](https://github.com/SecureAuthCorp/impacket) is a collection of open-source tools for working with network protocols. Impacket's scripts come by default on security-oriented distros like Parrot and Kali. However, installing them is just a matter of cloning the repository if you are using any other distro.

Impacket-collection is a must-have for pentesting Active Directory. You can get more information on what every tool does by having a look at [this link](https://www.secureauth.com/labs/open-source-tools/impacket/).

For ASRepRoasting, we have to use **impacket-GetNPUsers.py** script. 

By typing `impacket-GetNPUsers.py` we can get a help message:

![[11getnpusers.png]]


```
There are a few modes for using this script

1. Get a TGT for a user:

        GetNPUsers.py contoso.com/john.doe -no-pass

For this operation you don't need john.doe's password. It is important tho, to specify -no-pass in the script, 
otherwise a badpwdcount entry will be added to the user

2. Get a list of users with UF_DONT_REQUIRE_PREAUTH set

        GetNPUsers.py contoso.com/emily:password or GetNPUsers.py contoso.com/emily

This will list all the users in the contoso.com domain that have UF_DONT_REQUIRE_PREAUTH set. 
However it will require you to have emily's password. (If you don't specify it, it will be asked by the script)

3. Request TGTs for all users

        GetNPUsers.py contoso.com/emily:password -request or GetNPUsers.py contoso.com/emily

4. Request TGTs for users in a file

        GetNPUsers.py contoso.com/ -no-pass -usersfile users.txt

For this operation you don't need credentials.

```

We don't have any credentials yet, so we can only go for the 4th option.

![[12verstappenvulnerable.png]]

And we see that it returns 2 remarkable things:
1. M.Verstappen's AS-REP hash, which is easily crackable by John or Hashcat!
2. L.Hamilton is not vulnerable to ASRepRoasting attack. This is because he doesn't have **DONT_REQ_PREAUTH** set.


###### Cracking
We can copy the hash and proceed to cracking it straight ahead with JTR.

![[13crackedhash.png]]

It took really little time to get the password. Now that we have credentials we can further continue with enumeration and exploitation of the domain. 

The ASRepRoasting attack has already finished here. However, I'm going to continue to give you some examples of what we can do with these credentials:
- Use LDAP to dump more information about the domain
- If winrm is enabled, use evil-winrm to spawn a shell
- Use Bloodhound to visualize the path to Domain Admins
- Use impacket tools to spawn pseudo-shells
- Check if these credentials are valid against other services (SSH, website authentication, FTP, SMB)
- Be creative!


###### Using LDAP to dump more information
LDAP is "LDAP’s primary function is enabling users to find data about organizations, persons, and more. It accomplishes this goal by storing data in the LDAP directory and authenticating users to access the directory. It also provides the communication language that applications require to send and receive information from directory services." [(Source)](https://sensu.io/blog/what-is-ldap)

Because LDAP is running on port 389 and we have valid credentials, we can dump that information! Note that sometimes credentials are not required, but it's a very uncommon thing.

There's a tool called **LDAPDomainDump** that generates html and xml files with all the information about the domain. Because it's HTML, we can use our browser to display those files beautifully. So, the first thing we do is move to */var/www/html* and remove anything that's in there. Then, run LDAPDomainDump  to generate juicy files.

![[14ldapdomaindump.png]]

Start apache2 service by running `sudo service apache2 start` and browse to http://localhost

![[15ldapdirectory.png]]

Clicking on one of these links will take us to tables showing users, groups, permissions and much more information about the domain. For example, this is the domain_users.html file:

![[15ldapusers.png]]

Again, there are several more interesting ways to continue the exploitation but this is a good starting point. Now that we have a list of all the usernames, we can try bruteforcing to see if any of them has a weak password maybe! 