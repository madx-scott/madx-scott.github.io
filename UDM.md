 Unknown Device Monitor

A lightweight Python script that alerts you **only when an unknown device is actively connected** to your network. Designed for homelabs and small networks.

---

## What It Does

- Scans the local network (LAN) for **active devices**
- Compares detected devices to an **approved allowlist**
- Optionally checks **Tailscale** peers
- Sends a **Discord alert only when a new unknown device is detected**
- No alerts if no unknown devices are active

---

## Requirements

- Linux  
- Python 3  
- `arp-scan` (recommended) or `nmap`  
- Optional: `tailscale`  

Install dependency:
```bash
sudo apt install arp-scan
```

---

## Setup

Clone the repository:
```bash
git clone https://github.com/<your-username>/unknown-device-watch.git
cd unknown-device-watch
```

Edit approved devices in `unknown_device_watch.py`:
```python
APPROVED = [
    {"name": "Router", "local_ip": "192.168.1.1", "ts_ip": None, "mac": "aa:bb:cc:dd:ee:ff"},
]
```

Set Discord webhook:
```bash
sudo bash -lc 'echo "DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/XXX/YYY" > /etc/net-watch.env'
```

---

## Usage

Manual run:
```bash
sudo -E python3 unknown_device_watch.py
```

Automatic monitoring (cron every 5 minutes):
```cron
*/5 * * * * set -a; . /etc/net-watch.env; set +a; /usr/bin/python3 /path/to/unknown_device_watch.py
```

---

## Alerts

- Alerts trigger **only when an unknown device is actively connected**
- No repeat alerts for the same device
- No historical tracking or uptime monitoring
