# Task 1: Library Member DocType

**Estimated Time**: 2 hours  
**Difficulty**: Easy

---

## Objective

Create a DocType to manage library members with their personal information and membership details.

---

## Requirements

### DocType Configuration

**DocType Name**: `Library Member`

**Module**: Library Management

**Naming Series**: `LIB-MEM-.#####`

**Settings**:
- [x] Allow Search
- [x] Allow Filters
- [x] Allow Sorting
- [x] Track Changes

---

## Fields

Create the following fields in this order:

### 1. Member ID
- **Field Name**: `member_id`
- **Type**: Data
- **Label**: Member ID
- **Read Only**: Yes
- **Description**: Auto-generated member identifier

### 2. First Name
- **Field Name**: `first_name`
- **Type**: Data
- **Label**: First Name
- **Mandatory**: Yes

### 3. Last Name
- **Field Name**: `last_name`
- **Type**: Data
- **Label**: Last Name
- **Mandatory**: Yes

### 4. Full Name
- **Field Name**: `full_name`
- **Type**: Data
- **Label**: Full Name
- **Read Only**: Yes
- **Description**: Auto-generated from first and last name

### 5. Email
- **Field Name**: `email`
- **Type**: Data
- **Label**: Email
- **Mandatory**: Yes
- **Unique**: Yes
- **Options**: Email

### 6. Phone
- **Field Name**: `phone`
- **Type**: Data
- **Label**: Phone Number
- **Options**: Phone

### 7. Membership Type
- **Field Name**: `membership_type`
- **Type**: Select
- **Label**: Membership Type
- **Options**: 
  ```
  Standard
  Premium
  VIP
  ```
- **Default**: Standard

### 8. Membership Start Date
- **Field Name**: `membership_start_date`
- **Type**: Date
- **Label**: Membership Start Date
- **Mandatory**: Yes
- **Default**: Today

### 9. Membership End Date
- **Field Name**: `membership_end_date`
- **Type**: Date
- **Label**: Membership End Date

### 10. Status
- **Field Name**: `status`
- **Type**: Select
- **Label**: Status
- **Options**:
  ```
  Active
  Inactive
  Suspended
  ```
- **Default**: Active
- **Mandatory**: Yes

---

## Implementation Steps

### Step 1: Create DocType

1. Login to ERPNext
2. Go to: **Home → Customization → DocType → New**
3. Enter DocType Name: `Library Member`
4. Select Module: `Library Management`
5. Set Naming: `LIB-MEM-.#####`

### Step 2: Add Fields

Add all fields listed above in the specified order.

**Tips**:
- Use the exact field names (snake_case)
- Set field types correctly
- Mark mandatory fields
- Set default values where specified

### Step 3: Configure Settings

Enable these options:
- Allow Search
- Allow Filters  
- Allow Sorting
- Track Changes

### Step 4: Save

Click **Save** to create the DocType.

---

## Testing

### Manual Tests

1. **Create Member**:
   - Go to: Library Management → Library Member → New
   - Fill in all mandatory fields
   - Save
   - Verify member_id is auto-generated (e.g., LIB-MEM-00001)

2. **Edit Member**:
   - Open an existing member
   - Change membership_type to "Premium"
   - Save
   - Verify changes are saved

3. **Unique Email**:
   - Try to create two members with same email
   - Verify error message appears

4. **List View**:
   - Create at least 3 members
   - Verify all appear in list view
   - Test search functionality
   - Test filters

---

## Deliverables

### Git Repository Setup

Before taking screenshots, initialize your Git repository:

```bash
cd apps/library_management

# Initialize git (if not already done)
git init

# Configure git
git config user.name "Your Name"
git config user.email "your.email@example.com"

# Create .gitignore
echo "*.pyc
__pycache__/
*.swp
.DS_Store" > .gitignore

# Initial commit
git add .
git commit -m "chore: initial commit"

# Create GitHub/GitLab repository and push
git remote add origin <your-repo-url>
git branch -M main
git push -u origin main
```

**Important**: Make your repository **public** so evaluators can access it.

### Screenshots Required

1. **DocType Form View**: Empty form showing all fields
2. **DocType List View**: List with at least 3 sample members
3. **Sample Member**: One filled member form

### Sample Data

Create at least 3 members with different membership types:
- 1 Standard member
- 1 Premium member  
- 1 VIP member

---

## Acceptance Criteria

- ✅ DocType created with correct name
- ✅ All 10 fields present with correct types
- ✅ Naming series works (LIB-MEM-.#####)
- ✅ Can create new members
- ✅ Can edit existing members
- ✅ Can delete members
- ✅ Email uniqueness enforced
- ✅ Search and filters work
- ✅ 3 screenshots provided

---

## Common Issues

### Issue: Naming series not working

**Solution**: 
- Check naming format is exactly: `LIB-MEM-.#####`
- Ensure "Auto Name" field is set to "Naming Series"

### Issue: Email not unique

**Solution**:
- Set field type to "Data"
- Set "Options" to "Email"
- Check "Unique" checkbox

### Issue: Fields not showing in form

**Solution**:
- Ensure fields are not hidden
- Check "In List View" and "In Standard Filter" if needed
- Clear cache: `bench --site library.localhost clear-cache`

---

## Next Steps

After completing this task:
1. ✅ Take required screenshots
2. ✅ Commit your changes:
   ```bash
   git add .
   git commit -m "feat(member): create Library Member DocType"
   git push origin main
   ```
3. ✅ Verify your public repository is accessible
4. ✅ Move to [Task 2: Book DocType](TASK_2_BOOK.md)
