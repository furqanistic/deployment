# VPS Deployment Guide

Example values used in this guide:

```text
Server IP: 89.167.30.15
Domain: example.com
API: api.example.com
Backend Port: 8800
Project: /var/www/website
Frontend: /var/www/website/client
Backend: /var/www/website/api
```

Replace them with your own values.

---

# 1. Connect to VPS

```bash
ssh root@89.167.30.15
```

Using an SSH key is recommended.

## Create SSH Key on Mac / Linux / Windows

```bash
ssh-keygen -t ed25519
```

Press `ENTER` for the default location.

### Copy Key on Mac

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

If you already use RSA:

```bash
pbcopy < ~/.ssh/id_rsa.pub
```

Add this public key to your VPS provider, then connect:

```bash
ssh root@89.167.30.15
```

Or specify the key:

```bash
ssh -i ~/.ssh/id_ed25519 root@89.167.30.15
```

---

# 2. Update Server

```bash
apt update && apt dist-upgrade -y
```

```bash
apt autoremove -y
apt clean
```

---

# 3. Remove Apache

Only if Apache is installed:

```bash
systemctl stop apache2
systemctl disable apache2
apt remove apache2 -y
apt autoremove -y
```

---

# 4. Install Nginx

```bash
apt install nginx -y
```

```bash
systemctl enable nginx
systemctl start nginx
```

Check:

```bash
systemctl status nginx
```

---

# 5. Setup Firewall

Install:

```bash
apt install ufw -y
```

**Allow SSH before enabling UFW:**

```bash
ufw allow OpenSSH
```

Allow Nginx:

```bash
ufw allow "Nginx Full"
```

Enable:

```bash
ufw enable
```

Check:

```bash
ufw status
```

---

# 6. Remove Default Nginx Config

```bash
rm -f /etc/nginx/sites-enabled/default
```

```bash
rm -f /etc/nginx/sites-available/default
```

---

# 7. Test Nginx With a Temporary Website

Create folder:

```bash
mkdir -p /var/www/website
```

Create config:

```bash
nano /etc/nginx/sites-available/website
```

Add:

```nginx
server {
    listen 80;
    server_name _;

    location / {
        root /var/www/website;
        index index.html index.htm;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;

        try_files $uri $uri/ /index.html;
    }
}
```

Here:

```nginx
server_name _;
```

means we can access the website using the VPS IP.

Example:

```text
http://89.167.30.15
```

Enable config:

```bash
ln -s /etc/nginx/sites-available/website /etc/nginx/sites-enabled/website
```

Test:

```bash
nginx -t
```

Reload:

```bash
systemctl reload nginx
```

---

# 8. Create Test Page

```bash
nano /var/www/website/index.html
```

Add:

```html
<h1>My VPS is working!</h1>
```

Visit:

```text
http://89.167.30.15
```

If you see the message, Nginx is working.

---

# 9. Delete Temporary Website

**Important:** this folder was only created for testing.

Delete it before cloning your real project:

```bash
rm -rf /var/www/website
```

Check:

```bash
ls /var/www
```

`website` should no longer exist.

---

# 10. Install Git

```bash
apt install git -y
```

Check:

```bash
git --version
```

---

# 11. GitHub SSH Setup for Private Repositories

### Skip this section if your repository is public.

If your GitHub repo is **private**, you need to give the VPS access **before cloning it**.

Generate an SSH key on the VPS:

```bash
ssh-keygen -t ed25519 -C "vps-deployment"
```

Press `ENTER` for the defaults.

Get the public key:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the complete output.

Go to:

```text
GitHub
→ Your Repository
→ Settings
→ Deploy Keys
→ Add Deploy Key
```

Name it:

```text
Production VPS
```

Paste the SSH public key.

If the VPS only needs to pull code, **do not enable write access**.

Test:

```bash
ssh -T git@github.com
```

The first time, enter:

```text
yes
```

Now the VPS can access your private repository.

---

# 12. Clone Your Project

Go to:

```bash
cd /var/www
```

## Public GitHub Repository

You can use HTTPS:

```bash
git clone https://github.com/yourusername/project.git website
```

Example:

```bash
git clone https://github.com/furqanistic/project-manara-AI.git website
```

## Private GitHub Repository

**First complete Step 11 — GitHub SSH Setup.**

Then clone using SSH:

```bash
git clone git@github.com:yourusername/project.git website
```

Example:

```bash
git clone git@github.com:furqanistic/project-manara-AI.git website
```

This creates:

```text
/var/www/website
```

Go inside:

```bash
cd /var/www/website
```

Check:

```bash
ls
```

---

# 13. Install Latest Stable Node.js LTS

Do not use:

```bash
apt install nodejs
```

because Ubuntu may provide an older version.

Install NVM:

```bash
apt install curl -y
```

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
```

Reload:

```bash
source ~/.bashrc
```

Check:

```bash
nvm --version
```

Install the **latest stable Node.js LTS**:

```bash
nvm install --lts
```

Make it default:

```bash
nvm alias default 'lts/*'
```

Use it:

```bash
nvm use --lts
```

Verify:

```bash
node -v
npm -v
```

npm is already included, so you do **not** need:

```bash
apt install npm
```

### Update Node Later

```bash
nvm install --lts
nvm use --lts
nvm alias default 'lts/*'
```

---

# 14. Setup Backend

Example:

```bash
cd /var/www/website/api
```

Install packages:

```bash
npm ci
```

If there is no `package-lock.json`:

```bash
npm install
```

Create `.env`:

```bash
nano .env
```

Example:

```env
PORT=8800
NODE_ENV=production
DATABASE_URL=your_database_url
JWT_SECRET=your_secret
```

Test API:

```bash
node index.js
```

Or if your entry file is different:

```bash
node src/server.js
```

Test:

```bash
curl http://127.0.0.1:8800
```

Stop it:

```text
CTRL + C
```

---

# 15. Install PM2

```bash
npm install -g pm2
```

For:

```text
index.js
```

run:

```bash
pm2 start index.js --name api
```

For:

```text
src/server.js
```

run:

```bash
pm2 start src/server.js --name api
```

Check:

```bash
pm2 status
```

---

# 16. Start PM2 After VPS Reboot

```bash
pm2 startup
```

PM2 will give you a command.

**Copy and run the exact command PM2 gives you.**

Then:

```bash
pm2 save
```

Useful commands:

```bash
pm2 status
pm2 restart api
pm2 logs api
```

---

# 17. Setup Vite Frontend

Go to frontend:

```bash
cd /var/www/website/client
```

Create `.env`:

```bash
nano .env
```

Example:

```env
VITE_API_URL=https://api.example.com
```

Install:

```bash
npm ci
```

If there is no lock file:

```bash
npm install
```

Build:

```bash
npm run build
```

Vite creates:

```text
/var/www/website/client/dist
```

This is the folder Nginx will serve.

---

# 18. Add DNS Records

Example VPS IP:

```text
89.167.30.15
```

Add:

```text
Type: A
Name: @
Value: 89.167.30.15
```

For WWW:

```text
Type: A
Name: www
Value: 89.167.30.15
```

For API:

```text
Type: A
Name: api
Value: 89.167.30.15
```

This gives:

```text
example.com
www.example.com
api.example.com
```

---

# 19. Final Nginx Configuration

Open:

```bash
nano /etc/nginx/sites-available/website
```

Use:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    location / {
        root /var/www/website/client/dist;
        index index.html index.htm;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;

        try_files $uri $uri/ /index.html;
    }
}


server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:8800;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Example:

```text
example.com        → Your Vite frontend
api.example.com    → Your Node.js API
```

And:

```nginx
proxy_pass http://127.0.0.1:8800;
```

means Nginx sends API requests to your Node.js app running on port `8800`.

---

# 20. Optional Upload Limit

If your API accepts large files, add inside the API `server` block:

```nginx
client_max_body_size 1G;
```

Example:

```nginx
server {
    listen 80;
    server_name api.example.com;

    client_max_body_size 1G;

    location / {
        proxy_pass http://127.0.0.1:8800;

        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

# 21. Test Nginx

Always run after changing Nginx:

```bash
nginx -t
```

If successful:

```bash
systemctl reload nginx
```

Or:

```bash
nginx -t && systemctl reload nginx
```

---

# 22. Install SSL

Install Certbot:

```bash
apt install certbot python3-certbot-nginx -y
```

For main website:

```bash
certbot --nginx -d example.com -d www.example.com
```

For API:

```bash
certbot --nginx -d api.example.com
```

Or together:

```bash
certbot --nginx -d example.com -d www.example.com -d api.example.com
```

Only run this after your DNS records point to the VPS.

Test automatic renewal:

```bash
certbot renew --dry-run
```

---

# 23. Deploy Updates Later

## On Your Computer

```bash
git add .
git commit -m "update"
git push origin main
```

## On VPS

```bash
cd /var/www/website
git pull origin main
```

### Backend Changed

```bash
cd api
npm ci
pm2 restart api
```

### Frontend Changed

```bash
cd ../client
npm ci
npm run build
```

That's it.

---

# Useful Commands

Check API:

```bash
pm2 status
```

API logs:

```bash
pm2 logs api
```

Check Nginx:

```bash
nginx -t
```

Nginx errors:

```bash
tail -f /var/log/nginx/error.log
```

Check RAM:

```bash
free -h
```

Check disk:

```bash
df -h
```

Check firewall:

```bash
ufw status
```

Restart VPS:

```bash
reboot
```

---

# Quick Setup Order

```text
SSH into VPS
↓
Update Ubuntu
↓
Install Nginx
↓
Configure Firewall
↓
Create temporary test website
↓
Test VPS IP
↓
DELETE temporary /var/www/website
↓
Install Git
↓
Private repo? → Setup GitHub SSH first
↓
Clone repo into /var/www/website
↓
Install NVM + latest Node LTS
↓
Setup backend
↓
Run backend with PM2
↓
Setup Vite frontend
↓
npm run build
↓
Add DNS
↓
Configure Nginx
↓
Install SSL
↓
Done
```
