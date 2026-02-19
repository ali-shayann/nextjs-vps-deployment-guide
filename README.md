# 🚀 Deploying a Next.js / Node.js App to VPS via SSH

A step-by-step deployment guide for junior developers.  
**Stack:** GoDaddy VPS · AlmaLinux 8 · cPanel · SSH · PM2 · Nginx/Apache

---

## 📋 Table of Contents

- [Prerequisites](#prerequisites)
- [Step 1 — Connect to Server via SSH](#step-1--connect-to-server-via-ssh)
- [Step 2 — Install Node.js with NVM](#step-2--install-nodejs-with-nvm)
- [Step 3 — Install PM2](#step-3--install-pm2)
- [Step 4 — Clone Your Project](#step-4--clone-your-project)
- [Step 5 — Configure Environment Variables](#step-5--configure-environment-variables)
- [Step 6 — Install & Build](#step-6--install--build)
- [Step 7 — Start with PM2](#step-7--start-with-pm2)
- [Step 8 — Connect Your Domain](#step-8--connect-your-domain)
- [Step 9 — Enable SSL (HTTPS)](#step-9--enable-ssl-https)
- [Updating Your App](#-updating-your-app)
- [PM2 Cheatsheet](#-pm2-cheatsheet)
- [Troubleshooting](#-troubleshooting)

---

## Prerequisites

Before starting, make sure you have:

| Requirement | Details |
|---|---|
| A VPS server | With SSH access enabled |
| SSH credentials | Host IP, username, password |
| Your project | Pushed to a Git repository (GitHub, GitLab, etc.) |
| A domain name | Pointed to your server's IP (optional but recommended) |

---

## Step 1 — Connect to Server via SSH

Open your terminal (Mac/Linux) or PowerShell (Windows):

```bash
ssh your_username@your_server_ip
```

**Example:**
```bash
ssh nowshad@100.101.101.101
```

Enter your password when prompted. *(The cursor won't move while typing — that's normal.)*

✅ **You're in when you see:** `[nowshad@101 ~]$`

---

## Step 2 — Install Node.js with NVM

NVM lets you install and switch between Node.js versions easily.

```bash
# 1. Download and install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 2. Reload your shell so nvm command works
source ~/.bashrc

# 3. Install the latest LTS version of Node.js
nvm install --lts
nvm use --lts

# 4. Verify installation
node -v   # Should print something like: v20.x.x
npm -v    # Should print something like: 10.x.x
```

> 💡 **Why NVM?** It lets you manage multiple Node.js versions and avoids permission issues with global packages.

---

## Step 3 — Install PM2

PM2 is a process manager that keeps your app alive 24/7, even after server reboots.

```bash
npm install -g pm2
```

---

## Step 4 — Clone Your Project

Navigate to where you want your app to live and clone your repo:

```bash
# Go to home directory
cd ~

# Clone your repo (replace with your actual repo URL)
git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git

# Enter the project folder
cd YOUR_REPO
```

> 💡 **Private repo?** You'll need a GitHub Personal Access Token (PAT).  
> Generate one at: **GitHub → Settings → Developer Settings → Personal Access Tokens**  
> Use the token as your password when git asks.

---

## Step 5 — Configure Environment Variables

Create your `.env` (or `.env.local` for Next.js) file on the server:

```bash
# Create the env file
nano .env.local
```

Add your variables inside:

```env
NODE_ENV=production
NEXT_PUBLIC_API_URL=https://yourdomain.com
DATABASE_URL=your_database_connection_string
SECRET_KEY=your_secret_key
```

Save and exit: **Ctrl + X → Y → Enter**

> ⚠️ **Never commit `.env` files to Git.** Make sure `.env*` is in your `.gitignore`.

---

## Step 6 — Install & Build

```bash
# Install all dependencies
npm install

# Build for production (Next.js)
npm run build
```

> ⏳ The build may take 1–3 minutes. Wait for it to complete before moving on.

✅ **Successful build looks like:**
```
▶ Next.js build complete
Route (app)  ...
λ  (Lambda functions)
```

---

## Step 7 — Start with PM2

```bash
# Start your app
pm2 start npm --name "my-app" -- start

# Save process list (so it survives reboots)
pm2 save

# Enable PM2 to auto-start on server reboot
pm2 startup
```

> ⚠️ **Important:** After running `pm2 startup`, it will print a long command starting with `sudo env PATH=...`  
> **Copy and run that entire command.** This is required for auto-restart to work.

✅ **App is running when you see:**

```
id  name     mode   status   cpu   memory
 0  my-app   fork   online   0%    30mb
```

---

## Step 8 — Connect Your Domain

Your app runs on port **3000** by default. You need a reverse proxy to serve it on port 80/443.

### Option A — Apache (.htaccess)

In cPanel → File Manager → your domain's `public_html` folder, create or edit `.htaccess`:

```apache
RewriteEngine On
RewriteRule ^(.*)$ http://localhost:3000/$1 [P,L]
```

### Option B — Nginx Config

If you have Nginx access, create a config file:

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Then enable and reload:

```bash
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## Step 9 — Enable SSL (HTTPS)

### Via cPanel (Easiest)

1. Login to cPanel
2. Go to **Security → SSL/TLS Status**
3. Click **Run AutoSSL**
4. Wait 1–2 minutes → your site will have a free Let's Encrypt certificate ✅

### Via Certbot (SSH)

```bash
# Install Certbot
sudo dnf install certbot python3-certbot-nginx -y

# Get certificate (replace with your domain)
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Auto-renewal test
sudo certbot renew --dry-run
```

---

## 🔄 Updating Your App

Every time you push new code, SSH in and run:

```bash
# 1. Go to your project folder
cd ~/YOUR_REPO

# 2. Pull latest changes
git pull

# 3. Install any new packages
npm install

# 4. Rebuild
npm run build

# 5. Restart the app
pm2 restart my-app
```

> 💡 **Pro tip:** Save this as a shell script `deploy.sh` so you only need to run one command!

```bash
#!/bin/bash
cd ~/YOUR_REPO
git pull
npm install
npm run build
pm2 restart my-app
echo "✅ Deployment complete!"
```

Make it executable and run:

```bash
chmod +x deploy.sh
./deploy.sh
```

---

## 📟 PM2 Cheatsheet

| Command | Description |
|---|---|
| `pm2 list` | Show all running apps |
| `pm2 logs my-app` | View live logs (Ctrl+C to exit) |
| `pm2 restart my-app` | Restart the app |
| `pm2 stop my-app` | Stop the app |
| `pm2 delete my-app` | Remove from PM2 |
| `pm2 monit` | Open real-time dashboard |
| `pm2 reload my-app` | Zero-downtime restart |
| `pm2 show my-app` | Detailed app info |

---

## 🛠 Troubleshooting

| Problem | Solution |
|---|---|
| `nvm: command not found` | Run `source ~/.bashrc` then try again |
| `npm run build` fails | Check your `.env` variables are all set |
| App not loading in browser | Run `pm2 logs my-app` to see error details |
| Port 3000 already in use | Run `kill $(lsof -t -i:3000)` then restart PM2 |
| 502 Bad Gateway | App crashed — check `pm2 logs` for the error |
| Changes not showing | Make sure you ran `npm run build` and `pm2 restart` |
| Git pull asks for password | Use a Personal Access Token instead of your password |

---

## 🔐 Security Best Practices

- [ ] Change your SSH password after first login
- [ ] Add your SSH public key to `~/.ssh/authorized_keys` for key-based login
- [ ] Disable password-based SSH login after setting up keys
- [ ] Never commit `.env` files to Git
- [ ] Keep Node.js and npm updated regularly
- [ ] Use strong, unique passwords for cPanel and SSH

---

## 📁 Recommended Project Structure

```
your-project/
├── .env.local          ← NOT committed to git
├── .gitignore          ← includes .env*
├── deploy.sh           ← your one-command deploy script
├── package.json
├── next.config.js
├── app/
│   └── ...
└── public/
    └── ...
```

---

## 🆘 Need Help?

- Check logs first: `pm2 logs my-app`
- Check server status: `pm2 list`
- Check port: `lsof -i :3000`
- Check disk space: `df -h`
- Check memory: `free -h`

---

*Made with ❤️ for junior developers. Happy shipping! 🚢*
