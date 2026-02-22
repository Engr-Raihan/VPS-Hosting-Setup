# 🚀 VPS Hosting Setup - Dev Guide

**Simplified Ubuntu 24.04 LTS VPS setup for development**

---

## 📖 About

**Target OS:** Ubuntu 24.04 LTS  
**Purpose:** Quick VPS setup for Node.js microservices development  
**Estimated Time:** 1-2 hours

### 🏗️ Architecture Decision: Why PM2 Instead of Docker for Microservices?

**For Node.js microservices on VPS, we use PM2 instead of Docker because:**

✅ **Lower Overhead**
- No container runtime overhead
- Direct Node.js process execution
- Lower memory footprint (~30-40% less RAM usage)

✅ **Better Performance**
- No network bridge overhead
- Faster startup times
- Direct access to system resources

✅ **Easier Debugging**
- Direct access to logs (`pm2 logs`)
- No container layers to navigate
- Simple process monitoring

✅ **Simpler Management**
- One command to restart: `pm2 reload service-name`
- Built-in cluster mode for load balancing
- Zero-downtime deployments out of the box

✅ **Cost Effective**
- Can run more services on the same VPS
- Better resource utilization

**We DO use Docker for:**
- **Databases** (MongoDB, RabbitMQ) - Easy version management, isolation, and updates
- **Stateful services** - Where isolation and data persistence matter

**When to consider Docker for microservices:**
- Multi-server orchestration (Kubernetes)
- Complex non-Node.js dependencies
- Need strict isolation between services
- 15+ microservices requiring complex orchestration

**For small to medium VPS setups (1-10 services), PM2 is the optimal choice!**

### 📊 Quick Comparison

