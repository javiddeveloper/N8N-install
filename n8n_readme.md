
<p align="center">
  <img src="https://n8n.io/images/n8n-logo.png" width="100" alt="n8n logo" />
</p>

<h1 align="center">⚙️ Self-hosted n8n on Ubuntu with Cloudflare SSL</h1>

> 🚀 This guide shows you how to install and configure [n8n](https://n8n.io) on a clean Ubuntu server with a real domain, secured using Cloudflare SSL certificates.

---

## 📦 What You'll Set Up

- ✅ **n8n** (Node.js workflow automation tool)
- ✅ **Ubuntu 20.04+** or any compatible Linux distro
- ✅ **Node.js 20+** & npm
- ✅ **nginx** as reverse proxy
- ✅ 🔐 **Cloudflare SSL** using `cert.pem` and `key.pem`
- ✅ A custom domain (e.g., `agent.fireforks.site`)

---

## 🚧 Prerequisites

- 🔹 A VPS with a clean Ubuntu installation
- 🔹 Domain name connected to Cloudflare
- 🔹 Access to `cert.pem` & `key.pem` from Cloudflare dashboard

---

## 🛠️ Installation Steps

### 1. Update your system
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install required packages
```bash
sudo apt install -y nodejs npm nginx
```

> ⚠️ Make sure Node.js version is 20+. You can install from Nodesource if needed.

### 3. Install `n8n`
```bash
npm install n8n -g
```

### 4. Create working directory
```bash
mkdir -p ~/n8n && cd ~/n8n
```

### 5. Create `.env` file
```bash
nano ~/n8n/.env
```

Paste this in:

```env
GENERIC_TIMEZONE=Asia/Tehran
N8N_PORT=5678
N8N_HOST=localhost
N8N_PROTOCOL=http
WEBHOOK_URL=https://agent.fireforks.site/
```

### 6. Run n8n to test
```bash
n8n
```

Check if it's running:
```bash
ss -tuln | grep 5678
```

---

## 🌐 Setup Nginx as Reverse Proxy

### 1. Create SSL folder & move certs
```bash
sudo mkdir -p /etc/ssl/n8n
sudo cp ~/Downloads/cert.pem /etc/ssl/n8n/
sudo cp ~/Downloads/key.pem /etc/ssl/n8n/
```

> Replace `~/Downloads/` with your actual cert path.

### 2. Create nginx config
```bash
sudo nano /etc/nginx/sites-available/n8n.conf
```

Paste:

```nginx
server {
    listen 443 ssl;
    server_name agent.fireforks.site;

    ssl_certificate /etc/ssl/n8n/cert.pem;
    ssl_certificate_key /etc/ssl/n8n/key.pem;

    location / {
        proxy_pass http://localhost:5678;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### 3. Enable site
```bash
sudo ln -s /etc/nginx/sites-available/n8n.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

---

## ✅ Done! Access n8n

Visit:

```
https://agent.fireforks.site
```

If n8n is running, you’ll see the workflow UI.

---

## 🔁 Optional: Keep it running

You can use [`pm2`](https://pm2.keymetrics.io/) to run n8n as a background service:

```bash
npm install pm2 -g
pm2 start n8n
pm2 startup
pm2 save
```

---

## 🙌 Credits

- Inspired by [this Medium post](https://medium.com/@sebastiantorralba/how-to-install-n8n-on-ubuntu-with-https-e4070beb51e4)
- Logo © [n8n.io](https://n8n.io)

---

Made with ☕ and terminal rage 🤬 by [YourName]
