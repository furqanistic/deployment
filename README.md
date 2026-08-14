# VPS Setup & Deployment Guide

This guide covers:

* SSH connection
* Ubuntu server setup
* Nginx
* Firewall
* Git/GitHub
* Latest stable Node.js LTS
* Node.js API
* PM2
* React/Vite deployment
* Domain setup
* SSL
* Private GitHub repositories

---

# Example Values Used in This Guide

Throughout this guide, I will use these example values:

```text
Server IP: 89.167.30.15

Main Domain:
example.com

WWW:
www.example.com

API Domain:
api.example.com

Backend Port:
8800

Project Directory:
/var/www/website

Frontend:
/var/www/website/client

Backend:
/var/www/website/api
```

Replace these example values with your own.

For example:

```bash
ssh root@89.167.30.15
```

means:

```bash
ssh root@YOUR_SERVER_IP
```

---

# 1. Connect to the VPS

The recommended way to connect to your server is using an SSH key.

## Create an SSH Key

### macOS / Linux / Windows 10+

```bash
ssh-keygen -t ed25519 -C "my-macbook"
```

Press `ENTER` to use the default location.

Example:

```text
/Users/furqan/.ssh/id_ed25519
```

This creates:

```text
Private Key:
/Users/furqan/.ssh/id_ed25519

Public Key:
/Users/furqan/.ssh/id_ed25519.pub
```

---

## Copy SSH Key on macOS

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

If you already use RSA:

```bash
pbcopy < ~/.ssh/id_rsa.pub
```

---

## Linux

```bash
cat ~/.ssh/id_ed25519.pub
```

---

