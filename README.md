# 🚀 VPS Hosting Setup - Dev Guide

**Simplified Ubuntu 24.04 LTS VPS setup for development**

---

## 📖 About

**Target OS:** Ubuntu 24.04 LTS  
**Purpose:** Quick VPS setup for Node.js microservices development  
**Estimated Time:** 1-2 hours

---

## 📋 Table of Contents

1. [Update System](#step-1-update-system)
2. [Create Non-Root User](#step-2-create-non-root-user)
3. [Setup SSH Keys](#step-3-setup-ssh-keys)
4. [Configure SSH Security](#step-4-configure-ssh-security)
5. [Configure Firewall](#step-5-configure-firewall)
6. [Configure Automatic Security Updates](#step-6-configure-automatic-security-updates)
7. [Install Fail2Ban](#step-7-install-fail2ban)
8. [Install Core Dependencies](#step-8-install-core-dependencies)
9. [Install Databases & Message Queues](#step-9-install-databases--message-queues)
10. [Setup Application Infrastructure](#step-10-setup-application-infrastructure)
11. [Deploy Your First Service](#step-11-deploy-your-first-service)
12. [Configure Nginx](#step-12-configure-nginx)
13. [Setup SSL](#step-13-setup-ssl)
14. [Additional Configurations](#step-14-additional-configurations)

---

## ⚠️ Important Notes

- **SSH service name in Ubuntu 24.04:** Use `ssh` not `sshd`
  ```bash
  sudo systemctl restart ssh  # ✅ CORRECT
  sudo systemctl restart sshd # ❌ WRONG
  ```
- Always test SSH changes in a new terminal before closing your current session
- Replace `YOUR_SERVER_IP` and `yourdomain.com` with your actual values

---

## Step 1: Update System

```bash
# Update package lists and upgrade
sudo apt update && sudo apt upgrade -y

# Install essential tools
sudo apt install -y build-essential curl wget git vim software-properties-common apt-transport-https ca-certificates gnupg lsb-release

# Set timezone
sudo timedatectl set-timezone UTC

# Verify
timedatectl
```

---

## Step 2: Create Non-Root User

```bash
# Create user
sudo adduser --disabled-password --gecos "" appadmin
sudo usermod -aG sudo appadmin

# Set password
sudo passwd appadmin

# Optional: Configure sudo without password
echo "appadmin ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/appadmin
```

---

## Step 3: Setup SSH Keys

**⚠️ CRITICAL: Complete this step and test before proceeding!**

### On Your Local Machine (Windows):

```powershell
# Generate SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy to server (replace YOUR_SERVER_IP)
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh appadmin@YOUR_SERVER_IP "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"

# Test connection (should NOT ask for password)
ssh appadmin@YOUR_SERVER_IP
```

### On Your Local Machine (Linux/Mac):

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy to server
ssh-copy-id appadmin@YOUR_SERVER_IP

# Test connection
ssh appadmin@YOUR_SERVER_IP
```

**✅ CHECKPOINT: Make sure SSH key login works before continuing!**

---

## Step 4: Configure SSH Security

**⚠️ Only proceed if Step 3 works!**

```bash
# Backup original config
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Edit SSH config
sudo nano /etc/ssh/sshd_config
```

**Edit these lines:**

```bash
# Add/modify these settings:
AllowUsers appadmin
PermitEmptyPasswords no
MaxAuthTries 3
LoginGraceTime 60
```

**Test and restart:**

```bash
# Test configuration
sudo sshd -t

# Restart SSH (Ubuntu 24.04 uses 'ssh' not 'sshd')
sudo systemctl restart ssh

# Open NEW terminal and test connection before closing current one!
```

### Step 4b: Disable Root Login (Optional)

**⚠️ Only after SSH keys work perfectly!**

```bash
sudo nano /etc/ssh/sshd_config

# Change this line:
PermitRootLogin no

# Test and restart
sudo sshd -t
sudo systemctl restart ssh
```

### Step 4c: Disable Password Authentication (Optional)

**⚠️ Maximum security - only if SSH keys have been working for days!**

```bash
sudo nano /etc/ssh/sshd_config

# Change this line:
PasswordAuthentication no

# Test and restart
sudo sshd -t
sudo systemctl restart ssh
```

---

## Step 5: Configure Firewall

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

# Enable logging
sudo ufw logging on
```

---

## Step 6: Configure Automatic Security Updates

```bash
# Install unattended-upgrades
sudo apt install -y unattended-upgrades

# Configure
sudo dpkg-reconfigure -plow unattended-upgrades
```

---

## Step 7: Install Fail2Ban

```bash
# Install
sudo apt install -y fail2ban

# Create local configuration
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# Edit configuration
sudo nano /etc/fail2ban/jail.local
```

**Add these settings:**

```ini
[sshd]
enabled = true
port = 22
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
```

**Start Fail2Ban:**

```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban

# Check status
sudo fail2ban-client status
```

---

## Step 8: Install Core Dependencies

### Install Node.js 20 LTS

```bash
# Add repository
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install
sudo apt install -y nodejs

# Verify
node --version  # Should show v20.x.x
npm --version

# Update npm
sudo npm install -g npm@latest
```

### Install PM2

```bash
# Install globally
sudo npm install -g pm2

# Setup startup script
pm2 startup systemd -u appadmin --hp /home/appadmin

# Verify
pm2 --version
```

### Install Nginx

```bash
# Install
sudo apt install -y nginx

# Start and enable
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify
nginx -v
sudo systemctl status nginx
```

### Install Docker

```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker appadmin

# Start and enable
sudo systemctl start docker
sudo systemctl enable docker

# Install Docker Compose
sudo apt install -y docker-compose-plugin

# Verify
docker --version
docker compose version
```

### Install Certbot

```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Verify
certbot --version
```

---

## Step 9: Install Databases & Message Queues

### Create Directory Structure

```bash
# Create directories
sudo mkdir -p /opt/databases/{mongodb,rabbitmq}
sudo mkdir -p /opt/backups/{mongodb,rabbitmq}
sudo chown -R appadmin:appadmin /opt/databases /opt/backups

cd /opt/databases
```

### Create Docker Compose Configuration

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
      - "127.0.0.1:5672:5672"    # AMQP
      - "127.0.0.1:15672:15672"  # Management UI
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
```

### Create Environment File

```bash
nano .env
```

**.env:**

```env
# Generate passwords with: openssl rand -base64 32
MONGO_ROOT_PASSWORD=YOUR_SECURE_MONGODB_PASSWORD_HERE
RABBIT_USER=admin
RABBIT_PASSWORD=YOUR_SECURE_RABBITMQ_PASSWORD_HERE
```

**Secure and start:**

```bash
# Secure .env file
chmod 600 .env

# Start services
docker compose up -d

# Verify
docker compose ps
docker logs mongodb
docker logs rabbitmq
```

### Configure MongoDB

```bash
# Connect to MongoDB
docker exec -it mongodb mongosh -u admin -p 'YOUR_MONGO_ROOT_PASSWORD' --authenticationDatabase admin

# Create database users (in MongoDB shell)
use auth_service_db
db.createUser({
  user: "auth_service_user",
  pwd: "GENERATE_SECURE_PASSWORD",
  roles: [ { role: "readWrite", db: "auth_service_db" } ]
})

# Exit
exit
```

---

## Step 10: Setup Application Infrastructure

```bash
# Create directories
sudo mkdir -p /opt/apps
sudo mkdir -p /var/log/apps
sudo chown -R appadmin:appadmin /opt/apps /var/log/apps

# Create PM2 ecosystem file
cd /opt/apps
nano ecosystem.config.js
```

**ecosystem.config.js:**

```javascript
module.exports = {
  apps: [
    {
      name: 'auth-service',
      cwd: '/opt/apps/auth-service',
      script: './dist/main.js',
      instances: 2,
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
  ],
};
```

---

## Step 11: Deploy Your First Service

### Clone Your Repository

```bash
cd /opt/apps
mkdir auth-service && cd auth-service

# Option 1: Clone from Git
git clone https://github.com/your-username/your-repo.git .

# Option 2: Upload via SCP (from local machine)
# scp -r ./your-service/* appadmin@YOUR_SERVER_IP:/opt/apps/auth-service/
```

### Setup Environment

```bash
# Create .env file
nano .env
```

**Production .env:**

```env
NODE_ENV=production
PORT=3001

# Database
MONGODB_URI=mongodb://auth_service_user:YOUR_PASSWORD@localhost:27017/auth_service_db?authSource=auth_service_db

# JWT (generate with: openssl rand -base64 48)
JWT_SECRET=YOUR_JWT_SECRET_HERE
JWT_EXPIRES_IN=3600
JWT_REFRESH_EXPIRES_IN=604800

# RabbitMQ
RABBIT_USERNAME=admin
RABBIT_PASSWORD=YOUR_RABBITMQ_PASSWORD
RABBIT_URL=localhost:5672
RABBIT_QUEUE_NAME=auth-queue

# Domain
COOKIE_DOMAIN=yourdomain.com
ALLOWED_ORIGINS=https://yourdomain.com,https://api.yourdomain.com
```

**Secure the file:**

```bash
chmod 600 .env
```

### Build and Start

```bash
# Install dependencies
npm ci --production=false

# Build
npm run build

# Install production dependencies only
rm -rf node_modules
npm ci --production

# Create log directory
sudo mkdir -p /var/log/apps/auth-service
sudo chown -R appadmin:appadmin /var/log/apps/auth-service

# Start with PM2
cd /opt/apps
pm2 start ecosystem.config.js --only auth-service
pm2 save

# Verify
pm2 status
pm2 logs auth-service --lines 50

# Test locally
curl http://localhost:3001/api
```

---

## Step 12: Configure Nginx

### Remove Default Site

```bash
sudo rm /etc/nginx/sites-enabled/default
```

### Create Upstream Configuration

```bash
sudo mkdir -p /etc/nginx/conf.d
sudo nano /etc/nginx/conf.d/upstream.conf
```

**upstream.conf:**

```nginx
# Auth Service Upstream
upstream auth_service {
    least_conn;
    server 127.0.0.1:3001 max_fails=3 fail_timeout=30s;
}
```

### Create Rate Limiting Configuration

```bash
sudo nano /etc/nginx/conf.d/rate-limit.conf
```

**rate-limit.conf:**

```nginx
# Rate limit zones
limit_req_zone $binary_remote_addr zone=general_limit:10m rate=100r/s;
limit_req_zone $binary_remote_addr zone=auth_limit:10m rate=5r/s;
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=50r/s;

# Connection limit
limit_conn_zone $binary_remote_addr zone=conn_limit:10m;
```

### Create Site Configuration

```bash
sudo nano /etc/nginx/sites-available/auth-api
```

**auth-api (replace `api.yourdomain.com` with your domain):**

```nginx
# HTTP - Redirect to HTTPS
server {
    listen 80;
    listen [::]:80;
    server_name api.yourdomain.com;

    # For Let's Encrypt
    location /.well-known/acme-challenge/ {
        root /var/www/html;
    }

    # Redirect to HTTPS
    location / {
        return 301 https://$server_name$request_uri;
    }
}

# HTTPS
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name api.yourdomain.com;

    # SSL config (added by Certbot)

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Strict-Transport-Security "max-age=31536000" always;

    # Gzip
    gzip on;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_types text/plain text/css application/json application/javascript;

    # Limits
    client_max_body_size 10M;
    limit_conn conn_limit 10;

    # Auth endpoints - strict rate limit
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
    }

    # General API endpoints
    location /api/ {
        limit_req zone=api_limit burst=50 nodelay;

        proxy_pass http://auth_service;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }

    # Health check
    location /health {
        access_log off;
        proxy_pass http://auth_service;
    }

    # Logging
    access_log /var/log/nginx/auth-api.access.log;
    error_log /var/log/nginx/auth-api.error.log warn;
}
```

### Enable Site

```bash
# Create symbolic link
sudo ln -s /etc/nginx/sites-available/auth-api /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload
sudo systemctl reload nginx
```

---

## Step 13: Setup SSL

**Prerequisites:**
- Domain DNS must point to server IP
- Wait 5-10 minutes for DNS propagation
- Port 80 must be open

```bash
# Get SSL certificate (replace with your domain)
sudo certbot --nginx -d api.yourdomain.com

# Test auto-renewal
sudo certbot renew --dry-run

# Check certificate
sudo certbot certificates
```

---

## Step 14: Additional Configurations

### Log Rotation

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

### PM2 Log Rotation

```bash
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 50M
pm2 set pm2-logrotate:retain 14
pm2 set pm2-logrotate:compress true
```

### System Optimization

```bash
sudo nano /etc/sysctl.conf
```

**Add:**

```
# Network Performance
net.core.somaxconn = 65535
net.ipv4.tcp_max_syn_backlog = 8192

# Security
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.rp_filter = 1

# File Descriptors
fs.file-max = 2097152
```

**Apply:**

```bash
sudo sysctl -p
```

### Backup Script

```bash
mkdir -p /opt/scripts
nano /opt/scripts/backup-mongodb.sh
```

**backup-mongodb.sh:**

```bash
#!/bin/bash

BACKUP_DIR="/opt/backups/mongodb"
DATE=$(date +%Y%m%d_%H%M%S)
MONGO_PASSWORD="YOUR_MONGO_ROOT_PASSWORD"

echo "Starting MongoDB backup..."

docker exec mongodb mongodump \
  --uri="mongodb://admin:${MONGO_PASSWORD}@localhost:27017/?authSource=admin" \
  --out=/data/backup/$DATE

docker cp mongodb:/data/backup/$DATE $BACKUP_DIR/
docker exec mongodb rm -rf /data/backup/$DATE

# Compress
cd $BACKUP_DIR
tar -czf mongodb-$DATE.tar.gz $DATE
rm -rf $DATE

# Keep last 7 days
find $BACKUP_DIR -name "*.tar.gz" -mtime +7 -delete

echo "Backup completed: mongodb-$DATE.tar.gz"
```

**Setup:**

```bash
chmod +x /opt/scripts/backup-mongodb.sh

# Test
/opt/scripts/backup-mongodb.sh

# Schedule daily at 2 AM
crontab -e
# Add: 0 2 * * * /opt/scripts/backup-mongodb.sh >> /var/log/apps/backup.log 2>&1
```

---

## 🚀 Quick Commands Reference

### PM2

```bash
pm2 status                    # Check status
pm2 logs service-name         # View logs
pm2 restart service-name      # Restart service
pm2 reload service-name       # Zero-downtime reload
pm2 monit                     # Monitor resources
```

### Docker

```bash
docker ps                     # List containers
docker logs mongodb           # View logs
docker restart mongodb        # Restart container
cd /opt/databases && docker compose up -d  # Start all
```

### Nginx

```bash
sudo nginx -t                 # Test config
sudo systemctl reload nginx   # Reload
sudo tail -f /var/log/nginx/error.log  # View logs
```

### System

```bash
df -h                         # Disk usage
free -h                       # Memory usage
htop                          # Process monitor
sudo ufw status               # Firewall status
```

---

## 🔧 Troubleshooting

### Service Won't Start

```bash
pm2 logs service-name --lines 100
pm2 restart service-name
sudo lsof -i :3001  # Check port
```

### 502 Bad Gateway

```bash
pm2 status  # Is service running?
curl http://localhost:3001  # Test direct
sudo tail -f /var/log/nginx/error.log
```

### Database Connection Failed

```bash
docker ps  # Is MongoDB running?
docker logs mongodb
docker restart mongodb
```

### Out of Disk Space

```bash
df -h
docker system prune -a
pm2 flush
sudo apt clean
```

---

## 📝 Important Notes

**Store These Credentials Securely:**
- Server IP
- SSH private key location
- MongoDB passwords
- RabbitMQ password
- JWT secrets
- Domain credentials

**Regular Maintenance:**
- Check logs weekly
- Update system monthly
- Test backups quarterly
- Review security quarterly

---

## ✅ Verification Checklist

- [ ] System updated and secured
- [ ] SSH keys working
- [ ] Firewall configured
- [ ] Fail2Ban running
- [ ] Node.js & PM2 installed
- [ ] Nginx running
- [ ] Docker containers running
- [ ] Service deployed and accessible
- [ ] SSL certificate obtained
- [ ] HTTPS working
- [ ] Backups scheduled

---

**Setup Complete! 🎉**

Your VPS is ready for development. Remember to keep your system updated and monitor logs regularly.
