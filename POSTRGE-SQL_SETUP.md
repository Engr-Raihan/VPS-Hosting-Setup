### Configure PostgreSQL

**✨ Best Practice: One PostgreSQL Instance, Multiple Databases**

- ✅ **One PostgreSQL Docker container**
- ✅ **One database per application** (inside that PostgreSQL)
- ✅ **Separate user per database** (security isolation)
- ✅ **All applications connect to `localhost:5432`**

---

#### 1. Install and Setup PostgreSQL

**Step 1.1: Create Directories and Set Permissions**

```bash
# Create directories (if not already created)
sudo mkdir -p /opt/databases/postgresql/{data,logs,config}
sudo mkdir -p /opt/backups/postgresql

# Set ownership
sudo chown -R $USER:$USER /opt/databases/postgresql
sudo chmod 755 /opt/databases/postgresql/logs

cd /opt/databases
```

**Step 1.2: Update Docker Compose Configuration**

```bash
nano docker-compose.yml
```

**Update PostgreSQL service in docker-compose.yml:**

```yaml
  postgresql:
    image: postgres:16-alpine
    container_name: postgresql
    restart: unless-stopped
    ports:
      - "127.0.0.1:5432:5432"  # Only bind to localhost (for local access)
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - ./postgresql/data:/var/lib/postgresql/data
      - ./postgresql/logs:/var/log/postgresql
    networks:
      - backend
```

**Step 1.3: Update Environment File**

```bash
nano .env
```

**Add PostgreSQL credentials to .env:**

```bash
# Generate passwords with: openssl rand -base64 32
POSTGRES_USER=postgres
POSTGRES_PASSWORD=YOUR_SECURE_POSTGRESQL_PASSWORD_HERE
POSTGRES_DB=postgres
```

**Step 1.4: Start PostgreSQL and Fix Permissions**

```bash
# Secure .env file
chmod 600 .env

# Start PostgreSQL
docker compose up -d postgresql

# Fix log directory permissions (postgres user needs write access)
if docker ps | grep -q postgresql; then
  POSTGRES_UID=$(docker exec postgresql id -u postgres 2>/dev/null || echo "999")
  POSTGRES_GID=$(docker exec postgresql id -g postgres 2>/dev/null || echo "999")
  sudo chown -R ${POSTGRES_UID}:${POSTGRES_GID} /opt/databases/postgresql/logs
  sudo chmod 755 /opt/databases/postgresql/logs
  echo "✅ Log directory permissions set (UID: ${POSTGRES_UID}, GID: ${POSTGRES_GID})"
fi

# Restart to apply permissions
docker compose restart postgresql
```

**Step 1.5: Verify PostgreSQL is Running**

```bash
# Check container status
docker compose ps

# Check logs (should not show permission denied)
docker logs postgresql | tail -20

# Test connection
docker exec postgresql psql -U postgres -d postgres -c "SELECT version();"
```

---

#### 2. Configure PostgreSQL

**Step 2.1: Connect to PostgreSQL**

```bash
# Connect using environment variables
cd /opt/databases
source .env
docker exec -it postgresql psql -U ${POSTGRES_USER} -d ${POSTGRES_DB}

# Or connect directly
docker exec -it postgresql psql -U postgres -d postgres
```

**Step 2.2: Create Database and User**

```sql
-- Create database
CREATE DATABASE your_database_name;

-- Create user
CREATE USER your_database_user WITH PASSWORD 'GENERATE_SECURE_PASSWORD_HERE';

-- Grant database privileges
GRANT ALL PRIVILEGES ON DATABASE your_database_name TO your_database_user;
```

**Step 2.3: Grant All Privileges**

```sql
-- Connect to the new database
\c your_database_name

-- Grant schema privileges (for existing and future tables)
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO your_database_user;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO your_database_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO your_database_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO your_database_user;
```

**Step 2.4: Verify Database Setup**

```sql
-- List all databases
\l

-- List all users
\du

-- Verify your database
\l your_database_name

-- Exit
\q
```

**Test connection:**
```bash
docker exec postgresql psql -U your_database_user -d your_database_name -c "SELECT version();"
```

---

#### 3. Expose PostgreSQL for External Connections

**⚠️ Security Warning:** Only expose if you need external access. Follow all security steps.

**Step 3.1: Update Docker Compose to Expose Port**

```bash
cd /opt/databases
nano docker-compose.yml
```

