# 🔐 Restricted Service User Setup

**Quick guide for creating users with access to only ONE specific service**

---

## 📋 Quick Setup (Copy-Paste Ready)

### For Service: attendance-system

```bash
# 1. Create user (will prompt for password - set a strong one)
sudo adduser attendance-admin
# Enter password, skip optional info (press Enter)

# 2. Create directories and set permissions
sudo mkdir -p /opt/apps/attendance-system
sudo mkdir -p /var/log/apps/attendance-system
sudo chown -R attendance-admin:attendance-admin /opt/apps/attendance-system
sudo chown -R attendance-admin:attendance-admin /var/log/apps/attendance-system

# 3. Manager generates SSH key (on their local machine)
# Windows (PowerShell):
#   ssh-keygen -t ed25519 -C "manager@company.com"  # Press Enter for all prompts
#   type $env:USERPROFILE\.ssh\id_ed25519.pub | clip  # Copies to clipboard
# Linux/Mac:
#   ssh-keygen -t ed25519 -C "manager@company.com"  # Press Enter for all prompts
#   cat ~/.ssh/id_ed25519.pub  # View public key

# 4. Add manager's public key to server
sudo su - attendance-admin
mkdir -p ~/.ssh && chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys  # Paste public key
chmod 600 ~/.ssh/authorized_keys
exit

# 4. Setup Git Access - RECOMMENDED: Deploy Key (Single Repo Only)
sudo su - attendance-admin
# Generate deploy key for attendance-system repo ONLY
ssh-keygen -t ed25519 -C "deploy-attendance" -f ~/.ssh/deploy_attendance
cat ~/.ssh/deploy_attendance.pub  # Add to GitHub repo as Deploy Key

# Configure SSH to use deploy key
echo 'Host github.com-attendance
    HostName github.com
    User git
    IdentityFile ~/.ssh/deploy_attendance
    IdentitiesOnly yes' > ~/.ssh/config
chmod 600 ~/.ssh/config

# Configure Git
git config --global user.name "Attendance Service"
git config --global user.email "attendance@company.com"
exit

# On GitHub: Repo Settings → Deploy keys → Add deploy key
# Paste public key, set title "VPS Production"
# This key can ONLY access attendance-system repo!

# 5. Configure sudo permissions
sudo visudo -f /etc/sudoers.d/attendance-admin
```

**Add to sudoers:**
```
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 restart attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 reload attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 stop attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 start attendance-system
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 logs attendance-system*
attendance-admin ALL=(appadmin) NOPASSWD: /usr/bin/pm2 status
```

```bash
# 6. Create helper scripts
sudo tee /home/attendance-admin/update-service.sh > /dev/null <<'EOF'
#!/bin/bash
echo "🔄 Updating Attendance System..."
cd /opt/apps/attendance-system
git pull origin main || exit 1
npm ci --production=false
npm run build || exit 1
rm -rf node_modules
npm ci --production
sudo -u appadmin pm2 reload attendance-system
echo "✅ Updated successfully!"
pm2 logs attendance-system --lines 20
EOF

sudo tee /home/attendance-admin/restart-service.sh > /dev/null <<'EOF'
#!/bin/bash
echo "🔄 Restarting Attendance System..."
sudo -u appadmin pm2 reload attendance-system
echo "✅ Service restarted!"
pm2 logs attendance-system --lines 20
EOF

sudo tee /home/attendance-admin/view-logs.sh > /dev/null <<'EOF'
#!/bin/bash
echo "📋 Viewing Attendance System Logs..."
sudo -u appadmin pm2 logs attendance-system
EOF

sudo chmod +x /home/attendance-admin/*.sh
sudo chown attendance-admin:attendance-admin /home/attendance-admin/*.sh

# 7. Deploy initial service code
sudo su - attendance-admin
cd /opt/apps/attendance-system
git clone YOUR_REPO_URL .
npm ci && npm run build
rm -rf node_modules && npm ci --production
nano .env  # Add environment variables
chmod 600 .env
exit

# 8. Add to main PM2 ecosystem (as appadmin)
nano /opt/apps/ecosystem.config.js
# Add attendance-system configuration
pm2 start ecosystem.config.js --only attendance-system
pm2 save
```

