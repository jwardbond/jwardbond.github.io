---
title: "Ubuntu Wifi Frequently Disconnecting Realtek Drivers Solution"
date: 2022-01-08 10:33 -0500
categories: programming
tags: ubuntu
excerpt: "There are a ton of reasons why you might end up with a finnicky WiFi connection when using Ubuntu (just try googling 'WiFi randomly disconnecting Ubuntu'), and I believe I have found yet another."
author_profile: false
---
There are a *ton* of reasons why you might end up with a finnicky WiFi connection when using Ubuntu (just try googling 'WiFi randomly disconnecting Ubuntu'), and I believe I have found yet another. I couldn't seem to find any existing solutions to this particular problem, so I figured this is a good opportunity for me to contribute something!

I recently built a desktop out of spare parts I had, which I intended to use solely to run Linux. Installation of Ubuntu 20.04 LTS was successfull, but when I tried to connect to my WiFi I ended up with a frustrating problem: I would successfully connect, use it for a few minutes, and then it would disconnect and re-prompt me for the password. I would input the password, and it *sometimes* reconnected, but sometimes it just immediately prompted me for the password again. Here is the hack I used to fix the problem.

# Problem
- Ubuntu 20.04 frequently disconnects from WiFi and prompts for password

# Equipment
- **OS**: Ubuntu 20.04 LTS / 21.04
- **Desktop**: Gnome
- **Network Manager**: NetworkManager ??
- **WiFi Card**: [tp-link Archer AC1200 T4E PCI Express Adapter](https://www.tp-link.com/ca/home-networking/adapter/archer-t4e/)
- **WiFi Driver**: rtl8821ae or rtl8821ce
- **Router**: Bell Home Hub 3000

# Things That Didn't Work
- Installing open source drivers (Ubuntu Kernel should already have rtl8821ae / ce)
- Turning off power-save mode
- Changing country code

# Solution
Force NetworkManager to connect to only working BSSID when connecting to the internet. This was done through the following steps: 

1. Identify working BSSID by reading through `sudo dmesg`
2. On your desktop, go into Settings -> Wi-Fi -> Settings (for your specific SSID) -> Identity
3. Input the working BSSID into the BSSID field
4. Click apply
5. Forget all other connections

This forces NetworkManager to use a specific BSSID when connecting to the SSID. After doing this, you will likely see two entries in your Visible Networks with the same SSID. One of these is the SSID+BSSID combination you just specified, and the other is the original SSID. **MAKE SURE YOU DON'T AUTO-CONNECT TO THE ORIGINAL SSID**. It's easies to just 'Forget' the original SSID.

<img src="https://jwardbond.github.io/assets/images/ubuntu-wifi-problem.png" alt="a visual description of the steps above">

# Detailed Investigation
When experiencing a disconnect / reauthentication event, run `sudo dmesg | grep 'wlp'`. Ideally, this should return something like the following:

```bash
...
[ 6.459445] wlp2s0: authenticate with 86:a0:6e:ee:a5:67
[ 6.460220] wlp2s0: send auth to 86:a0:6e:ee:a5:67 (try 1/3)
[ 6.460943] wlp2s0: authenticated
[ 6.461252] wlp2s0: associate with 86:a0:6e:ee:a5:67 (try 1/3)
[ 6.463908] wlp2s0: RX AssocResp from 86:a0:6e:ee:a5:67 (capab=0x511 status=0 aid=23)
[ 6.464130] wlp2s0: associated
[ 6.505707] wlp2s0: Limiting TX power to 30 (30 - 0) dBm as advertised by 86:a0:6e:ee:a5:67
[ 6.705437] IPv6: ADDRCONF(NETDEV_CHANGE): wlp2s0: link becomes ready
...
```
If this command doesn't return anything, try `sudo dmesg | grep 'wlan0'`, or just manually search through the results of `sudo dmesg` until you find the section of the log similar to the snippet above. In the above, perfectly healthy example, my computer first attempts to authenticate with the WiFi address 86:a0:6e:ee:a5:67. After my computer authenticates successfully, it then attempts to fully connect (associate) with this same address. 

When I was running into connectivity problems, my log looked like and endless cycle of this:

```bash
...
[ 7103.149967] wlp2s0: authenticate with 86:a0:6e:ee:a5:67
[ 7103.150789] wlp2s0: send auth to 86:a0:6e:ee:a5:67 (try 1/3)
[ 7103.152583] wlp2s0: authenticated
[ 7103.152985] wlp2s0: associate with 86:a0:6e:ee:a5:67 (try 1/3)
[ 7103.156295] wlp2s0: RX AssocResp from 86:a0:6e:ee:a5:67 (capab=0x411 status=0 aid=1)
[ 7103.156535] wlp2s0: associated
[ 7103.217188] IPv6: ADDRCONF(NETDEV_CHANGE): wlp2s0: link becomes ready
[ 7122.629633] wlp2s0: disconnect from AP 86:a0:6e:ee:a5:67 for new auth to 86:a0:6e:ee:a6:60
[ 7122.738309] wlp2s0: authenticate with 86:a0:6e:ee:a6:60
[ 7122.739114] wlp2s0: send auth to 86:a0:6e:ee:a6:60 (try 1/3)
[ 7122.740504] wlp2s0: authenticated
[ 7122.741644] wlp2s0: associate with 86:a0:6e:ee:a6:60 (try 1/3)
[ 7122.742540] wlp2s0: RX ReassocResp from 86:a0:6e:ee:a6:60 (capab=0x11 status=0 aid=1)
[ 7122.742777] wlp2s0: associated
[ 7122.816266] wlp2s0: Limiting TX power to 30 (30 - 0) dBm as advertised by 86:a0:6e:ee:a6:60
[ 7131.750819] wlp2s0: deauthenticated from 86:a0:6e:ee:a6:60 (Reason: 15=4WAY_HANDSHAKE_TIMEOUT)
[ 7138.294643] wlp2s0: authenticate with 86:a0:6e:ee:a5:67
[ 7138.295497] wlp2s0: send auth to 86:a0:6e:ee:a5:67 (try 1/3)
[ 7138.297133] wlp2s0: authenticated
[ 7138.301721] wlp2s0: associate with 86:a0:6e:ee:a5:67 (try 1/3)
[ 7138.305027] wlp2s0: RX AssocResp from 86:a0:6e:ee:a5:67 (capab=0x411 status=0 aid=1)
[ 7138.305836] wlp2s0: associated
[ 7307.187362] wlp2s0: deauthenticating from 86:a0:6e:ee:a5:67 by local choice (Reason: 3=DEAUTH_LEAVING)
...
```
In this case, my computer connects successfully to `86:a0:6e:ee:a5:67` (I'll call it **Address 1**), and then after a brief period of calm, attempts to switch the WiFi connection to address `86:a0:6e:ee:a6:60` (Line 9, **Address 2**). After apparently successfully connecting to **Address 2**, I would get `deauthenticated from 86:a0:6e:ee:a6:60 (Reason: 15=4WAY_HANDSHAKE_TIMEOUT)` and then my computer would deauthenticate, prompt me for the password, and connect to **Address 1** again. This cycle just kept repeating over and over and over, leading to a very unstable connection.

During troubleshooting, I connected to another WiFi SSID in my house, and got the same problem: my connection would work for a bit, then try to switch to some alternate address, at which point it would start a cycle of prompting for passwords/connecting/disconnecting *ad infinitum*.

So what is happening here? What are these suspicious addresses that break my connection? 

These addresses are MAC addresses for wireless access points on my router, also known as [**B**SSIDs addresses](https://www.juniper.net/documentation/en_US/junos-space-apps/network-director4.0/topics/concept/wireless-ssid-bssid-essid.html). There are two SSID addresses set up on my router, each of which contains a couple BSSIDs. Specifically MySSID-1 has 2 BSSID, and MySSID-2 has 3 BSSID. When I connect to MySSID-1 or MySSID-2, I am actually connecting to one of the possible BSSID within those SSID. This is pretty normal, and this technology is what allows you to stay connected to the same SSID when you change routers (i.e. walking around campus or the office): the SSID would stay the same, but since you are connecting to a new router, the BSSID would change. This is also how modern routers allow you to dynamically switch between 5 GHz and 2.4 GHz connections on the same SSID. This latter point is important, my landlord has set up our router to allow for this dynamic switching activity.

By looking through the results of `dmesg` I identified which BSSIDs were working, and which were causing problems. It turned that MySSID-1 and MySSID-2 only had one problematic BSSID each. When I attempted to connect to either MySSID-1 or -2, I either: a) successfully connected to a working BSSID within them, or b) connected to the problematic BSSID, which then caused a whole bunch of errors. To complicate things further, even if my computer connected to a working BSSID, it would roam for the BSSID with the strongest signal strength and then attempt to connect to it (see line 9 above). If it then tried to switch to the one problematic BSSID, I once again got thrown into a cycle of disconnecting / reauthenticating.

A this point, I could have just forced my computer to only use working BSSID when connecting to the internet, which is what I ultimately did to solve the problem (see the **Soluction** section), but I was curious why certain BSSIDs were causing these problems. I ran `sudo iw wlp2s0 scan` and looked at the details for each of the BSSID I had identified (Note that you would have to replace `wlp2s0` in the above with whatever your wireless interface is named, run `iwconfig` if you aren't sure). It turned out that MySSID-1 was set up with 2.4GHz and 5.2 GHz channels, and MySSID-2 was set up with 2.4, 5.2, and 5.8 GHz channels (results in table below). In both cases, the problematic SSID was **the one coressponding to the 5.2 GHz channel!** I have absolutely no idea why this is the case, please email me if you do.

<table>
    <thead>
        <tr>
            <th style="text-align: center;" colspan="2">MySSID-1</th>
            <th style="text-align: center;" colspan="2">MySSID-2</th>
        </tr>
        <tr>
            <th>BSSID (MAC)</th>
            <th>Freq</th>
            <th>BSSID (MAC)</th>
            <th>Freq</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>86:a0:6e:ee:a5:67</td>
            <td>2.4 GHz</td>
            <td>84:a0:6e:ee:a4:66</td>
            <td>2.4 GHz</td>
        </tr>
        <tr>
            <td><b>86:a0:6e:ee:a6:60</b></td>
            <td><b>5.2 GHz</b></td>
            <td><b>84:a0:6e:ee:a4:67</b></td>
            <td><b>5.2 GHz</b></td>
        </tr>
        <tr>
            <td>-</td>
            <td>5.8 GHz</td>
            <td>86:a0:6e:ee:a4:65</td>
            <td>5.8 GHz</td>
        </tr>
    </tbody>
</table>