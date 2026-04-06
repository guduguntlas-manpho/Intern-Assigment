# Environment Verification Checklist

Use this checklist to verify your ERPNext development environment is set up correctly.

## ✅ System Verification

### Docker Setup

- [ ] Docker installed and running
  ```bash
  docker --version
  # Expected: Docker version 20.x or higher
  ```

- [ ] Docker Compose installed
  ```bash
  docker-compose --version
  # Expected: docker-compose version 1.29.x or higher
  ```

- [ ] Containers running
  ```bash
  docker-compose -f pwd.yml ps
  # Expected: All containers in "Up" state
  ```

### Bare-Metal Setup

- [ ] Python 3.10+ installed
  ```bash
  python3 --version
  # Expected: Python 3.10.x or higher
  ```

- [ ] Node.js 16 or 18 installed
  ```bash
  node --version
  # Expected: v16.x or v18.x
  ```

- [ ] MariaDB running
  ```bash
  sudo service mysql status
  # Expected: active (running)
  ```

- [ ] Redis running
  ```bash
  sudo service redis-server status
  # Expected: active (running)
  ```

- [ ] Bench installed
  ```bash
  bench --version
  # Expected: 5.x.x or higher
  ```

---

## ✅ ERPNext Verification

### Web Access

- [ ] ERPNext accessible in browser
  - URL: `http://localhost:8000` (Docker) or `http://library.localhost:8000` (Bare-metal)
  - Login successful with Administrator/admin

- [ ] No console errors
  - Open browser DevTools (F12)
  - Check Console tab for errors
  - Expected: No red errors

### Developer Mode

- [ ] Developer mode enabled
  ```bash
  # Docker
  docker-compose -f pwd.yml exec backend bench --site library.localhost get-config developer_mode
  
  # Bare-metal
  bench --site library.localhost get-config developer_mode
  
  # Expected: 1
  ```

### Installed Apps

- [ ] Frappe and ERPNext installed
  ```bash
  # Docker
  docker-compose -f pwd.yml exec backend bench --site library.localhost list-apps
  
  # Bare-metal
  bench --site library.localhost list-apps
  
  # Expected output:
  # frappe
  # erpnext
  ```

---

## ✅ Custom App Verification

### App Creation

- [ ] `library_management` app created
  ```bash
  # Docker
  docker-compose -f pwd.yml exec backend ls apps/
  
  # Bare-metal
  ls apps/
  
  # Expected: library_management folder exists
  ```

- [ ] App installed on site
  ```bash
  # Docker
  docker-compose -f pwd.yml exec backend bench --site library.localhost list-apps
  
  # Bare-metal
  bench --site library.localhost list-apps
  
  # Expected output includes:
  # library_management
  ```

### Folder Structure

- [ ] Correct folder structure exists
  ```bash
  # Docker
  docker-compose -f pwd.yml exec backend ls -la apps/library_management/
  
  # Bare-metal
  ls -la apps/library_management/
  
  # Expected structure:
  # library_management/
  # ├── library_management/
  # │   ├── __init__.py
  # │   ├── hooks.py
  # │   ├── modules.txt
  # │   └── library_management/
  # ├── setup.py
  # └── README.md
  ```

---

## ✅ DocType Creation Test

### Create Test DocType

- [ ] Can create DocType via UI
  1. Go to: Home → Customization → DocType → New
  2. Create a test DocType named "Test Item"
  3. Add fields:
     - `item_name` (Data, Mandatory)
     - `description` (Text)
  4. Save

- [ ] DocType appears in module
  - Navigate to: Home → Library Management → Test Item
  - Expected: List view loads without errors

- [ ] Can create document
  1. Click "New"
  2. Fill in item_name
  3. Save
  4. Expected: Document saves successfully

- [ ] Can delete test DocType
  1. Go to DocType list
  2. Delete "Test Item"
  3. Expected: Deletion successful

---

## ✅ Database Access

### Database Connection

