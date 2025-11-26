# 🚀 VPS Setup Checklist

Use this checklist to track your progress through the setup process.

## 📋 Pre-Setup (Before You Begin)

- [ ] VPS purchased with Ubuntu 24.04 LTS installed
- [ ] Root or sudo access credentials available
- [ ] Domain name purchased (if using custom domain)
- [ ] DNS records configured and propagating
- [ ] Local SSH client installed (Terminal, PuTTY, etc.)
- [ ] Text editor ready (VS Code, Notepad++, etc.)
- [ ] Password manager ready for storing credentials

---

## 🔐 Phase 1: Initial Server Setup (~30 minutes)

### System Update
- [ ] Connect to server via SSH
- [ ] Update package lists (`apt update`)
- [ ] Upgrade existing packages (`apt upgrade`)
- [ ] Install essential tools
- [ ] Set correct timezone

### User Management
- [ ] Create non-root user `appadmin`
- [ ] Add user to sudo group
- [ ] Set strong password
- [ ] Configure sudo without password (optional)

### SSH Security
- [ ] Generate SSH key pair on local machine
- [ ] Copy public key to server
- [ ] Test SSH key authentication
- [ ] Configure SSH security settings
- [ ] Disable root login (after testing)
- [ ] Disable password authentication (optional)

### Firewall Setup
- [ ] Enable UFW firewall
- [ ] Allow SSH port 22
- [ ] Allow HTTP port 80
- [ ] Allow HTTPS port 443
- [ ] Verify firewall status

### Security Tools
- [ ] Install unattended-upgrades
- [ ] Configure automatic updates
- [ ] Install Fail2Ban
- [ ] Configure Fail2Ban rules
- [ ] Verify Fail2Ban is running

---

## 📦 Phase 2: Install Core Dependencies (~20 minutes)

### Node.js
- [ ] Add NodeSource repository
- [ ] Install Node.js 20 LTS
- [ ] Verify Node.js version
- [ ] Update npm to latest

### PM2
- [ ] Install PM2 globally
- [ ] Setup PM2 startup script
- [ ] Verify PM2 installation

### Nginx
- [ ] Install Nginx
- [ ] Start Nginx service
- [ ] Enable Nginx on boot
- [ ] Verify Nginx is running

### Docker
- [ ] Install Docker
- [ ] Add user to docker group
- [ ] Start Docker service
- [ ] Install Docker Compose plugin
- [ ] Verify Docker installation

### SSL Tools
- [ ] Install Certbot
- [ ] Install Nginx plugin for Certbot
- [ ] Verify Certbot installation

---

## 🗄️ Phase 3: Setup Databases (~15 minutes)

### Directory Structure
- [ ] Create `/opt/databases` directory
- [ ] Create `/opt/backups` directory
- [ ] Set proper ownership

### Docker Compose
- [ ] Create `docker-compose.yml` file
- [ ] Create `.env` file with credentials
- [ ] Generate strong MongoDB password
- [ ] Generate strong RabbitMQ password
- [ ] Secure `.env` file (chmod 600)

### Start Services
- [ ] Start Docker Compose services
- [ ] Verify MongoDB is running
- [ ] Verify RabbitMQ is running
- [ ] Check Docker logs for errors

### MongoDB Configuration
- [ ] Connect to MongoDB shell
- [ ] Create database users for each service
- [ ] Test database connections
- [ ] Document all credentials securely

### RabbitMQ Verification
- [ ] Check RabbitMQ status
- [ ] Test management UI via SSH tunnel
- [ ] Verify login credentials

---

## 🏢 Phase 4: Application Infrastructure (~10 minutes)

### Directory Setup
- [ ] Create `/opt/apps` directory
- [ ] Create `/var/log/apps` directory
- [ ] Set proper ownership

### PM2 Configuration
- [ ] Create `ecosystem.config.js` file
- [ ] Configure first service
- [ ] Set resource limits
- [ ] Configure logging paths

---

## 🚀 Phase 5: Deploy First Service (~20 minutes)

### Code Deployment
- [ ] Create service directory
- [ ] Clone repository OR upload files
- [ ] Verify files are present

### Environment Configuration
- [ ] Create `.env` file
- [ ] Fill in all environment variables
- [ ] Generate JWT secrets
- [ ] Configure database connection
- [ ] Configure RabbitMQ connection
- [ ] Secure `.env` file (chmod 600)

### Build & Install
- [ ] Install dependencies
- [ ] Build application
- [ ] Install production dependencies only
- [ ] Verify build output

### Start Service
- [ ] Create log directory for service
- [ ] Start service with PM2
- [ ] Save PM2 configuration
- [ ] Verify service is running
- [ ] Check logs for errors
- [ ] Test local API endpoint

---

## 🌐 Phase 6: Configure Nginx (~30 minutes)

### Basic Configuration
- [ ] Remove default Nginx site
- [ ] Create upstream configuration
- [ ] Create rate limiting configuration

### Site Configuration
- [ ] Create site config for service
- [ ] Configure HTTP → HTTPS redirect
- [ ] Configure HTTPS server block
- [ ] Set security headers
- [ ] Configure rate limiting
- [ ] Enable gzip compression

