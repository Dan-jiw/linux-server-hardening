<img width="706" height="197" alt="image" src="https://github.com/user-attachments/assets/5aea41a3-1130-44bd-a738-7f9cb416871c" /># linux-server-hardening

A practical, step-by-step guide to hardening a fresh Ubuntu/Debian server.  
Includes real configuration files and commands used in a home lab environment.

---

## What this covers

- SSH hardening (key-based auth, disable root, change port)
- Firewall setup with `ufw`
- Brute-force protection with `fail2ban`
- Basic `nginx` install and configuration
- System updates and unnecessary service removal

---

## Requirements

- Ubuntu 22.04 / Debian 12 (fresh install)
- Root or sudo access
- SSH client on your local machine

---

## Step 1 — Update the system

```bash
apt update && apt upgrade -y
apt install -y ufw fail2ban nginx curl git
```

---

## Step 2 — Create a non-root sudo user

```bash
adduser danylo
usermod -aG sudo danylo
```

---

## Step 3 — SSH Hardening

### 3.1 Generate SSH key on your local machine

```bash
ssh-keygen -t ed25519 -C "your@email.com"
```

### 3.2 Copy public key to server

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub danylo@YOUR_SERVER_IP
```

### 3.3 Edit SSH config on the server

```bash
nano /etc/ssh/sshd_config
```

Change or add these lines:

```
Port 2222                    # change default port
PermitRootLogin no           # disable root login
PasswordAuthentication no    # keys only, no passwords
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys
X11Forwarding no
AllowUsers danylo
```



### 3.4 Restart SSH


```bash
systemctl restart sshd
```

[SSH Status](screenshots/ssh_status.png)

> ⚠️ Before logging out — open a second terminal and test the new connection:
> `ssh -p 2222 danylo@YOUR_SERVER_IP`

---

[Log in by SSH](screenshots/ssh_login.jpg)



## Step 4 — Firewall (ufw)

```bash
ufw default deny incoming
ufw default allow outgoing
ufw allow 2222/tcp     # your new SSH port
ufw allow 80/tcp       # HTTP
ufw allow 443/tcp      # HTTPS
ufw enable
ufw status verbose
```

---

[UFW rules](screenshots/ufw.jpg)

## Step 5 — fail2ban

```bash
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
nano /etc/fail2ban/jail.local
```

Find the `[sshd]` section and set:

```ini
[sshd]
enabled  = true
port     = 2222
maxretry = 5
bantime  = 1h
findtime = 10m
```

[jail.local](screenshots/sshd_failt2ban.jpg)

```bash
systemctl enable fail2ban
systemctl restart fail2ban
fail2ban-client status sshd
```

---

## Step 6 — Remove unnecessary services

```bash
# Check what's listening
ss -tlnp

# Disable services you don't need, example:
systemctl disable --now avahi-daemon
systemctl disable --now cups
```

---

## Step 7 — Automatic security updates

```bash
apt install -y unattended-upgrades
dpkg-reconfigure --priority=low unattended-upgrades
```

---

## Step 8 — Basic nginx check

```bash
systemctl status nginx
curl -I http://localhost
```

Expected: `HTTP/1.1 200 OK`

---

[Nginx status](screenshots/nginx.jpg)

## Security checklist

- [ ] Root login disabled
- [ ] SSH password auth disabled
- [ ] SSH key works on new port
- [ ] ufw enabled with minimal open ports
- [ ] fail2ban running and monitoring SSH
- [ ] System packages up to date
- [ ] Unneeded services removed

---

## Files in this repo

| File | Description |
|---|---|
| `sshd_config` | Hardened SSH config example |
| `jail.local` | fail2ban config for SSH |
| `ufw-rules.sh` | Script to apply ufw rules |

> 📸 Screenshots of each step added in `/screenshots/`

---

## Author

**Dan-jiw** · [GitHub](https://github.com/Dan-jiw) · Available for freelance sysadmin work
