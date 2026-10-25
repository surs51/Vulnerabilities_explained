# ARP Spoofing / MITM

## The main idea behind ARP spoofing is that ARP has no memory of who asked whereas the host just believes the last "X is at Y" it hears. So we just keep telling the victim that we are the gateway, and the gateway that we are the victim. Now each packet flows Victim <-> Us <-> Gateway.

<img width="920" height="560" alt="arp-spoofing-mitm" src="https://github.com/user-attachments/assets/dfde82c8-62e6-48ac-af2d-105dbfca8b35" />

#### Layer 2 - no port - no auth

ARP maps IP-MAC on the local segment, no signature, no session, just a reply

e.g.
> - (Victim) Who has 10.0.0.1? Tell me (10.0.0.5)
> - (Gateway) Me, 10.0.0.1 is at aa:bb:cc...
> - (Us) actually 10.0.0.1 is at OurMAC

The Victim and the Gateway cache our reply and we sit in the middle of both

## So how it all works

Two forged replies do the whole job

- 1st to the Victim -> "10.0.0.1 (The Gateway) is at OurMAC"
- 2nd to the Gateway -> "10.0.0.5 (the Victim) is at OurMAC"

Now the victim's traffic comes to us, and we forward it to the Gateway, so it can also be called a relay

#### Forwarding
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

## Commands

### Bettercap (easy way)
```bash
sudo bettercap -iface wlan0
# we get into the session of Better cap

set arp.spoof.targets <victim's IP> #we can also leave it empty to try and poison the whole subnet
arp.spoof on
net.sniff on   # Watch for information
```
---
But of course - you'll get nothing, just because HTTPS and TLS exist, so we have to go a little further than that
So also check 

- DNS-Spoofing
- HTTP Proxy (Caplets)

### Arpspoof (dsniff, manual)
You'll have to set-up 2 directions, so 2 terminals
```bash
sudo arpspoof -i wlan0 -t 10.0.0.5 10.0.0.1 #show the gateway
sudo arpspoof -i wlan0 -t 10.0.0.1 10.0.0.5 #shot the victim
```
## May be combined with
- SSLStrip / sslstrip2 - (downgrade the HTTPS you now sit in front of)
- DNS Spoofing & Cache Poisoning  - (answer DNS yourself once you're in the path)
- NTLM Relay / SMB Relay - (catch + forward any auth that passes through)
- [Evil Twin / Rogue AP](https://github.com/surs51/Vulnerabilities_explained/blob/fc43f8b006abdd64a64b8e216d39fbe16b694f24/Vulberabilities/Network/Evil_Twin%20%7C%20Rogue_AP.md) - (same MiTM position, done over Wi-Fi instead)


## Gotchas
- `ip_forward` must be 1 or you're just DoSing the victim, not reading them
- Static ARP entries and DAI (Dynamic ARP Inspection) on the switch kill it dead
- It's loud - gratuitous ARP storms light up any IDS, don't spray the whole /24 unless you mean it
- HTTPS/HSTS means you see the flow but not the plaintext - you still need SSLStrip or a cert trick for the juicy bits


