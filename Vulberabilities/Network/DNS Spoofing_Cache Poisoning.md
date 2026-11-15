# DNS Spoofing / Cache Poisoning

## DNS - Domain Name System

#### 53 UDP (mostly), TCP for big answers.

DNS turns `bank.com` into an IP. But the problem is - on the wire it's just an unauthenticated Q&A. Nothing is signed on the last hop, so whoever sends the asnwer first is trusted. And if we're part of the path - we're always first.

But there's two flavors of Spoofing

- In-Path Spoofing - We are MITM (ARP, Evil Twin, rogue DHCP..), we see the query and fire the answer instead of DNS resolver.
- Cache poisoning - We just put our forged answer into a resolver's cache, so everyone who asks it - follows your forgery. Tho it becomes much more difficult when we deal with source-port randomization, 0x20 casing, DNSSEC.

## So how it all works

- victim asks for `bank.com`
- we answer first with a forged A record - `bank.com = attacker IP`
- victim connects straight to us, thinking we're the site
- optionally we proxy to the real site so nothing looks broken
- we sslstrip / serve a fake login
- creds and cookies land in our lap

<img width="1000" height="700" alt="dns-spoofing" src="https://github.com/user-attachments/assets/bf217d87-c493-4eda-b620-56a1f0b810e4" />

The redirect is the easy half. Getting the browser to *trust* what's on our IP (past the TLS warning) is the half that actually matters - that's where sslstrip / a hostile cert / an HTTP-only target comes in.

## Commands

You need a MITM position first - here it's `arp.spoof`, then bettercap does the DNS
```bash
sudo bettercap -iface eth0
# in the session
set arp.spoof.targets 10.0.0.5
arp.spoof on

set dns.spoof.domains bank.com,*.bank.com
set dns.spoof.address 10.0.0.66      # your box
dns.spoof on
net.sniff on
```

Old-school route with ettercap (edit `/etc/ettercap/etter.dns`, then)
```bash
# etter.dns:  bank.com   A   10.0.0.66
sudo ettercap -T -q -i eth0 -M arp /10.0.0.5// /10.0.0.1// -P dns_spoof
```

## May be combined with
- [ARP Spoofing / MITM](https://github.com/surs51/Vulnerabilities_explained/blob/533e2148009276ea01fbc3cd085fd529191d9f66/Vulberabilities/Network/ARP%20Spoofing%20%7C%20MITM.md) - (the usual way to get in the path so you even see the query)
- [Evil Twin / Rogue AP](https://github.com/surs51/Vulnerabilities_explained/blob/533e2148009276ea01fbc3cd085fd529191d9f66/Vulberabilities/Network/Evil_Twin%20%7C%20Rogue_AP.md) - (own the DHCP, just hand out yourself as the DNS server)
- HTTP Proxy / Caplets - (where the victim lands after you redirect them)
- mitm6 / IPv6 DNS Takeover - (the IPv6 way to become their DNS with no spoofing at all)

## Gotchas
- You almost always need a MITM position first - pure cache poisoning against a modern resolver is rough (port randomization, 0x20, DNSSEC)
- HTTPS + HSTS - pointing `bank.com` at your IP just throws a cert error unless you also strip or substitute TLS. The redirect is easy, the trust is not.
- DoH / DoT (DNS over HTTPS/TLS) sidesteps you entirely - can't spoof what you can't even read
- TTLs - a good cached answer sticks around, you may have to wait it out or force a re-lookup
