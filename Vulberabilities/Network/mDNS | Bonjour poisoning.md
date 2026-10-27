# mDNS / Bonjour poisoning

## mDNS (multicast DNS)

#### 5353 - UDP (224.0.0.251 / ff02::fb)

So we've heard of LLMNR poisoning, but it only worked for windows, so here's Mac/Linux cousing. Anything that ends with `.local` never goes through normal DNS server - the host just multicasts "Who has X.local?" to the whole LAN, and whoever answers 1st - wins. (Bonjour/avahi runs on it)

Same logic as with LLMNR - doesn't matter who's right, only who's first.

e.g. 
> - (Victim) printer.local? anyone? (to 224.0.0.251:5353 - and everyone hears it)
> - (Printer) me (slow responce)
> - (Us) Me printer.local = my IP (fast)

---

## So how it all works

1. Victim multicasts the querry - `printer.local`
2. Legit owner replies slower and loses
3. We answer first - `.local == my IP`
4. Victim connects to us

<img width="920" height="560" alt="mdns-bonjour-poisoning" src="https://github.com/user-attachments/assets/66249b69-9860-44bb-a826-51421d62a6c0" />

---

## Commands 

Responder already speaks mDNS, so we only need 1 tool

```bash
sudo responder -I wlan0 -mv # -m - Answer DNS, -v - verbose

# or just the normal full run - Responder poisons LLMNR + NBT-NS + mDNS all at once
sudo responder -I wlan0 -wF
```
And de-hash
```bash
hashcat -m 5600 hash.txt rockyou.txt
```
---
If you don't wanna be noisy - you can just analyze the traffic
`sudo responder -I wlan0 -A` 

## May be combined with
- LLMNR / NBT-NS Poison - (Responder does all three in one run)
- NTLM Relay / SMB Relay - (forward the captured auth instead of cracking it)
- WPAD Hijacking - (hand back a proxy config once they trust us)

## Gotchas
- It's a race - you must answer before the real `.local` owner, so closer / faster = win
- mDNS is link-local only - it won't cross subnets or VLANs (224.0.0.251 isn't routed)
- Mostly hits Apple / Linux / IoT - Windows leans on LLMNR and NBT-NS more
- Fairly quiet, but duplicate-name answers can trip conflict detection on the real device and tip someone off
