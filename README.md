# VPS Setup & Deployment Guide

This guide covers:

* Connecting securely using SSH
* Initial Ubuntu server setup
* Nginx
* Firewall
* Git/GitHub
* Latest stable Node.js LTS
* Node.js API deployment
* PM2
* React/Vite deployment
* Domain configuration
* SSL with Certbot
* Private GitHub repository access

---

# 1. Connecting to the VPS

You can connect to your VPS using a root password, but **SSH keys are more secure and recommended**.

## Create an SSH Key

### macOS / Linux / Windows 10+

Open Terminal or PowerShell:

```bash
ssh-keygen -t ed25519 -C "your-device"
```

Press `ENTER` to save it in the default location.

Usually:

```text
~/.ssh/id_ed25519
```

You can add a passphrase for extra security.

### If you prefer RSA

```bash
ssh-keygen -t rsa -b 4096
```

This creates:

```text
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

---

## Copy Your Public SSH Key

### macOS — ED25519

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

### macOS — RSA

```bash
pbcopy < ~/.ssh/id_rsa.pub
```

### Linux

```bash
cat ~/.ssh/id_ed25519.pub
```

### Windows PowerShell

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```

Copy the output.

---

# 2. Add SSH Key to VPS Provider

Go to your VPS provider dashboard, such as:

* Hetzner
* Linode
* DigitalOcean
* Vultr

Add your **public SSH key** when creating the server.

Then connect:

```bash
ssh root@SERVER_IP
```

Example:

```bash
ssh root@89.167.30.15
```

If you specifically need to choose the key:

```bash
ssh -i ~/.ssh/id_ed25519 root@SERVER_IP
```

For RSA:

```bash
ssh -i ~/.ssh/id_rsa root@SERVER_IP
```

---

# 3. Initial Server Setup

Update the server first:

```bash
apt update && apt full-upgrade -y
```

Clean unused packages:

```bash
apt autoremove -y
apt clean
```

---

# 4. Remove Apache

Only do this if Apache is installed and you want to use Nginx.

```bash
systemctl stop apache2
```

```bash
systemctl disable apache2
```

```bash
apt purge apache2 apache2-utils apache2-bin apache2.2-common -y
```

```bash
apt autoremove -y
```

If Apache is not installed, simply skip this section.

---

# 5. Install Nginx

```bash
apt install nginx -y
```

Enable Nginx at startup:

```bash
systemctl enable nginx
```

Start it:

```bash
systemctl start nginx
```

Check status:

```bash
systemctl status nginx
```

Press `q` to exit the status screen.

---

# 6. Configure Firewall

Install UFW:

```bash
apt install ufw -y
```

## IMPORTANT: Allow SSH before enabling the firewall

```bash
ufw allow OpenSSH
```

Allow Nginx:

```bash
ufw allow "Nginx Full"
```

Now enable the firewall:

```bash
ufw enable
```

Check:

```bash
ufw status
```

You should see something similar to:

```text
OpenSSH                    ALLOW
Nginx Full                 ALLOW
```

---

# 7. Create Website Directory

Remove the default Nginx website if you do not need it:

```bash
rm -f /etc/nginx/sites-enabled/default
```

You can also remove its configuration:

```bash
rm -f /etc/nginx/sites-available/default
```

Create your application directory:

```bash
mkdir -p /var/www/website
```

---

# 8. Create Initial Nginx Configuration

Create:

```bash
nano /etc/nginx/sites-available/website
```

Add:

```nginx
server {
    listen 80;
    server_name _;

    root /var/www/website;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Save:

```text
CTRL + O
ENTER
CTRL + X
```

Enable the configuration:

```bash
ln -sf /etc/nginx/sites-available/website /etc/nginx/sites-enabled/website
```

Always test Nginx:

```bash
nginx -t
```

If successful:

```bash
systemctl reload nginx
```

---

# 9. Create a Test Page

```bash
nano /var/www/website/index.html
```

Add:

```html
<h1>Server is working!</h1>
```

Now visit:

```text
http://SERVER_IP
```

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

# 11. Clone Your Application

Go to `/var/www`:

```bash
cd /var/www
```

Clone:

```bash
git clone YOUR_REPOSITORY_URL website
```

Example:

```bash
git clone https://github.com/username/project.git website
```

Then:

```bash
cd /var/www/website
```

---

# 12. Install Latest Stable Node.js

For production servers, use the **latest Node.js LTS version**.

Using Ubuntu's normal:

```bash
apt install nodejs
```

can sometimes install an older Node.js version.

Instead, install **NVM**, which allows us to install and update to the latest stable LTS version.

## Install NVM

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
```

