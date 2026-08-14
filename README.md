# Complete VPS Setup & Deployment Guide

This guide covers:

* Connecting to a VPS with SSH
* Creating SSH keys
* Initial Ubuntu configuration
* Nginx
* UFW Firewall
* Testing the server
* Git & GitHub
* Installing the latest stable Node.js LTS
* Node.js API deployment
* PM2
* React + Vite deployment
* Domain configuration
* SSL with Certbot
* Private GitHub repository setup
* Updating your application later

---

# Example Values Used in This Guide

To make everything easier to understand, these example values will be used:

```text
Server IP:
89.167.30.15

Main Domain:
example.com

WWW Domain:
www.example.com

API Domain:
api.example.com

Admin Domain (optional):
admin.example.com

Backend Port:
8800

Project Folder:
/var/www/website

Frontend Folder:
/var/www/website/client

Backend Folder:
/var/www/website/api
```

Replace these examples with your actual values.

For example:

```bash
ssh root@89.167.30.15
```

means:

```bash
ssh root@YOUR_SERVER_IP
```

---

# 1. Connecting to the VPS

You can connect to your VPS using the root password.

Example:

```bash
ssh root@89.167.30.15
```

However, using an **SSH key** is more secure and recommended.

---

# 2. Creating an SSH Key

## macOS / Linux / Windows 10+

Open Terminal or PowerShell.

Run:

```bash
ssh-keygen -t ed25519
```

Press `ENTER` to save the key in the default location.

Example on macOS:

```text
/Users/furqan/.ssh/id_ed25519
```

You can enter a passphrase or press `ENTER` to leave it empty.

This creates:

```text
Private Key:
~/.ssh/id_ed25519

Public Key:
~/.ssh/id_ed25519.pub
```

---

## If You Already Use RSA

You can also use:

```bash
ssh-keygen -t rsa -b 4096
```

This creates:

```text
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

---

# 3. Copy Your SSH Public Key

## macOS — ED25519

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

## macOS — RSA

If you already use RSA:

```bash
pbcopy < ~/.ssh/id_rsa.pub
```

## Linux

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the output.

## Windows PowerShell

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```

Copy the output.

---

# 4. Add SSH Key to Your VPS

Go to your hosting provider dashboard.

For example:

```text
Hetzner
Linode
DigitalOcean
Vultr
```

Add your **public SSH key**.

Then connect:

```bash
ssh root@YOUR_SERVER_IP
```

Example:

```bash
ssh root@89.167.30.15
```

If you specifically want to use your ED25519 key:

```bash
ssh -i ~/.ssh/id_ed25519 root@89.167.30.15
```

If using RSA:

```bash
ssh -i ~/.ssh/id_rsa root@89.167.30.15
```

---

# 5. First Server Configuration

Once connected to your VPS, update the server.

```bash
apt update && apt dist-upgrade -y
```

Clean unused packages:

```bash
apt autoremove -y
```

```bash
apt clean
```

---

# 6. Remove Apache

If Apache is installed and you want to use Nginx, remove it.

Stop Apache:

```bash
systemctl stop apache2
```

Disable Apache:

```bash
systemctl disable apache2
```

Remove Apache:

```bash
apt remove apache2 -y
```

Delete unnecessary dependencies:

```bash
apt autoremove -y
```

If Apache is not installed, skip this section.

---

# 7. Install Nginx

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

Check Nginx:

```bash
systemctl status nginx
```

Press:

```text
q
```

to exit.

---

# 8. Install and Configure Firewall

Install UFW:

```bash
apt install ufw -y
```

## IMPORTANT — Allow SSH First

Before enabling UFW, allow SSH:

```bash
ufw allow OpenSSH
```

Otherwise, you could lock yourself out of your VPS.

Allow Nginx:

```bash
ufw allow "Nginx Full"
```

Now enable UFW:

```bash
ufw enable
```

Check:

```bash
ufw status
```

Example:

```text
OpenSSH                    ALLOW
Nginx Full                 ALLOW
```

---

# 9. Delete Default Nginx Configuration

Delete the default enabled configuration:

```bash
rm -f /etc/nginx/sites-enabled/default
```

