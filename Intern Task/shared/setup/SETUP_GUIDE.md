# ERPNext Environment Setup Guide

This guide will help you set up a working ERPNext development environment for completing the technical assignment.

## Prerequisites

- **OS**: Linux (Ubuntu 20.04+), macOS, or Windows with WSL2
- **RAM**: Minimum 4GB (8GB recommended)
- **Disk Space**: 10GB free space
- **Internet**: Stable connection for downloads

## Option A: Docker Setup (Recommended)

### Why Docker?

- ✅ Faster setup
- ✅ Consistent environment
- ✅ Easy to reset
- ✅ No system-wide dependencies

### Step 1: Install Docker

**Ubuntu/Debian**:
```bash
sudo apt-get update
sudo apt-get install docker.io docker-compose -y
sudo usermod -aG docker $USER
# Log out and log back in
```

**macOS**:
```bash
# Download and install Docker Desktop from:
# https://www.docker.com/products/docker-desktop
```

**Windows (WSL2)**:
```bash
# Install Docker Desktop for Windows with WSL2 backend
# https://docs.docker.com/desktop/windows/install/
```

### Step 2: Clone Frappe Docker

```bash
git clone https://github.com/frappe/frappe_docker.git
cd frappe_docker
```

### Step 3: Setup Development Environment

```bash
# Copy example environment file
cp example.env .env

# Start containers
docker-compose -f pwd.yml up -d

# Wait for services to start (2-3 minutes)
docker-compose -f pwd.yml ps
```

### Step 4: Create a New Site

```bash
# Create site
docker-compose -f pwd.yml exec backend bench new-site library.localhost \
  --admin-password admin \
  --db-root-password admin

# Set site as current site
docker-compose -f pwd.yml exec backend bench use library.localhost

# Install ERPNext
docker-compose -f pwd.yml exec backend bench get-app erpnext --branch version-15
docker-compose -f pwd.yml exec backend bench --site library.localhost install-app erpnext

# Enable developer mode
docker-compose -f pwd.yml exec backend bench --site library.localhost set-config developer_mode 1

# Clear cache
docker-compose -f pwd.yml exec backend bench --site library.localhost clear-cache
```

### Step 5: Access ERPNext

1. Open browser: `http://localhost:8000`
2. Login:
   - Username: `Administrator`
   - Password: `admin`

### Step 6: Create Custom App

```bash
# Create new app
docker-compose -f pwd.yml exec backend bench new-app library_management

# Install app on site
docker-compose -f pwd.yml exec backend bench --site library.localhost install-app library_management

# Restart bench
docker-compose -f pwd.yml restart backend
```

---

## Option B: Bare-Metal Setup (Fallback)

### Why Bare-Metal?

- Better performance on some systems
- Direct access to files
- More control over configuration

### Step 1: Install System Dependencies

**Ubuntu 20.04/22.04**:
```bash
sudo apt-get update
sudo apt-get install -y \
    git python3-dev python3-pip python3-venv \
    redis-server mariadb-server \
    libmysqlclient-dev \
    curl wget \
    nodejs npm

# Install specific Node version (16 or 18)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Install yarn
sudo npm install -g yarn
```

**macOS**:
```bash
# Install Homebrew if not installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install python@3.10 mariadb redis node@18 git

# Start services
brew services start mariadb
brew services start redis
```

### Step 2: Configure MariaDB

```bash
# Secure installation
sudo mysql_secure_installation

# Create database user
sudo mysql -u root -p
```

In MySQL prompt:
```sql
CREATE USER 'frappe'@'localhost' IDENTIFIED BY 'frappe';
GRANT ALL PRIVILEGES ON *.* TO 'frappe'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

Edit MariaDB config:
```bash
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Add under `[mysqld]`:
```ini
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

[mysql]
default-character-set = utf8mb4
```

Restart MariaDB:
```bash
sudo service mysql restart
```

### Step 3: Install Bench

```bash
# Install bench CLI
sudo pip3 install frappe-bench

# Initialize bench
bench init frappe-bench --frappe-branch version-15

# Navigate to bench directory
cd frappe-bench
```

### Step 4: Create Site

```bash
# Create new site
bench new-site library.localhost \
  --db-name library_db \
  --admin-password admin

# Set as current site
bench use library.localhost
```

### Step 5: Install ERPNext

```bash
# Get ERPNext app
bench get-app erpnext --branch version-15

# Install on site
bench --site library.localhost install-app erpnext

# Enable developer mode
bench --site library.localhost set-config developer_mode 1

# Start bench
bench start
```

### Step 6: Access ERPNext

1. Open browser: `http://library.localhost:8000`
2. Login:
   - Username: `Administrator`
   - Password: `admin`

### Step 7: Create Custom App

```bash
# In a new terminal (keep bench running in first terminal)
cd frappe-bench

# Create app
bench new-app library_management

# Install app
bench --site library.localhost install-app library_management

# Restart bench (Ctrl+C in first terminal, then bench start)
```

---

## Verification

After setup, verify your environment using the [VERIFICATION.md](VERIFICATION.md) checklist.

---

## Common Issues

### Docker Issues

**Port 8000 already in use**:
```bash
# Find process using port
sudo lsof -i :8000
# Kill process or change port in docker-compose file
```

**Permission denied**:
```bash
# Add user to docker group
sudo usermod -aG docker $USER
# Log out and log back in
```

**Containers not starting**:
```bash
# Check logs
docker-compose -f pwd.yml logs backend
# Restart containers
docker-compose -f pwd.yml restart
```

### Bare-Metal Issues

**MariaDB connection error**:
```bash
# Check MariaDB is running
sudo service mysql status
# Restart if needed
sudo service mysql restart
```

**Redis connection error**:
```bash
# Check Redis is running
sudo service redis-server status
# Restart if needed
sudo service redis-server restart
```

**Node version mismatch**:
```bash
# Check Node version
node --version
# Should be v16.x or v18.x

# Install correct version using nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 18
nvm use 18
```

**Bench command not found**:
```bash
# Install bench globally
sudo pip3 install frappe-bench

# Or add to PATH
export PATH=$PATH:~/.local/bin
```

---

## Next Steps

1. ✅ Complete verification checklist
2. ✅ Read code quality guidelines
3. ✅ Review Git workflow
4. ✅ Start your level's tasks

---

## Resources

- [Frappe Framework Documentation](https://frappeframework.com/docs)
- [ERPNext Documentation](https://docs.erpnext.com/)
- [Frappe Docker Repository](https://github.com/frappe/frappe_docker)
- [Bench CLI Guide](https://github.com/frappe/bench)