**Update PostgreSQL ports:**

```yaml
  postgresql:
    image: postgres:16-alpine
    container_name: postgresql
    restart: unless-stopped
    ports:
      - "5432:5432"  # Expose to external connections (was: "127.0.0.1:5432:5432")
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    volumes:
      - ./postgresql/data:/var/lib/postgresql/data
      - ./postgresql/logs:/var/log/postgresql
      - ./postgresql/config/postgresql.conf:/etc/postgresql/postgresql.conf
      - ./postgresql/config/pg_hba.conf:/etc/postgresql/pg_hba.conf
    networks:
      - backend
    command: postgres -c config_file=/etc/postgresql/postgresql.conf -c hba_file=/etc/postgresql/pg_hba.conf
```

**Step 3.2: Create Configuration Files**

**Create postgresql.conf:**

```bash
mkdir -p /opt/databases/postgresql/config
nano /opt/databases/postgresql/config/postgresql.conf
```

**postgresql.conf:**

```conf
# Listen on all interfaces (inside container)
listen_addresses = '*'

# Connection settings
max_connections = 100
shared_buffers = 256MB
effective_cache_size = 1GB
maintenance_work_mem = 64MB
checkpoint_completion_target = 0.9
wal_buffers = 16MB
default_statistics_target = 100
random_page_cost = 1.1
effective_io_concurrency = 200
work_mem = 4MB
min_wal_size = 1GB
max_wal_size = 4GB

# Logging
logging_collector = on
log_directory = '/var/log/postgresql'
log_filename = 'postgresql-%Y-%m-%d.log'
log_statement = 'ddl'  # Log DDL statements (CREATE, ALTER, DROP)
log_connections = on
log_disconnections = on
log_duration = on

# SSL Configuration (recommended for external access)
# Note: SSL certificates will be created in next step
ssl = on
ssl_cert_file = '/var/lib/postgresql/data/server.crt'
ssl_key_file = '/var/lib/postgresql/data/server.key'
```

**Create pg_hba.conf:**

```bash
nano /opt/databases/postgresql/config/pg_hba.conf
```

**pg_hba.conf (Specific IP):**

```conf
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# Local connections (from inside container)
local   all             all                                     trust

# IPv4 local connections (from VPS itself)
host    all             all             127.0.0.1/32            md5

# IPv6 local connections
host    all             all             ::1/128                  md5       

# Option 1: Allow specific IP (RECOMMENDED) - Use `/32` for single IP
host    bldg_real_estate  bldg_real_estate_user  103.109.238.99/32    md5

# Option 2: Allow specific IP range (e.g., your office network `/24`)
# host    bldg_real_estate  bldg_real_estate_user  192.168.1.0/24   md5    

# Option 3: Allow from anywhere (NOT RECOMMENDED - use only for testing)   
# host    bldg_real_estate  bldg          0.0.0.0/0         md5

# Default deny all other connections
host    all             all             0.0.0.0/0               reject   
```



**Step 3.3: Generate SSL Certificates**

```bash
# Navigate to PostgreSQL data directory
cd /opt/databases/postgresql/data

# Generate self-signed SSL certificate
openssl req -new -x509 -days 365 -nodes -text -out server.crt -keyout server.key -subj '/CN=postgresql'

# Set permissions
chmod 600 server.key
chmod 644 server.crt

# Get container UID/GID (if container is running)
if docker ps | grep -q postgresql; then
  CONTAINER_UID=$(docker exec postgresql id -u postgres 2>/dev/null || echo "999")
  CONTAINER_GID=$(docker exec postgresql id -g postgres 2>/dev/null || echo "999")
else
  # Use existing data directory ownership
  CONTAINER_UID=$(stat -c '%u' /opt/databases/postgresql/data 2>/dev/null || echo "999")
  CONTAINER_GID=$(stat -c '%g' /opt/databases/postgresql/data 2>/dev/null || echo "999")
fi

# Set ownership
sudo chown ${CONTAINER_UID}:${CONTAINER_GID} server.crt server.key

# Verify
ls -la server.crt server.key
```

**Step 3.4: Configure Firewall (UFW)**

