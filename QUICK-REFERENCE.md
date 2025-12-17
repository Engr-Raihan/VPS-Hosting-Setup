# 🚀 Quick Reference

Essential commands for VPS management.

**Note:** We use PM2 for Node.js microservices (efficient, low overhead) and Docker only for databases (MongoDB, RabbitMQ).

---

## 🔑 SSH Connection

```bash
# Connect to server
ssh appadmin@YOUR_SERVER_IP

# SSH tunnel for RabbitMQ UI
ssh -L 15672:localhost:15672 appadmin@YOUR_SERVER_IP
```

---

## 📦 PM2 Commands

```bash
pm2 status                          # View all processes
pm2 logs                            # Live logs
pm2 logs service-name --lines 100  # Specific service
pm2 restart service-name            # Restart
pm2 reload service-name             # Zero-downtime reload
pm2 stop service-name               # Stop
pm2 save                            # Save config
pm2 monit                           # Monitor
pm2 flush                           # Clear logs
```

---

## 🐳 Docker Commands

**Note:** Docker runs MongoDB/RabbitMQ (shared by all microservices). Microservices run via PM2.

```bash
docker ps                           # Running containers
docker logs mongodb                 # View logs
docker restart mongodb              # Restart
docker stats                        # Resource usage

# Start all databases
cd /opt/databases && docker compose up -d

# Stop all databases (WARNING: stops all microservices' DB access!)
docker compose down

# Connect to MongoDB (serves all microservices)
docker exec -it mongodb mongosh -u admin -p

# View all databases (shared + isolated)
docker exec -it mongodb mongosh -u admin -p --eval "show dbs"

# Check which services use which database
docker exec -it mongodb mongosh -u admin -p --eval "
  use core_shared_db;
  print('Shared DB for Auth/User/Profile');
  use order_service_db;
  print('Isolated DB for Order Service');
"

docker system prune -a              # Clean up
```

---

## 🌐 Nginx Commands

```bash
sudo nginx -t                       # Test config
sudo systemctl reload nginx         # Reload
sudo systemctl restart nginx        # Restart
sudo tail -f /var/log/nginx/error.log  # Error logs
sudo tail -f /var/log/nginx/access.log # Access logs
```

---

## 🔒 SSL Commands

```bash
sudo certbot certificates           # List certs
sudo certbot renew                  # Renew all
sudo certbot renew --dry-run        # Test renewal
```

---

## 🛡️ Firewall Commands

```bash
sudo ufw status                     # Check status
sudo ufw allow 22/tcp               # Allow port
sudo ufw deny 3000/tcp              # Deny port
```

---

## 📊 System Monitoring

```bash
htop                                # Process viewer
df -h                               # Disk usage
free -h                             # Memory usage
netstat -tulpn                      # Open ports
sudo lsof -i :3001                  # What's using port
```

---

## 📂 Important Directories

```
/opt/apps/                    # Application code
/opt/databases/               # Database setup
/opt/backups/                 # Backups
/opt/scripts/                 # Scripts
/var/log/apps/                # App logs
/var/log/nginx/               # Nginx logs
/etc/nginx/sites-available/   # Nginx configs
```

---

## 🗄️ Database Architecture Patterns

### Your Setup (Hybrid):

```
One MongoDB Container:
├─ core_shared_db (Auth, User, Profile share this)
├─ order_service_db (isolated)
├─ payment_service_db (isolated)
└─ inventory_service_db (isolated)
```

### Connection Strings:

**Shared Database Services:**
```bash
# Auth, User, Profile all use SAME connection:
MONGODB_URI=mongodb://core_services_user:PASSWORD@localhost:27017/core_shared_db?authSource=core_shared_db
```

**Isolated Database Services:**
```bash
# Each service has UNIQUE connection:
# Order:
MONGODB_URI=mongodb://order_user:PASSWORD1@localhost:27017/order_service_db?authSource=order_service_db

# Payment:
MONGODB_URI=mongodb://payment_user:PASSWORD2@localhost:27017/payment_service_db?authSource=payment_service_db
```

---

## 🔧 Common Tasks

### Update Service

```bash
cd /opt/apps/service-name
git pull
npm ci && npm run build
pm2 reload service-name
```

### Check Health

```bash
pm2 status
docker ps
curl http://localhost:3001/health
```

### Manual Backup

```bash
/opt/scripts/backup-mongodb.sh
```

---

## 🚨 Troubleshooting

### Service Won't Start

```bash
pm2 logs service-name --lines 100
pm2 restart service-name
sudo lsof -i :3001
```

### 502 Bad Gateway

```bash
pm2 status
curl http://localhost:3001
sudo tail -f /var/log/nginx/error.log
```

### Database Connection Failed

```bash
docker ps
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

## 🔐 Security Checks

```bash
sudo ufw status                     # Firewall
sudo fail2ban-client status         # Fail2Ban
ls -la /opt/apps/*/.env             # Check permissions
sudo apt list --upgradable          # Pending updates
```

---

## 💡 Pro Tips

1. Always test Nginx: `sudo nginx -t`
2. Keep SSH session open when testing SSH changes
3. Check logs after deploy: `pm2 logs service-name`
4. Use `pm2 reload` for zero-downtime
5. Never commit `.env` files
6. Backup before major changes

---

## 📝 Quick Notes

```
Server IP: _______________
MongoDB Password: (in password manager)
Last Updated: _______________

Custom Ports:
- Service 1: 3001
- Service 2: 3002
```

---

---

## 👥 Restricted Service Users

**Give team members access to ONLY their specific service:**

```bash
# Create restricted user (example: attendance-admin)
sudo adduser attendance-admin  # No sudo group!

# Setup directories
sudo mkdir -p /opt/apps/attendance-system
sudo chown -R attendance-admin:attendance-admin /opt/apps/attendance-system

# Configure limited sudo access
sudo visudo -f /etc/sudoers.d/attendance-admin
```

**Allow only PM2 commands for their service:**
```
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 restart attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 reload attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 stop attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 start attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 logs attendance-system*
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 status
```

**Manager commands:**
```bash
# Update service
cd /opt/apps/attendance-system
git pull && npm ci && npm run build
sudo -u appadmin pm2 reload attendance-system

# View logs
sudo -u appadmin pm2 logs attendance-system
```

📖 **Full guide:** See README.md "Creating Restricted Service Users"

---

**💾 Bookmark this for quick access!**
