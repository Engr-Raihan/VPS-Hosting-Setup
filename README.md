<div align="center">

# 🚀 Complete Ubuntu 24.04 VPS Setup Guide
### Production-Ready Multi-Microservice Architecture

![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04%20LTS-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-20.x%20LTS-339933?style=for-the-badge&logo=node.js&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Latest-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-Latest-009639?style=for-the-badge&logo=nginx&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)

**A comprehensive, production-ready guide for deploying multiple Node.js microservices and Next.js applications on Ubuntu 24.04 LTS**

[Features](#-features) • [Quick Start](#-quick-start) • [Documentation](#-table-of-contents) • [Contributing](#-contributing)

</div>

---

## 📖 About This Guide

**Target OS:** Ubuntu 24.04 LTS (Noble Numbat)  
**Purpose:** Host multiple Node.js microservices + Next.js applications  
**Architecture:** Production-ready multi-service infrastructure  
**Tested:** ✅ Verified on Ubuntu 24.04 LTS  
**Last Updated:** November 2025  
**Estimated Setup Time:** 2-3 hours

### ✨ Features

- 🔒 **Security-First Approach** - SSH hardening, UFW firewall, Fail2Ban
- 🚀 **Zero-Downtime Deployments** - PM2 cluster mode with hot reloading
- 📊 **Production Monitoring** - Health checks, log rotation, disk monitoring
- 💾 **Automated Backups** - Daily backups with retention policies
- 🌐 **SSL/TLS Ready** - Let's Encrypt integration with auto-renewal
- ⚡ **High Performance** - Nginx reverse proxy with rate limiting
- 🐳 **Containerized Databases** - MongoDB and RabbitMQ via Docker
- 📈 **Scalable Architecture** - Easy to add new microservices

---

## ⚠️ Disclaimer

**IMPORTANT - READ BEFORE PROCEEDING:**

- This guide is provided "AS IS" without warranty of any kind, express or implied.
- The author(s) and contributors assume **NO RESPONSIBILITY** for any damages, data loss, security breaches, or system failures that may occur from following this guide.
- You are **SOLELY RESPONSIBLE** for:
  - Understanding each command before executing it
  - Backing up your data before making changes
  - Securing your server and credentials
  - Complying with applicable laws and regulations
  - Testing in a non-production environment first
  - Maintaining and updating your systems
- **Security**: This guide includes security best practices, but no system is 100% secure. Stay informed about security updates and monitor your systems regularly.
- **Production Use**: While this guide is tested and production-ready, you should thoroughly test all procedures in a development/staging environment first.
- By using this guide, you acknowledge that you understand these risks and accept full responsibility for your actions.

---

## 👨‍💻 Author Information

**Created by:** Engr. Raihan Mahamud  
**GitHub:** [@Engr-Raihan](https://github.com/Engr-Raihan)  
**LinkedIn:** [linkedin.com/in/engr-raihan](https://www.linkedin.com/in/engr-raihan/)  
**Contact:** engr.raihan.buet@gmail.com

### 🤝 Contributing

Contributions are welcome! If you find issues or have improvements:

1. Fork this repository: [VPS-Hosting-Setup](https://github.com/Engr-Raihan/VPS-Hosting-Setup)
2. Create your feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -m 'Add some improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed guidelines.

### 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

### ⭐ Support

If this guide helped you, please consider:
- Starring the repository ⭐
- Sharing it with others
- Contributing improvements
- Reporting issues

---

## 🎯 Ubuntu 24.04 LTS Specific Notes

This guide is specifically optimized for **Ubuntu 24.04 LTS (Noble Numbat)**. Key differences from previous versions:

- ✅ **Node.js 20 LTS** - Latest stable version with improved performance
- ✅ **Updated systemd** - Better service management
- ✅ **Enhanced security** - AppArmor profiles enabled by default
- ✅ **Kernel 6.8** - Better hardware support and performance
- ✅ **Updated package repositories** - Latest versions of all tools
- ⚠️ **SSH service name** - Use `ssh` not `sshd` (e.g., `systemctl restart ssh`)

**Migration from older Ubuntu versions?** This guide handles all breaking changes automatically.

### Important Service Name Change

**Ubuntu 24.04 uses `ssh` as the service name (not `sshd`):**

```bash
# ✅ CORRECT for Ubuntu 24.04
sudo systemctl restart ssh
sudo systemctl status ssh
sudo systemctl enable ssh

# ❌ WRONG - This will fail
sudo systemctl restart sshd
```

**Note:** The config file is still `/etc/ssh/sshd_config` and the test command is still `sudo sshd -t`.

---

## 📋 Table of Contents

1. [🏗️ Architecture Overview](#️-architecture-overview)
2. [🔐 Initial Server Setup](#-initial-server-setup-30-minutes)
3. [📦 Install Core Dependencies](#-install-core-dependencies-20-minutes)
4. [🗄️ Install Databases & Message Queues](#️-install-databases--message-queues-15-minutes)
5. [🏢 Setup Application Infrastructure](#-setup-application-infrastructure-10-minutes)
6. [🚀 Deploy First Microservice](#-deploy-first-microservice-auth-service-20-minutes)
7. [🌐 Configure Nginx for Multiple Services](#-configure-nginx-for-multiple-services-30-minutes)
8. [🔒 Security Hardening](#-security-hardening-20-minutes)
9. [📊 Monitoring & Logging](#-monitoring--logging-15-minutes)
10. [💾 Backup Strategy](#-backup-strategy-15-minutes)
11. [🔄 Maintenance & Updates](#-maintenance--updates-10-minutes)
12. [📝 Deploying Additional Services](#-deploying-additional-services)
13. [✅ Verification Checklist](#-verification-checklist)
14. [🎯 Testing Your Setup](#-testing-your-setup)
15. [📚 Useful Commands Reference](#-useful-commands-reference)
16. [🚨 Troubleshooting Guide](#-troubleshooting-guide)

---

## 🚀 Quick Start

Already familiar with server setups? Here's the TL;DR:

```bash
# 1. Update system
sudo apt update && sudo apt upgrade -y

# 2. Install core stack
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs nginx
sudo npm install -g pm2
curl -fsSL https://get.docker.com | sudo sh

# 3. Setup firewall
sudo ufw allow 22,80,443/tcp && sudo ufw enable

# 4. Deploy your app
cd /opt/apps/your-service
npm ci && npm run build
pm2 start ecosystem.config.js

# 5. Configure Nginx + SSL
sudo certbot --nginx -d your-domain.com
```

📖 **First time?** Follow the detailed guide below!

---

## 🏗️ Architecture Overview

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                         🌐 Internet                             │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                    ┌───────────▼──────────┐
                    │   🔒 Cloudflare      │ (Optional)
                    │   DDoS Protection    │
                    └───────────┬──────────┘
                                │
                    ┌───────────▼──────────┐
                    │   🛡️ UFW Firewall    │
                    │   Ports: 22,80,443   │
                    └───────────┬──────────┘
                                │
                    ┌───────────▼─────────────────────┐
                    │   🌐 Nginx Reverse Proxy        │
                    │   - SSL/TLS Termination         │
                    │   - Load Balancing              │
                    │   - Rate Limiting               │
                    │   - Gzip Compression            │
                    └──────┬──────────┬────────┬──────┘
                           │          │        │
              ┌────────────▼──┐  ┌───▼────┐  ┌▼────────────┐
              │ 📦 Service 1  │  │ Service│  │ Service 3   │
              │ PM2 Cluster   │  │   2    │  │ (Next.js)   │
              │ Port: 3001    │  │ :3002  │  │ Port: 3003  │
              └───────┬───────┘  └───┬────┘  └─────┬───────┘
                      │              │             │
                      └──────┬───────┴─────────────┘
                             │
                    ┌────────▼────────────────┐
                    │  🔌 localhost:27017     │
                    │  🐳 Docker Network      │
                    ├─────────────────────────┤
                    │  🗄️ MongoDB             │
                    │  📨 RabbitMQ           │
                    └─────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │  💾 Backups     │
                    │  - Local        │
                    │  - Cloud (S3)   │
                    └─────────────────┘
```

### Directory Structure
```
📁 /opt/
├── 📁 apps/                           # Application code
│   ├── 📁 auth-service/               # Authentication microservice
│   │   ├── 📄 .env                    # Environment variables (chmod 600)
│   │   ├── 📁 dist/                   # Compiled code
│   │   └── 📁 node_modules/           # Dependencies
│   ├── 📁 user-service/               # User management service
│   ├── 📁 business-app/               # Next.js application
│   ├── 📄 ecosystem.config.js         # PM2 configuration
│   └── 📁 [future-services]/
│
├── 📁 databases/                      # Database storage
│   ├── 📁 mongodb/
│   │   ├── 📁 data/                   # MongoDB data files
│   │   └── 📁 logs/                   # MongoDB logs
│   ├── 📁 rabbitmq/
│   │   ├── 📁 data/                   # RabbitMQ data
│   │   └── 📁 logs/                   # RabbitMQ logs
│   ├── 📄 docker-compose.yml          # Docker services config
│   └── 📄 .env                        # Database credentials
│
├── 📁 backups/                        # Backup storage
│   ├── 📁 mongodb/                    # Database backups
│   ├── 📁 configs/                    # Config backups
│   └── 📁 ssl/                        # SSL certificate backups
│
└── 📁 scripts/                        # Maintenance scripts
    ├── 📄 backup-all.sh               # Backup script
    ├── 📄 health-check.sh             # Health monitoring
    ├── 📄 disk-monitor.sh             # Disk space checker
    └── 📄 update-service.sh           # Service updater

📁 /etc/
├── 📁 nginx/
│   ├── 📁 sites-available/            # Nginx site configs
│   │   ├── 📄 auth-api
│   │   └── 📄 business-app
│   ├── 📁 sites-enabled/              # Enabled sites (symlinks)
│   └── 📁 conf.d/                     # Additional configs
│       ├── 📄 upstream.conf           # Load balancer config
│       └── 📄 rate-limit.conf         # Rate limiting rules
│
└── 📁 letsencrypt/                    # SSL certificates
    └── 📁 live/
        └── 📁 yourdomain.com/

📁 /var/
├── 📁 log/
│   ├── 📁 apps/                       # Application logs
│   │   ├── 📁 auth-service/
│   │   │   ├── 📄 pm2-error.log
│   │   │   └── 📄 pm2-out.log
│   │   ├── 📄 health-check.log
│   │   └── 📄 backup.log
│   └── 📁 nginx/                      # Nginx logs
│       ├── 📄 access.log
│       ├── 📄 error.log
│       └── 📄 auth-api.access.log
│
└── 📁 www/
    └── 📁 html/                       # Web root for Let's Encrypt
```

### Request Flow

```
1. User Request
   └─> HTTPS (443) → Nginx
        ├─> SSL Termination
        ├─> Rate Limiting Check
        └─> Route to Service
             └─> PM2 Process (Round-robin)
                  ├─> Instance 1 (CPU Core 1)
                  └─> Instance 2 (CPU Core 2)
                       └─> MongoDB Connection Pool
                            └─> Response ← ← ← ← ← ← Back to User

2. WebSocket Connection
   └─> WSS (443) → Nginx
        └─> Upgrade Connection
             └─> PM2 Process (Sticky Session)
                  └─> Persistent Connection

3. Static Files (Next.js)
   └─> HTTPS (443) → Nginx
        ├─> Gzip Compression
        ├─> Cache Headers
        └─> Direct Serve (no PM2)
```

### Technology Stack
- **OS:** Ubuntu 24.04 LTS
- **Node.js:** v20.x LTS
- **Process Manager:** PM2 (for Node.js apps)
- **Web Server:** Nginx (reverse proxy + load balancer)
- **Database:** MongoDB 7.0 (Docker)
- **Message Queue:** RabbitMQ 3.x (Docker)
- **SSL:** Let's Encrypt (Certbot)
- **Firewall:** UFW
- **Container Runtime:** Docker (for databases only)

---

## 🔐 Initial Server Setup (30 minutes)

---

### Step 1: Update System
```bash
# Update package lists and upgrade existing packages
sudo apt update && sudo apt upgrade -y

# Install essential build tools
sudo apt install -y build-essential curl wget git vim software-properties-common apt-transport-https ca-certificates gnupg lsb-release

# Set timezone (adjust as needed)
sudo timedatectl set-timezone UTC

# Verify
timedatectl
```

### Step 2: Create Non-Root User (Security Best Practice)
```bash
# Create dedicated user for applications
sudo adduser --disabled-password --gecos "" appadmin
sudo usermod -aG sudo appadmin

# Set strong password
sudo passwd appadmin

# Configure sudo without password (optional, for automation)
echo "appadmin ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/appadmin
```

### Step 3: Setup SSH Keys (Do This BEFORE Securing SSH!)

**⚠️ CRITICAL: Complete this step and test access before proceeding to Step 4!**

#### On Your Local Machine (Windows):

**A. First, fix the known_hosts issue if you get "HOST IDENTIFICATION HAS CHANGED" error:**

```powershell
# Open PowerShell as Administrator and run:
# Remove old host key for this IP (replace with your server IP)
ssh-keygen -R YOUR_SERVER_IP

# Or manually edit and remove the offending line:
notepad C:\Users\YourUsername\.ssh\known_hosts
# Delete the line mentioned in the error
```

**B. Generate SSH key (if not already done):**

```powershell
# Generate ED25519 key (replace with your email)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Press Enter to accept default location: C:\Users\YourUsername\.ssh\id_ed25519
# Enter a strong passphrase (optional but HIGHLY recommended)
```

**C. Copy public key to server (Windows method):**

```powershell
# Method 1: Using PowerShell (RECOMMENDED for Windows)
# Replace YOUR_SERVER_IP with your actual server IP address
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh appadmin@YOUR_SERVER_IP "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"

# Method 2: Manual copy (if Method 1 fails)
# First, display your public key:
type $env:USERPROFILE\.ssh\id_ed25519.pub

# Then, SSH to server with password (replace YOUR_SERVER_IP):
ssh appadmin@YOUR_SERVER_IP

# On server, run these commands:
mkdir -p ~/.ssh
nano ~/.ssh/authorized_keys
# Paste your public key (from step above), save and exit (Ctrl+X, Y, Enter)

chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
exit
```

**D. Test SSH key authentication:**

```powershell
# Try to connect using SSH key (should NOT ask for password)
# Replace YOUR_SERVER_IP with your actual server IP
ssh appadmin@YOUR_SERVER_IP

# If it works without password, you're good!
# If it asks for password, something went wrong - DON'T proceed to Step 4!
```

#### On Your Local Machine (Linux/Mac):

```bash
# Generate SSH key (replace with your email)
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy public key to server (replace YOUR_SERVER_IP)
ssh-copy-id appadmin@YOUR_SERVER_IP

# Test SSH key login (should NOT ask for password)
ssh appadmin@YOUR_SERVER_IP
```

**✅ CHECKPOINT: Make sure you can SSH into the server WITHOUT entering a password before proceeding!**

### Step 4: Configure SSH Security (ONLY After Step 3 Works!)

**⚠️ WARNING: Only proceed if Step 3 is working perfectly!**

```bash
# Backup original SSH config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Edit SSH config
sudo nano /etc/ssh/sshd_config
```

**Edit these lines in /etc/ssh/sshd_config:**

```bash
# Find and modify these lines (use Ctrl+W to search in nano):

# 1. Allow specific user (ADD this line if not present)
AllowUsers appadmin

# 2. Disable empty passwords (should already be set, verify it's "no")
PermitEmptyPasswords no

# 3. Limit authentication attempts (ADD this line)
MaxAuthTries 3

# 4. Set login grace time (ADD this line)
LoginGraceTime 60

# ⚠️ DO NOT CHANGE THESE YET - Leave them commented for now:
# PermitRootLogin yes  (leave this as-is for now)
# PasswordAuthentication yes  (leave this as-is for now)
```

**Test SSH restart WITHOUT logging out:**

```bash
# Restart SSH service (Ubuntu 24.04 uses 'ssh' not 'sshd')
sudo systemctl restart ssh

# Open a NEW terminal window (don't close current one!)
# Try to connect from new window (replace YOUR_SERVER_IP):
ssh appadmin@YOUR_SERVER_IP

# If new connection works, close the new window and continue
# If it FAILS, you still have your original connection to fix it!
```

**✅ CHECKPOINT: Only after testing above works, proceed below**

### Step 4b: Disable Root Login (Optional - Only if SSH keys work!)

**⚠️ FINAL WARNING: Only do this if you're 100% sure SSH key access works!**

```bash
# Edit SSH config again
sudo nano /etc/ssh/sshd_config

# Find and change this line:
PermitRootLogin no

# Save and exit (Ctrl+X, Y, Enter)

# Test the change
sudo sshd -t

# If test passes, restart SSH (Ubuntu 24.04 uses 'ssh' not 'sshd')
sudo systemctl restart ssh

# Keep your current session open!
# Open NEW terminal and test (replace YOUR_SERVER_IP):
ssh appadmin@YOUR_SERVER_IP

# If it works, you're good!
# If it fails, use your open session to revert changes
```

### Step 4c: Disable Password Authentication (Optional - Maximum Security)

**⚠️ EXTREME WARNING: Only do this if SSH key access has been working perfectly for several days!**

```bash
# Edit SSH config
sudo nano /etc/ssh/sshd_config

# Find and change:
PasswordAuthentication no

# Save, test, and restart
sudo sshd -t
sudo systemctl restart ssh

# Test from NEW terminal before closing current one!
```

### Step 5: Configure Firewall (Safe to do anytime)
```bash
# Enable UFW
sudo ufw enable

# Allow SSH (CRITICAL - do this first!)
sudo ufw allow 22/tcp

# Allow HTTP and HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Check status
sudo ufw status verbose

# Enable firewall logging (optional)
sudo ufw logging on
```

### Step 6: Configure Automatic Security Updates
```bash
# Install unattended-upgrades
sudo apt install -y unattended-upgrades

# Configure automatic updates
sudo dpkg-reconfigure -plow unattended-upgrades

# Edit configuration (optional)
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

### Step 7: Install Fail2Ban (Brute Force Protection)
```bash
# Install fail2ban
sudo apt install -y fail2ban

# Create local configuration
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit configuration
sudo nano /etc/fail2ban/jail.local
```

**Recommended Fail2Ban Settings:**
```ini
[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600

[nginx-http-auth]
enabled = true
filter = nginx-http-auth
port = http,https
logpath = /var/log/nginx/error.log
maxretry = 5
bantime = 3600
```

**Start Fail2Ban:**
```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban

# Check status
sudo fail2ban-client status
```

---

## 📦 Install Core Dependencies (20 minutes)

### Step 1: Install Node.js 20 LTS
```bash
# Add NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js
sudo apt install -y nodejs

# Verify installation
node --version  # Should show v20.x.x
npm --version   # Should show 10.x.x

# Update npm to latest
sudo npm install -g npm@latest
```

### Step 2: Install PM2 (Process Manager)
```bash
# Install PM2 globally
sudo npm install -g pm2

# Setup PM2 startup script
pm2 startup systemd -u appadmin --hp /home/appadmin
# Run the command that PM2 outputs

# Verify
pm2 --version
```

### Step 3: Install Nginx
```bash
# Install Nginx
sudo apt install -y nginx

# Start and enable
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify
nginx -v
sudo systemctl status nginx
```

### Step 4: Install Docker (for Databases)
```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker appadmin

# Start and enable Docker
sudo systemctl start docker
sudo systemctl enable docker

# Verify (may need to log out and back in for group to take effect)
docker --version

# Install Docker Compose
sudo apt install -y docker-compose-plugin

# Verify
docker compose version
```

### Step 5: Install Certbot (SSL Certificates)
```bash
# Install Certbot with Nginx plugin
sudo apt install -y certbot python3-certbot-nginx

# Verify
certbot --version
```

---

## 🗄️ Install Databases & Message Queues (15 minutes)

### Docker Compose Setup (Recommended for Multiple Services)

Create directory structure:
```bash
# Create directories
sudo mkdir -p /opt/databases/{mongodb,rabbitmq}
sudo mkdir -p /opt/backups/{mongodb,rabbitmq}
sudo chown -R appadmin:appadmin /opt/databases /opt/backups

cd /opt/databases
```

Create `docker-compose.yml`:
```bash
nano docker-compose.yml
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:7.0
    container_name: mongodb
    restart: unless-stopped
    ports:
      - "127.0.0.1:27017:27017"  # Only bind to localhost
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_ROOT_PASSWORD}
    volumes:
      - ./mongodb/data:/data/db
      - ./mongodb/logs:/var/log/mongodb
    networks:
      - backend
    command: --auth --bind_ip_all

  rabbitmq:
    image: rabbitmq:3-management-alpine
    container_name: rabbitmq
    restart: unless-stopped
    ports:
      - "127.0.0.1:5672:5672"    # AMQP - Only bind to localhost
      - "127.0.0.1:15672:15672"  # Management UI - Only bind to localhost
    environment:
      RABBITMQ_DEFAULT_USER: ${RABBIT_USER}
      RABBITMQ_DEFAULT_PASS: ${RABBIT_PASSWORD}
    volumes:
      - ./rabbitmq/data:/var/lib/rabbitmq
      - ./rabbitmq/logs:/var/log/rabbitmq
    networks:
      - backend

networks:
  backend:
    driver: bridge

volumes:
  mongodb-data:
  rabbitmq-data:
```

Create `.env` file for Docker Compose:
```bash
nano .env
```

**Docker Compose .env:**
```env
# MongoDB Configuration
# Generate strong password: openssl rand -base64 32
MONGO_ROOT_PASSWORD=YOUR_SECURE_MONGODB_PASSWORD_HERE

# RabbitMQ Configuration
RABBIT_USER=admin
# Generate strong password: openssl rand -base64 32
RABBIT_PASSWORD=YOUR_SECURE_RABBITMQ_PASSWORD_HERE
```

**💡 TIP:** Generate secure passwords using:
```bash
# Generate a strong 32-character password
openssl rand -base64 32
```

**Secure the .env file:**
```bash
chmod 600 .env
```

**Start services:**
```bash
# Start all services
docker compose up -d

# Verify services are running
docker compose ps

# View logs
docker compose logs -f

# Check individual service logs
docker logs mongodb
docker logs rabbitmq
```

### Configure MongoDB
```bash
# Wait for MongoDB to fully start
sleep 10

# Connect to MongoDB (replace with password from your .env file)
docker exec -it mongodb mongosh -u admin -p 'YOUR_MONGO_ROOT_PASSWORD' --authenticationDatabase admin

# Inside MongoDB shell:
use admin

# Create database users for each microservice
# Replace passwords with secure ones (use: openssl rand -base64 32)

use auth_service_db
db.createUser({
  user: "auth_service_user",
  pwd: "GENERATE_SECURE_PASSWORD_FOR_AUTH_SERVICE",
  roles: [ { role: "readWrite", db: "auth_service_db" } ]
})

use user_service_db
db.createUser({
  user: "user_service_user",
  pwd: "GENERATE_SECURE_PASSWORD_FOR_USER_SERVICE",
  roles: [ { role: "readWrite", db: "user_service_db" } ]
})

# Repeat for additional services as needed

# Exit
exit
```

**📝 Note:** Save all database usernames and passwords securely. You'll need them for your application .env files.

### Verify RabbitMQ
```bash
# Check RabbitMQ is running
docker exec rabbitmq rabbitmqctl status

# Access management UI via SSH tunnel (from your local machine):
# Replace YOUR_SERVER_IP with your actual server IP
ssh -L 15672:localhost:15672 appadmin@YOUR_SERVER_IP

# Then open in browser: http://localhost:15672
# Username: admin
# Password: (use the RABBIT_PASSWORD from your .env file)
```

**💡 TIP:** The SSH tunnel allows you to securely access RabbitMQ management UI without exposing it to the internet.

---

## 🏢 Setup Application Infrastructure (10 minutes)

### Create Application Directory Structure
```bash
# Create main apps directory
sudo mkdir -p /opt/apps
sudo chown -R appadmin:appadmin /opt/apps

# Create logs directory
sudo mkdir -p /var/log/apps
sudo chown -R appadmin:appadmin /var/log/apps

# Create directory for first microservice
mkdir -p /opt/apps/auth-service
cd /opt/apps/auth-service
```

### Setup Centralized PM2 Configuration
```bash
# Create PM2 ecosystem file for all services
cd /opt/apps
nano ecosystem.config.js
```

**ecosystem.config.js (Multi-Service):**
```javascript
module.exports = {
  apps: [
    {
      name: 'auth-service',
      cwd: '/opt/apps/auth-service',
      script: './dist/main.js',
      instances: 2,  // Run 2 instances
      exec_mode: 'cluster',
      autorestart: true,
      watch: false,
      max_memory_restart: '500M',
      env: {
        NODE_ENV: 'production',
        PORT: 3001,
      },
      error_file: '/var/log/apps/auth-service/pm2-error.log',
      out_file: '/var/log/apps/auth-service/pm2-out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      merge_logs: true,
    },
    // Add more services here as you deploy them
    // {
    //   name: 'user-service',
    //   cwd: '/opt/apps/user-service',
    //   script: './dist/main.js',
    //   instances: 2,
    //   exec_mode: 'cluster',
    //   env: {
    //     NODE_ENV: 'production',
    //     PORT: 3002,
    //   },
    //   ...
    // },
  ],
};
```

---

## 🚀 Deploy First Microservice (Auth Service) (20 minutes)

### Step 1: Clone Repository
```bash
cd /opt/apps/auth-service

# Option 1: Clone your repository (replace with your actual repo URL)
git clone https://github.com/your-username/your-auth-service-repo.git .

# Option 2: Upload files manually via SCP (from local machine)
# Replace YOUR_SERVER_IP and adjust paths as needed
scp -r ./your-auth-service/* appadmin@YOUR_SERVER_IP:/opt/apps/auth-service/

# Option 3: Use rsync for better performance
rsync -avz --progress ./your-auth-service/ appadmin@YOUR_SERVER_IP:/opt/apps/auth-service/
```

**💡 TIP:** If using a private repository, set up SSH keys for GitHub/GitLab first:
```bash
# Generate SSH key for Git
ssh-keygen -t ed25519 -C "your_email@example.com" -f ~/.ssh/github_key

# Add to GitHub: cat ~/.ssh/github_key.pub and add to GitHub SSH keys
```

### Step 2: Setup Environment Configuration
```bash
# Create .env file
nano .env
```

**Production .env for Auth Service:**
```env
# Application Configuration
NODE_ENV=production
PORT=3001
ORIGIN=auth-service

# Domain Configuration (replace with your actual domain)
COOKIE_DOMAIN=yourdomain.com
ALLOWED_ORIGINS=https://yourdomain.com,https://api.yourdomain.com

# Database Configuration
# Replace with the password you created for auth_service_user
MONGODB_TENANT_URI=mongodb://auth_service_user:YOUR_AUTH_SERVICE_DB_PASSWORD@localhost:27017/auth_service_db?authSource=auth_service_db

# JWT Configuration (GENERATE using command below!)
JWT_SECRET=GENERATE_JWT_SECRET_USING_OPENSSL_COMMAND
JWT_EXPIRES_IN=3600
JWT_REFRESH_EXPIRES_IN=604800

# RabbitMQ Configuration (use password from Docker .env)
RABBIT_USERNAME=admin
RABBIT_PASSWORD=YOUR_SECURE_RABBITMQ_PASSWORD_HERE
RABBIT_URL=localhost:5672
RABBIT_QUEUE_NAME=auth-email-queue

# Cookie Configuration (in milliseconds)
COOKIE_TOKEN_EXPIRE=3600000        # 1 hour
COOKIE_REFRESH_TOKEN_EXPIRE=604800000  # 7 days

# Account Activation
ACCOUNT_ACTIVATION=false

# Optional: Email Configuration (if using email features)
# SMTP_HOST=smtp.yourprovider.com
# SMTP_PORT=587
# SMTP_USER=your_email@example.com
# SMTP_PASSWORD=your_email_password
```

**Generate Secrets:**
```bash
# Generate JWT Secret (copy output to .env)
openssl rand -base64 48

# Generate additional secrets if needed
openssl rand -hex 32
```

**⚠️ SECURITY REMINDER:**
- Never commit `.env` files to version control
- Use different secrets for each environment
- Rotate secrets regularly
- Store backups of `.env` files securely

**Secure the .env file:**
```bash
chmod 600 .env
```

### Step 3: Install Dependencies and Build
```bash
# Install dependencies
npm ci --production=false

# Build application
npm run build

# Install production dependencies only
rm -rf node_modules
npm ci --production

# Verify build
ls -la dist/
```

### Step 4: Create Log Directory
```bash
sudo mkdir -p /var/log/apps/auth-service
sudo chown -R appadmin:appadmin /var/log/apps/auth-service
```

### Step 5: Start with PM2
```bash
# From /opt/apps directory
cd /opt/apps

# Start the auth service
pm2 start ecosystem.config.js --only auth-service

# Save PM2 configuration
pm2 save

# Verify service is running
pm2 status
pm2 logs auth-service --lines 50

# Test the service locally
curl http://localhost:3001/api
```

---

## 🌐 Configure Nginx for Multiple Services (30 minutes)

### Step 1: Remove Default Nginx Configuration
```bash
sudo rm /etc/nginx/sites-enabled/default
```

### Step 2: Create Nginx Configuration Template
```bash
sudo mkdir -p /etc/nginx/conf.d
sudo nano /etc/nginx/conf.d/upstream.conf
```

**upstream.conf (Load Balancer Configuration):**
```nginx
# Auth Service Upstream
upstream auth_service {
    least_conn;  # Use least connections algorithm
    server 127.0.0.1:3001 max_fails=3 fail_timeout=30s;
    # Add more instances if you scale:
    # server 127.0.0.1:3002 max_fails=3 fail_timeout=30s;
}

# Add more upstreams for other services
# upstream user_service {
#     least_conn;
#     server 127.0.0.1:3002 max_fails=3 fail_timeout=30s;
# }

# upstream business_app {
#     least_conn;
#     server 127.0.0.1:3003 max_fails=3 fail_timeout=30s;
# }
```

### Step 3: Configure Rate Limiting
```bash
sudo nano /etc/nginx/conf.d/rate-limit.conf
```

**rate-limit.conf:**
```nginx
# Define rate limit zones
limit_req_zone $binary_remote_addr zone=general_limit:10m rate=100r/s;
limit_req_zone $binary_remote_addr zone=auth_limit:10m rate=5r/s;
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=50r/s;

# Connection limit
limit_conn_zone $binary_remote_addr zone=conn_limit:10m;
```

### Step 4: Create Site Configuration for Auth Service
```bash
sudo nano /etc/nginx/sites-available/auth-api
```

**auth-api (Production-Ready Configuration):**

**⚠️ IMPORTANT:** Replace `api.yourdomain.com` with your actual domain name throughout this configuration.

```nginx
# HTTP Server - Redirect to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name api.yourdomain.com;  # REPLACE with your domain

    # For Let's Encrypt verification
    location /.well-known/acme-challenge/ {
        root /var/www/html;
    }

    # Redirect all other traffic to HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS Server
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name api.yourdomain.com;  # REPLACE with your domain

    # SSL Configuration (will be added automatically by Certbot)
    # ssl_certificate /etc/letsencrypt/live/api.yourdomain.com/fullchain.pem;
    # ssl_certificate_key /etc/letsencrypt/live/api.yourdomain.com/privkey.pem;

    # SSL Security Settings
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_stapling on;
    ssl_stapling_verify on;

    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;

    # Gzip Compression
    gzip on;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_min_length 1000;
    gzip_proxied any;
    gzip_types 
        text/plain 
        text/css 
        text/xml 
        text/javascript 
        application/json 
        application/javascript 
        application/xml+rss 
        application/rss+xml 
        font/truetype 
        font/opentype 
        application/vnd.ms-fontobject 
        image/svg+xml;

    # Client body size limit
    client_max_body_size 10M;

    # Timeouts
    client_body_timeout 12;
    client_header_timeout 12;
    keepalive_timeout 15;
    send_timeout 10;

    # Buffer sizes
    client_body_buffer_size 10K;
    client_header_buffer_size 1k;
    large_client_header_buffers 2 1k;

    # Connection limits
    limit_conn conn_limit 10;

    # Auth endpoints - Stricter rate limiting
    location ~ ^/api/(users/signin|users/signup) {
        limit_req zone=auth_limit burst=10 nodelay;
        limit_req_status 429;

        proxy_pass http://auth_service;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Timeouts for auth endpoints
        proxy_connect_timeout 10s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }

    # General API endpoints
    location /api/ {
        limit_req zone=api_limit burst=50 nodelay;
        limit_req_status 429;

        proxy_pass http://auth_service;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Health check endpoint (no rate limit)
    location /health {
        access_log off;
        proxy_pass http://auth_service;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
    }

    # Logging
    access_log /var/log/nginx/auth-api.access.log;
    error_log /var/log/nginx/auth-api.error.log warn;
}
```

### Step 5: Enable Site
```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/auth-api /etc/nginx/sites-enabled/

# Test Nginx configuration
sudo nginx -t

# If test passes, reload Nginx
sudo systemctl reload nginx
```

### Step 6: Obtain SSL Certificate

**⚠️ PREREQUISITES:**
1. Your domain DNS must point to your server IP address
2. Wait 5-10 minutes for DNS propagation after updating DNS records
3. Port 80 must be open in your firewall (already done if you followed Step 5 in Initial Setup)

```bash
# Replace api.yourdomain.com with your actual domain
sudo certbot --nginx -d api.yourdomain.com

# For multiple domains/subdomains:
sudo certbot --nginx -d api.yourdomain.com -d api2.yourdomain.com

# Follow the prompts:
# - Enter your email (for renewal notifications)
# - Agree to terms of service
# - Choose whether to redirect HTTP to HTTPS (recommend: 2 for redirect)

# Test automatic renewal (dry run - doesn't actually renew)
sudo certbot renew --dry-run

# Check certificate status and expiry
sudo certbot certificates
```

**💡 TIP:** Certificates expire every 90 days, but Certbot sets up auto-renewal automatically via systemd timer.

### Step 7: Setup Auto-Renewal Cron Job (Certbot usually does this automatically)
```bash
# Check if renewal timer is active
sudo systemctl status certbot.timer

# If not, add manual cron job
sudo crontab -e

# Add this line:
0 3 * * * /usr/bin/certbot renew --quiet --post-hook "systemctl reload nginx"
```

---

## 🔒 Security Hardening (20 minutes)

### Step 1: Configure System Limits
```bash
sudo nano /etc/security/limits.conf
```

**Add these lines:**
```
* soft nofile 65536
* hard nofile 65536
* soft nproc 65536
* hard nproc 65536
```

### Step 2: Kernel Optimization
```bash
sudo nano /etc/sysctl.conf
```

**Add these optimizations:**
```
# Network Performance
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 8192
net.core.netdev_max_backlog = 5000

# Connection Handling
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_keepalive_intvl = 15

# Security
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0

# File Descriptors
fs.file-max = 2097152
```

**Apply changes:**
```bash
sudo sysctl -p
```

### Step 3: Secure MongoDB and RabbitMQ Access
```bash
# Edit Docker Compose to ensure ports are bound to localhost only
cd /opt/databases
nano docker-compose.yml

# Ensure ports are configured as:
# ports:
#   - "127.0.0.1:27017:27017"  # MongoDB
#   - "127.0.0.1:5672:5672"    # RabbitMQ
#   - "127.0.0.1:15672:15672"  # RabbitMQ Management

# Restart if you made changes
docker compose down
docker compose up -d
```

### Step 4: Setup Log Rotation for Application Logs
```bash
sudo nano /etc/logrotate.d/nodejs-apps
```

**Add:**
```
/var/log/apps/*/*.log {
    daily
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 appadmin appadmin
    sharedscripts
    postrotate
        su - appadmin -c "pm2 reloadLogs"
    endscript
    su appadmin appadmin
}
```

### Step 5: Configure PM2 Log Rotation
```bash
# Install PM2 logrotate module
pm2 install pm2-logrotate

# Configure
pm2 set pm2-logrotate:max_size 50M
pm2 set pm2-logrotate:retain 14
pm2 set pm2-logrotate:compress true
pm2 set pm2-logrotate:workerInterval 3600
```

---

## 📊 Monitoring & Logging (15 minutes)

### Step 1: Setup PM2 Monitoring
```bash
# Install PM2 monitoring modules
pm2 install pm2-server-monit

# View real-time monitoring
pm2 monit

# Setup PM2 web dashboard (optional)
pm2 web
# Access at http://localhost:9615 (use SSH tunnel)
```

### Step 2: Create Health Check Scripts
```bash
mkdir -p /opt/scripts
nano /opt/scripts/health-check.sh
```

**health-check.sh:**
```bash
#!/bin/bash

LOG_FILE="/var/log/apps/health-check.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

echo "[$DATE] Starting health check..." >> $LOG_FILE

# Check PM2 processes
if ! pm2 status | grep -q "online"; then
    echo "[$DATE] ERROR: Some PM2 processes are not online" >> $LOG_FILE
    pm2 restart all
fi

# Check MongoDB
if ! docker exec mongodb mongosh --eval "db.adminCommand('ping')" > /dev/null 2>&1; then
    echo "[$DATE] ERROR: MongoDB is not responding" >> $LOG_FILE
    docker restart mongodb
fi

# Check RabbitMQ
if ! docker exec rabbitmq rabbitmqctl status > /dev/null 2>&1; then
    echo "[$DATE] ERROR: RabbitMQ is not responding" >> $LOG_FILE
    docker restart rabbitmq
fi

# Check Nginx
if ! systemctl is-active --quiet nginx; then
    echo "[$DATE] ERROR: Nginx is not running" >> $LOG_FILE
    sudo systemctl restart nginx
fi

echo "[$DATE] Health check completed" >> $LOG_FILE
```

**Make executable and schedule:**
```bash
chmod +x /opt/scripts/health-check.sh

# Add to crontab
crontab -e

# Run every 5 minutes
*/5 * * * * /opt/scripts/health-check.sh
```

### Step 3: Setup Disk Space Monitoring
```bash
nano /opt/scripts/disk-monitor.sh
```

**disk-monitor.sh:**
```bash
#!/bin/bash

THRESHOLD=80
ALERT_LOG="/var/log/apps/disk-alert.log"
DATE=$(date '+%Y-%m-%d %H:%M:%S')

df -H | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{ print $5 " " $1 }' | while read output; do
  usage=$(echo $output | awk '{ print $1}' | cut -d'%' -f1)
  partition=$(echo $output | awk '{ print $2 }')
  if [ $usage -ge $THRESHOLD ]; then
    echo "[$DATE] WARNING: $partition is ${usage}% full" >> $ALERT_LOG
  fi
done
```

```bash
chmod +x /opt/scripts/disk-monitor.sh

# Add to crontab (run every hour)
0 * * * * /opt/scripts/disk-monitor.sh
```

---

## 💾 Backup Strategy (15 minutes)

### Step 1: Create Backup Scripts
```bash
mkdir -p /opt/backups/{mongodb,configs,ssl}
nano /opt/scripts/backup-all.sh
```

**backup-all.sh:**
```bash
#!/bin/bash

BACKUP_ROOT="/opt/backups"
DATE=$(date +%Y%m%d_%H%M%S)
RETENTION_DAYS=7

# REPLACE WITH YOUR ACTUAL MONGODB PASSWORD
MONGO_PASSWORD="YOUR_MONGO_ROOT_PASSWORD_FROM_ENV_FILE"

echo "Starting backup at $DATE"

# Backup MongoDB
echo "Backing up MongoDB..."
docker exec mongodb mongodump --uri="mongodb://admin:${MONGO_PASSWORD}@localhost:27017/?authSource=admin" --out=/data/backup/$DATE
docker cp mongodb:/data/backup/$DATE $BACKUP_ROOT/mongodb/
docker exec mongodb rm -rf /data/backup/$DATE

# Backup application configs
echo "Backing up application configs..."
mkdir -p $BACKUP_ROOT/configs/$DATE
cp -r /opt/apps/*/package.json $BACKUP_ROOT/configs/$DATE/ 2>/dev/null
cp -r /opt/apps/*/.env $BACKUP_ROOT/configs/$DATE/ 2>/dev/null
cp /opt/apps/ecosystem.config.js $BACKUP_ROOT/configs/$DATE/

# Backup Nginx configs
cp -r /etc/nginx/sites-available $BACKUP_ROOT/configs/$DATE/nginx-sites/
cp -r /etc/nginx/conf.d $BACKUP_ROOT/configs/$DATE/nginx-conf/

# Backup SSL certificates
echo "Backing up SSL certificates..."
sudo cp -r /etc/letsencrypt $BACKUP_ROOT/ssl/$DATE/

# Compress backups
echo "Compressing backups..."
cd $BACKUP_ROOT
tar -czf mongodb-$DATE.tar.gz mongodb/$DATE
tar -czf configs-$DATE.tar.gz configs/$DATE
tar -czf ssl-$DATE.tar.gz ssl/$DATE

# Remove uncompressed backups
rm -rf mongodb/$DATE configs/$DATE ssl/$DATE

# Remove old backups (older than RETENTION_DAYS)
find $BACKUP_ROOT -name "*.tar.gz" -mtime +$RETENTION_DAYS -delete

echo "Backup completed at $(date +%Y%m%d_%H%M%S)"
```

**Make executable and schedule:**
```bash
chmod +x /opt/scripts/backup-all.sh

# Test the backup script
/opt/scripts/backup-all.sh

# Schedule daily backups at 2 AM
crontab -e

# Add:
0 2 * * * /opt/scripts/backup-all.sh >> /var/log/apps/backup.log 2>&1
```

### Step 2: Setup Remote Backup (Optional but Recommended)
```bash
# Install rclone for cloud backup
curl https://rclone.org/install.sh | sudo bash

# Configure rclone (for S3, Google Drive, etc.)
rclone config

# Create remote backup script
nano /opt/scripts/backup-to-cloud.sh
```

**backup-to-cloud.sh:**
```bash
#!/bin/bash

BACKUP_ROOT="/opt/backups"
# Replace with your rclone remote name (from rclone config)
REMOTE_NAME="your-remote-name"
REMOTE_PATH="vps-backups"

# Sync to remote storage
rclone sync $BACKUP_ROOT $REMOTE_NAME:$REMOTE_PATH \
    --log-file=/var/log/apps/rclone.log \
    --log-level INFO

echo "Cloud backup completed at $(date)"
```

**💡 Supported Cloud Providers for rclone:**
- Amazon S3
- Google Drive
- Dropbox
- Microsoft OneDrive
- Backblaze B2
- DigitalOcean Spaces
- Wasabi
- And 40+ more providers

```bash
chmod +x /opt/scripts/backup-to-cloud.sh

# Schedule (run after local backup)
crontab -e

# Add (run 30 minutes after local backup):
30 2 * * * /opt/scripts/backup-to-cloud.sh >> /var/log/apps/cloud-backup.log 2>&1
```

---

## 🔄 Maintenance & Updates (10 minutes)

### Automated Update Script
```bash
nano /opt/scripts/update-service.sh
```

**update-service.sh:**
```bash
#!/bin/bash

SERVICE_NAME=$1
SERVICE_PATH="/opt/apps/$SERVICE_NAME"

if [ -z "$SERVICE_NAME" ]; then
    echo "Usage: ./update-service.sh <service-name>"
    exit 1
fi

echo "Updating $SERVICE_NAME..."

cd $SERVICE_PATH

# Pull latest changes
git pull origin main

# Install dependencies
npm ci --production=false

# Build
npm run build

# Install production dependencies
rm -rf node_modules
npm ci --production

# Reload with PM2 (zero-downtime)
pm2 reload ecosystem.config.js --only $SERVICE_NAME

echo "$SERVICE_NAME updated successfully!"
```

```bash
chmod +x /opt/scripts/update-service.sh

# Usage:
# /opt/scripts/update-service.sh auth-service
```

### System Maintenance Script
```bash
nano /opt/scripts/system-maintenance.sh
```

**system-maintenance.sh:**
```bash
#!/bin/bash

echo "Starting system maintenance..."

# Update system packages
sudo apt update && sudo apt upgrade -y

# Update Node.js global packages
sudo npm update -g

# Update PM2
sudo npm install -g pm2@latest
pm2 update

# Clean up Docker
docker system prune -af --volumes

# Clean up old logs
find /var/log/apps -name "*.log" -mtime +30 -delete

# Clean up old backups
find /opt/backups -name "*.tar.gz" -mtime +30 -delete

# Update Nginx
sudo apt install --only-upgrade nginx -y

# Restart services if needed
pm2 restart all

echo "System maintenance completed!"
```

```bash
chmod +x /opt/scripts/system-maintenance.sh

# Schedule monthly (first Sunday of the month at 3 AM)
crontab -e

# Add:
0 3 1-7 * 0 [ "$(date +\%u)" = "7" ] && /opt/scripts/system-maintenance.sh >> /var/log/apps/maintenance.log 2>&1
```

---

## 📝 Deploying Additional Services

### Quick Guide for Adding New Microservice

Follow these steps to add any new microservice to your infrastructure:

#### 1. Create Service Directory
```bash
# Create directory for new service
mkdir -p /opt/apps/new-service
cd /opt/apps/new-service
```

#### 2. Clone/Upload Code
```bash
# Option 1: Clone from Git
git clone https://github.com/your-username/your-service-repo.git .

# Option 2: Upload via SCP (from local machine)
scp -r ./your-service/* appadmin@YOUR_SERVER_IP:/opt/apps/new-service/

# Option 3: Use rsync (recommended for large projects)
rsync -avz --progress ./your-service/ appadmin@YOUR_SERVER_IP:/opt/apps/new-service/
```

#### 3. Setup Environment
```bash
# Copy example env file
cp .env.example .env

# Edit with production values
nano .env

# Secure the file
chmod 600 .env

# Verify permissions
ls -la .env  # Should show: -rw------- 1 appadmin appadmin
```

#### 4. Install and Build
```bash
# Install all dependencies (including dev dependencies)
npm ci --production=false

# Build the application
npm run build

# Verify build output
ls -la dist/

# Remove dev dependencies and reinstall only production
rm -rf node_modules
npm ci --production --omit=dev

# Final size check
du -sh node_modules/
```

#### 5. Create Log Directory
```bash
# Create logs directory
sudo mkdir -p /var/log/apps/new-service

# Set proper ownership
sudo chown -R appadmin:appadmin /var/log/apps/new-service

# Verify
ls -ld /var/log/apps/new-service
```

#### 6. Add to PM2 Ecosystem
```bash
# Edit ecosystem config
nano /opt/apps/ecosystem.config.js
```

**Add this configuration:**
```javascript
{
  name: 'new-service',
  cwd: '/opt/apps/new-service',
  script: './dist/main.js',  // Adjust to your entry point
  instances: 2,               // Number of instances
  exec_mode: 'cluster',
  autorestart: true,
  watch: false,
  max_memory_restart: '500M',
  env: {
    NODE_ENV: 'production',
    PORT: 3003,               // Unique port for this service
  },
  error_file: '/var/log/apps/new-service/pm2-error.log',
  out_file: '/var/log/apps/new-service/pm2-out.log',
  log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
  merge_logs: true,
},
```

#### 7. Configure Nginx

**Add upstream:**
```bash
sudo nano /etc/nginx/conf.d/upstream.conf
```

Add:
```nginx
# New Service Upstream
upstream new_service {
    least_conn;
    server 127.0.0.1:3003 max_fails=3 fail_timeout=30s;
}
```

**Create site configuration:**
```bash
sudo nano /etc/nginx/sites-available/new-service
```

**Basic configuration template:**
```nginx
# HTTP - Redirect to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name service.yourdomain.com;

    location /.well-known/acme-challenge/ {
        root /var/www/html;
    }

    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name service.yourdomain.com;

    # SSL certificates (added by Certbot)
    
    # Proxy to service
    location / {
        limit_req zone=api_limit burst=50 nodelay;
        
        proxy_pass http://new_service;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
        
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # Logging
    access_log /var/log/nginx/new-service.access.log;
    error_log /var/log/nginx/new-service.error.log warn;
}
```

**Enable site:**
```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/new-service /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

#### 8. Obtain SSL Certificate
```bash
# Get SSL certificate (make sure DNS points to your server first!)
sudo certbot --nginx -d service.yourdomain.com

# Verify certificate
sudo certbot certificates
```

#### 9. Start Service
```bash
# Start the new service
cd /opt/apps
pm2 start ecosystem.config.js --only new-service

# Save PM2 configuration
pm2 save

# Verify it's running
pm2 status
pm2 logs new-service --lines 50

# Test the service
curl http://localhost:3003/health  # Adjust URL as needed
curl https://service.yourdomain.com/health
```

#### 10. Update Firewall (if using custom ports)
```bash
# Usually not needed if proxying through Nginx
# But if you need direct access to the service port:
sudo ufw allow 3003/tcp
```

---

## 🔧 Advanced Configurations

### Load Balancing Multiple Instances

If you need to scale a service across multiple ports:

```nginx
upstream scaled_service {
    least_conn;
    server 127.0.0.1:3001 weight=1 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:3002 weight=1 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:3003 weight=1 max_fails=3 fail_timeout=30s;
    
    keepalive 32;  # Keep connections alive
}
```

### Nginx Caching for Better Performance

```nginx
# Add to /etc/nginx/nginx.conf (http block)
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=api_cache:10m max_size=1g inactive=60m;

# In your server block
location /api/public/ {
    proxy_cache api_cache;
    proxy_cache_valid 200 10m;
    proxy_cache_use_stale error timeout http_500 http_502 http_503;
    add_header X-Cache-Status $upstream_cache_status;
    
    proxy_pass http://your_service;
}
```

### WebSocket Support

```nginx
location /ws {
    proxy_pass http://your_service;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    
    # WebSocket timeout
    proxy_read_timeout 86400s;
    proxy_send_timeout 86400s;
}
```

### Custom Error Pages

```nginx
# Create error pages directory
sudo mkdir -p /var/www/errors

# Create custom error page
sudo nano /var/www/errors/50x.html

# In Nginx config
error_page 500 502 503 504 /50x.html;
location = /50x.html {
    root /var/www/errors;
    internal;
}
```

### PM2 Advanced Features

```javascript
// ecosystem.config.js advanced options
{
  name: 'advanced-service',
  script: './dist/main.js',
  
  // Auto-restart on file changes (use with caution in production)
  watch: ['dist'],
  ignore_watch: ['node_modules', 'logs'],
  
  // Environment-specific configs
  env_production: {
    NODE_ENV: 'production',
    PORT: 3001,
  },
  env_staging: {
    NODE_ENV: 'staging',
    PORT: 4001,
  },
  
  // Graceful shutdown
  kill_timeout: 5000,
  wait_ready: true,
  listen_timeout: 10000,
  
  // Resource limits
  max_memory_restart: '500M',
  min_uptime: '10s',
  max_restarts: 10,
  
  // Cron restart (restart at specific time)
  cron_restart: '0 3 * * *',  // 3 AM daily
  
  // Logging
  log_type: 'json',
  merge_logs: true,
  
  // Source map support
  source_map_support: true,
}
```

### MongoDB Connection Pooling

```javascript
// In your application code
const mongoOptions = {
  maxPoolSize: 50,
  minPoolSize: 10,
  maxIdleTimeMS: 30000,
  serverSelectionTimeoutMS: 5000,
  socketTimeoutMS: 45000,
  family: 4, // Use IPv4
};
```

### Environment-Specific Configurations

```bash
# Create separate env files
/opt/apps/service/
  ├── .env.production
  ├── .env.staging
  └── .env.development

# Use with PM2
pm2 start ecosystem.config.js --env production
pm2 start ecosystem.config.js --env staging
```

---

## ❓ Frequently Asked Questions (FAQ)

### General Questions

**Q: Can I use this guide for Ubuntu 22.04 or 20.04?**  
A: While most steps will work, this guide is optimized for Ubuntu 24.04 LTS. Some package versions and configurations may differ in older versions.

**Q: How much does it cost to run this setup?**  
A: VPS hosting typically ranges from $5-20/month depending on resources. You can start with a 2GB RAM / 2 CPU VPS and scale up as needed.

**Q: Can I run this on a different Linux distribution?**  
A: Yes, but you'll need to adjust package manager commands (apt → yum/dnf for RHEL-based distros) and some configuration paths.

**Q: How many microservices can I run on one VPS?**  
A: Depends on your VPS resources and service requirements. A 4GB RAM VPS can typically handle 5-10 lightweight microservices.

### Security Questions

**Q: Is this setup secure enough for production?**  
A: Yes, this guide follows security best practices, but always perform your own security audit and keep systems updated.

**Q: Do I need to disable IPv6?**  
A: Not necessary, but if you're not using it, you can disable it in `/etc/sysctl.conf`.

**Q: Should I use Cloudflare in front of my server?**  
A: Recommended! Cloudflare adds DDoS protection, CDN, and additional security features at no cost.

**Q: How do I handle secrets management?**  
A: For small setups, `.env` files with proper permissions (600) are sufficient. For larger deployments, consider HashiCorp Vault or AWS Secrets Manager.

### Technical Questions

**Q: Why Docker only for databases and not for applications?**  
A: PM2 provides better performance and easier debugging for Node.js apps. Docker adds unnecessary complexity for simple microservices. However, you can containerize apps if preferred.

**Q: Can I use PostgreSQL instead of MongoDB?**  
A: Absolutely! Just replace MongoDB Docker container with PostgreSQL and adjust connection strings.

**Q: How do I handle database migrations?**  
A: Run migrations before restarting services:
```bash
cd /opt/apps/service-name
npm run migrate
pm2 restart service-name
```

**Q: What about zero-downtime deployments?**  
A: PM2's `reload` command provides zero-downtime restarts:
```bash
pm2 reload ecosystem.config.js --only service-name
```

**Q: Can I use this with TypeScript?**  
A: Yes! Build your TypeScript to JavaScript first (`npm run build`), then run the compiled code with PM2.

**Q: How do I handle CORS in production?**  
A: Configure CORS in your application code, not in Nginx. Set `ALLOWED_ORIGINS` in your `.env` file.

### Scaling Questions

**Q: When should I move to Kubernetes?**  
A: When you're managing 15+ microservices, need auto-scaling, or have complex orchestration needs. This setup works great for small to medium deployments.

**Q: Can I add a load balancer?**  
A: Yes! Use external load balancers (AWS ALB, Cloudflare) or run Nginx on multiple servers with a load balancer in front.

**Q: How do I scale MongoDB?**  
A: Start with MongoDB replica sets, then move to sharding when you hit performance limits.

### Backup & Recovery Questions

**Q: How do I restore from backup?**  
A: Extract backup file and restore:
```bash
cd /opt/backups
tar -xzf mongodb-YYYYMMDD_HHMMSS.tar.gz
docker exec -i mongodb mongorestore --uri="mongodb://admin:PASSWORD@localhost:27017" /data/backup/YYYYMMDD_HHMMSS/
```

**Q: Should I backup to the same server?**  
A: No! Always maintain off-site backups. Use cloud storage (S3, Backblaze B2) with the rclone script provided.

**Q: How long should I keep backups?**  
A: Recommended retention: Daily for 7 days, Weekly for 4 weeks, Monthly for 12 months.

### Troubleshooting Questions

**Q: My service keeps restarting in PM2. Why?**  
A: Check logs: `pm2 logs service-name`. Common causes: port conflicts, missing env variables, database connection issues.

**Q: Nginx shows 502 Bad Gateway**  
A: Service is down or unreachable. Check: `pm2 status`, `curl http://localhost:PORT`, and Nginx error logs.

**Q: SSL certificate renewal failed**  
A: Ensure ports 80/443 are open, domain DNS is correct, and Nginx is running. Test with: `sudo certbot renew --dry-run`

**Q: High CPU usage on PM2 processes**  
A: Check for infinite loops, inefficient queries, or too many instances. Reduce instances or optimize code.

---

## ✅ Verification Checklist

### Initial Setup
- [ ] System updated and secured
- [ ] Non-root user created with sudo access
- [ ] SSH keys configured
- [ ] Firewall (UFW) enabled and configured
- [ ] Fail2Ban installed and running
- [ ] Automatic security updates enabled

### Core Dependencies
- [ ] Node.js 20 LTS installed
- [ ] PM2 installed and configured
- [ ] Nginx installed and running
- [ ] Docker installed and running
- [ ] Certbot installed

### Databases
- [ ] MongoDB running in Docker
- [ ] RabbitMQ running in Docker
- [ ] Database users created
- [ ] Ports bound to localhost only

### Application
- [ ] Auth service deployed
- [ ] Environment variables configured
- [ ] Application builds successfully
- [ ] PM2 managing processes
- [ ] PM2 startup script configured

### Nginx & SSL
- [ ] Nginx configuration valid
- [ ] SSL certificates obtained
- [ ] HTTPS working correctly
- [ ] HTTP redirects to HTTPS
- [ ] Rate limiting configured

### Monitoring & Backups
- [ ] Health check script running
- [ ] Disk monitoring configured
- [ ] Log rotation configured
- [ ] Backup script tested
- [ ] Backup schedule configured

### Security
- [ ] Firewall rules correct
- [ ] Database ports not exposed
- [ ] .env files secured (chmod 600)
- [ ] Security headers configured
- [ ] Rate limiting tested

---

## 🎯 Testing Your Setup

### Test Auth Service

**Replace `api.yourdomain.com` with your actual domain in all commands below:**

```bash
# Test locally (from server)
curl http://localhost:3001/api

# Test via Nginx (from server)
curl https://api.yourdomain.com/api

# Test from your local machine
curl https://api.yourdomain.com/api

# Test signup endpoint
curl -X POST https://api.yourdomain.com/api/users/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"Test@123456","name":"Test User"}'

# Test signin endpoint
curl -X POST https://api.yourdomain.com/api/users/signin \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"Test@123456"}' \
  -v

# Test rate limiting (should block after threshold)
for i in {1..10}; do 
  echo "Request $i:"
  curl -w "\nHTTP Status: %{http_code}\n" https://api.yourdomain.com/api/users/signin
  sleep 0.2
done
```

**Expected Responses:**
- ✅ **200/201:** Success
- ⚠️ **429:** Rate limit exceeded (this is good - means protection is working!)
- ❌ **502/503:** Service down - check PM2 logs
- ❌ **SSL Error:** Certificate issue - check Certbot

### Check Services Status
```bash
# PM2 processes
pm2 status
pm2 logs --lines 50

# Docker containers
docker ps
docker stats --no-stream

# Nginx
sudo nginx -t
sudo systemctl status nginx

# Firewall
sudo ufw status verbose

# Fail2Ban
sudo fail2ban-client status
```

### Performance Testing

```bash
# Install Apache Bench (if not already installed)
sudo apt install -y apache2-utils

# Basic load test (1000 requests, 10 concurrent)
ab -n 1000 -c 10 https://api.yourdomain.com/api/

# More aggressive test (10000 requests, 100 concurrent)
ab -n 10000 -c 100 https://api.yourdomain.com/api/

# Test with keepalive connections
ab -k -n 5000 -c 50 https://api.yourdomain.com/api/

# Monitor during test (open in separate terminals)
pm2 monit              # Monitor Node.js processes
docker stats           # Monitor Docker containers
htop                   # Monitor system resources
```

**Understanding Results:**
- **Requests per second:** Higher is better
- **Time per request:** Lower is better
- **Failed requests:** Should be 0 (non-zero indicates issues)
- **Transfer rate:** Network throughput

**💡 Alternative Tools:**
```bash
# Install wrk for better load testing
sudo apt install -y wrk

# Test with wrk (more modern alternative)
wrk -t4 -c100 -d30s https://api.yourdomain.com/api/
# -t4: 4 threads
# -c100: 100 connections
# -d30s: 30 seconds duration
```

---

## 📚 Useful Commands Reference

### PM2 Commands
```bash
pm2 status                          # Show all processes
pm2 logs                            # View logs
pm2 logs auth-service --lines 100  # View specific service
pm2 monit                          # Real-time monitoring
pm2 restart all                    # Restart all services
pm2 reload all                     # Zero-downtime reload
pm2 stop all                       # Stop all services
pm2 delete all                     # Remove all from PM2
pm2 save                           # Save current config
pm2 resurrect                      # Restore saved config
```

### Docker Commands
```bash
docker ps                          # List running containers
docker ps -a                       # List all containers
docker logs mongodb                # View MongoDB logs
docker exec -it mongodb bash       # Access MongoDB shell
docker restart mongodb             # Restart MongoDB
docker stats                       # Resource usage
docker system prune -a             # Clean up
```

### Nginx Commands
```bash
sudo nginx -t                      # Test configuration
sudo systemctl reload nginx        # Reload config
sudo systemctl restart nginx       # Restart Nginx
sudo systemctl status nginx        # Check status
sudo tail -f /var/log/nginx/error.log  # View errors
```

### SSL/Certbot Commands
```bash
sudo certbot certificates          # List certificates
sudo certbot renew                 # Renew all certificates
sudo certbot renew --dry-run       # Test renewal
sudo certbot delete                # Delete certificate
```

---

## 🚨 Troubleshooting Guide

### Service Won't Start
```bash
# Check logs
pm2 logs service-name --lines 100

# Check environment
pm2 env 0

# Check port availability
sudo lsof -i :3001

# Restart service
pm2 restart service-name
```

### Database Connection Issues
```bash
# Check MongoDB
docker logs mongodb
docker exec -it mongodb mongosh -u admin -p

# Test connection
telnet localhost 27017
```

### Nginx Issues
```bash
# Check configuration
sudo nginx -t

# View error log
sudo tail -f /var/log/nginx/error.log

# Check if port 80/443 is available
sudo netstat -tulpn | grep :80
sudo netstat -tulpn | grep :443
```

### High Memory Usage
```bash
# Check memory
free -h

# Check which process is using memory
ps aux --sort=-%mem | head

# Restart high-memory services
pm2 restart <service-name>
```

### Disk Space Issues
```bash
# Check disk usage
df -h

# Find large files
sudo du -ah / | sort -rh | head -n 20

# Clean up
docker system prune -a --volumes
pm2 flush  # Clear PM2 logs
```

---

## 🔐 Security Best Practices Summary

<div align="center">

### Essential Security Checklist

</div>

| # | Security Practice | Status | Priority |
|---|-------------------|--------|----------|
| 1 | Use non-root user for all operations | ☐ | 🔴 Critical |
| 2 | Configure SSH keys, disable password auth | ☐ | 🔴 Critical |
| 3 | Enable UFW firewall | ☐ | 🔴 Critical |
| 4 | Install Fail2Ban for brute force protection | ☐ | 🟡 High |
| 5 | Bind database ports to localhost only | ☐ | 🔴 Critical |
| 6 | Use strong passwords (32+ characters) | ☐ | 🔴 Critical |
| 7 | Secure .env files with `chmod 600` | ☐ | 🔴 Critical |
| 8 | Enable automatic security updates | ☐ | 🟡 High |
| 9 | Configure Nginx security headers | ☐ | 🟡 High |
| 10 | Implement rate limiting | ☐ | 🟡 High |
| 11 | Use HTTPS only (force redirect) | ☐ | 🔴 Critical |
| 12 | Setup automated backups | ☐ | 🟡 High |
| 13 | Configure log monitoring | ☐ | 🟢 Medium |
| 14 | Regular software updates | ☐ | 🟡 High |

**Password Generation:**
```bash
# Generate a strong 32-character password
openssl rand -base64 32

# Generate multiple passwords at once
for i in {1..5}; do openssl rand -base64 32; done
```

---

## 🆘 Common Issues & Solutions

### Issue: "Permission Denied" when deploying

**Solution:**
```bash
# Fix ownership of app directory
sudo chown -R appadmin:appadmin /opt/apps

# Fix .env file permissions
chmod 600 /opt/apps/*/. env
```

### Issue: PM2 process crashes immediately

**Solution:**
```bash
# Check detailed logs
pm2 logs service-name --lines 200

# Common causes:
# 1. Missing environment variables
cat /opt/apps/service-name/.env

# 2. Port already in use
sudo lsof -i :3001

# 3. Database connection failed
docker logs mongodb
```

### Issue: Nginx returns 502 Bad Gateway

**Solution:**
```bash
# Check if service is running
pm2 status

# Check Nginx error logs
sudo tail -f /var/log/nginx/error.log

# Test upstream connection
curl http://localhost:3001/api

# Restart services
pm2 restart all
sudo systemctl restart nginx
```

### Issue: SSL certificate not auto-renewing

**Solution:**
```bash
# Check certbot timer status
sudo systemctl status certbot.timer

# Test renewal manually
sudo certbot renew --dry-run

# Force renewal (if needed)
sudo certbot renew --force-renewal

# Enable timer if disabled
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer
```

### Issue: High memory usage

**Solution:**
```bash
# Identify memory hogs
ps aux --sort=-%mem | head -10

# Reduce PM2 instances if needed
pm2 scale service-name 1

# Set max memory restart in ecosystem.config.js
max_memory_restart: '300M'

# Clear PM2 logs
pm2 flush

# Restart with updated config
pm2 reload ecosystem.config.js
```

### Issue: Disk space full

**Solution:**
```bash
# Check disk usage
df -h

# Find large files
sudo du -h / | sort -rh | head -20

# Clean Docker
docker system prune -a --volumes

# Clean old logs
sudo find /var/log -type f -name "*.log" -mtime +30 -delete

# Clean PM2 logs
pm2 flush

# Clean apt cache
sudo apt clean
sudo apt autoclean
```

---

## 📞 Resources & Documentation

### Official Documentation

| Tool | Documentation | Description |
|------|--------------|-------------|
| **Ubuntu** | [help.ubuntu.com](https://help.ubuntu.com) | Official Ubuntu documentation |
| **Node.js** | [nodejs.org/docs](https://nodejs.org/docs) | Node.js API reference |
| **PM2** | [pm2.keymetrics.io](https://pm2.keymetrics.io) | Process manager documentation |
| **Nginx** | [nginx.org/en/docs](https://nginx.org/en/docs) | Nginx configuration guide |
| **Docker** | [docs.docker.com](https://docs.docker.com) | Docker reference |
| **MongoDB** | [docs.mongodb.com](https://docs.mongodb.com) | MongoDB manual |
| **Let's Encrypt** | [letsencrypt.org/docs](https://letsencrypt.org/docs) | SSL certificate docs |

### Community & Support

- 🔷 **DigitalOcean Tutorials:** [digitalocean.com/community/tutorials](https://www.digitalocean.com/community/tutorials)
- 🔷 **Stack Overflow:** [stackoverflow.com](https://stackoverflow.com) - Tag: `ubuntu`, `nginx`, `pm2`
- 🔷 **Reddit:** [r/selfhosted](https://reddit.com/r/selfhosted), [r/node](https://reddit.com/r/node)
- 🔷 **GitHub Discussions:** Check individual tool repositories

### Recommended Reading

- 📖 [Linux Security Hardening Guide](https://www.cyberciti.biz/tips/linux-security.html)
- 📖 [Nginx Performance Tuning](https://www.nginx.com/blog/tuning-nginx/)
- 📖 [Node.js Production Best Practices](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)
- 📖 [MongoDB Security Checklist](https://docs.mongodb.com/manual/administration/security-checklist/)

---

## 📈 Performance Benchmarks

### Expected Performance (on 2 CPU / 4GB RAM VPS)

| Metric | Value | Notes |
|--------|-------|-------|
| **Concurrent Users** | 500-1000 | With PM2 cluster mode |
| **Response Time** | < 50ms | Simple API calls |
| **Requests/Second** | 2000+ | With Nginx caching |
| **Uptime** | 99.9%+ | With health checks |
| **Memory Usage** | < 60% | Under normal load |

### Optimization Tips

1. **Enable Nginx Caching** for static content
2. **Use PM2 Cluster Mode** to utilize all CPU cores
3. **Optimize MongoDB** indexes and queries
4. **Implement Redis** for session storage and caching
5. **Use CDN** for static assets
6. **Monitor with Grafana** + Prometheus for detailed metrics

---

## 🎓 Learning Resources

### Video Tutorials

- 🎥 [Linux Server Administration](https://www.youtube.com/results?search_query=linux+server+administration)
- 🎥 [Nginx Configuration Tutorial](https://www.youtube.com/results?search_query=nginx+tutorial)
- 🎥 [Docker for Beginners](https://www.youtube.com/results?search_query=docker+tutorial)

### Courses

- 🎓 **Linux Academy** - System Administration
- 🎓 **Udemy** - Complete Node.js Developer Course
- 🎓 **Coursera** - Cloud Computing Specialization

---

## 🔄 Changelog

### Version 2.0 (November 2025)
- ✨ Updated for Ubuntu 24.04 LTS
- ✨ Added Docker Compose for databases
- ✨ Enhanced security configurations
- ✨ Added automated backup scripts
- ✨ Improved monitoring setup
- 🐛 Fixed SSL renewal issues
- 📝 Better documentation with examples

### Version 1.0 (June 2024)
- 🎉 Initial release for Ubuntu 22.04

---

## 📊 Project Stats

<div align="center">

![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)
![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04%20LTS-E95420)

**Setup Time:** ~2-3 hours (first time)  
**Maintenance:** ~30 minutes/week  
**Backup:** Automated daily (2 AM)  
**Updates:** Automated security, manual service updates  
**Support:** Community-driven

</div>

---

<div align="center">

## 🎉 Your Ubuntu 24.04 VPS is Now Production-Ready!

**Thank you for using this guide!**

If you found this helpful, please ⭐ star the repository and share it with others!

---

**Author:** Engr. Raihan Mahamud | **Last Updated:** November 2025  
**Tested On:** Ubuntu 24.04 LTS (Noble Numbat)  
**Status:** ✅ Production Ready | **License:** MIT

[Report Issue](https://github.com/Engr-Raihan/VPS-Hosting-Setup/issues) • [Contribute](https://github.com/Engr-Raihan/VPS-Hosting-Setup/pulls) • [Contact](mailto:engr.raihan.buet@gmail.com)

</div>

