# Hotspot setup

## Introduction

In this section, you'll set up your Raspberry Pi as a WiFi hotspot using the `wlan0` interface. This lets other devices connect to your Pi for internet access or local networking.

> **Note:** You need a working internet connection (via Ethernet or WiFi WAN) before starting.

---

## Step 1: Install required packages

Install the software needed for the hotspot:

- **hostapd** runs the WiFi access point.
- **dnsmasq** gives out IP addresses and handles DNS for clients.
- **nftables** manages firewall rules and network security.

```bash
sudo apt install hostapd dnsmasq nftables
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
```

---

## Step 2: Assign a static IP to wlan0

**Goal:** Give your hotspot interface a fixed address (10.77.0.1) so clients always know the gateway.

Why?

- Your Pi acts as the gateway for all hotspot clients.
- Devices need a fixed address to send their traffic for internet access.
- Without a static IP, clients won’t know where to find the gateway and won’t get online.

How:

1. Create the service file:
   ```bash
   sudo nano /etc/systemd/system/wlan0-static-ip.service
   ```
2. Paste:

   ```ini
   [Unit]
   Description=Assign static IP to wlan0 for hotspot
   After=network-online.target sys-subsystem-net-devices-wlan0.device
   Wants=network-online.target sys-subsystem-net-devices-wlan0.device
   Before=dnsmasq.service

   [Service]
   Type=oneshot
   ExecStart=/sbin/ip link set wlan0 up
   ExecStart=/sbin/ip address replace 10.77.0.1/24 dev wlan0
   RemainAfterExit=yes

   [Install]
   WantedBy=multi-user.target
   ```

3. Enable and start it:

   ```bash
   sudo systemctl enable --now wlan0-static-ip.service
   ```

4. Verify:
   ```bash
   ip -4 addr show dev wlan0 | grep 10.77.0.1
   ```
   You should see `10.77.0.1/24`.

---

## Step 3: Disable wlan0 Power Saving

**Goal:** Prevent the built-in WiFi adapter from entering power-saving mode, which can cause random disconnects.

**Why?** Power saving can make the hotspot unreliable, causing clients to disconnect.

How:

1. Create the service file:

   ```bash
   sudo nano /etc/systemd/system/wlan0-ps-off.service
   ```

   Paste:

   ```ini
   [Unit]
   Description=Disable power saving on wlan0
   After=network.target

   [Service]
   Type=oneshot
   ExecStart=/sbin/iw dev wlan0 set power_save off

   [Install]
   WantedBy=multi-user.target
   ```

   Save and exit nano.

   Then enable and start the service:

   ```bash
   sudo systemctl enable --now wlan0-ps-off.service
   ```

This will ensure power saving is always disabled for `wlan0`, even after reboot.

## Step 4: Enable IPv4 forwarding

**Goal:** Make your Pi forward internet traffic for devices connected to its hotspot.

**Why?** By default, Linux does not forward packets between interfaces. Enabling IPv4 forwarding lets your Pi act as a gateway for connected devices.

How:

1. Permanently enable IPv4 forwarding:
   ```bash
   echo 'net.ipv4.ip_forward=1' | sudo tee /etc/sysctl.d/99-ipforward.conf
   sudo sysctl --system
   ```

---

## Step 5: Configure DHCP and DNS

**Goal:** Set up automatic IP assignment and local DNS for devices joining your hotspot.

**Why?** This establishes your Pi as the network hub for all hotspot clients.

1. Create the dnsmasq config file:

   ```bash
   sudo nano /etc/dnsmasq.d/hotspot.conf
   ```

   Paste and review each line:

   ```ini
   # Use wlan0 for the hotspot
   interface=wlan0

   # Wait for interfaces to appear before binding (prevents boot errors if wlan0 isn't up yet)
   bind-dynamic

   # Act as the main DHCP server
   dhcp-authoritative

   # IP range for clients (adjust if needed)
   dhcp-range=10.77.0.10,10.77.0.200,255.255.255.0,12h

   # Set gateway and DNS server to Pi's IP
   dhcp-option=option:router,10.77.0.1
   dhcp-option=option:dns-server,10.77.0.1

   # Block invalid domains and private IP leaks
   domain-needed
   bogus-priv
   stop-dns-rebind
   ```

   > **Tip:** Change the IP range if you want a different subnet for your hotspot.

2. Enable and start dnsmasq:
   ```bash
   sudo systemctl enable --now dnsmasq
   ```

---

## Step 6: Make dnsmasq wait for wlan0

**Goal:** Ensure `dnsmasq` only starts after the `wlan0` device exists.

**Why?** Without this, `dnsmasq` may start too early and complain that `wlan0` doesn't exist. This guarantees clean startup and avoids race conditions.

Create the override:

```bash
sudo mkdir -p /etc/systemd/system/dnsmasq.service.d
sudo nano /etc/systemd/system/dnsmasq.service.d/wlan0-device.conf
```

Paste:

```ini
[Unit]
After=sys-subsystem-net-devices-wlan0.device
Requires=sys-subsystem-net-devices-wlan0.device
```

Save and exit nano.

Reload systemd and restart dnsmasq:

```bash
sudo systemctl daemon-reload
sudo systemctl restart dnsmasq
```

---