## Windows PowerShell

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```

Copy the output.

---

# 2. Add SSH Key to VPS Provider

Go to your hosting provider:

* Hetzner
* Linode
* DigitalOcean
* Vultr

Add the public SSH key.

Then connect:

```bash
ssh root@YOUR_SERVER_IP
```

Example:

```bash
ssh root@89.167.30.15
```

If you want to explicitly use a specific SSH key:

```bash
ssh -i ~/.ssh/id_ed25519 root@89.167.30.15
```

For RSA:

```bash
ssh -i ~/.ssh/id_rsa root@89.167.30.15
```

---

# 3. Update the Server

```bash
apt update && apt full-upgrade -y
```

Clean unnecessary packages:

```bash
apt autoremove -y
apt clean
```

---

# 4. Remove Apache

Only do this if Apache is installed.

```bash
systemctl stop apache2
```

```bash
systemctl disable apache2
```

```bash
apt purge apache2 apache2-utils apache2-bin -y
```

```bash
apt autoremove -y
```

If Apache is not installed, skip this step.

---

# 5. Install Nginx

```bash
apt install nginx -y
```

Enable Nginx:

```bash
systemctl enable nginx
```

Start Nginx:

```bash
systemctl start nginx
```

Check:

```bash
systemctl status nginx
```

Press:

```text
q
```

to exit.

---

# 6. Install Firewall

Install UFW:

```bash
apt install ufw -y
```

## IMPORTANT

Allow SSH **before** enabling UFW.

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

Example output:

```text
OpenSSH       ALLOW
Nginx Full    ALLOW
```

---

# 7. Remove Default Nginx Website

```bash
rm -f /etc/nginx/sites-enabled/default
```

```bash
rm -f /etc/nginx/sites-available/default
```

---

# 8. Create Website Directory

```bash
mkdir -p /var/www/website
```

Example structure:

```text
/var/www/website/
```

Later it may look like:

```text
/var/www/website/
├── api/
└── client/
```

---

# 9. Create First Nginx Configuration

Open:

```bash
nano /etc/nginx/sites-available/website
```

For now, we are creating a simple configuration that works directly with the server IP.

Example:

```nginx
server {
    listen 80;

    # This means accept requests coming to this server
    # even before we connect a domain.
    server_name _;

    # Example:
    # Our files are stored inside /var/www/website
    root /var/www/website;

    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

### What this means

```nginx
listen 80;
```

means:

```text
Listen for normal HTTP traffic.
```

This:

```nginx
server_name _;
```

means:

```text
Accept requests even if we are accessing the server using its IP.

Example:
http://89.167.30.15
```

This:

```nginx
root /var/www/website;
```

means Nginx will serve files from:

```text
/var/www/website
```

For example:

```text
/var/www/website/index.html
```

---

# 10. Enable Nginx Configuration

```bash
ln -sf /etc/nginx/sites-available/website /etc/nginx/sites-enabled/website
```

Always test configuration:

```bash
nginx -t
```

Expected:

```text
syntax is ok
test is successful
```

Then reload:

```bash
systemctl reload nginx
```

---

# 11. Create Test Website

Create:

```bash
nano /var/www/website/index.html
```

Add:

```html
<h1>My VPS is working!</h1>
```

Now visit your server IP.

Example:

```text
http://89.167.30.15
```

You should see:

```text
My VPS is working!
```

---

# 12. Install Git

```bash
apt install git -y
```

Check:

```bash
git --version
```

---

# 13. Clone Your Project

Go to:

```bash
cd /var/www
```

Clone your repository.

Example:

```bash
git clone https://github.com/yourusername/my-project.git website
```

This means Git will clone the project into:

```text
/var/www/website
```

So your structure could become:

```text
/var/www/website/
├── api/
├── client/
├── README.md
└── package.json
```

Go inside:

```bash
cd /var/www/website
```

---

# 14. Install Latest Stable Node.js LTS

Do **not** simply use:

```bash
apt install nodejs
```

because Ubuntu may provide an older version.

We will use NVM so we can install the **latest stable LTS version of Node.js**.

## Install NVM

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
```

Reload terminal:

```bash
source ~/.bashrc
```

Check:

```bash
nvm --version
```

---

## Install Latest Stable LTS

```bash
nvm install --lts
```

Make it the default version:

```bash
nvm alias default 'lts/*'
```

Use it:

```bash
nvm use --lts
```

Check Node.js:

```bash
node -v
```

Example output:

```text
v24.x.x
```

Check npm:

```bash
npm -v
```

Example:

```text
11.x.x
```

You do **not** need:

```bash
apt install npm
```

because npm is already installed with Node.js.

---

# 15. Update Node.js to Latest LTS Later

In the future, run:

```bash
nvm install --lts
```

Then:

```bash
nvm use --lts
```

Then:

```bash
nvm alias default 'lts/*'
```

Verify:

```bash
node -v
npm -v
```

---

# 16. Backend Setup

Assume your project looks like:

```text
/var/www/website/
├── api/
└── client/
```

Go to backend:

```bash
cd /var/www/website/api
```

Example:

```text
Your backend files could be:

/var/www/website/api/index.js
/var/www/website/api/package.json
/var/www/website/api/.env
```

Install dependencies:

```bash
npm ci
```

If you do not have `package-lock.json`:

```bash
npm install
```

---

# 17. Add Backend Environment Variables

Create:

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

The important example here is:

```env
PORT=8800
```

So our API will be running at:

```text
http://127.0.0.1:8800
```

---

# 18. Test Backend

If your entry file is:

```text
index.js
```

run:

```bash
node index.js
```

If your entry file is:

```text
src/server.js
```

run:

```bash
node src/server.js
```

You may see something like:

```text
Server running on port 8800
```

From another terminal, or after stopping it, you can test:

```bash
curl http://127.0.0.1:8800
```

Example response:

```text
API is running
```

Stop the manually started server:

```text
CTRL + C
```

---

# 19. Install PM2

If you start your API with:

```bash
node index.js
```

the API will stop when you close SSH.

PM2 keeps it running.

Install:

```bash
npm install -g pm2
```

Check:

```bash
pm2 -v
```

---

# 20. Start Backend with PM2

If your backend starts with:

```text
index.js
```

use:

```bash
pm2 start index.js --name api
```

Example:

```text
PM2 App Name:
api

Running File:
index.js
```

If your backend starts from:

```text
src/server.js
```

use:

```bash
pm2 start src/server.js --name api
```

Check:

```bash
pm2 status
```

Example:

```text
api    online
```

---

# 21. Make PM2 Start After Server Reboot

Run:

```bash
pm2 startup
```

PM2 will show you another command.

For example, it may show something similar to:

```bash
sudo env PATH=$PATH:/root/.nvm/versions/node/v24.x.x/bin pm2 startup systemd -u root --hp /root
```

**Do not blindly copy the example above.**

Copy and run the exact command PM2 gives you.

Then:

```bash
pm2 save
```

Now if the VPS restarts, PM2 can automatically restart your API.

---

# 22. Useful PM2 Commands

Check:

```bash
pm2 status
```

Restart API:

```bash
pm2 restart api
```

Logs:

```bash
pm2 logs api
```

Stop:

```bash
pm2 stop api
```

Delete:

```bash
pm2 delete api
```

---

# 23. Nginx Configuration for API

Assume:

```text
Backend:
Node.js

Backend Port:
8800

Local Backend URL:
http://127.0.0.1:8800
```

Instead of using your public VPS IP like:

```nginx
proxy_pass http://89.167.30.15:8800;
```

use:

```nginx
proxy_pass http://127.0.0.1:8800;
```

because Nginx and Node.js are running on the same VPS.

Example configuration:

```nginx
location /api/ {

    # Our Node.js backend is running locally on port 8800.
    proxy_pass http://127.0.0.1:8800/;

    proxy_http_version 1.1;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
}
```

So:

```text
http://89.167.30.15/api/users
```

would be forwarded by Nginx to something like:

```text
http://127.0.0.1:8800/users
```

---

# 24. React / Vite Frontend

Go to frontend:

```bash
cd /var/www/website/client
```

Example:

```text
/var/www/website/client/package.json
/var/www/website/client/src/
/var/www/website/client/.env
```

Create environment file:

```bash
nano .env
```

Example for Vite:

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

---

# 25. Vite Build Folder

Vite normally generates:

```text
dist/
```

Example:

```text
/var/www/website/client/dist
```

Inside it:

```text
/var/www/website/client/dist/index.html
/var/www/website/client/dist/assets/
```

Therefore our Nginx configuration will use:

```nginx
root /var/www/website/client/dist;
```

---

# 26. Create React App Build Folder

Older React apps using Create React App normally generate:

```text
build/
```

Example:

```text
/var/www/website/client/build
```

Then use:

```nginx
root /var/www/website/client/build;
```

### Use only one

For Vite:

```text
dist
```

For Create React App:

```text
build
```

---

# 27. Add Your Domain

Assume:

```text
Server IP:
89.167.30.15

Main Website:
example.com

API:
api.example.com
```

Create DNS records.

## Main Domain

```text
Type: A
Name: @
Value: 89.167.30.15
```

This connects:

```text
example.com
```

to:

```text
89.167.30.15
```

---

## WWW

```text
Type: A
Name: www
Value: 89.167.30.15
```

This connects:

```text
www.example.com
```

to your VPS.

---

## API

```text
Type: A
Name: api
Value: 89.167.30.15
```

This connects:

```text
api.example.com
```

to your VPS.

---

# 28. Final Nginx Configuration

Assume we have:

```text
Server IP:
89.167.30.15

Frontend:
example.com

Frontend Folder:
/var/www/website/client/dist

API:
api.example.com

Node API:
127.0.0.1:8800
```

Open:

```bash
nano /etc/nginx/sites-available/website
```

Use:

```nginx
# ==============================
# FRONTEND
# ==============================

server {
    listen 80;

    # Example:
    # Main website: https://example.com
    # WWW: https://www.example.com
    server_name example.com www.example.com;

    # Example Vite production build:
    # /var/www/website/client/dist/index.html
    root /var/www/website/client/dist;

    index index.html;

    # Example:
    # Allows uploads up to 1 GB.
    client_max_body_size 1G;

    location / {

        # Example:
        # /dashboard
        # /profile
        # /settings
        #
        # If Nginx cannot find an actual file,
        # it sends the request back to index.html
        # so React Router can handle it.
        try_files $uri $uri/ /index.html;
    }
}


# ==============================
# BACKEND API
# ==============================

server {
    listen 80;

    # Example:
    # https://api.example.com
    server_name api.example.com;

    # Allows API uploads up to 1 GB.
    client_max_body_size 1G;

    location / {

        # Example:
        #
        # Request:
        # https://api.example.com/api/users
        #
        # Nginx forwards it to:
        # http://127.0.0.1:8800/api/users
        #
        # Node.js is running locally on port 8800.
        proxy_pass http://127.0.0.1:8800;

        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### Replace these:

```text
example.com
www.example.com
api.example.com
```

with your domains.

For example, if your domain is:

```text
myapp.com
```

then it becomes:

```nginx
server_name myapp.com www.myapp.com;
```

and:

```nginx
server_name api.myapp.com;
```

---

# 29. Test Nginx

Every time you edit Nginx configuration, run:

```bash
nginx -t
```

If you see:

```text
syntax is ok
test is successful
```

reload Nginx:

```bash
systemctl reload nginx
```

A useful one-line command is:

```bash
nginx -t && systemctl reload nginx
```

This only reloads Nginx if the configuration is valid.

---

# 30. Understanding a 502 Error

If you visit:

```text
https://api.example.com
```

and receive:

```text
502 Bad Gateway
```

it normally means:

```text
Nginx is working.

BUT

Nginx cannot connect to your Node.js application.
```

Check:

```bash
pm2 status
```

Then:

```bash
pm2 logs api
```

Also test:

```bash
curl http://127.0.0.1:8800
```

If this fails, the problem is most likely your Node.js application rather than Nginx.

---

# 31. Install SSL

Install Certbot:

```bash
apt install certbot python3-certbot-nginx -y
```

Check firewall:

```bash
ufw status
```

You should see:

```text
Nginx Full    ALLOW
```

---

# 32. SSL for Main Website

Example domain:

```text
example.com
www.example.com
```

Run:

```bash
certbot --nginx -d example.com -d www.example.com
```

---

# 33. SSL for API

Example:

```text
api.example.com
```

Run:

```bash
certbot --nginx -d api.example.com
```

Or do everything together:

```bash
certbot --nginx \
-d example.com \
-d www.example.com \
-d api.example.com
```

Only run Certbot after the DNS records are pointing to your VPS.

---

# 34. Test SSL Renewal

Check:

```bash
systemctl status certbot.timer
```

Test:

```bash
certbot renew --dry-run
```

---

# 35. Private GitHub Repository

If the repository is private, your VPS needs permission to pull it.

A good approach is using a **GitHub Deploy Key**.

---

# 36. Generate GitHub SSH Key on VPS

Run on the VPS:

```bash
ssh-keygen -t ed25519 -C "production-vps"
```

Press `ENTER` for defaults.

Example:

```text
Private:
/root/.ssh/id_ed25519

Public:
/root/.ssh/id_ed25519.pub
```

---

# 37. Copy VPS Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

Example output:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA.... production-vps
```

Copy the whole line.

---

# 38. Add Deploy Key to GitHub

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

If the VPS only needs to:

```text
git pull
```

leave:

```text
Allow write access
```

disabled.

---

# 39. Change Git Remote

Go to project:

```bash
cd /var/www/website
```

Check:

```bash
git remote -v
```

You may currently have:

```text
https://github.com/yourusername/my-project.git
```

Change it:

```bash
git remote set-url origin git@github.com:yourusername/my-project.git
```

Example:

```bash
git remote set-url origin git@github.com:furqanistic/xocial.git
```

Check again:

```bash
git remote -v
```

---

# 40. Test GitHub SSH

```bash
ssh -T git@github.com
```

The first time you may see:

```text
Are you sure you want to continue connecting?
```

Type:

```text
yes
```

Then pull:

```bash
git pull origin main
```

---

# 41. Normal Deployment Workflow

After everything has been configured once, deployments become very simple.

## On Your Mac

```bash
git add .
```

```bash
git commit -m "update"
```

```bash
git push origin main
```

---

## On VPS

```bash
cd /var/www/website
```

```bash
git pull origin main
```

---

# 42. Backend Update

```bash
cd /var/www/website/api
```

```bash
npm ci
```

```bash
pm2 restart api
```

Example:

```text
Code pulled
↓
Dependencies installed
↓
API restarted
```

---

# 43. Frontend Update

```bash
cd /var/www/website/client
```

Install dependencies:

```bash
npm ci
```

Build:

```bash
npm run build
```

Example Vite output:

```text
/var/www/website/client/dist/
```

Because Nginx already points to:

```text
/var/www/website/client/dist
```

you don't need to copy the build anywhere else.

---

# 44. Useful Server Commands

## Check Disk

```bash
df -h
```

Example:

```text
Filesystem      Size  Used  Avail
/dev/sda1       150G   25G   125G
```

---

## Check RAM

```bash
free -h
```

---

## Check CPU

Install:

```bash
apt install htop -y
```

Run:

```bash
htop
```

---

## Check API

```bash
pm2 status
```

---

## API Logs

```bash
pm2 logs api
```

---

## Nginx Error Logs

```bash
tail -f /var/log/nginx/error.log
```

---

## Nginx Access Logs

```bash
tail -f /var/log/nginx/access.log
```

---

## Check Nginx

```bash
nginx -t
```

---

## Reload Nginx

```bash
nginx -t && systemctl reload nginx
```

---

## Check Firewall

```bash
ufw status
```

---

## Check Node

```bash
node -v
```

---

## Check npm

```bash
npm -v
```

---

## Restart VPS

```bash
reboot
```

---

# Final Server Structure

A normal MERN/Vite project could look like:

```text
/var/www/website/
│
├── api/
│   ├── index.js
│   ├── package.json
│   ├── package-lock.json
│   └── .env
│
└── client/
    ├── src/
    ├── public/
    ├── package.json
    ├── package-lock.json
    ├── .env
    └── dist/
```

---

# How Everything Connects

Example:

```text
User visits:

https://example.com
        │
        ▼
      Nginx
        │
        ▼
/var/www/website/client/dist
        │
        ▼
    React / Vite
```

API:

```text
Frontend sends request:

https://api.example.com/api/users
              │
              ▼
            Nginx
              │
              ▼
    http://127.0.0.1:8800
              │
              ▼
         Node.js API
              │
              ▼
             PM2
```

---

# Quick Deployment Cheat Sheet

After initial setup:

```bash
cd /var/www/website
git pull origin main
```

Backend:

```bash
cd api
npm ci
pm2 restart api
```

Frontend:

```bash
cd ../client
npm ci
npm run build
```

Check:

```bash
pm2 status
```

```bash
nginx -t
```

Done.