Delete the default available configuration:

```bash
rm -f /etc/nginx/sites-available/default
```

---

# 10. Create a Temporary Test Website

Before deploying your real application, we will test whether Nginx is working.

Create the website directory:

```bash
mkdir -p /var/www/website
```

Create the Nginx configuration:

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

### What does this mean?

This:

```nginx
server_name _;
```

allows us to access the website using the server IP before connecting a domain.

Example:

```text
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

# 11. Enable the Nginx Configuration

Run:

```bash
ln -s /etc/nginx/sites-available/website /etc/nginx/sites-enabled/website
```

Always test your Nginx configuration:

```bash
nginx -t
```

You should see something similar to:

```text
syntax is ok
test is successful
```

Reload Nginx:

```bash
systemctl reload nginx
```

---

# 12. Create the Test Page

Create:

```bash
nano /var/www/website/index.html
```

Add:

```html
<h1>My VPS is working!</h1>
```

Save the file.

Now visit your server IP.

Example:

```text
http://89.167.30.15
```

You should see:

```text
My VPS is working!
```

This confirms:

```text
VPS ✅
Nginx ✅
Firewall ✅
Server IP ✅
```

---

# 13. IMPORTANT — Delete the Temporary Test Website

The `/var/www/website` directory was only created to test Nginx.

We now need to delete it before cloning our real project.

Run:

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

This step is important.

Otherwise, this command:

```bash
git clone https://github.com/yourusername/project.git website
```

would fail because:

```text
/var/www/website
```

already exists.

---

# 14. Install Git

```bash
apt install git -y
```

Check:

```bash
git --version
```

---

# 15. Clone Your Application

Go to:

```bash
cd /var/www
```

Now clone your repository.

Example:

```bash
git clone https://github.com/yourusername/project.git website
```

Real-looking example:

```bash
git clone https://github.com/furqanistic/project-manara-AI.git website
```

This:

```text
website
```

at the end means Git will create:

```text
/var/www/website
```

So after cloning, your structure might look like:

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

Check:

```bash
ls
```

---

# If Your Repository Is Private

If the GitHub repository is private, HTTPS cloning may require authentication.

In that case, follow the **Private GitHub Repository Setup** section later in this guide and clone using SSH.

Example:

```bash
git clone git@github.com:yourusername/project.git website
```

Example:

```bash
git clone git@github.com:furqanistic/project-manara-AI.git website
```

---

# 16. Install Latest Stable Node.js LTS

For a production VPS, we want the latest stable **LTS** version of Node.js.

Instead of:

```bash
apt install nodejs
```

we will use NVM.

This makes it easier to install and update Node.js.

Node recommends production applications use an LTS release, and NVM supports installing the latest LTS automatically.

---

## Install curl

```bash
apt install curl -y
```

---

## Install NVM

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.6/install.sh | bash
```

The official NVM documentation currently provides this installer version.

Reload your shell:

```bash
source ~/.bashrc
```

Check NVM:

```bash
command -v nvm
```

You should see:

```text
nvm
```

---

# 17. Install Latest Node.js LTS

Run:

```bash
nvm install --lts
```

This automatically installs the newest available LTS version rather than hard-coding an old Node version.

Set LTS as the default:

```bash
nvm alias default 'lts/*'
```

Use it:

```bash
nvm use --lts
```

Now check Node:

```bash
node -v
```

Example:

```text
v24.x.x
```

Check npm:

```bash
npm -v
```

You do **not** need:

```bash
apt install npm
```

because npm comes with Node.js installed through NVM.

---

# 18. Update Node.js Later

Whenever you want to move to the newest LTS:

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

Check:

```bash
node -v
```

```bash
npm -v
```

---

# 19. Backend / API Setup

Assume your project looks like this:

```text
/var/www/website/
├── api/
└── client/
```

Go to the API:

```bash
cd /var/www/website/api
```

Example:

```text
/var/www/website/api/
├── index.js
├── package.json
├── package-lock.json
└── .env
```

---

# 20. Install Backend Dependencies

If you have:

```text
package-lock.json
```

use:

```bash
npm ci
```

Otherwise:

```bash
npm install
```

---

# 21. Create Backend Environment File

Create:

```bash
nano .env
```

Paste your production environment variables.

Example:

```env
PORT=8800
NODE_ENV=production
DATABASE_URL=your_database_url
JWT_SECRET=your_secret
```

Here:

```env
PORT=8800
```

means your Node.js API runs on:

```text
http://127.0.0.1:8800
```

---

# 22. Test Your API

If your entry file is:

```text
index.js
```

run:

```bash
node index.js
```

Example output:

```text
Server running on port 8800
```

If your project uses:

```text
src/server.js
```

run:

```bash
node src/server.js
```

Test the API:

```bash
curl http://127.0.0.1:8800
```

Once you know it works, stop the manually running Node process:

```text
CTRL + C
```

---

# 23. Install PM2

If you run:

```bash
node index.js
```

and then close your SSH connection, the Node.js application will stop.

PM2 keeps the application running.

Install PM2:

```bash
npm install -g pm2
```

Check:

```bash
pm2 -v
```

---

# 24. Start API Using PM2

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

Example:

```text
api       online
```

Check logs:

```bash
pm2 logs api
```

---

# 25. Make PM2 Start After VPS Reboot

Run:

```bash
pm2 startup
```

PM2 will give you another command.

It may look similar to:

```bash
sudo env PATH=$PATH:/root/.nvm/versions/node/... pm2 startup systemd -u root --hp /root
```

Run the **exact command PM2 gives you**.

Then save your running applications:

```bash
pm2 save
```

Now PM2 can restore your API after the server reboots.

---

# 26. Useful PM2 Commands

Check running applications:

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

Start API:

```bash
pm2 start api
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

# 27. React + Vite Deployment

This guide uses **Vite**.

Assume your frontend is:

```text
/var/www/website/client
```

Go there:

```bash
cd /var/www/website/client
```

---

# 28. Create Frontend Environment File

```bash
nano .env
```

Example:

```env
VITE_API_URL=https://api.example.com
```

Replace:

```text
api.example.com
```

with your real API domain.

---

# 29. Install Frontend Dependencies

If you have `package-lock.json`:

```bash
npm ci
```

Otherwise:

```bash
npm install
```

---

# 30. Build the Vite Application

Run:

```bash
npm run build
```

Vite will normally create:

```text
dist
```

Example:

```text
/var/www/website/client/dist
```

Inside it:

```text
/var/www/website/client/dist/
├── index.html
└── assets/
```

This is the folder Nginx will serve.

You do **not** need to copy the files somewhere else.

---

# 31. Configure Nginx for Your Domain

Now we will replace the temporary IP-based Nginx configuration with the real domain configuration.

Open:

```bash
nano /etc/nginx/sites-available/website
```

Assume:

```text
Main Domain:
example.com

API:
api.example.com

Frontend:
/var/www/website/client/dist

API Port:
8800
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

This is intentionally kept very close to the original configuration.

---

# 32. Understanding the Frontend Configuration

This:

```nginx
server_name example.com www.example.com;
```

means this server block handles:

```text
http://example.com
http://www.example.com
```

This:

```nginx
root /var/www/website/client/dist;
```

means the frontend is served from:

```text
/var/www/website/client/dist
```

which was created by:

```bash
npm run build
```

---

# 33. Understanding the API Configuration

This:

```nginx
server_name api.example.com;
```

means this block handles:

```text
http://api.example.com
```

This:

```nginx
proxy_pass http://127.0.0.1:8800;
```

means requests are forwarded to your Node.js API running on port:

```text
8800
```

For example:

```text
https://api.example.com/api/users
```

is sent to your backend running locally on the VPS.

---

# 34. Why We Use 127.0.0.1

Instead of:

```nginx
proxy_pass http://89.167.30.15:8800;
```

use:

```nginx
proxy_pass http://127.0.0.1:8800;
```

because Nginx and Node.js are running on the same VPS.

There is no need for Nginx to connect back through the server's public IP.

---

# 35. Optional Admin Vite App

If you also have a separate Vite admin application:

