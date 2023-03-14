---
published: false
---
## Active Directory Pentesting: Gathering Credentials in 2023

One of the primary tasks for a penetration tester during internal penetration tests is to gather or extract credentials. These credentials can be found in different locations, including scripts in SMB shares, kerberoastable users, or directly from the host in memory. In this blog post, we will explore various methods for obtaining credentials in Active Directory environments, which involves attacks over the wire and in memory.


## Network-Based

First, let's talk about network-based attacks and initial access vectors. 

To begin, we will explore two commonly used Windows protocols, NetBIOS Name Service (NBNS) and Link-Local Multicast Name Resolution (LLMNR). These protocols are used by Windows hosts to perform name resolution as a fallback when DNS fails. Although it may seem unlikely that organizations with properly configured DNS would fall victim to attacks that exploit these protocols, as long as these default protocols remain enabled, they can be vulnerable to such attacks.

Now that we have discussed these protocols, we will focus on attacks specific to them, which typically involve either relaying or poisoning. In this post, we will cover the later method, but I encourage readers to explore blog posts on relaying, such as [ANNO 2022 LINK] and [BLACK HILLS Gabriel].


# NetBIOS and LLMNR Spoofing

An NBNS and LLMNR spoofing attack occurs when an attack manipulates broadcast traffic being sent over the network to force an authentication attempt to the attacker machine, rather the requested resource on the respective server. This type of attack is known as a Man-in-the-Middle (MitM). Typically the go-to tool for most testers using Linux is [responder], however, a new tool namely, [pretender], has started to gain some traction. For now we will stick with responder. 

*To perform this attack we must be present on the same local network as our victim machine*

Let's start out by running responder in analyze mode, this mode is preferred when first running the tool to identify if LLMNR/NBNS protocols are in use without affecting the network.

[PIC OF RESPONDER IN ANALYZE MODE]

As you can see responder is actively listening for various network protocols that have been highlighted. Let's simulate an authentication attempt from a our victim machine by browsing a non-existent share in our explorer window by specifying a UNC path to our attacker machine.

[AUTH SIMULATION]

Now if we go back to our responder tab, we will notice a user authentication attempt via SMB to our kali machine. Additionally we notice an NTLMv2 hash which we can take offline and crack with a tool like [hashcat], [john the ripper], or even [npk] (cloud ftw!). For this illustration we will utilize hashcat with the rockyou wordlist. Take the full NTLMv2 hash as is, and put this into a text file named hash.txt

`hash.txt`
```
NTLMV2 HASH SIR
```

To crack the NTLMv2 Hash we first need to identify which hashcat mode to use, we can simply use grep for this.

```
HASHCAT MODE GREP
```

Looks like our mode will be 5600, let's setup hashcat accordingly

```
HASHCAT CRACKING COMMAND
```

In about 5 minutes with a fairly normal GPU (2070 Super) we crack our user's hash in about five minutes. During real engagements I have found success using the original rockyou.txt list. However, it doesn't work the best and I have had to resort to a bigger wordlist like RockYou2021.txt which is over 90GB of data! This combined with [clem6969 ruleset] has yielded great results.

Even though we got an authentication attempt this was only because we directly attempt to access a SMB resource on our attacker machine. 

In a real-world situation we would utilize responder without the `-A` flag like so:

```
RESPONDER WITH NO A FLAG
```

This way we are actively spoofing any LLMNR/NBNS and even MDNS requests that are broadcasted through out the local network. 

Along with relaying hashes, we can also perform protocol downgrade attacks in an active directory environment that still has NTLMv1 in use, which is common for older networks that contain legacy devices such as Windows NT 4, Windows 2000, Windows XP, and Windows Server 2003. 

I will leave this an exercise to the reader with the following resources:

- [NTLM Downgrade Attack Explained]
- [Practical Attacks Against NTLMv1]

Additionally a larger resource regarding attacks with responder can be found at the following [hackerarticles link]

The next protocol attack we will cover is DHCPv6 Poisoining. By default on Windows environments, IPv6 is enabled and even preffered over IPv6. And furthermore, when a comptuer is boots up, restarts, or a network cable is attached it makes a request for an IPv6 configuration. We can abuse this (feature) in Windows to intercept a machine account authentication attempt over HTTP, which can later be relayed with a tool like [impacket's ntlmrelayx] to perform actions via LDAP. Relaying authentication to LDAP allows attackers to perform attacks such as domain enumeration, ACL attacks, and computer takeover via RBCD abuse. 


```
MITM6 COMMAND
```



Now that we have covered the two most common poisoning attacks for initial access let's look at an alternative path to gaining credentials initially. Introducting LDAP Pass back attacks.

# LDAP Pass Back

Often we see printers.

We use a tool like gowitness or eyewitness to identify all webservers. 



```
NETCAT COMMAND LISTENER
```

[CHANGE SETTINGS]

```
NETCAT RECEIVING CREDS
```

yay!

```
TESTING CREDS WITH SOME TOOL
```

and we can now begin enumeration of the environment.


We have now covered three possible ways to gain access from an uncredentialed perspective, it's time to focus on gaining additional credentials from an authenticated perspective.

# Kerberoasting

Once credentials have been obtained, a very common attack for adversaries to perform is kerberoasting. Kerberoasting is an attack whereby an adversary is able to obtain the password by requesting a service ticket for any account configured with a Service Principal Name (SPN). 



```
impacket getuserspns.py
```

The output above shows users configured with an SPN along with the respective hashes for each account configured with an SPN.

```
HASHCAT
```

Now we have additional cred!




# GPP

you'd be surprised man.


```
impacket gpppassword.py
```

# Exposed shares

manspider dat jawn.
attackers do this all the time


searching for keywords creds
```
manspider keyword search
```

searching for files

```
manspider files
```

searching for domain admins in powershell scripts
```
maspider domain admin
```


```
manspider f
```

*Side Note*

PowerHuntShares good post-da tool


# Sniffing Packets

often overlooked, still handy
pcredz



I saved this attack for last, because we dont like to lock out accounts


# Password spraying

pw spraying is when we try one password with many accounts. we use talon or kerbrute for this.



# Host-Based
- extract creds from pw managers, browsers, dpapi
- lsass (dragoncastle, lsassy)
- searching for creds manually

NOT FINISHED....
