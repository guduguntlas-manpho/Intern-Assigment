# Testing Guidelines

This document outlines testing expectations for different seniority levels.

## Testing Philosophy

- **Write tests that provide value**: Focus on business logic, edge cases, and critical paths
- **Test behavior, not implementation**: Tests should verify outcomes, not internal details
- **Keep tests maintainable**: Clear, readable tests are easier to maintain

---

## Entry Level: Manual Testing

### Requirements

Create a `TESTING.md` file documenting all manual test cases.

### Test Case Format

```markdown
## Test Case: [Feature Name]

**Objective**: What you're testing

**Prerequisites**:
- List any setup required

**Steps**:
1. Step one
2. Step two
3. Step three

**Expected Result**:
- What should happen

**Actual Result**:
- What actually happened

**Status**: ✅ Pass / ❌ Fail
```

### Example Test Cases

````markdown
## Test Case: Issue Book to Active Member

**Objective**: Verify that an active member can successfully borrow an available book

**Prerequisites**:
- Library Member exists with status "Active"
- Book exists with available_copies > 0

**Steps**:
1. Navigate to Book Transaction → New
2. Select the active member
3. Select an available book
4. Set transaction_type to "Issue"
5. Click Save

**Expected Result**:
- Transaction saves successfully
- Book's available_copies decremented by 1
- Due date calculated and set
- Status set to "Issued"

**Actual Result**:
- Transaction saved successfully
- Available copies: 5 → 4
- Due date: 2024-02-14 (14 days from today)
- Status: Issued

**Status**: ✅ Pass

---

## Test Case: Prevent Inactive Member from Borrowing

**Objective**: Verify that inactive members cannot borrow books

**Prerequisites**:
- Library Member exists with status "Inactive"
- Book exists with available_copies > 0

**Steps**:
1. Navigate to Book Transaction → New
2. Select the inactive member
3. Select an available book
4. Set transaction_type to "Issue"
5. Click Save

**Expected Result**:
- Validation error displayed
- Message: "Cannot issue book to inactive member"
- Transaction not saved

**Actual Result**:
- Error message displayed: "Cannot issue book to inactive member LIB-MEM-00001"
- Transaction not saved

**Status**: ✅ Pass
````

### Minimum Test Coverage

- [ ] Happy path for all features
- [ ] Basic validation errors
- [ ] Permission checks
- [ ] UI functionality

---

## Junior Level: Basic Unit Tests

### Requirements

- Write unit tests for all DocType methods
- Minimum 60% code coverage
- Tests must pass before submission

### Test File Structure

```python
# library_management/library_management/doctype/book_transaction/test_book_transaction.py

import frappe
import unittest
from frappe.utils import today, add_days

class TestBookTransaction(unittest.TestCase):
    
    def setUp(self):
        """Set up test data before each test"""
        self.member = create_test_member()
        self.book = create_test_book()
    
    def tearDown(self):
        """Clean up after each test"""
        frappe.db.rollback()
    
    def test_issue_book_decrements_available_copies(self):
        """Test that issuing a book decrements available copies"""
        initial_copies = self.book.available_copies
        
        transaction = create_transaction(
            member=self.member.name,
            book=self.book.name,
            transaction_type="Issue"
        )
        
        self.book.reload()
        self.assertEqual(
            self.book.available_copies,
            initial_copies - 1,
            "Available copies should be decremented by 1"
        )
    
    def test_return_book_increments_available_copies(self):
        """Test that returning a book increments available copies"""
        # First issue the book
        transaction = create_transaction(
            member=self.member.name,
            book=self.book.name,
            transaction_type="Issue"
        )
        
        copies_after_issue = self.book.reload().available_copies
        
        # Return the book
        transaction.transaction_type = "Return"
        transaction.return_date = today()
        transaction.save()
        
        self.book.reload()
        self.assertEqual(
            self.book.available_copies,
            copies_after_issue + 1,
            "Available copies should be incremented by 1"
        )

# Helper functions
def create_test_member(**kwargs):
    """Create a test library member"""
    defaults = {
        "doctype": "Library Member",
        "first_name": "Test",
        "last_name": "Member",
        "email": f"test{frappe.generate_hash()}@example.com",
        "membership_type": "Standard",
        "membership_start_date": today(),
        "status": "Active"
    }
    defaults.update(kwargs)
    
    member = frappe.get_doc(defaults)
    member.insert()
    return member

def create_test_book(**kwargs):
    """Create a test book"""
    defaults = {
        "doctype": "Book",
        "title": f"Test Book {frappe.generate_hash(length=5)}",
        "author": "Test Author",
        "isbn": f"978-{frappe.generate_hash(length=10)}",
        "total_copies": 5,
        "available_copies": 5,
        "status": "Available"
    }
    defaults.update(kwargs)
    
    book = frappe.get_doc(defaults)
    book.insert()
    return book

def create_transaction(**kwargs):
    """Create a test transaction"""
    defaults = {
        "doctype": "Book Transaction",
        "transaction_date": today()
    }
    defaults.update(kwargs)
    
    transaction = frappe.get_doc(defaults)
    transaction.insert()
    return transaction
```

