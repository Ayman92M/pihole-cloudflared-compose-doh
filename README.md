# pihole-cloudflared-compose-doh
Pi-hole with DNS over HTTPS (DoH) using Cloudflared

This repository sets up [Pi-hole](https://pi-hole.net/) using [Cloudflared](https://developers.cloudflare.com/cloudflared/) for secure DNS-over-HTTPS (DoH) resolution on your local network using Docker.

> ✅ **Tested only on Ubuntu**

---

## 🔧 Prerequisites

Before starting, disable the default systemd stub listener on port 53 to free it for Pi-hole:

```bash
sudo nano /etc/systemd/resolved.conf
```

Update or ensure the following lines exist:

```ini
DNSStubListener=no
DNS=127.0.0.1
```

Then restart the service:

```bash
sudo systemctl restart systemd-resolved
```

---

## 📥 Clone the Repository

```bash
git clone https://github.com/Ayman92M/pihole-cloudflared-compose-doh.git
cd pihole-cloudflared-compose-doh
```


---

## 📦 Docker Compose Setup

### File: `docker-compose.yml`

```yaml
version: '3'

services:
  cloudflared:
    image: cloudflare/cloudflared:latest
    container_name: cloudflared
    command: >
      proxy-dns
      --address 0.0.0.0
      --port 5053
      --upstream https://1.1.1.1/dns-query
      --upstream https://1.0.0.1/dns-query
    expose:
      - "5053"
    networks:
      - pihole_net
    restart: unless-stopped

  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    environment:
      TZ: Europe/Stockholm
      DNS1: cloudflared#5053
      DNS2: no
      FTLCONF_dns_listeningMode: all
    volumes:
      - ./etc-pihole:/etc/pihole
    cap_add:
      - NET_ADMIN
      - SYS_TIME
      - SYS_NICE
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80"
      - "8443:443"
    networks:
      - pihole_net
    restart: unless-stopped

networks:
  pihole_net:
    driver: bridge
```

Start the stack:

```bash
docker compose up -d
```

---

## 🌐 Access the Pi-hole Web UI

Navigate to:

```
http://<your-server-ip>:80/admin
```


---

## 🛠 Pi-hole Configuration

1. Go to **Settings → DNS**
2. Uncheck all default *Upstream DNS Servers*
3. Add a **Custom DNS server**:

```
cloudflared#5053
```

---

## ✅ Done!

You now have Pi-hole running with secure DNS-over-HTTPS using Cloudflare's DNS!