```text
/var/www/website/admin
```

and after building:

```text
/var/www/website/admin/dist
```

you can add:

```nginx
server {
    listen 80;
    server_name admin.example.com;

    location / {
        root /var/www/website/admin/dist;
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

If you do not have an admin application, do not add this block.

---

# 36. Upload Size Limit

If your application accepts large uploads, add:

```nginx
client_max_body_size 1G;
```

Example API configuration:

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

Change:

```text
1G
```

according to your application's requirements.

---

# 37. Test Nginx Configuration

Every time you change Nginx:

```bash
nginx -t
```

If you see:

```text
syntax is ok
test is successful
```

reload:

```bash
systemctl reload nginx
```

You can also use:

```bash
nginx -t && systemctl reload nginx
```

This checks the configuration first and only reloads if the check succeeds.

---

# 38. 502 Bad Gateway

If:

```text
api.example.com
```

returns:

```text
502 Bad Gateway
```

Nginx is normally unable to reach your Node.js API.

Check PM2:

```bash
pm2 status
```

Check API logs:

```bash
pm2 logs api
```

Check the API directly:

```bash
curl http://127.0.0.1:8800
```

If this does not work, check your Node.js application.

---

# 39. Add Your Domain DNS

Go to your domain provider.

Assume your VPS IP is:

```text
89.167.30.15
```

## Main Domain

Create:

```text
Type: A
Name: @
Value: 89.167.30.15
```

This connects:

```text
example.com
```

to your VPS.

---

## WWW

Create:

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

Create:

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

## Optional Admin

If you have:

```text
admin.example.com
```

create:

```text
Type: A
Name: admin
Value: 89.167.30.15
```

---

# 40. Check DNS

You can check:

```bash
ping example.com
```

Or:

```bash
dig example.com +short
```

Example result:

```text
89.167.30.15
```

---

# 41. SSL Certification

Install Certbot:

```bash
apt install certbot python3-certbot-nginx -y
```

Check firewall:

```bash
ufw status
```

Make sure:

```text
Nginx Full
```

is allowed.

---

# 42. Install SSL for Website

For:

```text
example.com
www.example.com
```

run:

```bash
certbot --nginx -d example.com -d www.example.com
```

---

# 43. Install SSL for API

For:

```text
api.example.com
```

run:

```bash
certbot --nginx -d api.example.com
```

---

# 44. Install SSL Together

You can also run:

```bash
certbot --nginx -d example.com -d www.example.com -d api.example.com
```

If you also have admin:

```bash
certbot --nginx -d example.com -d www.example.com -d api.example.com -d admin.example.com
```

Only include domains that already point to the VPS.

---

# 45. Check Automatic SSL Renewal

Check:

```bash
systemctl status certbot.timer
```

Test renewal:

```bash
certbot renew --dry-run
```

---

# 46. Private GitHub Repository Setup

If your repository is private, your VPS needs permission to pull it.

The cleaner approach is to use a GitHub **Deploy Key**.

---

# 47. Generate GitHub SSH Key on VPS

On your VPS:

```bash
ssh-keygen -t ed25519 -C "vps-deployment"
```

Press `ENTER` for the default location.

It will create:

```text
/root/.ssh/id_ed25519
/root/.ssh/id_ed25519.pub
```

---

# 48. Get the VPS Public Key

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire output.

Example:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... vps-deployment
```

---

# 49. Add Deploy Key to GitHub

Go to your repository:

```text
GitHub
→ Repository
→ Settings
→ Deploy keys
→ Add deploy key
```

Example title:

```text
Production VPS
```

Paste the public key.

If the VPS only needs to pull updates, leave:

```text
Allow write access
```

disabled.

---

# 50. Test GitHub Connection

Run:

```bash
ssh -T git@github.com
```

The first time, you may be asked:

```text
Are you sure you want to continue connecting?
```

Enter:

```text
yes
```

---

# 51. Change Existing Repository to SSH

If you originally cloned using HTTPS:

```bash
cd /var/www/website
```

Check:

```bash
git remote -v
```

You may see:

```text
https://github.com/yourusername/project.git
```

Change it:

