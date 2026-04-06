# Task 3: Book Transaction DocType

**Estimated Time**: 3 hours  
**Difficulty**: Medium

---

## Objective

Create a DocType to record book checkouts and returns with basic validation logic.

---

## Requirements

### DocType Configuration

**DocType Name**: `Book Transaction`

**Module**: Library Management

**Naming Series**: `TXN-.#####`

**Settings**:
- [x] Allow Search
- [x] Allow Filters
- [x] Allow Sorting
- [x] Track Changes

---

## Fields

### 1. Transaction ID
- **Field Name**: `transaction_id`
- **Type**: Data
- **Label**: Transaction ID
- **Read Only**: Yes

### 2. Member
- **Field Name**: `member`
- **Type**: Link
- **Label**: Member
- **Options**: Library Member
- **Mandatory**: Yes
- **In List View**: Yes

### 3. Book
- **Field Name**: `book`
- **Type**: Link
- **Label**: Book
- **Options**: Book
- **Mandatory**: Yes
- **In List View**: Yes

### 4. Transaction Type
- **Field Name**: `transaction_type`
- **Type**: Select
- **Label**: Transaction Type
- **Options**:
  ```
  Issue
  Return
  ```
- **Mandatory**: Yes
- **In List View**: Yes

### 5. Transaction Date
- **Field Name**: `transaction_date`
- **Type**: Date
- **Label**: Transaction Date
- **Default**: Today
- **Mandatory**: Yes
- **In List View**: Yes

### 6. Due Date
- **Field Name**: `due_date`
- **Type**: Date
- **Label**: Due Date

### 7. Return Date
- **Field Name**: `return_date`
- **Type**: Date
- **Label**: Return Date

### 8. Status
- **Field Name**: `status`
- **Type**: Select
- **Label**: Status
- **Options**:
  ```
  Issued
  Returned
  Overdue
  ```
- **Default**: Issued
- **In List View**: Yes

---

## Business Logic

Add validation logic to the Python file.

### File Location

```
apps/library_management/library_management/doctype/book_transaction/book_transaction.py
```

### Validation Code

```python
# -*- coding: utf-8 -*-
# Copyright (c) 2024, [Your Name] and contributors
# For license information, please see license.txt

from __future__ import unicode_literals
import frappe
from frappe.model.document import Document
from frappe.utils import getdate

class BookTransaction(Document):
    def validate(self):
        """Validate transaction data"""
        
        # Validation 1: If Issue, due_date is mandatory
        if self.transaction_type == "Issue":
            if not self.due_date:
                frappe.throw("Due Date is mandatory for Issue transactions")
        
        # Validation 2: If Return, return_date is mandatory
        if self.transaction_type == "Return":
            if not self.return_date:
                frappe.throw("Return Date is mandatory for Return transactions")
        
        # Validation 3: return_date cannot be before transaction_date
        if self.return_date:
            if getdate(self.return_date) < getdate(self.transaction_date):
                frappe.throw("Return Date cannot be before Transaction Date")
```

---

## Implementation Steps

### Step 1: Create DocType

1. Go to: **Home → Customization → DocType → New**
2. Enter DocType Name: `Book Transaction`
3. Module: `Library Management`
4. Naming: `TXN-.#####`
5. Add all fields in order

### Step 2: Save DocType

Click **Save**.

### Step 3: Add Python Validation

1. Open `book_transaction.py`
2. Add the validation code above
3. Save the file

### Step 4: Restart & Clear Cache

```bash
# Docker
docker-compose -f pwd.yml exec backend bench --site library.localhost clear-cache
docker-compose -f pwd.yml restart backend

# Bare-metal
bench --site library.localhost clear-cache
# Restart bench (Ctrl+C then bench start)
```

---

## Testing

### Test Case 1: Issue Transaction

**Steps**:
1. Go to: Library Management → Book Transaction → New
2. Fill in:
   - Member: Select any active member
   - Book: Select any available book
   - Transaction Type: Issue
   - Transaction Date: Today
   - Due Date: (leave empty)
3. Try to save

**Expected**:
- Error: "Due Date is mandatory for Issue transactions"

**Fix & Retry**:
4. Set Due Date: 14 days from today
5. Save

**Expected**:
- Transaction saves successfully
- Status automatically set to "Issued"

### Test Case 2: Return Transaction

**Steps**:
1. Create new transaction
2. Fill in:
   - Member: Select member
   - Book: Select book
   - Transaction Type: Return
   - Transaction Date: Today
   - Return Date: (leave empty)
3. Try to save

**Expected**:
- Error: "Return Date is mandatory for Return transactions"

**Fix & Retry**:
4. Set Return Date: Today
5. Save

**Expected**:
- Transaction saves successfully

### Test Case 3: Invalid Return Date

**Steps**:
1. Create new transaction
2. Fill in:
   - Transaction Type: Return
   - Transaction Date: 2024-02-01
   - Return Date: 2024-01-15 (before transaction date)
3. Try to save

**Expected**:
- Error: "Return Date cannot be before Transaction Date"

---

## Deliverables

### Screenshots Required

1. **Issue Transaction Form**: Filled form with transaction_type = "Issue"
2. **Return Transaction Form**: Filled form with transaction_type = "Return"
3. **Validation Error**: Screenshot showing one of the validation error messages

### Sample Data

Create at least 5 transactions:
- 3 Issue transactions (with due dates)
- 2 Return transactions (with return dates)

Use different members and books for variety.

---

## Acceptance Criteria

- ✅ DocType created with all fields
- ✅ Link fields work (Member, Book)
- ✅ Naming series works (TXN-.#####)
- ✅ Validation: Due date required for Issue
- ✅ Validation: Return date required for Return
- ✅ Validation: Return date not before transaction date
- ✅ Can create Issue transactions
- ✅ Can create Return transactions
- ✅ 3 screenshots provided
- ✅ 5 sample transactions created

---

## Common Issues

### Issue: Link fields not showing options

**Solution**:
- Ensure "Library Member" and "Book" DocTypes exist
- Field type must be "Link"
- Options must match DocType name exactly
- Clear cache

### Issue: Validation not working

**Solution**:
```bash
# Check for Python syntax errors
bench --site library.localhost console

# In console:
from library_management.library_management.doctype.book_transaction.book_transaction import BookTransaction
# If no errors, validation code is correct

# Clear cache
bench --site library.localhost clear-cache
```

### Issue: Date comparison error

**Solution**:
- Import `getdate` from `frappe.utils`
- Use `getdate()` to convert date strings to date objects
- Example: `getdate(self.return_date) < getdate(self.transaction_date)`

---

## Bonus (Optional)

If you finish early, try adding:

1. **Auto-calculate due date**: Set due_date to 14 days from transaction_date for Issue transactions
2. **Status logic**: Auto-set status to "Returned" when transaction_type is "Return"

**Hint**: Use the `before_save` method like in the Book DocType.

---

## Next Steps

After completing this task:
1. ✅ Take required screenshots
2. ✅ Commit and push changes:
   ```bash
   git add .
   git commit -m "feat(transaction): create Book Transaction DocType with validation"
   git push origin main
   ```
3. ✅ Move to [Task 4: Testing Documentation](TASK_4_TESTING.md)
