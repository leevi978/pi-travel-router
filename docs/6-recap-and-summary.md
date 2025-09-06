# 6. Where We Are Now & What Comes Next

Use this as a grounding point before we add smarter features.

## Quick Recap

You have a Raspberry Pi acting as a small travel router:

- `wlan0` (built‑in): runs the hotspot via hostapd (your private WiFi for phone / laptop / streaming devices).
- `eth0`: optional wired uplink (preferred when available).
- `wlan1`: external USB WiFi adapter used to join hotel / public WiFi (uplink when no ethernet).
- `dnsmasq`: hands out DHCP + provides DNS to your devices.
- `nftables`: does NAT + basic forwarding/filtering.
- `sysctl` enables IP forwarding.

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

- When you plug in the USB WiFi adapter, NetworkManager connects it to the configured upstream (`wan-hotel` on `wlan1`).
- When you connect an ethernet cable, `eth0` (`Wired connection 1`) becomes the uplink automatically.
- Both modes feed the same hotspot on `wlan0`.

## Interference Note

`wlan0` (hotspot) and `wlan1` (USB adapter) can sit on overlapping 2.4 GHz spectrum. Even if channels differ, simultaneous transmit/receive in close physical proximity (same board + USB dongle) reduces effective throughput and can increase latency / packet loss.
Recommendation: If an ethernet jack is available, prefer it and unplug the USB WiFi adapter to free RF space and maximize stability & speed.

## When To Use Each Uplink

- Use ethernet (`eth0`) whenever a jack is present. Lowest latency, least interference.
- Use the USB WiFi adapter only when WiFi is the only option.
- If only WiFi exists, the Pi effectively becomes a WiFi “stability layer” / extender. It usually won’t raise raw Mbps beyond what the upstream allows, but it can:
  - Provide stronger local signal for your devices (closer, cleaner RF path)
  - Reduce random disconnects
  - Normalize multiple devices behind one MAC / session (sometimes helping captive portals)

## Hostapd Profile Status

Current hotspot tuning is optimized for a wired uplink scenario (assumes stable backhaul). When using `wlan1` as uplink, different airtime / channel / rate control trade‑offs may be better. We’ll automate that soon.

## Coming Enhancements (Next Parts)

We will layer quality‑of‑life + resilience features:

1. Full‑tunnel option: route all client traffic through a WireGuard (VPN) tunnel; quick commands to toggle / switch configs.
2. Simple commands to switch & log into new public WiFis (modify upstream without manual editing).
3. Auto hostapd profile switching: detect whether uplink is `eth0` or `wlan1` and load the matching optimized config.
4. Media / streaming mode: control streaming or downloads (e.g. to HDMI TV or local storage) from your phone’s browser.
5. (Optional) Further RF optimizations: adaptive channel selection, transmit power adjustments, coexistence tweaks.

## What You Should Do Right Now

Nothing more—just verify both uplink paths work:

1. Test with ethernet plugged in (unplug USB WiFi adapter): confirm internet from a client on the hotspot.
2. Unplug ethernet, plug in USB adapter: confirm it joins the configured public WiFi and hotspot clients still reach the internet.
3. Observe any speed / stability difference so you have a baseline before adding VPN overhead later.

Once that feels solid, proceed to the next section where we start adding the VPN layer.
