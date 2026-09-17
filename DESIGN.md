# Design & Documentation

Paste these Mermaid blocks into the report (Word/Docs, GitHub markdown, or
https://mermaid.live) to render them as images for the PDF report.

## 1. System Architecture

```mermaid
flowchart TB
    UI["Main.java (Console UI)"] --> ENGINE["Library.java (Core Engine)"]
    ENGINE --> CATALOG["Catalog Module\n(Book, HashMap<String,Book>)"]
    ENGINE --> MEMBERSHIP["Membership Module\n(Member, HashMap<String,Member>)"]
    ENGINE --> CIRC["Circulation Module\n(Transaction, HashMap<String,Transaction>)"]
    ENGINE --> LOGGER["Logger.java"]
    CATALOG --> FM["FileManager.java"]
    MEMBERSHIP --> FM
    CIRC --> FM
    FM --> FILES[("data/*.txt files")]
    LOGGER --> LOGFILE[("data/activity.log")]
```

## 2. Process Flow — Issue Book

```mermaid
flowchart TD
    A[Librarian selects Issue Book] --> B[Enter ISBN and Member ID]
    B --> C{Book exists?}
    C -- No --> X1[Show BookNotFoundException]
    C -- Yes --> D{Member exists?}
    D -- No --> X2[Show MemberNotFoundException]
    D -- Yes --> E{Copies available AND\nmember under borrow limit?}
    E -- No --> X3[Show BookNotAvailableException]
    E -- Yes --> F[Decrement available copies]
    F --> G[Increment member's issued count]
    G --> H[Create Transaction, due date = today + 14 days]
    H --> I[Persist to files, write log entry]
    I --> J[Show transaction ID and due date]
```

## 3. Class Diagram

```mermaid
classDiagram
    class Persistable {
        <<interface>>
        +toFileLine() String
    }
    class Person {
        <<abstract>>
        #id String
        #name String
        #email String
        +getRole() String*
    }
    class Member {
        -booksIssuedCount int
        +canBorrow() boolean
        +incrementBooksIssued()
        +decrementBooksIssued()
    }
    class Librarian {
        -staffCode String
    }
    class Book {
        -isbn String
        -title String
        -author String
        -totalCopies int
        -availableCopies int
        +issueCopy()
        +returnCopy()
        +compareTo(Book) int
    }
    class Transaction {
        -transactionId String
        -isbn String
        -memberId String
        -dueDate LocalDate
        +markReturned(LocalDate) double
        +calculateFine(LocalDate) double
    }
    class Library {
        -catalog Map~String,Book~
        -members Map~String,Member~
        -transactions Map~String,Transaction~
        +addBook(Book)
        +registerMember(Member)
        +issueBook(String, String) Transaction
        +returnBook(String) double
    }
    class FileManager {
        +writeAll(String, List)$
        +readAll(String, Function)$
    }

    Person <|-- Member
    Person <|-- Librarian
    Persistable <|.. Person
    Persistable <|.. Book
    Persistable <|.. Transaction
    Book "1" <.. "*" Transaction : referenced by
    Member "1" <.. "*" Transaction : referenced by
    Library "1" o-- "*" Book
    Library "1" o-- "*" Member
    Library "1" o-- "*" Transaction
    Library ..> FileManager : uses
```

## 4. Sequence Diagram — Return Book (with fine)

```mermaid
sequenceDiagram
    actor Librarian
    participant Main
    participant Library
    participant Transaction
    participant Book
    participant Member
    participant FileManager

    Librarian->>Main: Choose "Return Book", enter Transaction ID
    Main->>Library: returnBook(transactionId)
    Library->>Transaction: markReturned(today)
    Transaction-->>Library: fine amount
    Library->>Book: returnCopy()
    Library->>Member: decrementBooksIssued()
    Library->>FileManager: writeAll(books/members/transactions)
    Library-->>Main: fine amount
    Main-->>Librarian: "Fine: X" or "No fine"
```

## Non-Functional Requirements Addressed

| Requirement      | How it's addressed |
|-------------------|---------------------|
| Performance        | `HashMap` lookups give O(1) access to books/members/transactions by key instead of linear scans. |
| Security           | Input validation on every menu entry point; no destructive operation (e.g. removing a member) proceeds while invariants are violated (e.g. unreturned books). |
| Reliability        | All state-changing operations immediately persist to disk, so a crash doesn't lose committed transactions. |
| Scalability        | Modules (`Catalog`, `Membership`, `Circulation`) are decoupled through `Library`, so swapping flat files for a real database later only touches `FileManager`. |
| Maintainability    | Single-responsibility classes, an interface (`Persistable`) for serialization, and generics in `FileManager` avoid duplicated I/O code. |
| Error handling     | Custom checked exceptions per failure mode, caught centrally in `Main` and shown as friendly messages, never raw stack traces. |
| Logging/monitoring | `Logger` appends a timestamped entry for every add/issue/return/error to `data/activity.log`. |
| Resource efficiency| Files are only opened for the duration of a read/write (try-with-resources), no lingering handles. |

## Note on Storage

This version uses flat text files (`data/*.txt`) rather than a relational
database, since Core Java/OOP courses typically don't require JDBC. If your
syllabus expects a database, the `FileManager` class is the only place that
would need to change — swap it for a `DatabaseManager` using JDBC, and
`Library.java` (the business logic) stays untouched. If you'd like, I can
build that JDBC version instead.