```bash
# Option 1: Allow from specific IP (RECOMMENDED), Replace YOUR_APP_IP with actual IP address
sudo ufw allow from YOUR_APP_IP to any port 5432 proto tcp comment 'PostgreSQL from app server'

# Option 2: Allow from IP range (e.g., office network)
# sudo ufw allow from 192.168.1.0/24 to any port 5432 proto tcp comment 'PostgreSQL from office network'

# Option 3: Allow from anywhere (NOT RECOMMENDED - only for testing)
# sudo ufw allow 5432/tcp comment 'PostgreSQL'

# Verify firewall rules
sudo ufw status numbered

# Check if rule was added
sudo ufw status | grep 5432
```


**Step 3.5: Restart PostgreSQL with New Configuration**

```bash
cd /opt/databases

# Stop container
docker compose stop postgresql

# Fix log directory permissions (if needed)
EXISTING_UID=$(stat -c '%u' /opt/databases/postgresql/data 2>/dev/null || echo "999")
EXISTING_GID=$(stat -c '%g' /opt/databases/postgresql/data 2>/dev/null || echo "999")
sudo chown -R ${EXISTING_UID}:${EXISTING_GID} /opt/databases/postgresql/logs
sudo chmod 755 /opt/databases/postgresql/logs

# Start with new configuration
docker compose up -d postgresql

# Verify it's running
docker compose ps
docker logs postgresql | tail -20
```

**Step 3.6: Verify Configuration**

```bash
# Check if listening on external interface
sudo netstat -tlnp | grep 5432
# Should show: 0.0.0.0:5432 (not just 127.0.0.1:5432)

# Verify SSL is enabled
docker exec postgresql psql -U postgres -c "SHOW ssl;"
# Should show: on

# Check SSL in logs
docker logs postgresql | grep -i ssl
```

---

#### 4. Create BLDG Real Estate Database (Example)

**Complete example with UUID extension:**

```bash
# Connect to PostgreSQL
docker exec -it postgresql psql -U postgres -d postgres
```

```sql
-- Create database with UTF8 encoding
CREATE DATABASE bldg_real_estate 
  WITH ENCODING 'UTF8' 
  LC_COLLATE='en_US.UTF-8' 
  LC_CTYPE='en_US.UTF-8'
  TEMPLATE template0;

-- Create dedicated user
CREATE USER bldg_real_estate_user WITH PASSWORD 'GENERATE_SECURE_PASSWORD_HERE';

-- Grant database privileges
GRANT ALL PRIVILEGES ON DATABASE bldg_real_estate TO bldg_real_estate_user;

-- Connect to the new database
\c bldg_real_estate

-- Enable UUID extension (required for UUID generation)
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Grant schema privileges
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO bldg_real_estate_user;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO bldg_real_estate_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON TABLES TO bldg_real_estate_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL ON SEQUENCES TO bldg_real_estate_user;

-- Verify database and extensions
\l bldg_real_estate
\dx

-- Exit
\q
```

**Connection Strings:**

```bash
# Local connection
postgresql://bldg_real_estate_user:PASSWORD@localhost:5432/bldg_real_estate

# External connection (if exposed)
postgresql://bldg_real_estate_user:PASSWORD@YOUR_VPS_IP:5432/bldg_real_estate

# External with SSL
postgresql://bldg_real_estate_user:PASSWORD@YOUR_VPS_IP:5432/bldg_real_estate?sslmode=require
```

---

### Troubleshooting PostgreSQL

**Connection refused from external IP:**

```bash
# Check if running
docker ps | grep postgresql

# Check if port is exposed
docker port postgresql

# Check firewall
sudo ufw status | grep 5432

# Check if listening on external interface
sudo netstat -tlnp | grep 5432
```

**Authentication failed:**

```bash
# Check pg_hba.conf rules
cat /opt/databases/postgresql/config/pg_hba.conf

# Verify user exists
docker exec postgresql psql -U postgres -c "\du"

# Test password locally first
docker exec postgresql psql -U your_database_user -d your_database_name
```

**SSL connection error:**

```bash
# Check if certificates exist
docker exec postgresql sh -c "ls -la /var/lib/postgresql/data/server.*"

# Check SSL status
docker exec postgresql psql -U postgres -c "SHOW ssl;"

# If SSL is off but should be on:
# - Verify certificates exist and have correct permissions
# - Check certificate paths in postgresql.conf
# - Restart container
```

**PostgreSQL container restarting continuously:**