Reload your shell:

```bash
source ~/.bashrc
```

Check:

```bash
nvm --version
```

---

## Install Latest Stable LTS Node.js

```bash
nvm install --lts
```

Set the latest LTS as the default:

```bash
nvm alias default 'lts/*'
```

Switch to it:

```bash
nvm use --lts
```

Now verify:

```bash
node -v
```

```bash
npm -v
```

### IMPORTANT

After this step, make sure:

```bash
node -v
```

shows the latest LTS version.

You **do not need** to separately run:

```bash
apt install npm
```

because npm comes with Node.js.

---

# 13. Updating Node.js Later

Whenever you want to update the server to the newest LTS:

```bash
nvm install --lts
```

Then:

```bash
nvm use --lts
```

Set it as default:

```bash
nvm alias default 'lts/*'
```

Verify:

```bash
node -v
npm -v
```

---

# 14. Deploy Node.js API

Example structure:

```text
/var/www/website/
├── api/
└── client/
```

Go to API:

```bash
cd /var/www/website/api
```

If you have `package-lock.json`, use:

```bash
npm ci
```

Otherwise:

```bash
npm install
```

Create environment file:

```bash
nano .env
```

Paste your production environment variables.

Then test the application:

```bash
node index.js
```

Or:

```bash
node src/server.js
```

If your API is running on port `8800`, test:

```bash
curl http://127.0.0.1:8800
```

Stop the test server with:

```text
CTRL + C
```

---

# 15. Install PM2

A normal:

```bash
node index.js
```

process stops when you close your SSH session.

Install PM2:

```bash
npm install -g pm2
```

Check:

```bash
pm2 -v
```

---

## Start API with PM2

If entry file is:

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

## Make PM2 Start Automatically After Reboot

Run:

```bash
pm2 startup
```

PM2 will print another command.

**Copy and run the command PM2 gives you.**

Then save your current applications:

```bash
pm2 save
```

Now your API should automatically start after the VPS reboots.

---

# 16. Useful PM2 Commands

Check apps:

```bash
pm2 status
```

Restart API:

```bash
pm2 restart api
```

Stop API:

```bash
pm2 stop api
```

Delete API:

```bash
pm2 delete api
```

View logs:

```bash
pm2 logs api
```

Clear logs:

```bash
pm2 flush
```

---

# 17. Configure Nginx for API

It is better to proxy to the local Node.js process instead of using the VPS public IP.

Use:

```text
127.0.0.1:8800
```

instead of:

```text
SERVER_PUBLIC_IP:8800
```

Example:

```nginx
location /api/ {
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

Test:

```bash
nginx -t
```

Reload:

```bash
systemctl reload nginx
```

A `502 Bad Gateway` usually means Nginx is working but cannot connect to your Node.js application.

Check:

```bash
pm2 status
```

and:

```bash
pm2 logs api
```

---

# 18. React / Vite App Deployment

Go to frontend:

```bash
cd /var/www/website/client
```

Create production `.env`:

```bash
nano .env
```

Install dependencies.

If you have `package-lock.json`:

```bash
npm ci
```

Otherwise:

```bash
npm install
```

Build:

```bash
npm run build
```

---

## Vite

Vite normally creates:

```text
dist/
```

Your Nginx root can point directly to:

```text
/var/www/website/client/dist
```

This is cleaner than manually copying the build every time.

Example:

```nginx
root /var/www/website/client/dist;
```

---

## Create React App

Older Create React App projects normally create:

```text
build/
```

Use:

```nginx
root /var/www/website/client/build;
```

---

# 19. Recommended Nginx Configuration

Assume:

```text
Frontend: example.com
API: api.example.com
API Port: 8800
Frontend build: /var/www/website/client/dist
```

Create/edit:

```bash
nano /etc/nginx/sites-available/website
```

Use:

```nginx
server {
    listen 80;
    server_name example.com www.example.com;

    root /var/www/website/client/dist;
    index index.html;

    # Optional: maximum upload size
    client_max_body_size 1G;

    location / {
        try_files $uri $uri/ /index.html;
    }
}


