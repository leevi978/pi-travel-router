# Setup network adapter

## What you'll do

1. Check if your Wi-Fi adapter and driver are present
2. Prevent power-saving dropouts
3. Load the driver
4. Connect to Wi-Fi using NetworkManager

> If you're using ethernet only for internet connection, you can skip this section.

## Verify that a driver is present

If you use the [TP-Link Archer T3U Plus](https://www.tp-link.com/us/home-networking/usb-adapter/archer-t3u-plus/), its chipset is **Realtek RTL8812BU**. This is supported by the built-in driver `rtw88_8822bu` in modern Raspberry Pi OS.

If you have a different adapter, you’ll need to find and install its driver (not covered here).

**Check your adapter and driver:**

1. Plug the adapter into a USB-3 port.
2. Run:

   ```bash
   lsusb
   ```

   Look for: `2357:0138 TP-Link 802.11ac NIC`

3. Run:
   ```bash
   modinfo rtw88_8822bu | head -n5
   ```
   Look for: `filename: /lib/modules/.../rtw88_8822bu.ko.xz`

If you see both, you’re good to go!

## Prevent power-saving dropouts

Some USB Wi-Fi adapters disconnect because of power saving. Fix it with a udev rule:

```bash
sudo tee /etc/udev/rules.d/50-rtl88x2bu-usb-powersave.rules >/dev/null <<'EOF'
ACTION=="add", SUBSYSTEM=="usb", ATTR{idVendor}=="2357", ATTR{idProduct}=="0138", TEST=="power/control", ATTR{power/control}="on"
EOF
sudo udevadm control --reload-rules
```

## Load the driver

> If you use multiple external Wi-Fi adapters, make sure your USB adapter is permanently bound to `wlan1` before continuing.

1. **Load the driver and check for your Wi-Fi adapter:**

   ```bash
   sudo modprobe rtw88_8822bu
   ip link show | grep wlan
   ```

   You should see a line with `wlan1`. If not, the driver isn’t loaded or the adapter isn’t detected.

2. **Set the driver to load automatically at boot:**

   ```bash
   echo "rtw88_8822bu" | sudo tee /etc/modules-load.d/rtw88_8822bu.conf
   ```

   (No output expected. This just creates a config file.)

3. **Reboot your Raspberry Pi:**

   ```bash
   sudo reboot
   ```

   (Wait for the system to restart.)

4. **After reboot, check for your Wi-Fi adapter again:**
   ```bash
   ip a | grep wlan
   ```
   You should see `wlan1` listed. If you only see `wlan0`, the USB adapter isn’t detected.

## Bind adapter to WiFi AP

Create the file `/etc/NetworkManager/system-connections/wan-hotel.nmconnection`.

If the hotel Wi-Fi is open (no password), delete the whole `[wifi-security]` section.

Example config:

```
[connection]
id=wan-hotel
type=wifi
interface-name=wlan1
autoconnect=true
autoconnect-priority=50

[wifi]
ssid=<HOTEL_SSID>
mode=infrastructure

[wifi-security]
key-mgmt=wpa-psk
psk=<HOTEL_PASSWORD>

[ipv4]
method=auto
route-metric=300
ignore-auto-dns=true

[ipv6]
method=ignore
```

Set permissions and load the connection:

```bash
sudo chmod 600 /etc/NetworkManager/system-connections/wan-hotel.nmconnection
sudo chown root:root /etc/NetworkManager/system-connections/wan-hotel.nmconnection
sudo nmcli connection reload
nmcli connection show
sudo nmcli connection up wan-hotel
```

Test your internet connection:

```bash
curl -4 --interface wlan1 ifconfig.io
```

You should see your public IP address.