- [ ] Can access database
  ```bash
  # Docker
  docker-compose -f pwd.yml exec backend bench --site library.localhost mariadb
  
  # Bare-metal
  bench --site library.localhost mariadb
  
  # In MariaDB prompt:
  SHOW TABLES;
  # Expected: List of tables including tabUser, tabDocType, etc.
  EXIT;
  ```

---

## ✅ File System Access

### Code Editing

- [ ] Can edit files in custom app
  ```bash
  # Docker
  docker-compose -f pwd.yml exec backend nano apps/library_management/library_management/hooks.py
  
  # Bare-metal
  nano apps/library_management/library_management/hooks.py
  
  # Expected: File opens for editing
  ```

- [ ] Changes reflect after restart
  1. Edit `hooks.py` (add a comment)
  2. Restart bench
  3. Verify change persists

---

## ✅ Git Setup

### Repository Initialization

- [ ] Git initialized in custom app
  ```bash
  # Docker
  docker-compose -f pwd.yml exec backend git -C apps/library_management status
  
  # Bare-metal
  cd apps/library_management
  git status
  
  # Expected: On branch main/master
  ```

- [ ] Can make commits
  ```bash
  git add .
  git commit -m "test: Initial commit"
  # Expected: Commit successful
  ```

---

## 📸 Required Screenshots

Take the following screenshots for your submission:

### Screenshot 1: Login Page
- [ ] ERPNext login page showing URL in address bar

### Screenshot 2: Installed Apps
- [ ] Navigate to: Home → Settings → About
- [ ] Screenshot showing installed apps including `library_management`

### Screenshot 3: Custom App Module
- [ ] Navigate to: Home → Library Management
- [ ] Screenshot showing the module page

### Screenshot 4: Developer Mode Enabled
- [ ] Navigate to: Home → Settings → System Settings
- [ ] Screenshot showing "Developer Mode" checkbox is checked

### Screenshot 5: CLI Output
- [ ] Terminal screenshot showing:
  ```bash
  bench version
  bench --site library.localhost list-apps
  ```

---

## 🚨 Common Failure Points

If any verification fails, check these common issues:

### Port Conflicts
- **Symptom**: Cannot access ERPNext at localhost:8000
- **Solution**: Check if port is in use: `sudo lsof -i :8000`

### Permission Issues
- **Symptom**: Permission denied errors
- **Solution**: 
  - Docker: `sudo usermod -aG docker $USER` (then logout/login)
  - Bare-metal: Check file ownership in frappe-bench folder

### Database Connection
- **Symptom**: Database connection errors
- **Solution**: 
  - Check MariaDB is running
  - Verify credentials in site_config.json

### Node Version Mismatch
- **Symptom**: Build errors, npm errors
- **Solution**: Use Node 16 or 18 (use nvm to switch versions)

### Cache Issues
- **Symptom**: Changes not reflecting
- **Solution**:
  ```bash
  # Docker
  docker-compose -f pwd.yml exec backend bench --site library.localhost clear-cache
  
  # Bare-metal
  bench --site library.localhost clear-cache
  bench restart
  ```

---

## ✅ Final Checklist

Before starting the assignment tasks:

- [ ] All system verification items passed
- [ ] All ERPNext verification items passed
- [ ] All custom app verification items passed
- [ ] All 5 screenshots taken
- [ ] Git repository initialized
- [ ] Can create and save DocTypes
- [ ] Developer mode enabled
- [ ] No console errors in browser

---

## 🎯 Next Steps

Once all items are checked:

1. ✅ Read [CODE_QUALITY.md](../guidelines/CODE_QUALITY.md)
2. ✅ Read [GIT_WORKFLOW.md](../guidelines/GIT_WORKFLOW.md)
3. ✅ Navigate to your level's tasks:
   - Entry: [assignments/entry/tasks/TASKS.md](../../entry/tasks/TASKS.md)
   - Junior: [assignments/junior/tasks/TASKS.md](../../junior/tasks/TASKS.md)
   - Mid: [assignments/mid/tasks/TASKS.md](../../mid/tasks/TASKS.md)
   - Senior: [assignments/senior/tasks/TASKS.md](../../senior/tasks/TASKS.md)

---

**Environment setup complete! You're ready to start coding.** 🚀
