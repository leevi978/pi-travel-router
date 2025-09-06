# Requirements

## VPN and Wireguard

You need a VPN service that supports [Wireguard](https://www.wireguard.com/).

- Example: [ProtonVPN](https://protonvpn.com/) (easy to use)

## Raspberry Pi

Tested with **Raspberry Pi 5 (16 GB)** and **Raspberry Pi OS Lite 64-bit (Bookworm)**.

> This setup may work on other Raspberry Pi models or Linux-based OSes, but only the above is tested. You may need to adapt the guide for other hardware.

### Initial configuration

See: [Set up a headless Raspberry Pi](https://www.raspberrypi.com/documentation/computers/configuration.html#setting-up-a-headless-raspberry-pi)

#### 1) Physical access during setup

You should have **physical access to the Pi console** during setup, in case network changes break SSH.

**Equipment needed:**

- HDMI / Micro HDMI cable
- Monitor
- Keyboard

#### 2) Enable SSH

Make sure **SSH is enabled** during initial setup.

- **Key authentication is recommended** (safer than password, especially on public networks)

> You can later restrict SSH to the hosted WLAN only.

#### 3) Internet connection

The Pi needs internet access to install packages and must be on the same LAN as your laptop for SSH.

## External network adapter (optional)

For this project, I used a [TP-Link Archer T3U Plus](https://www.tp-link.com/us/home-networking/usb-adapter/archer-t3u-plus/) as an extra network adapter to connect the Pi to WiFi.

Any adapter can work, but look for these features:

1. **Dual-band**
   - Supports both 2.4GHz and 5GHz WiFi
2. **Linux support**
   - Uses built-in Linux drivers, or
   - Has well-maintained community drivers
3. **Signal strength**
   - Hotel WiFi can be weak
   - Tiny USB dongles may struggle with walls

### Why is it needed?

The Raspberry Pi 5 has two built-in network interfaces:

- WLAN (`wlan0`)
- Ethernet (`eth0`)

The built-in WLAN can:

1. Connect to a WiFi access point
2. Host its own WiFi hotspot

But it **can't do both at once** without reduced performance and stability.

To fix this, you can:

**A) Use Ethernet**

- If there's an Ethernet jack, just use a cable.
- `wlan0` hosts the hotspot
- `eth0` connects to the internet

**B) Add a network adapter**

- Extra adapter gives you `wlan1` (second WiFi interface)
- `wlan0` hosts the hotspot
- `eth0` connects to the internet (if available)
- `wlan1` connects to WiFi (if available)

This setup lets you use both Ethernet and WiFi flexibly as you travel.