```bash
git remote set-url origin git@github.com:yourusername/project.git
```

Example:

```bash
git remote set-url origin git@github.com:furqanistic/project-manara-AI.git
```

Check again:

```bash
git remote -v
```

---

# 52. Normal Development Workflow

## On Your MacBook

Make your changes.

Then:

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

# 53. Pull Updates on VPS

Connect to VPS:

```bash
ssh root@89.167.30.15
```

Go to project:

```bash
cd /var/www/website
```

Pull:

```bash
git pull origin main
```

---

# 54. Backend Update

If backend code changed:

```bash
cd /var/www/website/api
```

Install dependencies:

```bash
npm ci
```

Restart:

```bash
pm2 restart api
```

Check:

```bash
pm2 status
```

---

# 55. Frontend Update

If frontend code changed:

```bash
cd /var/www/website/client
```

Install dependencies:

```bash
npm ci
```

Build again:

```bash
npm run build
```

Vite will update:

```text
/var/www/website/client/dist
```

Nginx is already serving that folder, so you do not need to copy anything.

---

# 56. Full Normal Deployment

After pushing your changes from your development machine:

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

---

# 57. Useful Server Commands

## Check Disk Space

```bash
df -h
```

---

## Check RAM

```bash
free -h
```

---

## Check CPU and Processes

Install:

```bash
apt install htop -y
```

Run:

```bash
htop
```

Press:

```text
q
```

to exit.

---

## Check Node.js

```bash
node -v
```

---

## Check npm

```bash
npm -v
```

---

## Check NVM

```bash
nvm --version
```

---

## Check PM2

```bash
pm2 status
```

---

## Check API Logs

```bash
pm2 logs api
```

---

## Check Nginx

```bash
systemctl status nginx
```

---

## Check Nginx Configuration

```bash
nginx -t
```

---

## Reload Nginx

```bash
systemctl reload nginx
```

---

## Safely Test + Reload Nginx

```bash
nginx -t && systemctl reload nginx
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

## Firewall Status

```bash
ufw status
```

---

## See Failed System Services

```bash
systemctl --failed
```

If you already fixed the failed service and only need to clear its failed state:

```bash
systemctl reset-failed
```

---

## Restart VPS

```bash
reboot
```

---

# 58. Recommended Project Structure

A normal Vite + Node.js application could look like:

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

If you also have an admin app:

```text
/var/www/website/
│
├── api/
│
├── client/
│   └── dist/
│
└── admin/
    └── dist/
```

---

# 59. How Everything Works

Frontend:

```text
User
 ↓
https://example.com
 ↓
Nginx
 ↓
/var/www/website/client/dist
 ↓
Vite / React App
```

API:

```text
Frontend
 ↓
https://api.example.com
 ↓
Nginx
 ↓
http://127.0.0.1:8800
 ↓
Node.js API
 ↓
PM2
```

---

# 60. Complete Setup Order

Follow this order on a new VPS:

```text
1. Connect using SSH
        ↓
2. Update Ubuntu
        ↓
3. Remove Apache if installed
        ↓
4. Install Nginx
        ↓
5. Configure UFW
        ↓
6. Create /var/www/website temporarily
        ↓
7. Create test index.html
        ↓
8. Test using VPS IP
        ↓
9. DELETE /var/www/website
        ↓
10. Install Git
        ↓
11. Clone real GitHub project as /var/www/website
        ↓
12. Install NVM
        ↓
13. Install latest Node.js LTS
        ↓
14. Install backend dependencies
        ↓
15. Add backend .env
        ↓
16. Test backend
        ↓
17. Install PM2
        ↓
18. Run backend with PM2
        ↓
19. Configure PM2 startup
        ↓
20. Install frontend dependencies
        ↓
21. Add frontend .env
        ↓
22. npm run build
        ↓
23. Configure domains in Nginx
        ↓
24. Add DNS records
        ↓
25. nginx -t
        ↓
26. Reload Nginx
        ↓
27. Install SSL using Certbot
        ↓
28. Deployment complete
```

---

# Quick Deployment Cheat Sheet

Once the VPS has been configured, future updates are simple.

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
