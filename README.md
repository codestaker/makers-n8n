# 🌐 Self-Hosting n8n with Docker, Nginx, and SSL on Ubuntu 22.04

![Docker](https://img.shields.io/badge/Docker-Supported-blue)
![Ubuntu](https://img.shields.io/badge/Ubuntu-22.04-orange)
![Let's Encrypt](https://img.shields.io/badge/SSL-Let's%20Encrypt-brightgreen)
![n8n](https://img.shields.io/badge/n8n-Automation-success)

> A complete beginner-friendly guide to self-hosting [n8n](https://n8n.io) — the powerful open-source workflow automation tool — on a Linux server using Docker, Nginx, and Let's Encrypt SSL.

🧪 Tested on: **Ubuntu 22.04 (GCP Free Tier VM)**  
🌐 With real domain: `n8n.example.com`

---

## 📚 Table of Contents

- [✅ Requirements](#-requirements)
- [📦 Step-by-Step Setup](#-step-by-step-setup)
  - [1. Install Docker](#1-install-docker)
  - [2. Run n8n with Docker](#2-run-n8n-with-docker)
  - [3. Install Nginx](#3-install-nginx)
  - [4. Configure Nginx Reverse Proxy](#4-configure-nginx-reverse-proxy)
  - [5. Enable and Restart Nginx](#5-enable-and-restart-nginx)
  - [6. Secure with Let's Encrypt SSL](#6-secure-with-lets-encrypt-ssl)
- [🧰 Common Errors & Fixes](#-common-errors--fixes)
- [🔄 Keep It Running](#-keep-it-running)
- [⚡ Optional Enhancements](#-optional-enhancements)
- [💬 Credits](#-credits)

---

## ✅ Requirements

- A registered domain (e.g. `n8n.example.com`)
- Ubuntu 22.04 VPS (GCP, AWS, etc.)
- SSH terminal access
- Basic command line experience

---

## 📦 Step-by-Step Setup

### 1. Install Docker

```bash
sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

2. Run n8n with Docker

Replace n8n.example.com with your actual domain.

sudo docker run -d --restart unless-stopped -it \
--name n8n \
-p 5678:5678 \
-e N8N_HOST="n8n.example.com" \
-e WEBHOOK_TUNNEL_URL="https://n8n.example.com/" \
-e WEBHOOK_URL="https://n8n.example.com/" \
-v ~/.n8n:/root/.n8n \
n8nio/n8n

3. Install Nginx

sudo apt install nginx -y

4. Configure Nginx Reverse Proxy

sudo nano /etc/nginx/sites-available/n8n.conf

Paste the config below:

server {
    listen 80;
    server_name n8n.example.com;

    location / {
        proxy_pass http://localhost:5678;
        proxy_http_version 1.1;
        chunked_transfer_encoding off;
        proxy_buffering off;
        proxy_cache off;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_read_timeout 86400;
    }
}

To Save & Exit nano:

    Press CTRL + O → then Enter to write (save)

    Press CTRL + X to exit

5. Enable and Restart Nginx

sudo ln -s /etc/nginx/sites-available/n8n.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx

6. Secure with Let's Encrypt SSL

sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d n8n.example.com

During setup:

    Enter your email

    Agree to terms (press Y + Enter)

    Choose if you want to share your email with EFF (Y/N)

🧰 Common Errors & Fixes
❌ Issue	✅ Solution
502 Bad Gateway	Ensure Docker is running: docker ps
Certbot fails	Make sure your domain points to the server IP (DNS may need time)
Can't edit with nano	Use CTRL + O, Enter, then CTRL + X
SSL renewal not working	Test with: sudo certbot renew --dry-run
Port 80 already in use	Run: sudo lsof -i :80 then stop conflicting service
Outdated Certbot (Snap Errors)	Run: sudo snap install core; sudo snap refresh core; sudo snap install --classic certbot
Then link: sudo ln -s /snap/bin/certbot /usr/bin/certbot
🔄 Keep It Running

    Docker auto-restarts n8n with --restart unless-stopped

    Monitor logs:

docker logs -f n8n

⚡ Optional Enhancements

    💾 External DB: Setup PostgreSQL or SQLite

    🧠 GitHub Sync: Version-control workflows

    🔄 Auto-backups: .n8n folder using cron

    📈 Monitoring: Use UptimeRobot or BetterStack

💬 Credits

Crafted with ❤️ by [YourName or @yourhandle]
Based on a real setup experience using GCP Free Tier Ubuntu 22.04
