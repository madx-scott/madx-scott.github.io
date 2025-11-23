# WireGuard VPN Lab — DigitalOcean + Docker

This GitHub Pages document covers my full WireGuard VPN deployment using Docker on a DigitalOcean droplet, including all required screenshots, configs, and verification steps.

---

## 1. DigitalOcean Droplet Setup

I created a DigitalOcean account using the free 60-day credit and deployed a droplet with:

- Ubuntu 22.04 LTS  
- Basic $6/mo droplet  
- 1 GB RAM / 1 vCPU  
- Root password login  
- Region closest to me  

I connected using:

ssh root@<DROPLET_PUBLIC_IP>

yaml
Copy code

---

## 2. Installed Docker and Docker Compose

apt update
apt install -y docker.io docker-compose
systemctl enable --now docker

yaml
Copy code

---

## 3. WireGuard Docker Setup

I created my directory:

mkdir -p ~/wireguard
cd ~/wireguard

yaml
Copy code

Then created `docker-compose.yml`:

YAML
version: "3.9"

services:
  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=0
      - PGID=0
      - TZ=America/Chicago
      - SERVERURL=<DROPLET_PUBLIC_IP>
      - SERVERPORT=51820
      - PEERS=2
      - PEERDNS=1.1.1.1
      - INTERNAL_SUBNET=10.13.13.0
    volumes:
      - ./config:/config
      - /lib/modules:/lib/modules:ro
    ports:
      - "51820:51820/udp"
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
Then started it:

Copy code
docker-compose up -d
This auto-generated peers for phone (peer1) and PC (peer2).

4. Enabled NAT on DigitalOcean
DigitalOcean droplets must NAT VPN traffic:

bash
Copy code
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
echo "net.ipv4.ip_forward=1" | tee /etc/sysctl.d/99-wireguard.conf
sysctl --system
docker restart wireguard
5. Phone Setup (Peer 1)
I exported the QR code:

bash
Copy code
docker cp wireguard:/config/peer1/peer1.png ./peer1.png
Then scanned it using the WireGuard iOS app.

Phone WireGuard UI
![Phone WireGuard UI]<img width="309" height="654" alt="Screenshot 2025-11-23 135336" src="https://github.com/user-attachments/assets/18d571b8-61b8-46a1-8959-f7f9f473d461" />

6. PC Setup (Peer 2)
I exported my PC config:

bash
Copy code
docker cp wireguard:/config/peer2/peer2.conf ./peer2.conf
Then imported it into the WireGuard Windows desktop app.

PC WireGuard UI
![PC WireGuard UI](images/Screenshot 2025-11-23 134143.png)

7. Public IP Verification
I used ifconfig.me before and after connecting on both devices.

Phone – Before VPN

Phone – After VPN

PC – Before VPN

PC – After VPN

These show that my internet traffic is being routed through the DigitalOcean WireGuard server.

8. Repository Files Included
docker-compose.yml

All screenshots inside images/ folder

This index.md (GitHub Pages)

No private keys or sensitive config fields are included.<img width="725" height="631" alt="Screenshot 2025-11-23 134143" src="https://github.com/user-attachments/assets/cc398b24-6d86-4dbb-b42d-80fac15237aa" />
