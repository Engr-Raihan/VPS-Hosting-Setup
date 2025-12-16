# 🚀 VPS Setup Checklist

Quick checklist to track your setup progress.

---

## 📋 Pre-Setup

- [ ] VPS with Ubuntu 24.04 LTS
- [ ] Root/sudo access
- [ ] Domain name (if using)
- [ ] DNS configured
- [ ] Password manager ready

---

## 🔐 Phase 1: Security Setup (~30 min)

### System Update
- [ ] Connect via SSH
- [ ] Update packages (`apt update && apt upgrade`)
- [ ] Install essential tools (build-essential, curl, git, ca-certificates, gnupg, lsb-release)
- [ ] Optional: Install wget, vim (if you prefer)
- [ ] Set timezone (UTC or your local timezone)

### User Management
- [ ] Create `appadmin` user
- [ ] Add to sudo group
- [ ] Set strong password

### SSH Security
- [ ] Generate SSH key locally
- [ ] Copy public key to server
- [ ] **TEST SSH key login**
- [ ] Configure SSH settings
- [ ] [ ] Optional: Disable root login
- [ ] [ ] Optional: Disable password auth

### Firewall
- [ ] Enable UFW
- [ ] Allow port 22 (SSH)
- [ ] Allow port 80 (HTTP)
- [ ] Allow port 443 (HTTPS)

### Security Tools
- [ ] Install unattended-upgrades
- [ ] Configure auto-updates
- [ ] Install Fail2Ban
- [ ] Configure Fail2Ban

---

## 📦 Phase 2: Core Dependencies (~20 min)

- [ ] Install Node.js 20 LTS
- [ ] Install PM2
- [ ] Install Nginx
- [ ] Install Docker
- [ ] Install Certbot

---

## 🗄️ Phase 3: Databases (~15 min)

- [ ] Create `/opt/databases` directory
- [ ] Create `docker-compose.yml`
- [ ] Create `.env` with passwords
- [ ] Secure `.env` (chmod 600)
- [ ] Start Docker services
- [ ] Verify MongoDB running
- [ ] Verify RabbitMQ running

### Database Pattern Setup (Choose One or Both):

**Option A: Shared Database (for 3-4 related services)**
- [ ] Create shared database (e.g., `core_shared_db`)
- [ ] Create one shared user for all core services

**Option B: Isolated Databases (for business services)**
- [ ] Create separate database per service
- [ ] Create unique user per service

---

## 🏢 Phase 4: Application Setup (~10 min)

- [ ] Create `/opt/apps` directory
- [ ] Create `/var/log/apps` directory
- [ ] Create `ecosystem.config.js`

---

## 🚀 Phase 5: Deploy Service (~20 min)

- [ ] Clone/upload code
- [ ] Create `.env` file
- [ ] Secure `.env` (chmod 600)
- [ ] Install dependencies
- [ ] Build application
- [ ] Create log directory
- [ ] Start with PM2
- [ ] Verify service running
- [ ] Test local endpoint

---

## 🌐 Phase 6: Nginx (~30 min)

- [ ] Remove default site
- [ ] Create upstream config
- [ ] Create rate-limit config
- [ ] Create site config
- [ ] Enable site
- [ ] Test Nginx config
- [ ] Reload Nginx

---

## 🔒 Phase 7: SSL (~10 min)

- [ ] Verify DNS points to server
- [ ] Run Certbot
- [ ] Test HTTPS access
- [ ] Test auto-renewal (dry-run)

---

## 🔧 Phase 8: Additional Config (~15 min)

- [ ] Configure log rotation
- [ ] Install PM2 logrotate
- [ ] Configure system optimization
- [ ] Create backup script
- [ ] Schedule backup cron job

---

## ✅ Final Verification

### Services
- [ ] PM2 processes online
- [ ] Docker containers running
- [ ] Nginx active
- [ ] UFW enabled
- [ ] Fail2Ban running

### Functionality
- [ ] API responding
- [ ] SSL valid
- [ ] HTTP → HTTPS redirect
- [ ] Rate limiting works
- [ ] Logs being written

### Security
- [ ] SSH keys working
- [ ] Database on localhost only
- [ ] `.env` files chmod 600
- [ ] Firewall rules correct

---

## 📝 Credentials to Store

1. Server IP: _______________
2. SSH key location: _______________
3. MongoDB root password: _______________
4. MongoDB user passwords: _______________
5. RabbitMQ password: _______________
6. JWT secrets: _______________

---

**Setup Time:** ~2 hours  
**Status:** ☐ In Progress  ☐ Completed

✅ **Done!** Your VPS is ready for development.
