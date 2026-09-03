# WPAD hijacking

<img width="1289" height="1000" alt="wpad_hijacking" src="https://github.com/user-attachments/assets/509f8e53-30b1-4381-901b-0ae9629db3bc" />

## WPAD (Web-Proxy Auto Discovery)

#### 80 - TCP (HTTP)

As you've seen - it's a Proxy, sooo

e.g.
> - Hey, who's the warden?
> - Me (WPAD)
> - How do I get to facebook?
> - If you're an admin - go straight, if you're an accountant - you don't

Now you see the drill - we can be the first to be that WPAD (file) and give instructions instead of the original WPAD.

And Windows really helps us, by handing user's NTLM challange. Same as LLMNR Poisoning

## May be combined with
- [LLMNR / NBT-NS Poison](LLMNR%20Poisoning.md) - (The name-resolution trick that hands victims to us)
- NTLM Relay     - (Relay the 407-captured auth to SMB/HTTP/LDAP/MSSQL/others...)
- SMB Relay      - (Relay it straight onto another host's SMB)
- IPv6 / mitm6   - (DHCPv6 + rogue DNS to deliver WPAD, bypasses the LLMNR patch)

