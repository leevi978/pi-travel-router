# Configure ethernet connection

## Introduction

In this section, you will configure the Raspberry Pi’s Ethernet connection. This ensures:

- Reliable internet access via cable
- Custom DNS settings for better control
- Proper routing priority when both Ethernet and WiFi are available

> Note: Skip this section if you plan to use WiFi only.

## Step 1: Plug in the ethernet cable

Make sure your Raspberry Pi is powered on and plug an ethernet cable into the Pi and your router or wall jack.

## Step 2: Rename and reload the connection

Rename the default wired connection for clarity:

```bash
sudo nmcli connection modify "Wired connection 1" connection.id "wired-1"
sudo nmcli connection reload
```

## Step 3: Set permissions and edit the config

Set correct permissions and ownership:

```bash
sudo chmod 600 /etc/NetworkManager/system-connections/wired-1.nmconnection
sudo chown root:root /etc/NetworkManager/system-connections/wired-1.nmconnection
```

Edit the connection config:

```bash
sudo nano /etc/NetworkManager/system-connections/wired-1.nmconnection
```

Paste or update the following content:

```ini
[ipv4]
method=auto
route-metric=100
dns=1.1.1.1
ignore-auto-dns=true

[ipv6]
method=ignore
```

## Step 4: Apply changes and bring up the connection

Reload and activate the connection:

```bash
sudo nmcli connection reload
sudo nmcli connection up wired-1
```

## Step 5: Test your connection

Check your internet connection:

```bash
curl -4 --interface eth0 ifconfig.io
```

You should see your IP address.

You now have a working ethernet connection with custom DNS and routing priority.