---

## 📖 Manager Instructions

**Send this to your manager:**

### Connection

```bash
# From your local machine
ssh attendance-admin@SERVER_IP_ADDRESS
```

### Daily Commands

**Update service (pull code + build + restart):**
```bash
~/update-service.sh
```

**Quick restart (no code changes):**
```bash
~/restart-service.sh
```

**View live logs:**
```bash
~/view-logs.sh
# Press Ctrl+C to exit
```

**Check service status:**
```bash
sudo -u appadmin pm2 status
```

**Manual Git operations:**
```bash
cd /opt/apps/attendance-system
git status
git pull origin main
git log --oneline -5
```

**Edit environment variables:**
```bash
cd /opt/apps/attendance-system
nano .env
~/restart-service.sh  # Restart after changes
```

---

## 🔒 Security Summary

### Manager CAN:
✅ Access attendance-system directory
✅ Pull code from Git
✅ Build and deploy their service
✅ Restart their service
✅ View their service logs
✅ Edit their .env file

### Manager CANNOT:
❌ Access other services
❌ Install system packages
❌ Modify system configs
❌ Access database directly
❌ Restart other services
❌ Use sudo for anything else
❌ View other service logs

---

## 🔄 Multiple Restricted Users

**For each additional service:**

```bash
# Replace 'attendance' with your service name
SERVICE_NAME="inventory"
USER_NAME="${SERVICE_NAME}-admin"

sudo adduser $USER_NAME
sudo mkdir -p /opt/apps/$SERVICE_NAME
sudo mkdir -p /var/log/apps/$SERVICE_NAME
sudo chown -R $USER_NAME:$USER_NAME /opt/apps/$SERVICE_NAME
sudo chown -R $USER_NAME:$USER_NAME /var/log/apps/$SERVICE_NAME

# Setup SSH, sudo rules, and scripts (follow steps above)
```

---

## ⚠️ Important Notes

1. **Each user owns ONLY their service directory**
2. **No sudo access** except for PM2 restart of their service
3. **All commands are logged** in system logs
4. **Easy to revoke:** `sudo userdel -r username`
5. **Test restrictions** before giving actual access
6. **Backup important data** before making changes

---

## 🧪 Testing Restrictions

```bash
# As appadmin, test the new user
sudo su - attendance-admin

# Should work:
cd /opt/apps/attendance-system  # ✅
ls -la  # ✅
git pull  # ✅
~/update-service.sh  # ✅

# Should fail:
cd /opt/apps/auth-service  # ❌ Permission denied
sudo apt update  # ❌ Not allowed
pm2 restart auth-service  # ❌ Not in sudoers
cat /opt/apps/auth-service/.env  # ❌ Permission denied

exit
```

---

## 📞 Troubleshooting

**Problem: SSH connection refused**
```bash
# Check SSH key is in authorized_keys
sudo cat /home/attendance-admin/.ssh/authorized_keys
```

**Problem: Git pull fails**
```bash
# Check GitHub SSH key
sudo su - attendance-admin
ssh -T git@github.com
```

**Problem: Permission denied when restarting**
```bash
# Check sudo configuration
sudo cat /etc/sudoers.d/attendance-admin
```

**Problem: Can't access service directory**
```bash
# Fix permissions
sudo chown -R attendance-admin:attendance-admin /opt/apps/attendance-system
```

---

## 🗑️ Removing User Access

```bash
# Stop their service first
pm2 delete attendance-system
pm2 save

# Remove user and all their files
sudo userdel -r attendance-admin

# Remove sudo configuration
sudo rm /etc/sudoers.d/attendance-admin

# Service directory still exists (decide what to do)
ls -la /opt/apps/attendance-system
```

---

**✅ Setup complete!** Manager now has secure, restricted access to only their service.

