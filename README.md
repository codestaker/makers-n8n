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
- [✨ Overview](#️-overview)
- [✅ Requirements](#-requirements)
- [📦 Step-by-Step Setup](#-step-by-step-setup)
  - [1. Install Docker](#1-install-docker)
  - [2. Run n8n](#2-run-n8n)
  - [3. Install Nginx](#3-install-nginx)
  - [4. Configure Nginx](#4-configure-nginx)
  - [5. Enable and Restart Nginx](#5-enable-and-restart-nginx)
  - [6. Add SSL with Certbot](#6-add-ssl-with-certbot)
- [🧰 Troubleshooting & Fixes](#-troubleshooting--fixes)
- [⚡ Optional Enhancements](#-optional-enhancements)
- [💬 Credits](#-credits)

---

## ✨ Overview

This guide walks you through **self-hosting [n8n](https://n8n.io)** on a Linux server with:

✅ Docker  
✅ Nginx (Reverse Proxy)  
✅ Let's Encrypt SSL (via Certbot)  
✅ Custom Domain (`n8n.makersgridstudio.com`)

No prior DevOps experience required.

---

## ✅ Requirements

- A domain name (e.g. `n8n.yourdomain.com`)
- Ubuntu 22.04 VPS (GCP, AWS, etc.)
- Terminal & SSH access
- Basic copy-paste CLI experience

---

## 📦 Step-by-Step Setup

### 1. Install Docker

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker

2. Run n8n

Replace n8n.yourdomain.com with your domain.

sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="n8n.yourdomain.com" \
-e WEBHOOK_TUNNEL_URL="https://n8n.yourdomain.com/" \
-e WEBHOOK_URL="https://n8n.yourdomain.com/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n

3. Install Nginx

sudo apt install nginx -y

4. Configure Nginx

Create config file:

sudo nano /etc/nginx/sites-available/n8n.conf

Paste:

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

Save & exit:

    CTRL + O → Enter to save

    CTRL + X to exit

5. Enable and Restart Nginx

sudo ln -s /etc/nginx/sites-available/n8n.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

6. Add SSL with Certbot

sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d n8n.yourdomain.com

✅ Enter your email
✅ Agree to TOS
✅ Choose to share (Y/N) with EFF
✅ Certbot installs SSL + reloads Nginx
🧰 Troubleshooting & Fixes
❌ Problem	✅ Solution
502 Bad Gateway	Make sure Docker is running: docker ps
Certbot fails	Wait for DNS propagation & try again
nano confusing	Use CTRL+O to save, CTRL+X to exit
Port 80 in use	Run sudo lsof -i :80 to see what's blocking
SSL doesn't renew	Test with: sudo certbot renew --dry-run
Snap errors / certbot outdated	Run: sudo apt purge certbot && sudo snap install core; sudo snap refresh core; sudo snap install --classic certbot; sudo ln -s /snap/bin/certbot /usr/bin/certbot
⚡ Optional Enhancements

    Use PostgreSQL instead of default SQLite

    Connect GitHub repo to version-control workflows

    Set up .n8n auto-backups via cron

    Add BetterStack or UptimeRobot monitoring

    Use Cloudflare DNS for extra protection

💬 Credits

Crafted with ❤️ by Mezzop
🔗 Live instance: n8n.makersgridstudio.com
🖥️ Hosted on: GCP Free Tier VM (Ubuntu 22.04)
