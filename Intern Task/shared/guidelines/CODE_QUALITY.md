# Code Quality Guidelines

This document defines the code quality standards expected for the ERPNext technical assignment.

## Python (Backend)

### Naming Conventions

```python
# Classes: PascalCase
class LibraryMember:
    pass

class BookTransaction:
    pass

# Functions/Methods: snake_case
def calculate_fine_amount(transaction):
    pass

def get_active_books(member):
    pass

# Constants: UPPER_SNAKE_CASE
MAX_BOOKS_PER_MEMBER = 3
FINE_PER_DAY = 10
DEFAULT_LOAN_PERIOD = 14

# Private methods: _leading_underscore
def _validate_internal_state(self):
    pass

# Variables: snake_case
member_name = "John Doe"
total_fine = 100
```

### Docstrings

Use Google-style docstrings:

```python
def issue_book(member, book, transaction_date=None):
    """
    Issue a book to a library member.
    
    This function creates a new book transaction, validates member eligibility,
    checks book availability, and updates the available copies count.
    
    Args:
        member (str): Member document name/ID
        book (str): Book document name/ID
        transaction_date (date, optional): Transaction date. Defaults to today.
    
    Returns:
        str: Transaction document name
    
    Raises:
        frappe.ValidationError: If member is inactive or book unavailable
        frappe.PermissionError: If user lacks permission to issue books
    
    Example:
        >>> issue_book("LIB-MEM-00001", "BOOK-00001")
        'TXN-00001'
    """
    pass
```

### Validation Patterns

```python
# Validate in the validate() method
def validate(self):
    """Called before insert/update"""
    self.validate_member_status()
    self.validate_book_availability()
    self.calculate_due_date()
    self.set_status()

def validate_member_status(self):
    """Validate member is active"""
    if self.member:
        member = frappe.get_doc("Library Member", self.member)
        if member.status != "Active":
            frappe.throw(
                _("Cannot issue book to inactive member {0}").format(self.member),
                frappe.ValidationError
            )

# Use frappe.throw for user-facing errors
def validate_book_availability(self):
    """Validate book has available copies"""
    book = frappe.get_doc("Book", self.book)
    if book.available_copies < 1:
        frappe.throw(
            _("Book {0} is not available").format(book.title),
            frappe.ValidationError
        )
```

### Error Handling

```python
# ✅ Good - Specific exceptions
try:
    book.db_set("available_copies", new_count)
except frappe.exceptions.LinkValidationError as e:
    frappe.log_error(f"Invalid book reference: {str(e)}")
    frappe.throw(_("Invalid book reference"))
except frappe.exceptions.ValidationError as e:
    frappe.throw(_("Validation failed: {0}").format(str(e)))

# ✅ Good - Log errors before re-raising
try:
    process_transaction(transaction)
except Exception as e:
    frappe.log_error(
        message=frappe.get_traceback(),
        title=f"Transaction Processing Failed: {transaction.name}"
    )
    raise

# ❌ Bad - Bare except
try:
    process_transaction()
except:
    pass

# ❌ Bad - Swallowing exceptions
try:
    risky_operation()
except Exception:
    return None
```

### Security Best Practices

```python
# ✅ Good - Parameterized queries
frappe.db.sql("""
    SELECT name, title, author
    FROM `tabBook`
    WHERE category = %s AND status = %s
""", (category, status))

# ✅ Good - Using ORM (preferred)
frappe.get_all(
    "Book",
    filters={"category": category, "status": status},
    fields=["name", "title", "author"]
)

# ❌ Bad - String formatting (SQL injection risk!)
frappe.db.sql(f"SELECT * FROM `tabBook` WHERE name = '{book_name}'")

# ✅ Good - Check permissions
if not frappe.has_permission("Book Transaction", "write"):
    frappe.throw(_("Insufficient permissions"), frappe.PermissionError)

# ✅ Good - Validate user input
def validate_isbn(isbn):
    """Validate ISBN format"""
    import re
    pattern = r'^(?:ISBN(?:-1[03])?:? )?(?=[0-9X]{10}$|(?=(?:[0-9]+[- ]){3})[- 0-9X]{13}$|97[89][0-9]{10}$|(?=(?:[0-9]+[- ]){4})[- 0-9]{17}$)(?:97[89][- ]?)?[0-9]{1,5}[- ]?[0-9]+[- ]?[0-9]+[- ]?[0-9X]$'
    if not re.match(pattern, isbn):
        frappe.throw(_("Invalid ISBN format"))
```

### Performance

```python
# ✅ Good - Single query with JOIN
def get_transactions_with_details(filters):
    return frappe.db.sql("""
        SELECT 
            bt.name, bt.transaction_date, bt.status,
            b.title, b.author,
            m.first_name, m.last_name
        FROM `tabBook Transaction` bt
        INNER JOIN `tabBook` b ON bt.book = b.name
        INNER JOIN `tabLibrary Member` m ON bt.member = m.name
        WHERE bt.status = %(status)s
        ORDER BY bt.transaction_date DESC
    """, filters, as_dict=True)

# ❌ Bad - N+1 queries
def get_transactions_with_details_bad(status):
    transactions = frappe.get_all("Book Transaction", filters={"status": status})
    for txn in transactions:
        txn.book_details = frappe.get_doc("Book", txn.book)  # N queries!
        txn.member_details = frappe.get_doc("Library Member", txn.member)  # N queries!
    return transactions

# ✅ Good - Bulk operations
def update_multiple_books(book_names, status):
    frappe.db.sql("""
        UPDATE `tabBook`
        SET status = %s
        WHERE name IN ({})
    """.format(','.join(['%s'] * len(book_names))), 
    tuple([status] + book_names))

# ✅ Good - Use caching for frequently accessed data
@frappe.whitelist()
def get_library_settings():
    return frappe.cache().get_value(
        "library_settings",
        generator=lambda: frappe.get_single("Library Settings")
    )
```

