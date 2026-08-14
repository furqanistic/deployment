# VPS Deployment Guide

Example values used in this guide:

```text
Server IP: 89.167.30.15
Domain: example.com
API: api.example.com
Backend Port: 8800

Project:
/var/www/website

Frontend:
/var/www/website/client

Backend:
/var/www/website/api
```

Replace these with your own values.

---

# 1. Connect to VPS

```bash
ssh root@89.167.30.15
```

Using an SSH key is recommended.

## Create SSH Key

Mac / Linux / Windows:

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

Add this public key to your VPS provider.

Then connect:

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

Press `q` to exit.

---

# 5. Setup Firewall

Install UFW:

```bash
apt install ufw -y
```

## Important — Allow SSH First

Do this before enabling UFW:

```bash
ufw allow OpenSSH
```

Allow Nginx:

```bash
ufw allow "Nginx Full"
```

Enable firewall:

```bash
ufw enable
```

Check:

```bash
ufw status
```

Example:

```text
OpenSSH       ALLOW
Nginx Full    ALLOW
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

# 7. Test Nginx With Temporary Website

Create the temporary website folder:

```bash
mkdir -p /var/www/website
```

Create Nginx config:

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

This:

```nginx
server_name _;
```

allows you to test using your VPS IP.

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

# 8. Create Test HTML Page

Create:

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

The HTML page and `/var/www/website` folder were only created for testing.

First delete the temporary HTML:

```bash
rm -f /var/www/website/index.html
```

Then delete the temporary website folder:

```bash
rm -rf /var/www/website
```

Check:

```bash
ls /var/www
```

You should no longer see:

```text
website
```

Now `/var/www/website` is free for your real project.

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

For a private repository, setup GitHub SSH access **before cloning**.

## Generate SSH Key on VPS

```bash
ssh-keygen -t ed25519 -C "vps-deployment"
```

Press `ENTER` for the defaults.

This creates:

```text
/root/.ssh/id_ed25519
/root/.ssh/id_ed25519.pub
```

---

## Copy Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the complete output.

Example:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... vps-deployment
```

---

## Add Key to GitHub

Go to:

```text
GitHub
→ Your Repository
→ Settings
→ Deploy Keys
→ Add Deploy Key
```

Example title:

```text
Production VPS
```

Paste the public key.

If the VPS only needs to pull code, leave:

```text
Allow write access
```

disabled.

---

## Test GitHub SSH

```bash
ssh -T git@github.com
```

The first time, enter:

```text
yes
```

---

## Get SSH Repository URL

Go to:

```text
GitHub Repository
→ Code
→ SSH
```

It looks like:

```text
git@github.com:USERNAME/REPOSITORY.git
```

Example:

```text
git@github.com:johndoe/my-app.git
```

---

## If Repository Was Already Cloned Using HTTPS

Go to the project:

```bash
cd /var/www/website
```

Check remote:

```bash
git remote -v
```

Example HTTPS remote:

```text
origin  https://github.com/johndoe/my-app.git
```

Change it to SSH:

```bash
git remote set-url origin git@github.com:johndoe/my-app.git
```

Check again:

```bash
git remote -v
```

You should now see:

```text
origin  git@github.com:johndoe/my-app.git
```

If you clone using SSH from the beginning, you do **not** need to change the remote manually.

---

# 12. Clone Your Project

Go to:

```bash
cd /var/www
```

## Public Repository

```bash
git clone https://github.com/USERNAME/REPOSITORY.git website
```

Example:

```bash
git clone https://github.com/johndoe/my-app.git website
```

## Private Repository

**Complete Step 11 first.**

Then:

```bash
git clone git@github.com:USERNAME/REPOSITORY.git website
```

Example:

```bash
git clone git@github.com:johndoe/my-app.git website
```

This creates:

```text
/var/www/website
```

Go inside:

```bash
cd /var/www/website
```

Check files:

```bash
ls
```

Check remote:

```bash
git remote -v
```

For a private repository, it should look similar to:

```text
origin  git@github.com:johndoe/my-app.git
```

---

# 13. Install Latest Stable Node.js LTS

Do not simply use:

```bash
apt install nodejs
```

because Ubuntu may install an older version.

Install curl:

```bash
apt install curl -y
```

Install NVM:

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

Install latest stable LTS:

```bash
nvm install --lts
```

Set it as default:

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

npm comes with Node.js, so you do not need:

```bash
apt install npm
```

## Update Node Later

```bash
nvm install --lts
nvm use --lts
nvm alias default 'lts/*'
```

Check:

```bash
node -v
npm -v
```

---

# 14. Setup Backend

Example backend folder:

```text
/var/www/website/api
```

Go there:

```bash
cd /var/www/website/api
```

If you have `package-lock.json`:

```bash
npm ci
```

Otherwise:

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

If your entry file is:

```text
src/server.js
```

use:

```bash
node src/server.js
```

Test:

```bash
curl http://127.0.0.1:8800
```

Stop the manual process:

```text
CTRL + C
```

---

# 15. Install PM2

```bash
npm install -g pm2
```

If your entry file is:

```text
index.js
```

run:

```bash
pm2 start index.js --name api
```

If your entry file is:

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

Logs:

```bash
pm2 logs api
```

---

# 16. Start PM2 After VPS Reboot

Run:

```bash
pm2 startup
```

PM2 will give you another command.

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

Go to:

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

Install dependencies:

```bash
npm ci
```

If there is no `package-lock.json`:

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

Example server IP:

```text
89.167.30.15
```

Main domain:

```text
Type: A
Name: @
Value: 89.167.30.15
```

WWW:

```text
Type: A
Name: www
Value: 89.167.30.15
```

API:

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
example.com      → Vite frontend
api.example.com  → Node.js API
```

This:

```nginx
root /var/www/website/client/dist;
```

serves your Vite build.

This:

```nginx
proxy_pass http://127.0.0.1:8800;
```

sends API requests to Node.js running on port `8800`.

---

# 20. Optional Upload Limit

If your API accepts large files, add:

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

Always run:

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

Main website:

```bash
certbot --nginx -d example.com -d www.example.com
```

API:

```bash
certbot --nginx -d api.example.com
```

Or together:

```bash
certbot --nginx -d example.com -d www.example.com -d api.example.com
```

Only run Certbot after your DNS records point to the VPS.

Test renewal:

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

## Backend Changed

```bash
cd api
npm ci
pm2 restart api
```

## Frontend Changed

```bash
cd ../client
npm ci
npm run build
```

Done.

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

RAM:

```bash
free -h
```

Disk:

```bash
df -h
```

Firewall:

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
Create temporary /var/www/website
↓
Create temporary index.html
↓
Test VPS IP
↓
DELETE index.html
↓
DELETE /var/www/website
↓
Install Git
↓
Private repo?
→ Generate SSH key on VPS
→ Add Deploy Key to GitHub
→ Test GitHub SSH
→ Get SSH repository URL
↓
Clone repo into /var/www/website
↓
Verify git remote -v
↓
Install NVM
↓
Install latest Node.js LTS
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