### Running Tests

```bash
# Run all tests for your app
bench --site library.localhost run-tests --app library_management

# Run specific test file
bench --site library.localhost run-tests --app library_management --module library_management.doctype.book_transaction.test_book_transaction

# Run with coverage
bench --site library.localhost run-tests --app library_management --coverage
```

### Minimum Test Coverage

- [ ] All DocType validation methods
- [ ] Business logic methods
- [ ] Basic edge cases
- [ ] 60%+ code coverage

---

## Mid Level: Comprehensive Testing

### Requirements

- Unit tests for all logic
- Integration tests for workflows
- Edge case coverage
- Minimum 75% code coverage

### Additional Test Types

#### Integration Tests

```python
def test_complete_checkout_workflow(self):
    """Test the complete checkout and return workflow"""
    # Create member and book
    member = create_test_member(membership_type="Premium")
    book = create_test_book(total_copies=1, available_copies=1)
    
    # Issue book
    issue_txn = create_transaction(
        member=member.name,
        book=book.name,
        transaction_type="Issue"
    )
    
    # Verify book state
    book.reload()
    self.assertEqual(book.available_copies, 0)
    self.assertEqual(issue_txn.status, "Issued")
    
    # Verify due date (Premium = 21 days)
    expected_due_date = add_days(today(), 21)
    self.assertEqual(issue_txn.due_date, expected_due_date)
    
    # Try to issue same book again (should fail)
    with self.assertRaises(frappe.ValidationError):
        create_transaction(
            member=member.name,
            book=book.name,
            transaction_type="Issue"
        )
    
    # Return book after due date
    issue_txn.transaction_type = "Return"
    issue_txn.return_date = add_days(expected_due_date, 5)
    issue_txn.save()
    
    # Verify fine calculation
    expected_fine = 5 * 10  # 5 days * ₹10
    self.assertEqual(issue_txn.fine_amount, expected_fine)
    
    # Verify book available again
    book.reload()
    self.assertEqual(book.available_copies, 1)
```

#### Edge Case Tests

```python
def test_member_borrowing_limit(self):
    """Test that members cannot borrow more than 3 books"""
    member = create_test_member()
    
    # Issue 3 books
    for i in range(3):
        book = create_test_book()
        create_transaction(
            member=member.name,
            book=book.name,
            transaction_type="Issue"
        )
    
    # Try to issue 4th book
    book4 = create_test_book()
    with self.assertRaises(frappe.ValidationError) as context:
        create_transaction(
            member=member.name,
            book=book4.name,
            transaction_type="Issue"
        )
    
    self.assertIn("borrowing limit", str(context.exception).lower())

def test_zero_available_copies(self):
    """Test that books with 0 copies cannot be issued"""
    book = create_test_book(total_copies=1, available_copies=0)
    member = create_test_member()
    
    with self.assertRaises(frappe.ValidationError):
        create_transaction(
            member=member.name,
            book=book.name,
            transaction_type="Issue"
        )
```

### Minimum Test Coverage

- [ ] All business logic methods
- [ ] Complete workflows
- [ ] Edge cases and error conditions
- [ ] Permission checks
- [ ] 75%+ code coverage

---

## Senior Level: Production-Grade Testing

### Requirements

- Comprehensive unit and integration tests
- Concurrency and race condition tests
- Performance tests
- Security tests
- Minimum 85% code coverage

### Advanced Test Types

#### Concurrency Tests

