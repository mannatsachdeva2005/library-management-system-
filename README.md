# Library Management System

A console-based Library Management System built in core Java, demonstrating
OOP principles (inheritance, polymorphism, abstraction, encapsulation),
custom exception handling, collections, and file-based persistence.

## Overview

The system lets a library maintain a book catalog, register members, and
manage the full lifecycle of issuing and returning books — including
automatic overdue-fine calculation. All data is persisted to plain text
files so it survives between runs, with every significant action recorded
to an activity log.

## Features

- **Catalog Management** — add, search (by title, case-insensitive
  substring match), list (alphabetically sorted), and remove books.
  Each title tracks total vs. currently available copies.
- **Member Management** — register and list members; each member is
  capped at 3 books borrowed at once.
- **Circulation (Issue / Return)** — issue a book to a member (blocked if
  no copies are available or the member is at their limit), return a book,
  and automatically calculate an overdue fine (₹5/day past a 14-day loan
  period).
- View a specific member's active (unreturned) loans.
- Persistent storage: books, members, and transactions are saved to
  `data/*.txt` and reloaded automatically on the next run.
- Activity logging to `data/activity.log` for monitoring/auditing.
- Input validation and custom checked exceptions
  (`BookNotFoundException`, `BookNotAvailableException`,
  `MemberNotFoundException`, `DuplicateEntryException`) with friendly
  console error messages instead of stack traces.

## Technologies / Tools Used

- Java 11+ (Standard Edition, no external dependencies)
- `java.time` for date handling and fine calculation
- `java.nio.file` for file I/O
- Git for version control

## Project Structure

```
LibraryManagementSystem/
├── src/
│   └── library/
│       ├── Main.java              # Console menu / entry point
│       ├── Library.java           # Core engine – all 3 functional modules
│       ├── Person.java            # Abstract base class
│       ├── Member.java            # extends Person
│       ├── Librarian.java         # extends Person
│       ├── Book.java              # Comparable + Persistable
│       ├── Transaction.java       # Issue/return + fine logic
│       ├── Persistable.java       # Interface for file serialization
│       ├── FileManager.java       # Generic file read/write helper
│       ├── Logger.java            # Append-only activity logger
│       └── exceptions/
│           ├── BookNotFoundException.java
│           ├── BookNotAvailableException.java
│           ├── MemberNotFoundException.java
│           └── DuplicateEntryException.java
├── data/                          # Auto-created: books.txt, members.txt,
│                                   # transactions.txt, activity.log
├── README.md
└── statement.md
```

## Steps to Install & Run

1. Ensure a JDK (11 or later) is installed: `java -version` / `javac -version`
2. From the project root, compile all sources:
   ```
   javac -d out $(find src -name "*.java")
   ```
   (On Windows, list the files explicitly or use a wildcard per folder.)
3. Run the program:
   ```
   java -cp out library.Main
   ```
4. Follow the on-screen menu. Data files are created automatically under
   `data/` on first run.

## Instructions for Testing

Manual test flow using the console menu:

1. **Add Book** — add a book, e.g. ISBN `B001`, title `Clean Code`,
   author `Robert Martin`, genre `Software`, copies `2`.
2. **Register Member** — register a member, e.g. ID `M001`, name `Asha`.
3. **Issue Book** — issue `B001` to `M001`. Note the transaction ID and due date.
4. **List All Books** — confirm available copies dropped from 2 to 1.
5. **Return Book** — return using the transaction ID. Confirm no fine
   (returned same day, before the 14-day due date).
6. **Error handling checks**:
   - Try issuing a non-existent ISBN → `BookNotFoundException` message.
   - Issue all copies of a book, then try issuing one more →
     `BookNotAvailableException` message.
   - Register the same member ID twice → `DuplicateEntryException` message.
   - Try removing a member who still has an unreturned book → blocked
     with a clear message.
7. Restart the program and confirm books/members/transactions were
   reloaded correctly from the `data/` files (persistence check).

## Screenshots

_Add terminal screenshots of the menu, a successful issue/return, and an
error case here before submission._
