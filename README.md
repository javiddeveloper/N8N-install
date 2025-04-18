# ⚙️ n8n Installation Guide on a Fresh Ubuntu Server

This guide helps you install and run **n8n** on a clean Ubuntu server with your own domain and HTTPS support.  
Let's automate like a boss!

---

## ⚡ Requirements
- Fresh Ubuntu 20.04+ server
- A domain name (`YOUR_DOMAIN`)
- SSL cert and key (`cert.pem` & `key.pem`)
- Node.js 20+

---

## 🚀 Installation Steps

### 1. Update the system
```bash
sudo apt update && sudo apt upgrade -y
```

### 2. Install dependencies
```bash
sudo apt install -y curl gnupg2 ca-certificates lsb-release build-essential
```

### 3. Install Node.js 20
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
```

### 4. Verify Node & NPM
```bash
node -v
npm -v
```

### 5. Install PM2 to manage the n8n process
```bash
sudo npm install pm2 -g
```

### 6. Create folder and init project
```bash
mkdir ~/n8n && cd ~/n8n
npm init -y
npm install n8n
```

### 7. Create `.env` file
```bash
nano ~/.env
```

Paste the following and change `YOUR_DOMAIN`:
```
DOMAIN_NAME=YOUR_DOMAIN
N8N_HOST=0.0.0.0
N8N_PORT=5678
VUE_APP_URL_BASE_API=https://YOUR_DOMAIN/
N8N_PROTOCOL=https
N8N_BASIC_AUTH_ACTIVE=true
N8N_BASIC_AUTH_USER=admin
N8N_BASIC_AUTH_PASSWORD=strongpassword
```

Save with `CTRL+X`, then `Y`, and `Enter`.

### 8. Start n8n with PM2
```bash
pm2 start ./node_modules/n8n/bin/n8n
pm2 save
pm2 startup
```

### 9. Set up Nginx reverse proxy
```bash
sudo apt install nginx -y
sudo nano /etc/nginx/sites-available/n8n.conf
```

Paste this:
```
server {
    listen 80;
    server_name YOUR_DOMAIN;

    location / {
        proxy_pass http://localhost:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    listen 443 ssl;
    ssl_certificate /etc/ssl/certs/cert.pem;
    ssl_certificate_key /etc/ssl/private/key.pem;
}
```

Then run:
```bash
sudo ln -s /etc/nginx/sites-available/n8n.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

Make sure to upload your cert files to:
- `/etc/ssl/certs/cert.pem`
- `/etc/ssl/private/key.pem`

---

## ✅ Access
Now visit:  
`https://YOUR_DOMAIN`

Login with:
- **Username:** `admin`
- **Password:** `strongpassword` (or your own from .env)

---

Made with love & automation magic by [n8n.io](https://n8n.io) ✨