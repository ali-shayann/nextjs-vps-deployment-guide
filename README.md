# Deploy Next.js / Node.js App to VPS via SSH

> **The simplest step-by-step guide** to deploying a Next.js or Node.js app on a Linux VPS using SSH, PM2, and cPanel — written for junior developers.

[![GitHub Stars](https://img.shields.io/github/stars/nowshad7/nextjs-vps-deployment-guide?style=social)](https://github.com/nowshad7/nextjs-vps-deployment-guide)
[![GitHub Forks](https://img.shields.io/github/forks/nowshad7/nextjs-vps-deployment-guide?style=social)](https://github.com/nowshad7/nextjs-vps-deployment-guide/fork)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
![Platform](https://img.shields.io/badge/platform-Linux%20VPS-orange)
![Node](https://img.shields.io/badge/Node.js-18%2B-green)

---

## Deployment Flow

Here's the big picture of what you'll be doing. Every step in this guide maps to a box below.

```mermaid
flowchart TD
    A([Your Local Machine]) -->|git push| B[GitHub Repository]
    B -->|git clone / git pull| C[Linux VPS via SSH]

    subgraph VPS [VPS Server]
        C --> D[Install Node.js with NVM]
        D --> E[Install PM2]
        E --> F[Set .env variables]
        F --> G[npm install + npm run build]
        G --> H[pm2 start → App on port 3000]

        H --> I{Apache Reverse Proxy}

        subgraph Apache [Apache - cPanel/WHM]
            I --> J[Create userdata proxy.conf\nfor your domain]
            J --> K[Copy config for SSL]
            K --> L[Rebuild Apache\n/scripts/rebuildhttpdconf]
        end

        L --> M[SSL via AutoSSL / Let's Encrypt]
    end

    M --> N([yourdomain.com - Live on HTTPS])

    style VPS fill:#f0f4ff,stroke:#4a6cf7,stroke-width:2px
    style Apache fill:#fff7e6,stroke:#f0a500,stroke-width:1px
    style A fill:#e6ffe6,stroke:#2e7d32
    style N fill:#e6ffe6,stroke:#2e7d32
    style H fill:#fff3e0,stroke:#e65100
    style M fill:#e8f5e9,stroke:#388e3c
```

> Read left to right: your code goes from GitHub → VPS → PM2 (runs on port 3000) → Apache proxies it to your domain → SSL makes it HTTPS.

---

## Who Is This For?

This guide is for developers who want to:

- Deploy a **Next.js or Node.js app** to a VPS for the first time
- Use **GoDaddy VPS, DigitalOcean, Linode, Vultr**, or any Linux server
- Set up **PM2** to keep their app running 24/7
- Configure **Apache** as a reverse proxy using cPanel/WHM
- Get **HTTPS / SSL** working on their domain
- Understand **SSH basics** without prior server experience

**Estimated time: 20–30 minutes**

---

## Stack & Requirements

| Tool | Purpose |
|---|---|
| **Linux VPS** | AlmaLinux 8 / Ubuntu / Debian |
| **SSH** | Secure remote access to the server |
| **NVM** | Install and manage Node.js versions |
| **PM2** | Keep your app alive 24/7 |
| **Apache** | Reverse proxy (port 3000 → 80/443) |
| **Let's Encrypt** | Free SSL certificate |
| **cPanel / WHM** | GUI for domain, files, and SSL |

---

## Table of Contents

- [Step 1 — Connect via SSH](#step-1--connect-via-ssh)
- [Step 2 — Install Node.js with NVM](#step-2--install-nodejs-with-nvm)
- [Step 3 — Install PM2](#step-3--install-pm2)
- [Step 4 — Clone Your Project](#step-4--clone-your-project)
- [Step 5 — Set Environment Variables](#step-5--set-environment-variables)
- [Step 6 — Install Dependencies & Build](#step-6--install-dependencies--build)
- [Step 7 — Start App with PM2](#step-7--start-app-with-pm2)
- [Step 8 — Connect Your Domain](#step-8--connect-your-domain)
- [Step 9 — Enable SSL / HTTPS](#step-9--enable-ssl--https)
- [Updating Your App](#updating-your-app)
- [One-Command Deploy Script](#one-command-deploy-script)
- [PM2 Cheatsheet](#pm2-cheatsheet)
- [Apache Cheatsheet](#apache-cheatsheet)
- [Troubleshooting](#troubleshooting)
- [Security Checklist](#security-checklist)

---

## Step 1 — Connect via SSH

Open **Terminal** (Mac/Linux) or **PowerShell** (Windows):

```bash
ssh your_username@your_server_ip
```

Example:
```bash
ssh john@192.168.1.100
```

> The cursor won't move when typing your password — that's normal. Press Enter when done.

You're in when you see something like: `[john@server ~]$`

> **cPanel users:** You can also use the built-in terminal at  
> `https://YOUR_SERVER_IP:2083` → **Advanced → Terminal**

---

## Step 2 — Install Node.js with NVM

NVM (Node Version Manager) is the recommended way to install Node.js on a server.

```bash
# 1. Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 2. Reload shell config
source ~/.bashrc

# 3. Install latest LTS Node.js
nvm install --lts
nvm use --lts

# 4. Verify
node -v    # should print v20.x.x
npm -v     # should print 10.x.x
```

> Why NVM? It avoids `sudo` permission issues and lets you switch Node versions easily.

---

## Step 3 — Install PM2

PM2 is a process manager that keeps your app alive after crashes and server reboots.

```bash
npm install -g pm2
```

---

## Step 4 — Clone Your Project

```bash
# Navigate to home directory
cd ~

# Clone your repository
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git

# Enter the project folder
cd YOUR_REPO
```

> **Private repo?** Use a GitHub Personal Access Token (PAT) as your password when prompted.  
> Generate one at: **GitHub → Settings → Developer Settings → Personal Access Tokens → Tokens (classic)**

---

## Step 5 — Set Environment Variables

Create your environment file directly on the server:

```bash
nano .env.local
```

Add your variables:

```env
NODE_ENV=production
NEXT_PUBLIC_API_URL=https://yourdomain.com
DATABASE_URL=your_database_url
NEXTAUTH_SECRET=your_secret
```

Save: **Ctrl + X → Y → Enter**

> Never commit `.env` files to Git. Add `.env*` to your `.gitignore`.

---

## Step 6 — Install Dependencies & Build

```bash
# Install packages
npm install

# Build for production
npm run build
```

> Build takes 1–3 minutes. Wait until it finishes.

Successful build output looks like:
```
Route (app)     Size
○  /             2.5 kB
λ  /api/hello   0 B
```

---

## Step 7 — Start App with PM2

```bash
# Start the app
pm2 start npm --name "my-app" -- start

# Save the process list (persists after reboot)
pm2 save

# Configure PM2 to auto-start on server reboot
pm2 startup
```

> **Important:** After running `pm2 startup`, copy and run the full `sudo env PATH=...` command it prints. Without this, your app won't restart after a server reboot.

App is live when you see:
```
┌────┬───────────┬───────┬────────┬─────┬──────────┐
│ id │ name      │ mode  │ status │ cpu │ memory   │
├────┼───────────┼───────┼────────┼─────┼──────────┤
│ 0  │ my-app    │ fork  │ online │ 0%  │ 30.0mb   │
└────┴───────────┴───────┴────────┴─────┴──────────┘
```

Test it locally on the server:
```bash
curl http://localhost:3000
```

---

## Step 8 — Connect Your Domain

Your app runs on port **3000**. You need Apache to proxy it to port **80/443** so your domain works in the browser.

This is the recommended approach for **cPanel/WHM servers** (GoDaddy VPS, etc.). It creates a proper per-domain proxy config that survives Apache rebuilds — unlike `.htaccess` hacks.

### Step 8.1 — Create the config directories for your domain

Replace `yourdomain.com` and `cpanelusername` with your actual domain and cPanel username throughout.

```bash
mkdir -p /etc/apache2/conf.d/userdata/std/2_4/cpanelusername/yourdomain.com
mkdir -p /etc/apache2/conf.d/userdata/ssl/2_4/cpanelusername/yourdomain.com
```

> These folders tell WHM/Apache where to find per-domain custom config for both HTTP and HTTPS.

### Step 8.2 — Create the proxy config file

```bash
nano /etc/apache2/conf.d/userdata/std/2_4/cpanelusername/yourdomain.com/proxy.conf
```

Paste this inside:

```apache
<IfModule mod_proxy.c>
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</IfModule>
```

Save: **Ctrl + X → Y → Enter**

### Step 8.3 — Copy the same config for SSL (HTTPS)

```bash
cp /etc/apache2/conf.d/userdata/std/2_4/cpanelusername/yourdomain.com/proxy.conf \
   /etc/apache2/conf.d/userdata/ssl/2_4/cpanelusername/yourdomain.com/
```

### Step 8.4 — Rebuild Apache (important!)

```bash
/scripts/ensure_vhost_includes --all-users
/scripts/rebuildhttpdconf
/scripts/restartsrv_httpd
```

Wait for Apache to restart successfully. Once it does, visiting `yourdomain.com` should load your Next.js app.

> **What this does:** WHM manages Apache's vhost config automatically. Putting your config in the `userdata` directory is the correct way to inject custom rules — they survive cPanel/WHM updates and rebuilds. Editing vhosts directly would get overwritten.

### Why not Nginx?

If your server uses Nginx instead of Apache, create a config like this:

```bash
sudo nano /etc/nginx/sites-available/my-app
```

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass         http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header   Upgrade $http_upgrade;
        proxy_set_header   Connection 'upgrade';
        proxy_set_header   Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Enable and reload:

```bash
sudo ln -s /etc/nginx/sites-available/my-app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Step 9 — Enable SSL / HTTPS

### Via cPanel (easiest — recommended for GoDaddy VPS)

1. Login to cPanel → **Security → SSL/TLS Status**
2. Click **Run AutoSSL**
3. Wait 1–2 minutes — free Let's Encrypt certificate applied

### Via Certbot (SSH)

```bash
# Install Certbot (AlmaLinux / CentOS)
sudo dnf install certbot python3-certbot-nginx -y

# Get and install certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Test auto-renewal
sudo certbot renew --dry-run
```

---

## Updating Your App

Every time you push new code to GitHub, SSH in and run:

```bash
cd ~/YOUR_REPO
git pull
npm install
npm run build
pm2 restart my-app
```

---

## One-Command Deploy Script

Save this as `deploy.sh` in your project root:

```bash
#!/bin/bash
set -e  # Stop on any error

echo "Pulling latest changes..."
git pull

echo "Installing dependencies..."
npm install

echo "Building..."
npm run build

echo "Restarting app..."
pm2 restart my-app

echo "Done! App is live."
```

Make it executable:
```bash
chmod +x deploy.sh
```

Deploy any time with:
```bash
./deploy.sh
```

---

## PM2 Cheatsheet

| Command | What it does |
|---|---|
| `pm2 list` | Show all running apps |
| `pm2 logs my-app` | View live logs (Ctrl+C to exit) |
| `pm2 restart my-app` | Restart the app |
| `pm2 reload my-app` | Zero-downtime restart |
| `pm2 stop my-app` | Stop the app |
| `pm2 delete my-app` | Remove from PM2 |
| `pm2 monit` | Real-time CPU/memory dashboard |
| `pm2 show my-app` | Detailed app info |
| `pm2 save` | Save process list |
| `pm2 startup` | Enable auto-start on reboot |

---

## Apache Cheatsheet

Useful Apache commands when you're on a cPanel/WHM server:

| Command | What it does |
|---|---|
| `/scripts/restartsrv_httpd` | Restart Apache (WHM way) |
| `/scripts/rebuildhttpdconf` | Rebuild Apache config from WHM settings |
| `/scripts/ensure_vhost_includes --all-users` | Apply userdata config for all accounts |
| `httpd -t` | Test Apache config for syntax errors |
| `apachectl -t` | Same as above (standard Linux) |
| `apachectl graceful` | Reload Apache without dropping connections |
| `systemctl status httpd` | Check if Apache is running |
| `tail -f /var/log/apache2/error.log` | Watch Apache error log live |
| `tail -f /usr/local/apache/logs/error_log` | Same, cPanel path |
| `grep "yourdomain" /usr/local/apache/conf/httpd.conf` | Find your domain's vhost in Apache config |

> On cPanel servers, always use `/scripts/` commands to restart/rebuild Apache — don't use `systemctl restart httpd` as it bypasses WHM's process watcher.

---

## Troubleshooting

| Problem | Solution |
|---|---|
| `nvm: command not found` | Run `source ~/.bashrc` then retry |
| `npm run build` fails | Check all `.env` variables are set |
| App not loading in browser | Run `pm2 logs my-app` to see errors |
| Port 3000 already in use | `kill $(lsof -t -i:3000)` then restart PM2 |
| 502 Bad Gateway | App crashed — check `pm2 logs` |
| Changes not showing | Run `npm run build` then `pm2 restart my-app` |
| Git pull asks for password | Use a Personal Access Token (not your GitHub password) |
| SSL not working | Check domain DNS is pointing to your server IP |
| Proxy not working after Apache rebuild | Re-run `/scripts/ensure_vhost_includes --all-users` |
| Another domain redirecting to your app | Double-check the `userdata` path has the correct domain folder name |

Useful diagnostic commands:
```bash
pm2 logs my-app              # App errors
pm2 list                     # Is the app running?
lsof -i :3000                # Who is on port 3000?
df -h                        # Disk space
free -h                      # RAM usage
curl http://localhost:3000   # Does the app respond?
tail -f /usr/local/apache/logs/error_log  # Apache errors
```

---

## Security Checklist

- [ ] Change default SSH password after first login
- [ ] Set up SSH key-based authentication
- [ ] Disable password SSH login after keys are working
- [ ] Never commit `.env` files to Git
- [ ] Add `.env*` to `.gitignore`
- [ ] Keep Node.js, npm, and PM2 updated
- [ ] Enable a firewall (`ufw allow 80`, `ufw allow 443`, `ufw allow 22`)

---

## Recommended Project Structure

```
your-project/
├── .env.local          <- NOT committed (add to .gitignore)
├── .gitignore          <- must include .env*
├── deploy.sh           <- one-command deploy script
├── package.json
├── next.config.js
├── app/
│   └── page.js
└── public/
    └── favicon.ico
```

---

## Contributing

Found a mistake or want to add something? PRs and Issues are welcome!

1. Fork the repo
2. Create your branch: `git checkout -b fix/typo`
3. Commit changes: `git commit -m 'fix: correct typo in step 3'`
4. Push: `git push origin fix/typo`
5. Open a Pull Request

---

## License

MIT © [Nowshad](https://github.com/nowshad7)

---

> Made for junior developers. If this helped you, consider giving it a star — it helps others find it too!
