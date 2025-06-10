<h1 align="center">🌐 Self-Hosting n8n with Docker, Nginx & SSL</h1>

<p align="center">
  <b>Automate Anything. Host it Yourself.</b><br>
  🔧 A clean, tested self-hosting guide for <a href="https://n8n.io/">n8n</a> using Docker, Nginx, and Let's Encrypt.
</p>

<p align="center">
  👨‍💻 Built by <a href="https://x.com/mezzo_man" target="_blank">@Mezzop</a> • 🧪 <a href="https://n8n.makersgridstudio.com" target="_blank">Live Demo</a>
</p>

---

## 📚 Table of Contents

- [📌 Overview](#-overview)
- [🧰 Requirements](#-requirements)
- [⚙️ Step-by-Step Setup](#️-step-by-step-setup)
  - [Install Docker](#install-docker)
  - [Run n8n](#run-n8n)
  - [Install Nginx](#install-nginx)
  - [Configure Nginx](#configure-nginx)
  - [Enable and Restart Nginx](#enable-and-restart-nginx)
  - [Add SSL with Certbot](#add-ssl-with-certbot)
- [🚑 Troubleshooting & Fixes](#-troubleshooting--fixes)
- [🌟 Optional Enhancements](#-optional-enhancements)
- [🧾 Credits](#-credits)

---

## 📌 Overview

This guide walks you through **self-hosting [n8n](https://n8n.io)** on a Linux server with:

- ✅ Docker  
- ✅ Nginx (Reverse Proxy)  
- ✅ Let's Encrypt SSL (via Certbot)  
- ✅ Custom Domain (`n8n.makersgridstudio.com`)

---

## 🧰 Requirements

- A domain name (e.g. `n8n.yourdomain.com`)
- Ubuntu 22.04 VPS (GCP, AWS, etc.)
- Terminal & SSH access

---

## ⚙️ Step-by-Step Setup

### Install Docker

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker

Run n8n

sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="n8n.yourdomain.com" \
-e WEBHOOK_TUNNEL_URL="https://n8n.yourdomain.com/" \
-e WEBHOOK_URL="https://n8n.yourdomain.com/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n

Install Nginx

sudo apt install nginx -y

Configure Nginx

sudo nano /etc/nginx/sites-available/n8n.conf

Paste this config:

server {
    listen 80;
    server_name n8n.yourdomain.com;

    location / {
        proxy_pass http://localhost:5678;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_read_timeout 86400;
    }
}

Enable and Restart Nginx

sudo ln -s /etc/nginx/sites-available/n8n.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

Add SSL with Certbot

sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d n8n.yourdomain.com

🚑 Troubleshooting & Fixes
❌ Issue	✅ Solution
502 Bad Gateway	Ensure Docker is running: docker ps
Certbot fails	Make sure DNS points to server IP
Port already in use	Check: sudo lsof -i :80
SSL renew fails	Test: sudo certbot renew --dry-run
🌟 Optional Enhancements

    Setup external DB (PostgreSQL)

    Auto-backup .n8n folder via cron

    Add a Uptime Monitor (e.g. UptimeRobot)

    Secure server with UFW (firewall)

🧾 Credits

Created by @Mezzop
🔗 Live n8n Instance

Built with ❤️ on Ubuntu 22.04, Docker, and Let's Encrypt
