# Git Workflow Guidelines

This document outlines the Git workflow and pull request standards for the technical assignment.

## Repository Setup

### Initial Setup

```bash
# Navigate to your custom app
cd apps/library_management

# Initialize git (if not already done)
git init

# Set your identity
git config user.name "Your Name"
git config user.email "your.email@example.com"

# Create initial commit
git add .
git commit -m "chore: Initial commit"

# Create main branch
git branch -M main
```

### Remote Repository

```bash
# Create a repository on GitHub/GitLab
# Then add remote
git remote add origin <your-repository-url>

# Push initial commit
git push -u origin main
```

---

## Branching Strategy

### Branch Naming

```bash
# Feature branches
feature/<task-name>
feature/book-checkout
feature/analytics-report

# Bug fix branches
bugfix/<issue-name>
bugfix/concurrent-checkout
bugfix/negative-copies

# Documentation branches
docs/<topic>
docs/setup-guide
```

### Creating Branches

```bash
# Always branch from main
git checkout main
git pull origin main

# Create feature branch
git checkout -b feature/book-checkout

# Work on your feature...

# Push branch to remote
git push -u origin feature/book-checkout
```

---

## Commit Guidelines

### Conventional Commits

Follow the [Conventional Commits](https://www.conventionalcommits.org/) specification:

**Format**: `<type>(<scope>): <subject>`

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, no logic change)
- `refactor`: Code refactoring
- `test`: Adding or updating tests
- `chore`: Maintenance tasks

**Examples**:

```bash
# Features
git commit -m "feat(book): create Book DocType with validation"
git commit -m "feat(transaction): add book checkout logic"
git commit -m "feat(report): implement library analytics report"

# Bug fixes
git commit -m "fix(book): prevent negative available_copies"
git commit -m "fix(transaction): correct fine calculation for overdue books"

# Documentation
git commit -m "docs(readme): add setup instructions"
git commit -m "docs(api): document public API methods"

# Tests
git commit -m "test(transaction): add concurrent checkout tests"
git commit -m "test(book): add validation test cases"

# Refactoring
git commit -m "refactor(transaction): extract fine calculation to utility"

# Chore
git commit -m "chore(deps): update frappe to v15.1"
```

### Atomic Commits

Make small, focused commits:

```bash
# ✅ Good - Atomic commits
git add library_management/doctype/book/
git commit -m "feat(book): create Book DocType"

git add library_management/doctype/book_transaction/
git commit -m "feat(transaction): create Book Transaction DocType"

git add library_management/doctype/book_transaction/book_transaction.py
git commit -m "feat(transaction): implement issue book logic"

# ❌ Bad - Monolithic commit
git add .
git commit -m "add all features"
```

### Commit Messages

**Good commit message structure**:

```
feat(transaction): add fine calculation for overdue books

- Calculate fine based on days overdue
- Premium members get 21 days, VIP get 30 days
- Fine rate: ₹10 per day
- Update transaction status to Overdue if applicable

Closes #12
```

**Guidelines**:
- First line: 50 characters or less
- Use imperative mood ("add" not "added")
- Body: Explain WHAT and WHY, not HOW
- Reference issues if applicable

---

## Pull Request Process

### Before Creating PR

```bash
# Ensure you're on your feature branch
git checkout feature/book-checkout

# Pull latest changes from main
git checkout main
git pull origin main
git checkout feature/book-checkout

# Rebase on main (optional but recommended)
git rebase main

# Run tests
bench --site library.localhost run-tests --app library_management

# Push to remote
git push origin feature/book-checkout
```

### Creating Pull Request

Use this template:

````markdown
## Summary

Brief description of what this PR does (2-3 sentences).

## Type of Change

- [ ] New feature
- [ ] Bug fix
- [ ] Documentation update
- [ ] Refactoring
- [ ] Performance improvement

## Changes Made

- Created Book Transaction DocType
- Implemented checkout/return logic
- Added fine calculation
- Created permissions for Librarian role
- Added client-side validation

## Related DocTypes/Files

- `Book Transaction` (new)
- `Book` (modified)
- `Library Member` (modified)

## Screenshots/GIFs

### Book Checkout Form
![Book Checkout](./screenshots/book-checkout.png)

### Validation Error
![Validation](./screenshots/validation-error.png)

## Testing Steps

1. Create a Library Member with status "Active"
2. Create a Book with 1 available copy
3. Navigate to Book Transaction → New
4. Select the member and book
5. Save the transaction
6. Verify:
   - Transaction created successfully
   - Book's available_copies decremented to 0
   - Due date calculated correctly

## Edge Cases Handled

- [x] Member with 3 active books cannot borrow more
- [x] Book with 0 copies cannot be issued
- [x] Suspended members cannot borrow books
- [x] Negative available_copies prevented
- [x] Duplicate transactions prevented

## Known Limitations

- Fine calculation doesn't account for library holidays
- No email notifications for overdue books
- Bulk return not yet implemented

## Performance Considerations