| Feature | PM2 | Docker |
|---------|-----|--------|
| **Memory per service** | ~50-100 MB | ~150-300 MB |
| **Startup time** | < 1 second | 2-5 seconds |
| **Zero-downtime reload** | ✅ Built-in | ⚠️ Requires orchestration |
| **CPU overhead** | Minimal | 5-10% per container |
| **Debugging** | Easy (direct logs) | Harder (container layers) |
| **Cluster mode** | ✅ Native | Requires setup |
| **Best for VPS** | ✅ Yes (1-10 services) | ❌ Overkill |
| **Best for Cloud/K8s** | ❌ Limited | ✅ Yes (10+ services) |

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
15. [Deploying Multiple Microservices](#-deploying-multiple-microservices-sharing-resources)
16. [Decision Guide: Shared vs Isolated Database](#-decision-guide-shared-vs-isolated-database)
17. [Quick Commands Reference](#-quick-commands-reference)
18. [Troubleshooting](#-troubleshooting)
19. [Verification Checklist](#-verification-checklist)

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
# ESSENTIAL (required for Node.js, Git, SSL):
sudo apt install -y build-essential curl git ca-certificates gnupg lsb-release

# OPTIONAL (can install later if needed):
# wget - Alternative to curl (usually not needed)
# vim - Text editor (use nano if you prefer, already installed)
# software-properties-common - For add-apt-repository (not needed here)
# apt-transport-https - Already included in Ubuntu 24.04

# Set timezone (use your timezone: Asia/Dhaka, America/New_York, etc.)
sudo timedatectl set-timezone UTC

# Verify
timedatectl
```

**💡 Minimal Installation (if you want bare minimum):**
```bash
# Absolutely required for this setup:
sudo apt install -y build-essential curl git ca-certificates gnupg lsb-release
```

**What each package does:**
- `build-essential` - ✅ **Required** - Compiles native Node.js modules (many npm packages need this)
- `curl` - ✅ **Required** - Downloads Node.js, Docker installation scripts
- `git` - ✅ **Required** - Clone repositories, deploy code
- `ca-certificates` - ✅ **Required** - SSL certificates for HTTPS connections
- `gnupg` - ✅ **Required** - Verify package signatures (security)
- `lsb-release` - ✅ **Required** - Used by Node.js and Docker installers
- `wget` - ⚠️ Optional - Alternative to curl (not needed if you have curl)
- `vim` - ⚠️ Optional - Text editor (nano is already installed, use what you prefer)
- `software-properties-common` - ⚠️ Optional - Not needed for this setup
- `apt-transport-https` - ⚠️ Optional - Already included in Ubuntu 24.04

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

**Note:** We use Docker ONLY for databases and stateful services (MongoDB, RabbitMQ, Redis, etc.), NOT for Node.js microservices. PM2 handles microservices more efficiently.

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

### 🤔 Docker vs Native Installation for Shared MongoDB?

**Question:** If 3-4 microservices share MongoDB, should it be in Docker or installed natively?

**Answer:** ✅ **Use Docker** - Here's why:

| Feature | Docker MongoDB | Native MongoDB |
|---------|----------------|----------------|
| **Version management** | ✅ Easy (change image tag) | ❌ Manual upgrade process |
| **Isolation** | ✅ Isolated from host | ❌ Affects system packages |
| **Backup/Restore** | ✅ Simple (volume copy) | ⚠️ Complex procedures |
| **Multiple versions** | ✅ Can run 2+ versions | ❌ Only one version |
| **Port binding** | ✅ Easy (localhost only) | ⚠️ Requires config |
| **Cleanup** | ✅ `docker compose down` | ❌ Manual uninstall |
| **Resource usage** | ~200-300 MB | ~150-250 MB |
| **Startup time** | < 5 seconds | < 3 seconds |

**Verdict:** Docker wins! The small overhead (~50MB extra RAM) is worth it for the flexibility and ease of management.

**How it works with multiple microservices:**
```
One MongoDB Docker Container
├── Database 1 (auth_service_db) → Microservice 1 (PM2)
├── Database 2 (user_service_db) → Microservice 2 (PM2)
├── Database 3 (order_service_db) → Microservice 3 (PM2)
└── Database 4 (payment_service_db) → Microservice 4 (PM2)

All microservices connect to: localhost:27017
Each has its own database + user inside the same MongoDB instance
```

### Create Directory Structure

```bash
# Create directories
sudo mkdir -p /opt/databases/{mongodb,rabbitmq,postgresql}
sudo mkdir -p /opt/backups/{mongodb,rabbitmq,postgresql}

# Create subdirectories for each service
sudo mkdir -p /opt/databases/mongodb/{data,logs}
sudo mkdir -p /opt/databases/rabbitmq/{data,logs}
sudo mkdir -p /opt/databases/postgresql/{data,logs,config}

# Set ownership (user will own the directories)
sudo chown -R $USER:$USER /opt/databases /opt/backups

# Set initial permissions for PostgreSQL log directory
# PostgreSQL Alpine image typically uses UID 999 or varies, we'll fix it after container starts
sudo chmod 755 /opt/databases/postgresql/logs

cd /opt/databases
```

**💡 Note:** PostgreSQL log directory permissions will be fine-tuned after the container starts. The postgres user in the container needs write access, which we'll set based on the actual UID/GID used by the container.

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

  postgresql:
    image: postgres:16-alpine
    container_name: postgresql
    restart: unless-stopped
    ports:
      - "127.0.0.1:5432:5432"  # Only bind to localhost
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - ./postgresql/data:/var/lib/postgresql/data
      - ./postgresql/logs:/var/log/postgresql
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
POSTGRES_USER=postgres
POSTGRES_PASSWORD=YOUR_SECURE_POSTGRESQL_PASSWORD_HERE
POSTGRES_DB=postgres
```

**Secure and start:**

```bash
# Secure .env file
chmod 600 .env

# Start services
docker compose up -d

# Fix PostgreSQL log directory permissions (postgres user needs write access)
# Get the actual UID/GID from the container
if docker ps | grep -q postgresql; then
  POSTGRES_UID=$(docker exec postgresql id -u postgres 2>/dev/null || echo "999")
  POSTGRES_GID=$(docker exec postgresql id -g postgres 2>/dev/null || echo "999")
  sudo chown -R ${POSTGRES_UID}:${POSTGRES_GID} /opt/databases/postgresql/logs
  sudo chmod 755 /opt/databases/postgresql/logs
  echo "✅ PostgreSQL log directory permissions set (UID: ${POSTGRES_UID}, GID: ${POSTGRES_GID})"
else
  # If container is not running, use data directory ownership (most reliable)
  if [ -d "/opt/databases/postgresql/data" ]; then
    EXISTING_UID=$(stat -c '%u' /opt/databases/postgresql/data 2>/dev/null || echo "999")
    EXISTING_GID=$(stat -c '%g' /opt/databases/postgresql/data 2>/dev/null || echo "999")
    sudo chown -R ${EXISTING_UID}:${EXISTING_GID} /opt/databases/postgresql/logs
    sudo chmod 755 /opt/databases/postgresql/logs
    echo "✅ PostgreSQL log directory permissions set from data directory (UID: ${EXISTING_UID}, GID: ${EXISTING_GID})"
  fi
fi

# Restart PostgreSQL to apply permissions
docker compose restart postgresql

# Verify all services are running
docker compose ps

# Check logs for each service
docker logs mongodb | tail -10
docker logs rabbitmq | tail -10
docker logs postgresql | tail -10

# Test PostgreSQL connection
docker exec postgresql psql -U postgres -d postgres -c "SELECT version();" 2>/dev/null && echo "✅ PostgreSQL is running" || echo "⚠️ PostgreSQL connection failed - check logs"

# Test MongoDB connection
docker exec mongodb mongosh --eval "db.version()" --quiet 2>/dev/null && echo "✅ MongoDB is running" || echo "⚠️ MongoDB connection failed - check logs"

# Test RabbitMQ connection
docker exec rabbitmq rabbitmq-diagnostics ping 2>/dev/null && echo "✅ RabbitMQ is running" || echo "⚠️ RabbitMQ connection failed - check logs"
```

**⚠️ If PostgreSQL is restarting continuously:**

```bash
# Check for permission errors
docker logs postgresql | grep -i "permission denied"

# Fix: Stop container first (can't exec into restarting container)
cd /opt/databases
docker compose stop postgresql

# Get UID from existing data directory (most reliable)
EXISTING_UID=$(stat -c '%u' /opt/databases/postgresql/data 2>/dev/null || echo "999")
EXISTING_GID=$(stat -c '%g' /opt/databases/postgresql/data 2>/dev/null || echo "999")
sudo chown -R ${EXISTING_UID}:${EXISTING_GID} /opt/databases/postgresql/logs
sudo chmod 755 /opt/databases/postgresql/logs

# Start again
docker compose up -d postgresql

# Wait and verify
sleep 5
docker compose ps
docker logs postgresql | tail -20
```

### Configure MongoDB

**✨ Best Practice: One MongoDB Instance, Multiple Databases**

When sharing MongoDB across 3-4 microservices:
- ✅ **One MongoDB Docker container**
- ✅ **One database per microservice** (inside that MongoDB)
- ✅ **Separate user per database** (security isolation)
- ✅ **All microservices connect to `localhost:27017`**

**Two Architecture Patterns Supported:**

**Pattern 1: Shared Database (Tightly Coupled Services)**
```
┌─────────────────────────────────────────────┐
│  🐳 Docker: MongoDB Container               │
│  Port: 127.0.0.1:27017 (localhost only)     │
├─────────────────────────────────────────────┤
│  📁 core_shared_db (user: core_user)        │ ← SHARED
│     ├─ users collection                     │
│     ├─ auth_tokens collection               │
│     ├─ profiles collection                  │
│     └─ permissions collection               │
└─────────────────────────────────────────────┘
         ↑          ↑          ↑
         │          │          │
    ┌────┘    ┌─────┘    ┌─────┘
    │         │          │
┌───┴───┐ ┌──┴────┐ ┌───┴────┐
│PM2    │ │PM2    │ │PM2     │
│Auth   │ │User   │ │Profile │ ← 3-4 services
│:3001  │ │:3002  │ │:3003   │   same DB
└───────┘ └───────┘ └────────┘
```

**Pattern 2: Isolated Databases (Business Services)**
```
┌─────────────────────────────────────────────┐
│  🐳 Docker: MongoDB Container               │
├─────────────────────────────────────────────┤
│  📁 order_service_db (user: order_user)     │ ← ISOLATED
│  📁 payment_service_db (user: payment_user) │ ← ISOLATED
│  📁 inventory_db (user: inventory_user)     │ ← ISOLATED
└─────────────────────────────────────────────┘
         ↑          ↑          ↑
         │          │          │
    ┌────┘    ┌─────┘    ┌─────┘
    │         │          │
┌───┴───┐ ┌──┴────┐ ┌───┴────┐
│PM2    │ │PM2    │ │PM2     │
│Order  │ │Payment│ │Inventory│ ← Each service
│:3004  │ │:3005  │ │:3006    │   own DB
└───────┘ └───────┘ └─────────┘
```

**Combined Architecture (Your Use Case):**
```
┌────────────────────────────────────────────────────┐
│       🐳 Docker: MongoDB Container                 │
│       Port: 127.0.0.1:27017 (localhost only)       │
├────────────────────────────────────────────────────┤
│  📁 core_shared_db (SHARED by 3-4 services)        │
│     ├─ users, auth_tokens, profiles, etc.          │
│                                                     │
│  📁 order_service_db (ISOLATED)                    │
│  📁 payment_service_db (ISOLATED)                  │
│  📁 inventory_service_db (ISOLATED)                │
│  📁 analytics_service_db (ISOLATED)                │
└────────────────────────────────────────────────────┘
```

**Why This Hybrid Works:**
- **Shared DB for core services:** Less data duplication, easier to maintain
- **Isolated DBs for business:** Clear boundaries, independent scaling
- **One MongoDB serves all:** Resource efficient (~200-300MB total)
- **Easy backup:** Single MongoDB backup includes everything

**Create Database Users (Choose Your Pattern):**

```bash
# Connect to MongoDB
docker exec -it mongodb mongosh -u admin -p 'YOUR_MONGO_ROOT_PASSWORD' --authenticationDatabase admin
```

**Option A: Shared Database for 3-4 Related Services (e.g., Auth, User, Profile)**

```javascript
// Create ONE shared database for tightly coupled services
use core_shared_db

// Create ONE shared user that all 3-4 services will use
db.createUser({
  user: "core_services_user",
  pwd: "GENERATE_SECURE_PASSWORD_1",
  roles: [ { role: "readWrite", db: "core_shared_db" } ]
})

// All 3-4 services will connect using SAME credentials
// They share collections: users, auth_tokens, profiles, permissions, etc.

// Example collections structure:
db.createCollection("users")
db.createCollection("auth_tokens")
db.createCollection("profiles")
db.createCollection("permissions")
```

**Option B: Isolated Databases for Business Services**

```javascript
// Create separate database for Order Service
use order_service_db
db.createUser({
  user: "order_service_user",
  pwd: "GENERATE_SECURE_PASSWORD_2",
  roles: [ { role: "readWrite", db: "order_service_db" } ]
})

// Create separate database for Payment Service
use payment_service_db
db.createUser({
  user: "payment_service_user",
  pwd: "GENERATE_SECURE_PASSWORD_3",
  roles: [ { role: "readWrite", db: "payment_service_db" } ]
})

// Create separate database for Inventory Service
use inventory_service_db
db.createUser({
  user: "inventory_service_user",
  pwd: "GENERATE_SECURE_PASSWORD_4",
  roles: [ { role: "readWrite", db: "inventory_service_db" } ]
})

// Each business service has its own isolated database + user
```

**Combined Setup (Hybrid Approach - Recommended):**

```javascript
// 1. Shared database for core services (Auth, User, Profile)
use core_shared_db
db.createUser({
  user: "core_services_user",
  pwd: "GENERATE_PASSWORD_1",
  roles: [ { role: "readWrite", db: "core_shared_db" } ]
})

// 2. Isolated databases for business services
use order_service_db
db.createUser({
  user: "order_service_user",
  pwd: "GENERATE_PASSWORD_2",
  roles: [ { role: "readWrite", db: "order_service_db" } ]
})

use payment_service_db
db.createUser({
  user: "payment_service_user",
  pwd: "GENERATE_PASSWORD_3",
  roles: [ { role: "readWrite", db: "payment_service_db" } ]
})

// Verify all users
use admin
db.system.users.find().pretty()

// Exit
exit
```

**💡 Pro Tip:** Generate unique passwords for each database user:
```bash
# Generate 4 different passwords at once
for i in {1..4}; do echo "Password $i:"; openssl rand -base64 32; echo ""; done
```

---


## Step 10: Setup Application Infrastructure

```bash
# Create directories
sudo mkdir -p /opt/apps
sudo mkdir -p /var/log/apps
sudo chown -R $USER:$USER /opt/apps /var/log/apps

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

**Production .env Examples:**

**Pattern A: Services Sharing the Same Database (e.g., Auth, User, Profile services)**

```env
# Auth Service (.env)
NODE_ENV=production
PORT=3001

# Connects to SHARED database with SAME credentials
MONGODB_URI=mongodb://core_services_user:SHARED_PASSWORD@localhost:27017/core_shared_db?authSource=core_shared_db

JWT_SECRET=YOUR_JWT_SECRET_HERE
RABBIT_QUEUE_NAME=auth-queue  # Each service gets unique queue name
```

```env
# User Service (.env)
NODE_ENV=production
PORT=3002

# Same database, same credentials as Auth Service
MONGODB_URI=mongodb://core_services_user:SHARED_PASSWORD@localhost:27017/core_shared_db?authSource=core_shared_db

RABBIT_QUEUE_NAME=user-queue  # Different queue
```

```env
# Profile Service (.env)
NODE_ENV=production
PORT=3003

# Same database, same credentials as Auth & User services
MONGODB_URI=mongodb://core_services_user:SHARED_PASSWORD@localhost:27017/core_shared_db?authSource=core_shared_db

RABBIT_QUEUE_NAME=profile-queue  # Different queue
```

**Pattern B: Business Services with Isolated Databases**

```env
# Order Service (.env)
NODE_ENV=production
PORT=3004

# ISOLATED database with UNIQUE credentials
MONGODB_URI=mongodb://order_service_user:UNIQUE_PASSWORD_1@localhost:27017/order_service_db?authSource=order_service_db

RABBIT_QUEUE_NAME=order-queue
```

```env
# Payment Service (.env)
NODE_ENV=production
PORT=3005

# ISOLATED database with UNIQUE credentials
MONGODB_URI=mongodb://payment_service_user:UNIQUE_PASSWORD_2@localhost:27017/payment_service_db?authSource=payment_service_db

RABBIT_QUEUE_NAME=payment-queue
```

```env
# Inventory Service (.env)
NODE_ENV=production
PORT=3006

# ISOLATED database with UNIQUE credentials
MONGODB_URI=mongodb://inventory_service_user:UNIQUE_PASSWORD_3@localhost:27017/inventory_service_db?authSource=inventory_service_db

RABBIT_QUEUE_NAME=inventory-queue
```

**🔑 Key Differences:**

| Aspect | Shared Database | Isolated Database |
|--------|----------------|-------------------|
| **User/Password** | Same for all 3-4 services | Unique per service |
| **Database Name** | Same (e.g., `core_shared_db`) | Different per service |
| **Collections** | Shared (users, auth, etc.) | Service-specific |
| **Use Case** | Tightly coupled services | Independent business logic |
| **Example** | Auth + User + Profile | Orders, Payments, Inventory |

**All services share:**
- ✅ Same MongoDB Docker container
- ✅ Same RabbitMQ (different queue names)
- ✅ Connect to `localhost:27017`

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
sudo chown -R $USER:$USER /var/log/apps/auth-service

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

## 🔄 Deploying Multiple Microservices (Sharing Resources)

### Hybrid Architecture (Your Use Case)

```
                    🌐 Nginx Reverse Proxy
                            │
        ┌───────┬───────────┼───────────┬───────────┬──────────┐
        │       │           │           │           │          │
    ┌───▼───┐┌──▼────┐┌────▼────┐┌────▼─────┐┌────▼────┐┌────▼────┐
    │ PM2   ││ PM2   ││ PM2     ││ PM2      ││ PM2     ││ PM2     │
    │ Auth  ││ User  ││ Profile ││ Order    ││ Payment ││Inventory│
    │ :3001 ││ :3002 ││ :3003   ││ :3004    ││ :3005   ││ :3006   │
    └───┬───┘└───┬───┘└────┬────┘└────┬─────┘└────┬────┘└────┬────┘
        │        │         │           │           │          │
        └────────┴─────────┘           │           │          │
               │                       │           │          │
        SHARED DATABASE         ISOLATED│    ISOLATED   ISOLATED
               │                       │           │          │
        ┌──────▼──────────────────────────────────────────────▼──┐
        │            🐳 Docker MongoDB Container                  │
        ├─────────────────────────────────────────────────────────┤
        │  📁 core_shared_db                                      │
        │     ├─ users (shared by Auth, User, Profile)            │
        │     ├─ auth_tokens (shared)                             │
        │     └─ profiles (shared)                                │
        │                                                          │
        │  📁 order_service_db (isolated - only Order service)    │
        │  📁 payment_service_db (isolated - only Payment)        │
        │  📁 inventory_service_db (isolated - only Inventory)    │
        ├─────────────────────────────────────────────────────────┤
        │  🐰 RabbitMQ (shared by all with different queues)      │
        └─────────────────────────────────────────────────────────┘
```

**Architecture Benefits:**
- ✅ **Core services share DB:** Auth, User, Profile use same collections (no duplication)
- ✅ **Business services isolated:** Each has own database (clear boundaries)
- ✅ **One MongoDB container:** Serves both patterns efficiently
- ✅ **Resource efficient:** ~250MB for entire MongoDB (all databases)

### Step-by-Step: Adding More Microservices

**🔀 Choose Your Pattern First:**

---

**OPTION A: Add to Shared Database Group (e.g., another core service)**

**1. No new database needed! Use existing shared database:**
```bash
# Already exists: core_shared_db with core_services_user
# Skip database creation step
```

**2. Deploy the new service:**
```bash
mkdir -p /opt/apps/notification-service && cd /opt/apps/notification-service
git clone YOUR_REPO_URL .
nano .env
```

```env
NODE_ENV=production
PORT=3007  # Unique port

# Use SAME credentials as other core services
MONGODB_URI=mongodb://core_services_user:SAME_PASSWORD@localhost:27017/core_shared_db?authSource=core_shared_db

# Shared RabbitMQ
RABBIT_USERNAME=admin
RABBIT_PASSWORD=RABBITMQ_PASSWORD
RABBIT_URL=localhost:5672
RABBIT_QUEUE_NAME=notification-queue  # Unique queue
```

**When to use:** Service is tightly coupled to existing core services (Auth, User, Profile)

---

**OPTION B: Add as Isolated Business Service (recommended for business logic)**

**1. Create new isolated database:**
```bash
# Connect to MongoDB
docker exec -it mongodb mongosh -u admin -p 'YOUR_PASSWORD' --authenticationDatabase admin

# Create NEW database with NEW user
use new_business_service_db
db.createUser({
  user: "new_business_service_user",
  pwd: "GENERATE_UNIQUE_PASSWORD",
  roles: [ { role: "readWrite", db: "new_business_service_db" } ]
})
exit
```

**2. Deploy the new service:**
```bash
mkdir -p /opt/apps/analytics-service && cd /opt/apps/analytics-service
git clone YOUR_REPO_URL .
nano .env
```

```env
NODE_ENV=production
PORT=3008  # Unique port

# ISOLATED database with UNIQUE credentials
MONGODB_URI=mongodb://analytics_service_user:UNIQUE_PASSWORD@localhost:27017/analytics_service_db?authSource=analytics_service_db

# Shared RabbitMQ
RABBIT_USERNAME=admin
RABBIT_PASSWORD=RABBITMQ_PASSWORD
RABBIT_URL=localhost:5672
RABBIT_QUEUE_NAME=analytics-queue  # Unique queue
```

**When to use:** Independent business logic service (Orders, Payments, Analytics, etc.)

---

**3. Add to PM2 ecosystem:**
```bash
nano /opt/apps/ecosystem.config.js
```

Add new service:
```javascript
{
  name: 'new-service',
  cwd: '/opt/apps/new-service',
  script: './dist/main.js',
  instances: 2,
  exec_mode: 'cluster',
  autorestart: true,
  max_memory_restart: '500M',
  env: {
    NODE_ENV: 'production',
    PORT: 3004,
  },
  error_file: '/var/log/apps/new-service/pm2-error.log',
  out_file: '/var/log/apps/new-service/pm2-out.log',
  merge_logs: true,
}
```

**4. Build and start:**
```bash
# Build
npm ci && npm run build

# Create logs
sudo mkdir -p /var/log/apps/new-service
sudo chown appadmin:appadmin /var/log/apps/new-service

# Start
cd /opt/apps
pm2 start ecosystem.config.js --only new-service
pm2 save
```

**5. Configure Nginx:**
```bash
# Add to upstream.conf
sudo nano /etc/nginx/conf.d/upstream.conf
```

Add:
```nginx
upstream new_service {
    least_conn;
    server 127.0.0.1:3004 max_fails=3 fail_timeout=30s;
}
```

**6. Create site config and get SSL:**
```bash
sudo nano /etc/nginx/sites-available/new-service
# (Add Nginx config similar to auth-api)

sudo ln -s /etc/nginx/sites-available/new-service /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx

sudo certbot --nginx -d newservice.yourdomain.com
```

### 📊 Resource Usage Example (6 Microservices - Hybrid Setup)

On a **4GB RAM VPS**:

```
Docker (Databases):
├─ MongoDB:    ~250 MB ← Serves ALL 6 services
│  ├─ core_shared_db (3 services share this)
│  ├─ order_service_db (isolated)
│  ├─ payment_service_db (isolated)
│  └─ inventory_service_db (isolated)
└─ RabbitMQ:   ~150 MB ← Shared by all (6 different queues)

PM2 (Microservices):
├─ Auth Service (2 instances):      ~150 MB ┐
├─ User Service (2 instances):      ~150 MB ├─ Share core_shared_db
├─ Profile Service (2 instances):   ~150 MB ┘
├─ Order Service (2 instances):     ~150 MB ← Isolated DB
├─ Payment Service (2 instances):   ~150 MB ← Isolated DB
└─ Inventory Service (2 instances): ~150 MB ← Isolated DB

System + Nginx + Others:            ~600 MB
─────────────────────────────────────────────
Total:                             ~1.9 GB

Available for growth:              ~2.1 GB ✅
```

**Architecture Summary:**

```
📦 Shared Database Pattern:
   └─ core_shared_db (1 database, 1 user)
      ├─ Used by: Auth Service
      ├─ Used by: User Service  
      └─ Used by: Profile Service

📦 Isolated Database Pattern:
   ├─ order_service_db (Order Service only)
   ├─ payment_service_db (Payment Service only)
   └─ inventory_service_db (Inventory Service only)
```

**Key Benefits:**
- ✅ **One MongoDB** serves 6 services (1 shared + 3 isolated databases)
- ✅ **Core services** share data (no duplication of users/auth)
- ✅ **Business services** isolated (independent scaling/changes)
- ✅ **One RabbitMQ** serves all (6 different queue names)
- ✅ Can scale to **10-12 services** on same 4GB VPS

**When to add more services:**
- **Add to shared DB:** Services that need access to users/auth data
- **Add isolated DB:** Services with independent business logic

---

## 🤔 Decision Guide: Shared vs Isolated Database

### Use SHARED Database When:

✅ **Services are tightly coupled**
- Auth, User Management, Profile services
- Need to share users, authentication, permissions
- Same domain/bounded context

✅ **Data consistency is critical**
- Changes in one service immediately visible to others
- No data duplication
- Simple ACID transactions

✅ **Examples:**
```
Shared: core_shared_db
├─ Auth Service (handles login)
├─ User Service (manages user data)
├─ Profile Service (user profiles)
└─ Notification Service (sends emails)
All need access to same users table
```

### Use ISOLATED Database When:

✅ **Services are independent**
- Business logic services (Orders, Payments, Inventory)
- Different bounded contexts
- Each service owns its data

✅ **Scalability matters**
- Can scale database per service independently
- Different performance requirements
- Potential for separate database servers later

✅ **Security/isolation required**
- Financial data (Payments)
- Sensitive business data (Analytics)
- Separate backup/restore needs

✅ **Examples:**
```
Isolated: order_service_db
└─ Order Service (only one that needs orders data)

Isolated: payment_service_db
└─ Payment Service (sensitive financial data)

Isolated: analytics_service_db
└─ Analytics Service (heavy read operations)
```

### Quick Decision Table:

| Question | Shared DB | Isolated DB |
|----------|-----------|-------------|
| Need to access users table? | ✅ Yes | ❌ No |
| Same business domain? | ✅ Yes | ❌ No |
| Tight coupling acceptable? | ✅ Yes | ❌ No |
| Need to scale independently? | ❌ No | ✅ Yes |
| Sensitive/isolated data? | ❌ No | ✅ Yes |
| Different performance needs? | ❌ No | ✅ Yes |

**💡 Pro Tip:** Start with shared database for core services, use isolated databases for everything else. You can always split a shared database later if needed.

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

---

## 🔐 Creating Restricted Service Users (Optional)

### Use Case: Give team members access to ONLY their specific service

If you want to allow someone (e.g., manager, developer) to manage only ONE service without access to others:

### Step 1: Create Restricted User

```bash
# Create user for attendance-system manager
# This will prompt for password - set a strong one
sudo adduser attendance-admin
# Enter password when prompted
# Skip optional info (press Enter for all)

# DO NOT add to sudo group (no admin privileges)
```

### Step 2: Setup SSH Keys for New User

**On manager's local machine (Windows):**

```powershell
# Generate SSH key (PowerShell or CMD)
ssh-keygen -t ed25519 -C "manager@company.com"

# Just press Enter for all prompts (uses defaults):
# - File location: Press Enter (uses default)
# - Passphrase: Press Enter twice (no passphrase) or type one for security

# Copy public key to clipboard
type $env:USERPROFILE\.ssh\id_ed25519.pub | clip

# Or view and copy manually
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

**On manager's local machine (Linux/Mac):**

```bash
# Generate SSH key
ssh-keygen -t ed25519 -C "manager@company.com"

# Press Enter for all prompts (uses defaults)

# Copy public key to clipboard (Mac)
cat ~/.ssh/id_ed25519.pub | pbcopy

# Or view and copy manually (Linux/Mac)
cat ~/.ssh/id_ed25519.pub
```

**On VPS (as appadmin):**

```bash
# Switch to new user
sudo su - attendance-admin

# Create SSH directory
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Add manager's public key
nano ~/.ssh/authorized_keys
# Paste the public key, save and exit

# Set permissions
chmod 600 ~/.ssh/authorized_keys

# Exit back to appadmin
exit
```

### Step 3: Setup Service Directory Permissions

```bash
# Create service directory if not exists
sudo mkdir -p /opt/apps/attendance-system
sudo mkdir -p /var/log/apps/attendance-system

# Give ownership to restricted user
sudo chown -R attendance-admin:attendance-admin /opt/apps/attendance-system
sudo chown -R attendance-admin:attendance-admin /var/log/apps/attendance-system

# Verify permissions
ls -la /opt/apps/ | grep attendance
```

### Step 4: Configure Sudo Permissions

**Configure sudo to allow attendance-admin to manage ONLY their service:**

```bash
# As appadmin, create sudoers file for attendance-admin
sudo visudo -f /etc/sudoers.d/attendance-admin
```

**Add these lines:**

```bash
# Allow attendance-admin to manage ONLY their service
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 restart attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 reload attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 stop attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 start attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 logs attendance-system*
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 status

# Note: Only the above commands are allowed. All other sudo commands are automatically denied.
```

**Save and exit** (Ctrl+X, Y, Enter in nano)

**Test the configuration:**
```bash
# Switch to attendance-admin
sudo su - attendance-admin

# Try allowed command (should work)
sudo -u appadmin pm2 status

# Try forbidden command (should fail)
sudo apt update
# Error: Sorry, user attendance-admin is not allowed to execute...

exit
```

**⚠️ Security Note about `pm2 status`:**

The `pm2 status` command shows information about ALL services. While attendance-admin **cannot control** other services, they **can see** information about them (names, ports, status).

**For maximum security (external contractors):**
```bash
# Edit sudoers again
sudo visudo -f /etc/sudoers.d/attendance-admin

# Remove the pm2 status line:
# attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 status

# Keep only these:
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 restart attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 reload attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 stop attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 start attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 logs attendance-system*
```

---

### Step 5: Configure PM2 for Restricted User

**🎯 RECOMMENDED: Option A - Shared PM2 with Sudo Access (Simpler)**

**Skip separate PM2 installation! Just use the system-wide PM2 installed by appadmin.**

```bash
# PM2 is already installed system-wide by appadmin
# attendance-admin will use it via sudo commands

# NO need to install PM2 separately!
# We configured sudo permissions in Step 4
```

**attendance-admin will manage their service with:**
```bash
sudo -u appadmin pm2 restart attendance-system
sudo -u appadmin pm2 reload attendance-system
sudo -u appadmin pm2 stop attendance-system
sudo -u appadmin pm2 start attendance-system
sudo -u appadmin pm2 logs attendance-system
sudo -u appadmin pm2 status
```

**Benefits:**
- ✅ Single PM2 instance (saves ~50MB RAM)
- ✅ All services visible in one `pm2 list`
- ✅ Easier monitoring for admin
- ✅ Simpler setup

**Continue to Step 6 →**

---

**Option B: Separate PM2 Instance (Advanced - Only if Needed)**

**⚠️ Only use this if you need complete PM2 isolation**

```bash
# As attendance-admin user
sudo su - attendance-admin

# Install PM2 for this user only (creates separate instance)
npm install -g pm2

# Create ecosystem file for this service only
cd /opt/apps/attendance-system
nano ecosystem.config.js
```

**ecosystem.config.js (for attendance-admin user):**

```javascript
module.exports = {
  apps: [
    {
      name: 'attendance-system',
      cwd: '/opt/apps/attendance-system',
      script: './dist/main.js',
      instances: 2,
      exec_mode: 'cluster',
      autorestart: true,
      watch: false,
      max_memory_restart: '500M',
      env: {
        NODE_ENV: 'production',
        PORT: 3010,
      },
      error_file: '/var/log/apps/attendance-system/pm2-error.log',
      out_file: '/var/log/apps/attendance-system/pm2-out.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      merge_logs: true,
    },
  ],
};
```

```bash
# Setup PM2 startup for attendance-admin user
pm2 startup systemd -u attendance-admin --hp /home/attendance-admin

# Start the service
pm2 start ecosystem.config.js
pm2 save

# Exit back to appadmin
exit
```

**With Option B (separate PM2):**
- attendance-admin runs: `pm2 restart attendance-system` (no sudo needed)
- But creates separate PM2 daemon
- Uses more memory (~50MB extra)

---

### Step 6: Setup Git Access (Choose One Method)

**🎯 RECOMMENDED: Option 1 - Deploy Key (Single Repo Access Only)**

**Best for: Restricting access to ONLY attendance-system repo**

```bash
# As attendance-admin
sudo su - attendance-admin

# Generate deploy key (read-only by default)
ssh-keygen -t ed25519 -C "deploy-attendance-system" -f ~/.ssh/deploy_attendance

# Display public key
cat ~/.ssh/deploy_attendance.pub
# Copy this key

# Configure Git to use deploy key
nano ~/.ssh/config
```

**Add to SSH config:**
```
Host github.com-attendance
    HostName github.com
    User git
    IdentityFile ~/.ssh/deploy_attendance
    IdentitiesOnly yes
```

```bash
chmod 600 ~/.ssh/config

# Configure Git
git config --global user.name "Attendance Service"
git config --global user.email "attendance@yourcompany.com"

exit
```

**On GitHub (as company admin):**
1. Go to: `https://github.com/your-company/attendance-system`
2. Settings → Deploy keys → Add deploy key
3. Title: `VPS Production Server`
4. Paste the public key from `deploy_attendance.pub`
5. ✅ Check "Allow write access" (if needed for deployments)
6. Click "Add key"

**Clone using deploy key:**
```bash
sudo su - attendance-admin
cd /opt/apps/attendance-system

# Use custom host from SSH config
git clone git@github.com-attendance:your-company/attendance-system.git .

# Test
git pull

exit
```

**✅ Benefits:**
- Only has access to attendance-system repo
- Cannot access other company repos
- Can be revoked anytime from GitHub settings
- Read-only by default (safer)

---

**Option 2 - Manager's Personal GitHub Account**

**Best for: Manager owns/maintains the service independently**

**Setup:**
```bash
# Manager creates fork or separate repo on their GitHub
# Repository: github.com/manager-username/attendance-system

# As attendance-admin (on VPS)
sudo su - attendance-admin

# Generate SSH key
ssh-keygen -t ed25519 -C "manager@company.com" -f ~/.ssh/github_key

cat ~/.ssh/github_key.pub
# Manager adds this to their GitHub account: Settings → SSH Keys

# Configure SSH
nano ~/.ssh/config
```

**SSH config:**
```
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/github_key
```

```bash
chmod 600 ~/.ssh/config

# Configure Git
git config --global user.name "Manager Name"
git config --global user.email "manager@company.com"

# Clone from manager's GitHub
cd /opt/apps/attendance-system
git clone git@github.com:manager-username/attendance-system.git .

exit
```

**✅ Benefits:**
- Manager has full control
- No company GitHub access needed
- Manager can manage their own repo
- Independent deployment

**⚠️ Considerations:**
- Manager leaves = Need to transfer repo
- Code is outside company GitHub org
- May complicate code ownership

---

**Option 3 - Personal Access Token (PAT)**

**Best for: HTTPS access with limited scope**

**On GitHub (as manager or admin):**
1. Settings → Developer settings → Personal access tokens → Tokens (classic)
2. Generate new token
3. Name: `Attendance System VPS`
4. Scopes: Select only `repo` (or just `public_repo` if public)
5. Click "Generate token"
6. Copy token immediately (won't see again!)

**On VPS:**
```bash
sudo su - attendance-admin
cd /opt/apps/attendance-system

# Clone using HTTPS with token
git clone https://TOKEN@github.com/your-company/attendance-system.git .

# Configure Git credential storage
git config --global credential.helper store

# Or use environment variable
echo "export GH_TOKEN=your_token_here" >> ~/.bashrc
source ~/.bashrc

exit
```

**✅ Benefits:**
- Easy to revoke
- Can limit permissions
- HTTPS (no SSH setup needed)

**⚠️ Considerations:**
- Token stored on server
- Need to rotate periodically

---

**🎯 Decision Guide:**

| Scenario | Best Option |
|----------|-------------|
| Manager works for company, needs limited access | **Deploy Key** (Option 1) ⭐ |
| Manager is contractor, maintains independently | **Manager's GitHub** (Option 2) |
| Simple HTTPS setup, easy revocation | **PAT Token** (Option 3) |
| Need write access for multiple people | **GitHub Team** with specific repo access |

---

**🔒 Security Best Practices:**

**With Deploy Key (Recommended):**
```bash
# Each service gets its own deploy key
/opt/apps/attendance-system → deploy_attendance key → Only attendance repo
/opt/apps/inventory-system → deploy_inventory key → Only inventory repo
/opt/apps/payment-system → deploy_payment key → Only payment repo
```

**Advantages:**
- ✅ Breached server = Only one repo compromised
- ✅ Easy to revoke one without affecting others
- ✅ Clear audit trail per service
- ✅ Read-only by default

---

### Step 7: Create Helper Scripts for Manager

```bash
# As appadmin, create helper scripts
sudo nano /home/attendance-admin/update-service.sh
```

**update-service.sh:**

```bash
#!/bin/bash

echo "🔄 Updating Attendance System..."

cd /opt/apps/attendance-system

# Pull latest code
echo "📥 Pulling latest code from Git..."
git pull origin main

if [ $? -ne 0 ]; then
    echo "❌ Git pull failed!"
    exit 1
fi

# Install dependencies
echo "📦 Installing dependencies..."
npm ci --production=false

# Build
echo "🔨 Building application..."
npm run build

if [ $? -ne 0 ]; then
    echo "❌ Build failed!"
    exit 1
fi

# Install production dependencies only
echo "🎯 Installing production dependencies..."
rm -rf node_modules
npm ci --production

# Restart with PM2
echo "🔄 Restarting service..."
if [ -f "ecosystem.config.js" ]; then
    # Option A: Own PM2 instance
    pm2 reload ecosystem.config.js --only attendance-system
else
    # Option B: Via sudo to appadmin's PM2
    sudo -u appadmin pm2 reload attendance-system
fi

echo "✅ Attendance System updated successfully!"
pm2 logs attendance-system --lines 20
```

**restart-service.sh:**

```bash
#!/bin/bash

echo "🔄 Restarting Attendance System..."

if [ -f "/opt/apps/attendance-system/ecosystem.config.js" ]; then
    # Option A: Own PM2 instance
    pm2 reload attendance-system
else
    # Option B: Via sudo to appadmin's PM2
    sudo -u appadmin pm2 reload attendance-system
fi

echo "✅ Service restarted!"
pm2 logs attendance-system --lines 20
```

**view-logs.sh:**

```bash
#!/bin/bash

echo "📋 Viewing Attendance System Logs..."
echo "Press Ctrl+C to exit"
echo ""

if [ -f "/opt/apps/attendance-system/ecosystem.config.js" ]; then
    pm2 logs attendance-system
else
    sudo -u appadmin pm2 logs attendance-system
fi
```

```bash
# Make scripts executable
sudo chmod +x /home/attendance-admin/*.sh
sudo chown attendance-admin:attendance-admin /home/attendance-admin/*.sh
```

### Step 8: Restrict SSH Access (Optional but Recommended)

```bash
# Edit SSH config
sudo nano /etc/ssh/sshd_config
```

**Add at the bottom:**

```bash
# Restrict attendance-admin user
Match User attendance-admin
    # Restrict to specific directory
    ChrootDirectory none
    # Allow only specific commands (if using forced commands)
    ForceCommand /bin/bash --restricted
    # Disable port forwarding
    AllowTcpForwarding no
    X11Forwarding no
    # Disable agent forwarding
    AllowAgentForwarding no
    # Allow only these specific things
    PermitTTY yes
```

```bash
# Test config
sudo sshd -t

# Restart SSH
sudo systemctl restart ssh
```

### Step 9: Test Restricted Access

**From manager's machine:**

```bash
# Test SSH connection
ssh -i ~/.ssh/id_ed25519_attendance attendance-admin@YOUR_SERVER_IP

# Once connected, test allowed operations:

# ✅ Should work:
cd /opt/apps/attendance-system
ls -la
git pull
~/update-service.sh
~/restart-service.sh
~/view-logs.sh

# ❌ Should NOT work (no access):
cd /opt/apps/auth-service  # Permission denied
cd /opt/apps/order-service  # Permission denied
sudo apt update  # Not allowed
pm2 restart auth-service  # Not allowed
```

### What Manager CAN Do:

✅ **SSH into the server** (with their key)
✅ **Access `/opt/apps/attendance-system/`** (their service only)
✅ **Git pull** (update code from repository)
✅ **Install dependencies** (npm install in their directory)
✅ **Build the application** (npm run build)
✅ **Restart their service** (PM2 reload via script)
✅ **View logs** (PM2 logs for their service)
✅ **Edit .env file** (in their service directory)
✅ **View service status** (PM2 status)

### What Manager CANNOT Do:

❌ **Access other services** (no permission)
❌ **Modify other services** (no access)
❌ **Install system packages** (no sudo access)
❌ **Change system configuration** (no sudo)
❌ **Access database directly** (no MongoDB admin)
❌ **View other service logs** (no permission)
❌ **Restart other services** (PM2 restricted)
❌ **Modify Nginx configs** (no access)

---

## 📋 Manager Access Summary

**Connection:**
```bash
ssh attendance-admin@YOUR_SERVER_IP
```

**Daily Operations:**
```bash
# Update service (pull + build + restart)
~/update-service.sh

# Quick restart
~/restart-service.sh

# View logs
~/view-logs.sh

# Manual operations
cd /opt/apps/attendance-system
git pull
npm ci && npm run build
pm2 reload attendance-system
```

**Security Benefits:**
- ✅ Manager has access only to their service
- ✅ Cannot break other services
- ✅ Cannot access sensitive system files
- ✅ Cannot modify database directly
- ✅ All actions are logged
- ✅ Can be revoked anytime by deleting user

---

**Setup Complete! 🎉**

Your VPS is ready for development. Remember to keep your system updated and monitor logs regularly.
