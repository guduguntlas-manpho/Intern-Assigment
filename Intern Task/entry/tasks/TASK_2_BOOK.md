# Task 2: Book DocType

**Estimated Time**: 2 hours  
**Difficulty**: Easy-Medium

---

## Objective

Create a DocType to manage library books with inventory tracking and basic validation logic.

---

## Requirements

### DocType Configuration

**DocType Name**: `Book`

**Module**: Library Management

**Naming Series**: `BOOK-.#####`

**Settings**:
- [x] Allow Search
- [x] Allow Filters
- [x] Allow Sorting
- [x] Track Changes

---

## Fields

### 1. Book ID
- **Field Name**: `book_id`
- **Type**: Data
- **Label**: Book ID
- **Read Only**: Yes

### 2. Title
- **Field Name**: `title`
- **Type**: Data
- **Label**: Book Title
- **Mandatory**: Yes
- **In List View**: Yes

### 3. Author
- **Field Name**: `author`
- **Type**: Data
- **Label**: Author
- **Mandatory**: Yes
- **In List View**: Yes

### 4. ISBN
- **Field Name**: `isbn`
- **Type**: Data
- **Label**: ISBN
- **Unique**: Yes

### 5. Publisher
- **Field Name**: `publisher`
- **Type**: Data
- **Label**: Publisher

### 6. Publication Year
- **Field Name**: `publication_year`
- **Type**: Int
- **Label**: Publication Year

### 7. Category
- **Field Name**: `category`
- **Type**: Select
- **Label**: Category
- **Options**:
  ```
  Fiction
  Non-Fiction
  Science
  Technology
  History
  Biography
  Other
  ```
- **In List View**: Yes

### 8. Total Copies
- **Field Name**: `total_copies`
- **Type**: Int
- **Label**: Total Copies
- **Default**: 1
- **Mandatory**: Yes

### 9. Available Copies
- **Field Name**: `available_copies`
- **Type**: Int
- **Label**: Available Copies
- **Read Only**: Yes
- **In List View**: Yes

### 10. Status
- **Field Name**: `status`
- **Type**: Select
- **Label**: Status
- **Options**:
  ```
  Available
  Checked Out
  Maintenance
  ```
- **Default**: Available
- **In List View**: Yes

---

## Business Logic

You need to add Python code to handle inventory logic.

### Step 1: Create Python File

The file should already exist at:
```
apps/library_management/library_management/doctype/book/book.py
```

### Step 2: Add Validation Logic

Edit `book.py` and add this logic:

```python
# -*- coding: utf-8 -*-
# Copyright (c) 2024, [Your Name] and contributors
# For license information, please see license.txt

from __future__ import unicode_literals
import frappe
from frappe.model.document import Document

class Book(Document):
    def before_save(self):
        """Set available copies on new book creation"""
        # If this is a new book, set available_copies = total_copies
        if self.is_new():
            self.available_copies = self.total_copies
        
        # Ensure available_copies never exceeds total_copies
        if self.available_copies > self.total_copies:
            self.available_copies = self.total_copies
    
    def validate(self):
        """Validate book data"""
        # Ensure total_copies is positive
        if self.total_copies < 0:
            frappe.throw("Total copies cannot be negative")
        
        # Ensure available_copies is not negative
        if self.available_copies < 0:
            frappe.throw("Available copies cannot be negative")
```

### Step 3: Test the Logic

After adding the code:
1. Restart bench (if bare-metal) or restart container (if Docker)
2. Clear cache: `bench --site library.localhost clear-cache`
3. Test creating a new book

---

## Implementation Steps

### Step 1: Create DocType via UI

1. Go to: **Home → Customization → DocType → New**
2. Enter DocType Name: `Book`
3. Module: `Library Management`
4. Naming: `BOOK-.#####`
5. Add all fields

### Step 2: Save DocType

Click **Save** to create the DocType.

### Step 3: Add Python Logic

1. Navigate to the book.py file
2. Add the validation logic shown above
3. Save the file

### Step 4: Restart & Test

```bash
# Docker
docker-compose -f pwd.yml restart backend

# Bare-metal
# Press Ctrl+C in terminal running bench
bench start
```

---

## Testing

### Test Case 1: Create New Book

**Steps**:
1. Go to: Library Management → Book → New
2. Fill in:
   - Title: "The Great Gatsby"
   - Author: "F. Scott Fitzgerald"
   - ISBN: "978-0-7432-7356-5"
   - Category: Fiction
   - Total Copies: 5
3. Save

**Expected**:
- Book saves successfully
- `available_copies` automatically set to 5
- `book_id` auto-generated (e.g., BOOK-00001)

### Test Case 2: Edit Total Copies

**Steps**:
1. Open an existing book
2. Change `total_copies` from 5 to 3
3. Save

**Expected**:
- If `available_copies` was 5, it should become 3
- Validation ensures available never exceeds total

### Test Case 3: Negative Copies

**Steps**:
1. Try to set `total_copies` to -1
2. Save

**Expected**:
- Error message: "Total copies cannot be negative"
- Book not saved

---

## Deliverables

### Screenshots Required

1. **Book Form View**: Empty form showing all fields
2. **Book List View**: List with at least 5 sample books
3. **Validation Working**: Screenshot showing available_copies auto-set

### Sample Data

Create at least 5 books with different categories:

| Title | Author | Category | Total Copies |
|-------|--------|----------|--------------|
| 1984 | George Orwell | Fiction | 3 |
| Sapiens | Yuval Noah Harari | Non-Fiction | 2 |
| A Brief History of Time | Stephen Hawking | Science | 1 |
| Clean Code | Robert C. Martin | Technology | 4 |
| The Diary of a Young Girl | Anne Frank | Biography | 2 |

---

## Acceptance Criteria

- ✅ DocType created with all fields
- ✅ Naming series works (BOOK-.#####)
- ✅ Python validation logic implemented
- ✅ `available_copies` auto-set on creation
- ✅ `available_copies` never exceeds `total_copies`
- ✅ Negative copies validation works
- ✅ Can create, edit, delete books
- ✅ 3 screenshots provided
- ✅ 5 sample books created

---

## Common Issues

### Issue: Python code not executing

**Solution**:
```bash
# Clear cache
bench --site library.localhost clear-cache

# Restart bench
bench restart
```

### Issue: Syntax errors in Python

**Solution**:
- Check indentation (use spaces, not tabs)
- Ensure all colons are present
- Check for typos in method names

### Issue: available_copies not updating

**Solution**:
- Verify `before_save` method is defined correctly
- Check that you're calling `self.available_copies = self.total_copies`
- Clear cache and restart

---

## Next Steps

After completing this task:
1. ✅ Take required screenshots
2. ✅ Commit and push changes:
   ```bash
   git add .
   git commit -m "feat(book): create Book DocType with inventory logic"
   git push origin main
   ```
3. ✅ Move to [Task 3: Book Transaction DocType](TASK_3_TRANSACTION.md)
