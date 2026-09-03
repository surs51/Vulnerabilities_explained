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

- WPAD Hijacking - (Force proxy auth over NTML)
- SMB Relay      - (Relay captured NTML auth to another SMB)
- NTML Relay     - (Relay auth to SMB/HTTP/LDAP/MSQL/others...)
- IPv6/mitm6     - (Abuse IPv6 NameResolver to trigger NTML auth)


