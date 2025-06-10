# 🌐 Self-Hosting n8n with Docker, Nginx, and SSL on a Linux Server

A complete guide to self-hosting **n8n**, the powerful open-source workflow automation tool, on a Linux VPS using **Docker**, **Nginx**, and **Let's Encrypt SSL**.

Tested on **Ubuntu 22.04 (GCP Free Tier VM)** with a real domain (`n8n.example.com`). Includes beginner-friendly notes, common error fixes, and helpful CLI keypresses.

---

## ✅ Requirements

- A registered domain name (e.g. `n8n.example.com`)
- Ubuntu 22.04 Linux VPS (GCP, AWS, or any provider)
- Basic SSH access to the server
- Terminal experience (copy/paste and some keypresses)

---

## 📦 Step-by-Step Setup

### 1. Update & Install Docker

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

4. Configure Nginx as a Reverse Proxy

sudo nano /etc/nginx/sites-available/n8n.conf

Paste the following content:

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

Save & Exit:

    Press CTRL + O to write (save)

    Press Enter to confirm

    Press CTRL + X to exit

5. Enable and Restart Nginx

sudo ln -s /etc/nginx/sites-available/n8n.conf /etc/nginx/sites-enabled/
sudo nginx -t     # (tests config)
sudo systemctl restart nginx

6. Secure with Let's Encrypt SSL

sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d n8n.example.com

    Enter your email

    Agree to Terms (press Y then Enter)

    Choose to share email with EFF or not (Y or N)

🧰 Common Errors & Fixes
❌ Issue	✅ Solution
502 Bad Gateway	Ensure Docker is running: docker ps
Certbot fails	Make sure your domain points to the server IP (DNS propagation may take minutes)
Can't edit file with nano	Use CTRL + O (write), CTRL + X (exit)
SSL renewal fails later	Run: sudo certbot renew --dry-run to test
Port already in use	Run: sudo lsof -i :80 to find what’s using it, then stop it
🔄 Keep It Always Running

    Docker restarts n8n automatically:
    --restart unless-stopped

    To monitor logs:

    docker logs -f n8n

⚡ Optional Enhancements

    Setup external DB: PostgreSQL or SQLite

    Use GitHub repo to version-control workflows

    Auto-backup .n8n folder using cron

    Add a Uptime Monitor (e.g. UptimeRobot, BetterStack)

💬 Credits

Crafted with ❤️ by [Pelumi @ https://x.com/mezzo_man]

    Hosted on a GCP Free Tier Ubuntu Server
    Based on the real setup journey with real errors & solutions