- Added index on `(member, transaction_date)` for faster queries
- Used single UPDATE query instead of multiple db_set calls
- Implemented database transaction for atomic operations

## Security Considerations

- Permission checks added for all operations
- SQL queries use parameterization
- User input validated before processing

## Tests Added

- [x] Unit tests for issue book logic
- [x] Unit tests for return book logic
- [x] Unit tests for fine calculation
- [x] Edge case tests for validation
- [ ] Integration tests (planned)

## Documentation Updated

- [x] Docstrings added to all functions
- [x] README updated with new features
- [ ] User guide (planned)

## Time Spent

- Setup: 30 minutes
- Implementation: 3 hours
- Testing: 1 hour
- Documentation: 30 minutes
- **Total: 5 hours**

## Checklist

- [ ] Code follows style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Tests added/updated and passing
- [ ] Documentation updated
- [ ] No console errors
- [ ] Screenshots attached
- [ ] Commit messages follow conventions

## Additional Notes

[Any other information reviewers should know]
````

### PR Best Practices

**Do**:
- ✅ One PR per feature/task
- ✅ Keep PRs focused and small
- ✅ Include screenshots for UI changes
- ✅ Write clear descriptions
- ✅ Reference related issues
- ✅ Respond to review comments promptly

**Don't**:
- ❌ Mix multiple unrelated changes
- ❌ Submit without testing
- ❌ Leave TODOs in production code
- ❌ Ignore review feedback
- ❌ Force push after review starts

---

## Code Review

### Responding to Feedback

```bash
# Make requested changes
git add <files>
git commit -m "fix: address review comments"

# Push changes
git push origin feature/book-checkout
```

### Updating PR

```bash
# If you need to add more commits
git add <files>
git commit -m "feat: add additional validation"
git push origin feature/book-checkout

# If you need to amend the last commit
git add <files>
git commit --amend --no-edit
git push origin feature/book-checkout --force-with-lease
```

---

## Commit History

### Viewing History

```bash
# View commit history
git log --oneline

# View with graph
git log --oneline --graph --all

# View specific file history
git log --oneline -- path/to/file.py
```

### Example Good History

```
a1b2c3d feat(report): add library analytics report
d4e5f6g test(report): add report test cases
g7h8i9j feat(transaction): implement fine calculation
j0k1l2m feat(transaction): add return book logic
m3n4o5p feat(transaction): add issue book logic
p6q7r8s feat(book): create Book DocType
s9t0u1v docs(setup): add installation instructions
v2w3x4y chore: initial commit
```

### Example Bad History

```
a1b2c3d fix typo
d4e5f6g more changes
g7h8i9j update
j0k1l2m WIP
m3n4o5p asdf
p6q7r8s final version
s9t0u1v final version 2
v2w3x4y actually final
```

---

## Git Best Practices

### Commit Often

```bash
# ✅ Good - Commit logical units
git add book.py
git commit -m "feat(book): add Book DocType"

git add book_transaction.py
git commit -m "feat(transaction): add Book Transaction DocType"

# ❌ Bad - One giant commit
git add .
git commit -m "add everything"
```

### Keep Commits Clean

```bash
# Before pushing, review your commits
git log --oneline

# If needed, squash WIP commits
git rebase -i HEAD~3

# In the interactive rebase, mark commits to squash:
# pick a1b2c3d feat(book): add Book DocType
# squash d4e5f6g WIP
# squash g7h8i9j fix typo
```

### Write Meaningful Messages

```bash
# ✅ Good
git commit -m "feat(transaction): prevent concurrent checkout of last copy

Use database row locking (SELECT FOR UPDATE) to prevent race condition
where two librarians simultaneously issue the last available copy.

Fixes #23"

# ❌ Bad
git commit -m "fix bug"
```

---

## Common Git Commands

### Checking Status

```bash
# View current status
git status

# View diff of unstaged changes
git diff

# View diff of staged changes
git diff --staged
```

### Stashing Changes

```bash
# Stash current changes
git stash

# List stashes
git stash list

# Apply latest stash
git stash pop

# Apply specific stash
git stash apply stash@{1}
```

### Undoing Changes

```bash
# Discard unstaged changes to a file
git checkout -- <file>

# Unstage a file
git reset HEAD <file>

# Amend last commit
git commit --amend

# Reset to previous commit (keep changes)
git reset --soft HEAD~1

# Reset to previous commit (discard changes)
git reset --hard HEAD~1
```

---

## Submission Checklist

Before submitting your assignment:

- [ ] All commits follow conventional commit format
- [ ] Commit history is clean and logical
- [ ] Each PR has detailed description
- [ ] Screenshots included for UI changes
- [ ] All tests passing
- [ ] No debug code or console.log statements
- [ ] No merge conflicts
- [ ] Branch is up to date with main

---

## Resources

- [Conventional Commits](https://www.conventionalcommits.org/)
- [Git Best Practices](https://git-scm.com/book/en/v2)
- [Writing Good Commit Messages](https://chris.beams.io/posts/git-commit/)
