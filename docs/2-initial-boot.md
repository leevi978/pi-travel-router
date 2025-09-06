# Initial boot

## Boot up and log in

1. Flash the OS and configure your Pi according to the requirements described in [1-requirements.md](/docs/1-requirements.md).
2. Power up and wait for the boot to finish.
3. Connect:
   - A) Use a monitor and keyboard, or
   - B) SSH from your laptop.
4. Log in with your username and password.

### Increase font size (optional)

If the text is too small (e.g. you're using a TV as monitor), change the font size:

```bash
sudo dpkg-reconfigure console-setup
# Choose: UTF-8 → Terminus → 32x16 (or the largest shown)
sudo setupcon
```

## Secure the Pi

For more details, see the [official documentation](https://www.raspberrypi.com/documentation/computers/configuration.html#secure-your-raspberry-pi).

### Install updates

1. Update packages:
   ```bash
   sudo apt update && sudo apt full-upgrade
   ```
2. Reboot:
   ```bash
   sudo reboot
   ```

### Require password for sudo

1. Edit sudoers file:
   ```bash
   sudo visudo -f /etc/sudoers.d/010_pi-nopasswd
   ```
2. Add this line (replace `<username>` with your user):
   ```
   <username> ALL=(ALL) PASSWD: ALL
   ```

### Automatically update SSH server

```bash
sudo apt install openssh-server
```

### Install fail2ban

Fail2ban helps prevent brute-force attacks.

1. Install and configure:
   ```bash
   sudo apt install fail2ban
   sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
   sudo nano /etc/fail2ban/jail.local
   ```
2. Scroll to the end (or find the `[ssh]` section). If it doesn't exist, add:
   ```
   [ssh]
   enabled  = true
   port     = ssh
   filter   = sshd
   backend  = systemd
   maxretry = 6
   ```