```bash
# Check logs for errors
docker logs postgresql | tail -50

# Common issue: Permission denied on log directory
docker logs postgresql | grep -i "permission denied"

# Fix: Stop container first (can't exec into restarting container)
cd /opt/databases
docker compose stop postgresql

# Get UID/GID from existing data directory (most reliable - PostgreSQL already created it)
if [ -d "/opt/databases/postgresql/data" ]; then
  EXISTING_UID=$(stat -c '%u' /opt/databases/postgresql/data 2>/dev/null || echo "999")
  EXISTING_GID=$(stat -c '%g' /opt/databases/postgresql/data 2>/dev/null || echo "999")
  POSTGRES_UID=$EXISTING_UID
  POSTGRES_GID=$EXISTING_GID
  echo "✅ Detected PostgreSQL UID: ${POSTGRES_UID}, GID: ${POSTGRES_GID}"
else
  POSTGRES_UID=999
  POSTGRES_GID=999
  echo "⚠️ Using default UID: ${POSTGRES_UID}"
fi

# Set correct ownership and permissions
sudo chown -R ${POSTGRES_UID}:${POSTGRES_GID} /opt/databases/postgresql/logs
sudo chmod 755 /opt/databases/postgresql/logs

# Also ensure data directory has correct ownership (if it exists)
if [ -d "/opt/databases/postgresql/data" ]; then
  sudo chown -R ${POSTGRES_UID}:${POSTGRES_GID} /opt/databases/postgresql/data
fi

# Start container after fixes
docker compose up -d postgresql

# Wait a few seconds
sleep 5

# Check status
docker compose ps

# Verify it's running (not restarting)
docker logs postgresql | tail -20
```

**View active connections:**

```bash
# See who's connected
docker exec postgresql psql -U postgres -c "SELECT * FROM pg_stat_activity;"

# View connection logs
docker logs postgresql | grep -i connection

# Check connection count
docker exec postgresql psql -U postgres -c "SELECT count(*) FROM pg_stat_activity;"
```

---

### Security Checklist

Before exposing PostgreSQL externally, verify:

- [ ] Strong password generated (`openssl rand -base64 32`)
- [ ] `pg_hba.conf` restricts to specific IPs (not `0.0.0.0/0` for production)
- [ ] Firewall (UFW) allows only specific IPs
- [ ] SSL certificates generated and configured
- [ ] Using dedicated database user (not `postgres` superuser)
- [ ] User has only necessary privileges (not superuser)
- [ ] Logging enabled (`log_connections = on`)
- [ ] Tested connection from allowed IP only
- [ ] Connection string uses SSL in production (`sslmode=require`)
- [ ] Removed any testing rules (like `0.0.0.0/0`) before production

---

### Production Recommendations

1. **Use VPN or Private Network:** If possible, connect through VPN instead of exposing to internet
2. **IP Whitelisting:** Always restrict to specific IPs in `pg_hba.conf` and firewall
3. **SSL Required:** Force SSL connections (`sslmode=require` or `verify-full`)
4. **Regular Updates:** Keep PostgreSQL Docker image updated
5. **Monitor Connections:** Set up alerts for suspicious connection attempts
6. **Backup Strategy:** Regular automated backups before exposing externally
7. **Rate Limiting:** Consider using a connection pooler (PgBouncer) for high traffic
8. **Audit Logging:** Enable detailed logging for security audits
9. **Trusted CA Certificates:** Use Let's Encrypt or organization CA for production (not self-signed)

---

### Quick Reference

**Connection Strings:**

```bash
# Local (from VPS)
postgresql://bldg_real_estate_user:PASSWORD@localhost:5432/bldg_real_estate

# External (from another server)
postgresql://bldg_real_estate_user:PASSWORD@YOUR_VPS_IP:5432/bldg_real_estate

# External with SSL
postgresql://bldg_real_estate_user:PASSWORD@YOUR_VPS_IP:5432/bldg_real_estate?sslmode=require
```

**Find your VPS IP:**
```bash
# On VPS
curl ifconfig.me
# Or
hostname -I
```

**Common Commands:**
```bash
# Restart PostgreSQL
cd /opt/databases && docker compose restart postgresql

# View logs
docker logs postgresql | tail -50

# Connect to database
docker exec -it postgresql psql -U bldg_real_estate_user -d bldg_real_estate

# Check SSL status
docker exec postgresql psql -U postgres -c "SHOW ssl;"
```

**💡 Generate secure password:**
```bash
openssl rand -base64 32
```