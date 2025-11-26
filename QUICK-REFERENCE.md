# 🚀 Quick Reference Guide

**One-page reference for common VPS management tasks**

---

## 🔑 Essential Commands

### SSH Connection
```bash
# Connect to server
ssh appadmin@YOUR_SERVER_IP

# Connect via SSH tunnel (for databases)
ssh -L 15672:localhost:15672 appadmin@YOUR_SERVER_IP
```

### PM2 Process Management
```bash
pm2 status                              # View all processes
pm2 logs                                # View all logs (live)
pm2 logs service-name                   # View specific service logs
pm2 logs service-name --lines 100       # View last 100 lines
pm2 restart service-name                # Restart service
pm2 reload service-name                 # Zero-downtime reload
pm2 stop service-name                   # Stop service
pm2 start ecosystem.config.js           # Start all services
pm2 save                                # Save current config
pm2 monit                               # Real-time monitoring
pm2 flush                               # Clear logs
```

### Docker Management
```bash
docker ps                               # List running containers
docker ps -a                            # List all containers
docker logs mongodb                     # View MongoDB logs
docker logs rabbitmq                    # View RabbitMQ logs
docker exec -it mongodb bash            # Access MongoDB shell
docker restart mongodb                  # Restart MongoDB
docker stats                            # Resource usage
cd /opt/databases && docker compose up -d  # Start all databases
docker compose down                     # Stop all databases
docker system prune -a                  # Clean up (frees space)
```

### Nginx Management
```bash
sudo nginx -t                           # Test configuration
sudo systemctl reload nginx             # Reload config
sudo systemctl restart nginx            # Restart Nginx
sudo systemctl status nginx             # Check status
sudo tail -f /var/log/nginx/error.log   # View error logs
sudo tail -f /var/log/nginx/access.log  # View access logs
```

### SSL Certificates
```bash
sudo certbot certificates               # List certificates
sudo certbot renew                      # Renew all certificates
sudo certbot renew --dry-run            # Test renewal
sudo certbot delete                     # Delete certificate
sudo systemctl status certbot.timer     # Check auto-renewal
```

### Firewall (UFW)
```bash
sudo ufw status                         # Check status
sudo ufw allow 22/tcp                   # Allow port
sudo ufw deny 3000/tcp                  # Deny port
sudo ufw delete allow 3000/tcp          # Remove rule
sudo ufw enable                         # Enable firewall
sudo ufw disable                        # Disable firewall
```

### System Monitoring
```bash
htop                                    # Interactive process viewer
df -h                                   # Disk usage
free -h                                 # Memory usage
top                                     # CPU usage
netstat -tulpn                          # Open ports
sudo lsof -i :3001                      # What's using port 3001
journalctl -xe                          # System logs
journalctl -u nginx                     # Service-specific logs
```

---

## 📂 Important Directories

```
/opt/apps/                              # Application code
/opt/databases/                         # Database Docker setup
/opt/backups/                           # Local backups
/opt/scripts/                           # Maintenance scripts
/var/log/apps/                          # Application logs
/var/log/nginx/                         # Nginx logs
/etc/nginx/sites-available/             # Nginx site configs
/etc/nginx/sites-enabled/               # Enabled sites
/etc/letsencrypt/live/                  # SSL certificates
```

---

## 🔧 Common Tasks

### Deploy New Service
```bash
# 1. Create directory
mkdir -p /opt/apps/new-service && cd /opt/apps/new-service

# 2. Clone code
git clone YOUR_REPO_URL .

# 3. Setup environment
nano .env && chmod 600 .env

# 4. Install & build
npm ci && npm run build

# 5. Add to PM2
nano /opt/apps/ecosystem.config.js
pm2 start ecosystem.config.js --only new-service && pm2 save
```

### Update Existing Service
```bash
# Use the update script
/opt/scripts/update-service.sh service-name

# Or manually:
cd /opt/apps/service-name
git pull
npm ci && npm run build
pm2 reload service-name
```

### Add New Domain/Subdomain
```bash
# 1. Create Nginx config
sudo nano /etc/nginx/sites-available/new-domain

# 2. Enable site
sudo ln -s /etc/nginx/sites-available/new-domain /etc/nginx/sites-enabled/

# 3. Test & reload
sudo nginx -t && sudo systemctl reload nginx

# 4. Get SSL certificate
sudo certbot --nginx -d new-domain.com
```

### Database Backup (Manual)
```bash
/opt/scripts/backup-all.sh
```

