# Problem Statement

Small and mid-sized libraries often rely on manual registers or spreadsheets
to track which books are on the shelf, who has borrowed what, and when a
book is overdue. This is error-prone: it's easy to lose track of overdue
books, double-issue the last copy of a title, or forget to charge a fine.
There is a need for a simple, reliable system that a librarian can use to
manage the book catalog, member records, and the day-to-day borrowing
workflow, applying core object-oriented design principles.

## Scope of the Project

The system covers the core circulation workflow of a single-branch library:

- Maintaining a catalog of book titles with multiple copies per title.
- Registering and managing members.
- Issuing and returning books, with a fixed loan period and automatic
  overdue fine calculation.
- Persisting all data between runs and logging every significant action.

Out of scope (possible future enhancements): multi-branch support, a
graphical or web interface, reservations/holds queue, and online payment
of fines.

## Target Users

- **Librarians / library staff** — the primary users, who perform all
  catalog, member, and circulation operations through the console menu.
- **Library administrators** — indirectly benefit from the activity log
  and persisted records for auditing and reporting.

## High-Level Features

1. **Catalog Management** — add, search, list, and remove books; track
   total and available copies per title.
2. **Member Management** — register, list, and remove members, with a
   per-member borrowing limit.
3. **Circulation Management** — issue books, return books, calculate
   overdue fines automatically, and view a member's active loans.
4. Persistent file-based storage and an append-only activity log.
5. Input validation and custom exceptions for all failure paths (book not
   found, book unavailable, member not found, duplicate registration).