---

## JavaScript (Frontend)

### Form Scripts

```javascript
// Use frappe.ui.form.on for form events
frappe.ui.form.on('Book Transaction', {
    // Refresh event - runs when form loads
    refresh: function(frm) {
        // Add custom buttons
        if (frm.doc.status === "Issued") {
            frm.add_custom_button(__('Return Book'), function() {
                frm.trigger('return_book');
            });
        }
        
        // Set field properties
        frm.set_df_property('fine_amount', 'read_only', 1);
    },
    
    // Field change events
    member: function(frm) {
        if (frm.doc.member) {
            frm.trigger('check_member_eligibility');
        }
    },
    
    book: function(frm) {
        if (frm.doc.book) {
            frm.trigger('check_book_availability');
        }
    },
    
    // Custom methods
    check_member_eligibility: function(frm) {
        frappe.call({
            method: 'library_management.api.get_member_active_books',
            args: {
                member: frm.doc.member
            },
            callback: function(r) {
                if (r.message >= 3) {
                    frappe.msgprint({
                        title: __('Borrowing Limit Reached'),
                        message: __('Member has already borrowed 3 books'),
                        indicator: 'red'
                    });
                    frm.set_value('member', '');
                }
            }
        });
    },
    
    return_book: function(frm) {
        frappe.confirm(
            __('Are you sure you want to return this book?'),
            function() {
                // On yes
                frm.set_value('transaction_type', 'Return');
                frm.set_value('return_date', frappe.datetime.get_today());
                frm.save();
            }
        );
    }
});
```

### Best Practices

```javascript
// ✅ Good - Use frm.set_value
frm.set_value('fine_amount', calculated_fine);

// ❌ Bad - Direct assignment
frm.doc.fine_amount = calculated_fine;

// ✅ Good - Use frappe.call for server methods
frm.call({
    method: 'calculate_fine',
    doc: frm.doc,
    callback: function(r) {
        if (r.message) {
            frm.set_value('fine_amount', r.message);
        }
    }
});

// ✅ Good - Scoped functions
frappe.ui.form.on('Book Transaction', {
    calculate_days_overdue: function(frm) {
        const due_date = frappe.datetime.str_to_obj(frm.doc.due_date);
        const return_date = frappe.datetime.str_to_obj(frm.doc.return_date);
        const diff = frappe.datetime.get_day_diff(return_date, due_date);
        return Math.max(0, diff);
    }
});

// ❌ Bad - Global function
function calculate_days_overdue(due_date, return_date) {
    // Pollutes global namespace
}

// ✅ Good - User feedback
frappe.show_alert({
    message: __('Book issued successfully'),
    indicator: 'green'
}, 5);

// ✅ Good - Disable fields conditionally
if (frm.doc.status === "Returned") {
    frm.set_df_property('return_date', 'read_only', 1);
}
```

---

## Database

### Indexing

Add indexes for frequently queried fields:

```python
# In DocType JSON or via migration
{
    "index": [
        ["member", "transaction_date"],
        ["book", "status"],
        ["status", "due_date"]
    ]
}
```

### Query Optimization

```python
# ✅ Good - Specific fields
frappe.db.get_value(
    "Book",
    {"name": book_name},
    ["title", "author", "available_copies"]
)

# ❌ Bad - SELECT *
frappe.db.sql("SELECT * FROM `tabBook` WHERE name = %s", book_name)

# ✅ Good - Use LIMIT for large datasets
frappe.db.sql("""
    SELECT name, title
    FROM `tabBook`
    WHERE status = 'Available'
    ORDER BY creation DESC
    LIMIT 100
""")

# ✅ Good - Use EXISTS for checking existence
exists = frappe.db.exists("Book Transaction", {
    "member": member,
    "book": book,
    "status": "Issued"
})
```

---

## File Organization

### DocType Structure

```
library_management/
└── library_management/
    └── doctype/
        └── book_transaction/
            ├── __init__.py
            ├── book_transaction.py          # Server-side logic
            ├── book_transaction.js          # Client-side logic
            ├── book_transaction.json        # DocType definition
            ├── test_book_transaction.py     # Unit tests
            └── book_transaction_list.js     # List view customization (optional)
```

### API Organization

```
library_management/
└── library_management/
    ├── api.py                  # Public API methods
    ├── utils.py                # Utility functions
    └── validations.py          # Shared validation logic
```

---

## Comments

```python
# ✅ Good - Explain WHY, not WHAT
# Use optimistic locking to prevent race conditions when multiple
# librarians try to issue the last available copy simultaneously
with frappe.db.transaction():
    book = frappe.get_doc("Book", book_name, for_update=True)
    if book.available_copies > 0:
        book.available_copies -= 1
        book.save()

# ❌ Bad - Obvious comment
# Decrement available copies by 1
book.available_copies -= 1

# ✅ Good - Document complex business logic
# Premium members get 21 days, VIP get 30 days, standard get 14 days
# This is based on the library policy document dated 2024-01-15
loan_periods = {
    "Standard": 14,
    "Premium": 21,
    "VIP": 30
}
```

---

## Code Review Checklist

Before submitting, verify:

- [ ] All functions have docstrings
- [ ] No hardcoded values (use constants or settings)
- [ ] Error messages are user-friendly and translatable
- [ ] No console.log or print statements (use frappe.log_error)
- [ ] Permissions checked before sensitive operations
- [ ] SQL queries use parameterization
- [ ] No N+1 query patterns
- [ ] Form scripts use frm.set_value, not direct assignment
- [ ] Code follows naming conventions
- [ ] Complex logic has explanatory comments
