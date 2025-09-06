# 6. Where We Are Now & What Comes Next

You now have a working travel router. It already does everything needed for day‑to‑day use. From here onward we add optional quality, privacy, and convenience upgrades.

## Current Components

- `wlan0` (built‑in): hotspot (your private SSID) via hostapd
- `eth0`: wired uplink (preferred when available)
- `wlan1`: USB adapter joining hotel / public WiFi when no ethernet
- `dnsmasq`: DHCP + DNS for your devices
- `nftables`: NAT + basic filtering
- `sysctl`: enables IP forwarding

You do NOT need both uplink interfaces. Possible modes:

- Wired‑only: Pi + ethernet (no USB WiFi adapter needed).
- WiFi‑only: Pi + USB adapter (no ethernet available at location).
- Dual: Keep both for flexibility; whichever comes up provides the uplink.

## Mental Model (Current Data Paths)

Wired uplink (best performance):

```
Internet ⇄ eth0 (Wired connection 1) → NAT/firewall → wlan0 (your hotspot SSID) → Your devices
```

Wireless uplink via USB adapter (hotel / café WiFi):

```
Hotel/Public AP ⇄ wlan1 (wan-hotel) → NAT/firewall → wlan0 (your hotspot SSID) → Your devices
```

## Automatic Interface Behavior

- If the USB WiFi adapter is present, NetworkManager connects it to `wan-hotel` on `wlan1`.
- If an ethernet cable is plugged in, `eth0` ("Wired connection 1") becomes the uplink.
- If only one exists, that one is used—no failure; the other steps just don’t apply.
- Both (when present) feed the same hotspot on `wlan0`; you can unplug / plug either without reconfiguring the hotspot.

## Interference Note

`wlan0` and `wlan1` compete for the same 2.4 GHz airtime. Both active = less throughput + more latency. If ethernet exists: use it and unplug the USB adapter.

## When To Use Each Uplink

Quick rule:

- Ethernet available? Use ethernet (best speed + stability).
- No ethernet? Use USB WiFi adapter.
- Have both? Prefer ethernet; keep adapter unplugged unless needed.
  WiFi‑only mode acts like a small extender: better local signal & fewer drops (not higher raw Mbps). Bonus: single MAC can simplify captive portals.

## Hostapd Profile Status

Current tuning favors wired uplink; WiFi uplink may benefit from different channel / airtime settings. We will automate profile switching later.

## Coming Enhancements

Optional add‑ons (stop here if you’re happy):

- VPN / WireGuard: full‑tunnel toggle + quick profile switch
- Upstream helper: change / log into new public WiFi fast
- Adaptive hotspot: auto profile based on `eth0` vs `wlan1`
- Media mode: phone‑controlled streaming / downloads
- RF tweaks: channel, power, coexistence

## Quick Self‑Check Before Moving On

Do only what applies:

1. Ethernet present? Test hotspot → internet with ONLY ethernet.
2. USB adapter present? Test hotspot → internet with ONLY WiFi uplink.
3. Have both? Compare speeds to form a baseline before VPN.
   Then move on to the VPN / WireGuard section.