```python
import threading

def test_concurrent_checkout_prevention(self):
    """Test that concurrent checkouts don't create negative inventory"""
    book = create_test_book(total_copies=1, available_copies=1)
    member1 = create_test_member()
    member2 = create_test_member()
    
    results = []
    errors = []
    
    def issue_book(member):
        try:
            txn = create_transaction(
                member=member.name,
                book=book.name,
                transaction_type="Issue"
            )
            results.append(txn.name)
        except Exception as e:
            errors.append(str(e))
    
    # Simulate concurrent requests
    t1 = threading.Thread(target=issue_book, args=(member1,))
    t2 = threading.Thread(target=issue_book, args=(member2,))
    
    t1.start()
    t2.start()
    t1.join()
    t2.join()
    
    # Only one should succeed
    self.assertEqual(len(results), 1, "Only one transaction should succeed")
    self.assertEqual(len(errors), 1, "One transaction should fail")
    
    # Verify available_copies is 0, not negative
    book.reload()
    self.assertEqual(book.available_copies, 0)
    self.assertGreaterEqual(book.available_copies, 0, "Copies should never be negative")
```

#### Performance Tests

```python
import time

def test_report_performance(self):
    """Test that report performs well with large datasets"""
    # Create test data
    members = [create_test_member() for _ in range(10)]
    books = [create_test_book() for _ in range(50)]
    
    # Create 1000 transactions
    for i in range(1000):
        member = members[i % len(members)]
        book = books[i % len(books)]
        create_transaction(
            member=member.name,
            book=book.name,
            transaction_type="Issue" if i % 2 == 0 else "Return"
        )
    
    # Measure report execution time
    start_time = time.time()
    
    result = frappe.get_list(
        "Book Transaction",
        filters={"transaction_date": [">=", add_days(today(), -30)]},
        fields=["name", "member", "book", "transaction_date"]
    )
    
    execution_time = time.time() - start_time
    
    # Should complete in under 2 seconds
    self.assertLess(execution_time, 2.0, f"Report took {execution_time}s, should be < 2s")
    self.assertGreater(len(result), 0, "Report should return results")
```

#### Security Tests

```python
def test_permission_enforcement(self):
    """Test that permissions are properly enforced"""
    # Create a member user (not librarian)
    member_user = create_test_user(role="Library Member")
    
    frappe.set_user(member_user.name)
    
    # Member should not be able to create transactions
    with self.assertRaises(frappe.PermissionError):
        create_transaction(
            member="LIB-MEM-00001",
            book="BOOK-00001",
            transaction_type="Issue"
        )
    
    # Reset to Administrator
    frappe.set_user("Administrator")

def test_sql_injection_prevention(self):
    """Test that SQL injection is prevented"""
    malicious_input = "'; DROP TABLE `tabBook`; --"
    
    # This should not execute the DROP TABLE
    result = frappe.get_all(
        "Book",
        filters={"title": malicious_input}
    )
    
    # Verify Book table still exists
    self.assertTrue(frappe.db.table_exists("tabBook"))
```

### Test Organization

```
library_management/
└── library_management/
    └── doctype/
        └── book_transaction/
            ├── test_book_transaction.py           # Unit tests
            ├── test_book_transaction_integration.py  # Integration tests
            ├── test_book_transaction_performance.py  # Performance tests
            └── test_book_transaction_security.py     # Security tests
```

### Minimum Test Coverage

- [ ] All business logic (100%)
- [ ] Integration tests for workflows
- [ ] Concurrency tests
- [ ] Performance benchmarks
- [ ] Security tests
- [ ] 85%+ overall code coverage

---

## Test Best Practices

### Do's

- ✅ Use descriptive test names
- ✅ Test one thing per test
- ✅ Use setUp and tearDown
- ✅ Clean up test data
- ✅ Use assertions with messages
- ✅ Test edge cases
- ✅ Mock external dependencies

### Don'ts

- ❌ Don't test framework code
- ❌ Don't write brittle tests
- ❌ Don't skip cleanup
- ❌ Don't test implementation details
- ❌ Don't leave failing tests

---

## Running Tests

```bash
# All tests
bench --site library.localhost run-tests --app library_management

# Specific module
bench --site library.localhost run-tests --app library_management --module book_transaction

# With coverage
bench --site library.localhost run-tests --app library_management --coverage

# Verbose output
bench --site library.localhost run-tests --app library_management --verbose

# Fail fast (stop on first failure)
bench --site library.localhost run-tests --app library_management --failfast
```

---

## Coverage Report

```bash
# Generate coverage report
bench --site library.localhost run-tests --app library_management --coverage

# View HTML report
cd apps/library_management
python -m http.server 8001
# Open http://localhost:8001/htmlcov in browser
```

---

## Submission Requirements

Include in your submission:

- [ ] All test files
- [ ] Test execution output (screenshot or log)
- [ ] Coverage report (screenshot)
- [ ] Any test data fixtures used
