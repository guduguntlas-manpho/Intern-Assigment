```markdown
# Frappe Framework Installation Guide

This guide provides a step-by-step walkthrough for setting up a Frappe development environment. This is a **Native (Bare-Metal)** setup, recommended for better performance and easier debugging.

## Prerequisites
- **RAM**: 4GB Minimum (8GB Recommended)
- **Disk**: 10GB Free Space
- **Python**: 3.10, 3.11, or 3.12
- **Node.js**: 18 or 20

---

## 💻 Option 1: Windows (via WSL2)
*Windows users must use WSL2 to run a Linux environment inside Windows.*

### 1. Setup WSL2
1. Open **PowerShell** as Administrator and run:
   ```powershell
   wsl --install
   ```
2. **Restart your PC.**
3. After restart, a terminal will open. Set your **Username** and **Password**.
4. Update the system:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

### 2. Integration with VS-Code
1. Install the **WSL Extension** in VS-Code.
2. In your Ubuntu terminal, type `code .` to open the current folder in VS-Code.
3. **Now follow the "Ubuntu / Linux" steps below inside your VS-Code Terminal.**

---

## 🐧 Option 2: Ubuntu / Linux (Native)
*Follow these steps if you are using Ubuntu or the WSL2 terminal.*

### 1. Install Dependencies
```bash
sudo apt update
sudo apt install -y python3-dev python3-pip python3-venv software-properties-common mariadb-server mariadb-client redis-server nodejs npm curl git libmysqlclient-dev

# Install Yarn globally
sudo npm install -g yarn
```

### 2. Configure MariaDB
1. Open config: `sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf`
2. Add these lines under the `[mysqld]` section:
   ```ini
   innodb-check-optimize-with-force = 1
   innodb-file-format = Barracuda
   innodb-file-per-table = 1
   innodb-large-prefix = 1
   character-set-client-handshake = FALSE
   character-set-server = utf8mb4
   collation-server = utf8mb4_unicode_ci
   ```
3. Save (`Ctrl+O`, `Enter`) and Exit (`Ctrl+X`).
4. Restart & Secure:
   ```bash
   sudo service mysql restart
   sudo mysql_secure_installation
   ```
   *(Set a root password and remember it!)*

### 3. Install Bench & Init
```bash
pip3 install frappe-bench
bench init frappe-bench --frappe-branch version-15
cd frappe-bench
```

---

## 🍎 Option 3: macOS
*Note: macOS uses Homebrew for package management.*

### 1. Install Homebrew & Dependencies
```bash
# Install Homebrew if you don't have it
/bin/bash -c "$(curl -fsSL [https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh](https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh))"

# Install core packages
brew install python@3.11 mariadb redis node@18 yarn
```

### 2. Start Services
```bash
brew services start mariadb
brew services start redis
```

### 3. Configure MariaDB
1. Open config: `nano /usr/local/etc/my.cnf` (or `/opt/homebrew/etc/my.cnf` for M1/M2/M3 Macs).
2. Add the same `[mysqld]` configuration used in the Ubuntu section above.
3. Restart MariaDB: `brew services restart mariadb`.

### 4. Install Bench
```bash
pip3 install frappe-bench
bench init frappe-bench --frappe-branch version-15
cd frappe-bench
```

---

## 🚀 Finalizing Setup (All Platforms)

Once the `bench` is initialized and you are inside the `frappe-bench` folder, run:

### 1. Create a Site
```bash
bench new-site mysite.localhost
```
*You will be asked for the **MariaDB root password** (from step 2) and to create a new **Administrator password** for the web login.*

### 2. Start the Server
```bash
bench start
```

### 3. Access the Web UI
- **URL**: `http://localhost:8000`
- **User**: `Administrator`
- **Password**: (The one you just created)

---

## 🛠️ Essential Bench Commands

| Action | Command |
| :--- | :--- |
| **Create New App** | `bench new-app [app_name]` |
| **Install App to Site** | `bench --site [site_name] install-app [app_name]` |
| **Enter Console** | `bench --site [site_name] console` |
| **Enable Dev Mode** | `bench --site [site_name] set-config developer_mode 1` |
| **Watch Logs** | `bench watch` |

---

## 💡 Pro Tips for Freshers
- **Don't use `sudo`** for `bench` commands. It will break file permissions.
- **Developer Mode**: Always enable `developer_mode` if you want to create new DocTypes.
- **Port 8000 Busy?**: If the port is taken, use `bench set-mariadb-host localhost` or check for hanging processes with `lsof -i :8000`.
```
- [Frappe Docker Repository](https://github.com/frappe/frappe_docker)
- [Bench CLI Guide](https://github.com/frappe/bench)