server {
    listen 80;
    server_name api.example.com;

    client_max_body_size 1G;

    location / {
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

Test:

```bash
nginx -t
```

If successful:

```bash
systemctl reload nginx
```

---

# 20. Add Your Domain

Go to your domain/DNS provider.

Create the following records.

## Main Domain

```text
Type: A
Name: @
Value: SERVER_IP
```

## WWW

```text
Type: A
Name: www
Value: SERVER_IP
```

## API

```text
Type: A
Name: api
Value: SERVER_IP
```

Wait for DNS propagation.

Then test:

```bash
ping example.com
```

Or:

```bash
dig example.com +short
```

---

# 21. SSL Certificate

Install Certbot:

```bash
apt install certbot python3-certbot-nginx -y
```

Check firewall:

```bash
ufw status
```

You should have:

```text
Nginx Full
```

---

## Install SSL

For the main website:

```bash
certbot --nginx -d example.com -d www.example.com
```

For the API:

```bash
certbot --nginx -d api.example.com
```

Or install everything together:

```bash
certbot --nginx -d example.com -d www.example.com -d api.example.com
```

Only include domains/subdomains that already have valid DNS records pointing to your VPS.

---

# 22. Check Automatic SSL Renewal

Let's Encrypt certificates are automatically renewed by Certbot's timer.

Check:

```bash
systemctl status certbot.timer
```

Test renewal:

```bash
certbot renew --dry-run
```

---

# 23. Final Checks

Check Nginx:

```bash
nginx -t
```

```bash
systemctl status nginx
```

Check Node.js:

```bash
node -v
```

Check npm:

```bash
npm -v
```

Check PM2:

```bash
pm2 status
```

Check firewall:

```bash
ufw status
```

Check API:

```bash
curl http://127.0.0.1:8800
```

---

# 24. Updating Your Application

## Development Machine

Make your changes:

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

## VPS

Go to project:

```bash
cd /var/www/website
```

Pull changes:

```bash
git pull origin main
```

---

## If Backend Changed

```bash
cd api
```

Install updated dependencies:

```bash
npm ci
```

Restart:

```bash
pm2 restart api
```

---

## If Frontend Changed

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

Because Nginx points directly to `dist`, no additional copying is required.

---

# 25. Private GitHub Repository Setup

For production VPS servers, a **GitHub Deploy Key** is safer than adding the VPS key to your entire GitHub account.

## Generate SSH Key on VPS

```bash
ssh-keygen -t ed25519 -C "vps-deployment"
```

Press `ENTER` for the default location.

---

## Get Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy it.

---

## Add Deploy Key to GitHub

Open your repository:

```text
Repository
→ Settings
→ Deploy keys
→ Add deploy key
```

Name:

```text
Production VPS
```

Paste the public key.

For a deployment server that only needs to pull code:

**Do NOT enable "Allow write access".**

This gives the VPS read-only access to that repository.

---

## Test GitHub Connection

On VPS:

```bash
ssh -T git@github.com
```

The first time, type:

```text
yes
```

---

## Change Repository Remote to SSH

Go to your project:

```bash
cd /var/www/website
```

Check current remote:

```bash
git remote -v
```

Change it:

```bash
git remote set-url origin git@github.com:YOUR_USERNAME/YOUR_REPOSITORY.git
```

Example:

```bash
git remote set-url origin git@github.com:furqanistic/my-app.git
```

Now:

```bash
git pull origin main
```

should work without entering your GitHub password.

---

# 26. Useful Server Commands

## Disk Usage

```bash
df -h
```

## RAM Usage

```bash
free -h
```

## CPU / Processes

```bash
htop
```

Install if needed:

```bash
apt install htop -y
```

## Nginx Errors

```bash
tail -f /var/log/nginx/error.log
```

## Nginx Access Logs

```bash
tail -f /var/log/nginx/access.log
```

## PM2 Logs

```bash
pm2 logs
```

## Restart Nginx

```bash
systemctl restart nginx
```

## Reload Nginx

Prefer reload after configuration changes:

```bash
nginx -t && systemctl reload nginx
```

## Restart Server

```bash
reboot
```

---

# Recommended Deployment Structure

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
    ├── package.json
    ├── package-lock.json
    ├── .env
    └── dist/
```

Traffic flow:

```text
User
  │
  ├── example.com
  │       │
  │       └── Nginx
  │               │
  │               └── React/Vite dist/
  │
  └── api.example.com
          │
          └── Nginx
                  │
                  └── 127.0.0.1:8800
                           │
                           └── Node.js API
                                  │
                                  └── PM2
```

---

# Quick Deployment Workflow

After the initial setup, most deployments are simply:

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

Check everything:

```bash
pm2 status
nginx -t
```

That's it.