## Step 7: Enable Internet Access for Hotspot Clients

**Goal:** Forward traffic from hotspot devices to the internet.

**Why?** So clients connected to your hotspot have internet access.

1. Edit nftables config:

   ```bash
   sudo nano /etc/nftables.conf
   ```

   Add these lines (adjust `eth0` or `wlan1` as needed for your upstream):

   ```bash
   # Enable NAT for hotspot clients
   table inet nat {
     chain postrouting {
       type nat hook postrouting priority 100;
       oifname "eth0" ip saddr 10.77.0.0/24 masquerade
       oifname "wlan1" ip saddr 10.77.0.0/24 masquerade
     }
   }
   ```

   > **Note:** Only add a rule for the interface(s) you use for internet access (`eth0` for Ethernet, `wlan1` for WiFi WAN).

2. Apply the rules:
   ```bash
   sudo nft -f /etc/nftables.conf
   ```

---

## Step 8: Configure the WiFi Hotspot

**Goal:** Set up your Pi to broadcast a WiFi network for other devices.

**Why?** This lets your Pi act as a wireless access point, so other devices can connect and use its network.

1. Edit the hostapd config:

   ```bash
   sudo nano /etc/hostapd/hostapd.conf
   ```

   Paste and edit as needed:

   ```ini
   # Country code (change to your country, e.g. US, GB, DE)
   country_code=NO

   # WiFi network name (SSID)
   ssid=<HOTSPOT NAME>

   # WiFi password (use a strong passphrase)
   wpa_passphrase=<HOTSPOT PASSWORD>

   # Wireless interface
   interface=wlan0

   # WiFi driver
   driver=nl80211

   # Enable WPA2 security
   wpa=2
   wpa_key_mgmt=WPA-PSK
   rsn_pairwise=CCMP

   # Disable WPS for security
   wps_state=0

   # 5 GHz access point settings
   hw_mode=a
   channel=36
   ieee80211n=1
   ieee80211ac=1
   ieee80211d=1
   ieee80211h=1
   wmm_enabled=1
   require_vht=1

   # Prefer wide channels (80 MHz)
   vht_oper_chwidth=1
   vht_oper_centr_freq_seg0_idx=42

   # Enable useful capabilities
   ht_capab=[HT40+][SHORT-GI-20][SHORT-GI-40]
   vht_capab=[SHORT-GI-80]
   ```

   > **Note:** Replace `<HOTSPOT NAME>` and `<HOTSPOT PASSWORD>` with your own values.

2. Point hostapd to this config file:
   ```bash
   sudo nano /etc/default/hostapd
   ```
   Add or update:
   ```ini
   DAEMON_CONF="/etc/hostapd/hostapd.conf"
   ```

---

## Step 9: Set Your Pi's Hostname

**Goal:** Give your Pi a unique name for easy identification on the network.

**Why?** So your Pi has a domain name on the hotspot network, not just an IP address.

1. Edit the hostname file:

   ```bash
   sudo nano /etc/hostname
   ```

   Delete the existing content and type your desired hostname (e.g. `raspberrypi-hotspot`). Save and exit.

2. Add your hostname to the hosts file:
   ```bash
   echo "127.0.0.1   <YOUR HOSTNAME>" | sudo tee -a /etc/hosts
   ```

---

## Step 10: Free up wlan0 for the hotspot

**Goal:** Stop NetworkManager from managing `wlan0` so the hotspot works reliably.

Why?

- By default, NetworkManager tries to manage all network interfaces—including `wlan0`.
- `hostapd` needs full control of `wlan0` to run the WiFi access point.
- If NetworkManager manages `wlan0`, the hotspot won’t work reliably.

How:

1. Create the config file:
   ```bash
   sudo nano /etc/NetworkManager/conf.d/unmanaged-wlan.conf
   ```
2. Paste:
   ```ini
   [keyfile]
   unmanaged-devices=interface-name:wlan0
   ```
3. Restart NetworkManager:
   ```bash
   sudo systemctl restart NetworkManager
   ```

---

## Step 11: Reboot and Test

1. Enable all services and reboot:

   ```bash
   sudo systemctl enable hostapd dnsmasq nftables ssh
   sudo systemctl daemon-reload
   sudo reboot
   ```

2. After reboot, connect a device to your hotspot. You should get an IP address and internet access (if WAN is set up).

3. If you have issues:
   - Check service status:
     ```bash
     sudo systemctl status hostapd dnsmasq nftables
     ```
   - View error logs:
     ```bash
     journalctl -xe
     ```

Your Pi is now a working WiFi hotspot!

---

## Step 12: Limit SSH Access to Hotspot Clients

**Goal:** Only allow SSH connections from devices on your hotspot.

**Why?** This protects your Pi from remote SSH login attempts on public networks.

> Note: If you are SSH'd into the Pi from outside the hotspot, this step will break the connection.

1. Edit the nftables config:

   ```bash
   sudo nano /etc/nftables.conf
   ```

   Add to the input chain:

   ```bash
   # Only allow SSH from hotspot clients
   iifname "wlan0" tcp dport 22 accept
   tcp dport 22 drop
   ```

2. Apply the rules and enable SSH:
   ```bash
   sudo nft -f /etc/nftables.conf
   sudo systemctl enable --now ssh
   ```
