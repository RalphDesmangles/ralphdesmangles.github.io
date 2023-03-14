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


- MITM
- Kerberoasting
- GPP
- Exposed shares
- Password spraying
- LDAP Pass Back
- Sniffing Packets


# Host-Based
- extract creds from pw managers, browsers, dpapi
- lsass (dragoncastle, lsassy)
- searching for creds manually

NOT FINISHED....
