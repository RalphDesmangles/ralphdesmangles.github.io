---
published: false
---
## Active Directory Pentesting: Gathering Credentials in 2023

One of the primary tasks for a penetration tester during internal penetration tests is to gather or extract credentials. These credentials can be found in different locations, including scripts in SMB shares, kerberoastable users, or directly from the host in memory. In this blog post, we will explore various methods for obtaining credentials in Active Directory environments, which involves attacks over the wire and in memory.


## Network-Based

First, let's talk about network-based attacks and initial access vectors. 

To begin, we will explore two commonly used Windows protocols, NetBIOS Name Service (NBNS) and Link-Local Multicast Name Resolution (LLMNR). These protocols are used by Windows hosts to perform name resolution as a fallback when DNS fails. Although it may seem unlikely that organizations with properly configured DNS would fall victim to attacks that exploit these protocols, as long as these default protocols remain enabled, they can be vulnerable to such attacks.

Now that we have discussed these protocols, we will focus on attacks specific to them, which typically involve either relaying or poisoning. In this post, we will cover the later method, but I encourage readers to explore blog posts on relaying, such as [ANNO 2022 LINK] and [BLACK HILLS Gabriel].


# NetBIOS / LLMNR



- MITM
- Kerberoasting
- GPP
- Exposed shares
- Password spraying
- LDAP Pass Back


# Host-Based
- extract creds from pw managers, browsers, dpapi
- lsass (dragoncastle, lsassy)
- searching for creds manually

NOT FINISHED....