### Enable Site
- [ ] Create symbolic link
- [ ] Test Nginx configuration
- [ ] Reload Nginx
- [ ] Verify no errors

### SSL Certificate
- [ ] Verify DNS points to server
- [ ] Run Certbot for domain
- [ ] Verify certificate obtained
- [ ] Test HTTPS access
- [ ] Test automatic renewal (dry run)
- [ ] Verify certificate auto-renewal configured

---

## 🔒 Phase 7: Security Hardening (~20 minutes)

### System Limits
- [ ] Configure file descriptors
- [ ] Configure process limits

### Kernel Optimization
- [ ] Configure network settings
- [ ] Configure connection handling
- [ ] Enable security features
- [ ] Apply sysctl changes

### Database Security
- [ ] Verify MongoDB bound to localhost
- [ ] Verify RabbitMQ bound to localhost
- [ ] Restart Docker if needed

### Log Rotation
- [ ] Configure logrotate for apps
- [ ] Install PM2 logrotate module
- [ ] Configure PM2 logrotate settings

---

## 📊 Phase 8: Monitoring & Logging (~15 minutes)

### PM2 Monitoring
- [ ] Install PM2 monitoring modules
- [ ] Test PM2 monit command

### Health Checks
- [ ] Create health-check script
- [ ] Make script executable
- [ ] Test health-check manually
- [ ] Schedule with cron

### Disk Monitoring
- [ ] Create disk-monitor script
- [ ] Make script executable
- [ ] Test disk-monitor manually
- [ ] Schedule with cron

---

## 💾 Phase 9: Backup Strategy (~15 minutes)

### Backup Scripts
- [ ] Create backup directories
- [ ] Create backup-all script
- [ ] Update MongoDB password in script
- [ ] Make script executable
- [ ] Test backup manually
- [ ] Verify backup files created

### Schedule Backups
- [ ] Add backup to crontab (2 AM daily)
- [ ] Verify cron job added

### Remote Backups (Optional)
- [ ] Install rclone
- [ ] Configure rclone remote
- [ ] Create cloud backup script
- [ ] Test cloud backup
- [ ] Schedule cloud backup

---

## 🔄 Phase 10: Maintenance Scripts (~10 minutes)

### Update Scripts
- [ ] Create update-service script
- [ ] Make script executable
- [ ] Test update script

### System Maintenance
- [ ] Create system-maintenance script
- [ ] Make script executable
- [ ] Schedule monthly maintenance

---

## ✅ Final Verification

### Service Status
- [ ] All PM2 processes online
- [ ] All Docker containers running
- [ ] Nginx active and running
- [ ] UFW firewall active
- [ ] Fail2Ban running

### Functionality Tests
- [ ] API endpoints responding
- [ ] SSL certificate valid
- [ ] HTTP redirects to HTTPS
- [ ] Rate limiting working
- [ ] Logs being written
- [ ] Health checks running

### Security Verification
- [ ] SSH key-only access working
- [ ] Root login disabled
- [ ] Database ports not externally accessible
- [ ] All .env files have 600 permissions
- [ ] Firewall rules correct
- [ ] Security headers present

### Performance Tests
- [ ] Load testing completed
- [ ] Response times acceptable
- [ ] Memory usage normal
- [ ] CPU usage normal

### Documentation
- [ ] All credentials documented securely
- [ ] Architecture diagram saved
- [ ] Backup procedure documented
- [ ] Recovery procedure documented
- [ ] Contact information updated

---

## 🎉 Post-Setup Tasks

### Monitoring Setup
- [ ] Setup external monitoring (UptimeRobot, etc.)
- [ ] Configure alerts for downtime
- [ ] Setup log aggregation (optional)

### Team Access
- [ ] Add team SSH keys
- [ ] Document access procedures
- [ ] Share credentials securely

### Regular Maintenance
- [ ] Schedule weekly log reviews
- [ ] Schedule monthly updates
- [ ] Schedule quarterly security audits

### Disaster Recovery
- [ ] Test backup restoration
- [ ] Document recovery procedures
- [ ] Setup off-site backup storage
- [ ] Test failover procedures

---

## 📞 Support Checklist

If something goes wrong, check:

- [ ] Service logs: `pm2 logs`
- [ ] Docker logs: `docker logs <container>`
- [ ] Nginx logs: `/var/log/nginx/error.log`
- [ ] System logs: `journalctl -xe`
- [ ] Firewall status: `sudo ufw status`
- [ ] Disk space: `df -h`
- [ ] Memory usage: `free -h`
- [ ] Process status: `htop`

---

## 📝 Notes

**Important Credentials to Store:**
1. Server IP address
2. SSH private key location
3. MongoDB root password
4. MongoDB service user passwords
5. RabbitMQ password
6. JWT secrets
7. Domain registrar credentials
8. Cloud backup credentials

**Keep Track Of:**
- Setup completion date: _______________
- Next maintenance date: _______________
- SSL certificate expiry: _______________
- Backup verification date: _______________

---

**Setup Estimated Time:** 2-3 hours  
**Actual Time Taken:** _______________ hours

**Status:** ☐ In Progress  ☐ Completed  ☐ Verified

---

✅ **Congratulations!** Once all items are checked, your VPS is production-ready!