### Check Service Health
```bash
# PM2 processes
pm2 status

# Docker containers
docker ps

# Test API endpoints
curl http://localhost:3001/health
curl https://api.yourdomain.com/health

# Check logs
pm2 logs service-name --lines 50
```

---

## 🚨 Troubleshooting Quick Fixes

### Service Won't Start
```bash
pm2 logs service-name --lines 100       # Check errors
pm2 restart service-name                # Force restart
sudo lsof -i :3001                      # Check port conflict
```

### 502 Bad Gateway
```bash
pm2 status                              # Is service running?
curl http://localhost:3001              # Test direct connection
sudo tail -f /var/log/nginx/error.log   # Check Nginx errors
pm2 restart all                         # Restart services
```

### Out of Disk Space
```bash
df -h                                   # Check usage
docker system prune -a                  # Clean Docker
pm2 flush                               # Clear PM2 logs
sudo apt clean                          # Clean apt cache
find /var/log -type f -name "*.log" -mtime +7 -delete  # Old logs
```

### High Memory Usage
```bash
ps aux --sort=-%mem | head -10          # Find memory hogs
pm2 scale service-name 1                # Reduce instances
pm2 restart service-name                # Restart service
```

### SSL Certificate Issues
```bash
sudo certbot renew --force-renewal      # Force renewal
sudo systemctl restart nginx            # Restart Nginx
sudo certbot certificates               # Check status
```

### Database Connection Failed
```bash
docker ps                               # Is MongoDB running?
docker restart mongodb                  # Restart MongoDB
docker logs mongodb                     # Check errors
# Test connection:
docker exec -it mongodb mongosh -u admin -p
```

---

## 🔐 Security Quick Checks

```bash
# Check firewall
sudo ufw status

# Check Fail2Ban
sudo fail2ban-client status

# Check SSH config
sudo sshd -t

# Check open ports
sudo netstat -tulpn

# Check .env permissions (should be 600)
ls -la /opt/apps/*/.env

# Check for updates
sudo apt update && sudo apt list --upgradable
```

---

## 📊 Performance Optimization

```bash
# Monitor real-time
pm2 monit

# Load test
ab -n 1000 -c 10 https://api.yourdomain.com/api/

# Check response times
curl -w "\nTime: %{time_total}s\n" https://api.yourdomain.com/api/

# Database performance
docker exec mongodb mongosh --eval "db.currentOp()"
```

---

## 🔄 Maintenance Schedule

### Daily
- Check PM2 status
- Review error logs
- Monitor disk space

### Weekly
- Review application logs
- Check backup logs
- Update services (if needed)

### Monthly
- System updates (`apt update && apt upgrade`)
- Review security logs
- Test backup restoration
- Review SSL certificates

### Quarterly
- Full security audit
- Performance testing
- Cleanup old backups
- Update documentation

---

## 📞 Emergency Contacts

**Important Numbers to Keep Handy:**
- VPS Provider Support: _______________
- Domain Registrar: _______________
- Team Lead: _______________

**Important URLs:**
- Server IP: _______________
- Domain: _______________
- Monitoring Dashboard: _______________
- Cloud Backup Location: _______________

---

## 💡 Pro Tips

1. **Always** test Nginx config before reloading: `sudo nginx -t`
2. **Always** keep your current SSH session open when testing SSH changes
3. **Always** check logs after deploying: `pm2 logs service-name --lines 50`
4. **Use** `pm2 reload` instead of `pm2 restart` for zero-downtime
5. **Never** commit `.env` files to Git
6. **Backup** before major changes
7. **Test** in staging before production
8. **Monitor** regularly, don't wait for problems

---

## 🎯 Resource Limits (Adjust for Your VPS)

**2GB RAM VPS:**
- PM2 instances per service: 1-2
- Max services: 3-5
- MongoDB max connections: 50

**4GB RAM VPS:**
- PM2 instances per service: 2-4
- Max services: 5-10
- MongoDB max connections: 100

**8GB RAM VPS:**
- PM2 instances per service: 4-8
- Max services: 10-20
- MongoDB max connections: 200

---

## 📝 Quick Notes Section

Use this space for your custom notes:

```
Server IP: _______________
Root Password: (In password manager)
MongoDB Password: (In password manager)
Last Updated: _______________
Next Maintenance: _______________

Custom Ports:
- Service 1: 3001
- Service 2: 3002
- Service 3: _______________

Important Changes:
- _______________
- _______________
- _______________
```

---

**💾 Save this file** and keep it accessible for quick reference!

**🔖 Bookmark** the full guide for detailed instructions.

**📱 Print** this page for physical reference.

