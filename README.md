# Bibliotekssystem - Relationsdatabas i SQLite

## 1. Databasdesign

### 1a. ER-diagram (Entity-Relationship Diagram)

Ett ER-diagram för bibliotekssystemet består av tre huvudentiteter: `Book`, `Borrower` och `Loan`. Här är deras relationer:

- En `Book` kan ha många `Loan` (en-till-många förhållande).
- En `Borrower` kan ha många `Loan` (en-till-många förhållande).
- `Loan` fungerar som en länk-tabell som kopplar `Book` och `Borrower`.

#### Entiteter och Attribut:
**Book**:
- `book_id` (primärnyckel)
- `title`
- `author`
- `publication_year`
- `isbn`

**Borrower**:
- `borrower_id` (primärnyckel)
- `name`
- `phone`
- `email`

**Loan**:
- `loan_id` (primärnyckel)
- `loan_date`
- `return_date`
- `due_date`
- `book_id` (främmande nyckel, kopplar till `Book`)
- `borrower_id` (främmande nyckel, kopplar till `Borrower`)
- ![ER-dia](https://github.com/user-attachments/assets/2a400678-e1a7-47a2-9791-642f538d1815)


### 1b. Normalisering till 3NF (Tredje Normalform)

- **1NF**: Alla attribut innehåller atomära värden (inga upprepningar av grupper eller listor inom rader).
- **2NF**: Alla icke-nyckelattribut är fullt beroende av hela primärnyckeln.
- **3NF**: Alla attribut är beroende av enbart primärnyckeln och inte av något annat icke-nyckelattribut.

Resultatet efter normalisering:
- `Book` innehåller endast attribut relaterade till böcker.
- `Borrower` innehåller endast attribut relaterade till låntagare.
- `Loan` representerar utlåningar och hanterar kopplingen mellan `Book` och `Borrower`.

### 1c. UML-klassdiagram

Ett UML-klassdiagram skulle visa de tre klasserna: `Book`, `Borrower`, och `Loan` med deras attribut och relationer, där `Loan` har två referenser: en till `Book` och en till `Borrower`.
![UML-klassdiagram](https://github.com/user-attachments/assets/6baa4ec1-47e5-4a91-bea1-bbba9c6fbe17)


## 2. Implementering

### 2a. Skapande av SQLite-tabeller

```sql
CREATE TABLE Book (
    book_id INTEGER PRIMARY KEY AUTOINCREMENT,
    title TEXT NOT NULL,
    author TEXT NOT NULL,
    publication_year INTEGER,
    isbn TEXT UNIQUE NOT NULL
);

CREATE TABLE Borrower (
    borrower_id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    phone TEXT NOT NULL,
    email TEXT UNIQUE NOT NULL
);

CREATE TABLE Loan (
    loan_id INTEGER PRIMARY KEY AUTOINCREMENT,
    loan_date DATE NOT NULL,
    return_date DATE,
    due_date DATE NOT NULL,
    book_id INTEGER,
    borrower_id INTEGER,
    FOREIGN KEY (book_id) REFERENCES Book(book_id),
    FOREIGN KEY (borrower_id) REFERENCES Borrower(borrower_id)
);

Här är din text formaterad i **Markdown-syntax**:

```markdown
# 2b. Primärnycklar och främmande nycklar
- **Primärnycklar**: `book_id`, `borrower_id`, och `loan_id`.
- **Främmande nycklar**: `book_id` och `borrower_id` i `Loan`, som kopplar till respektive `Book` och `Borrower`.

# 3. Datamanipulation

## 3a. CRUD-operationer

- **Create**: Lägg till en ny bok:
  ```sql
  INSERT INTO Book (title, author, publication_year, isbn) 
  VALUES ('Moby Dick', 'Herman Melville', 1851, '9781234567890');
  ```

- **Read**: Hämta alla böcker:
  ```sql
  SELECT * FROM Book;
  ```

- **Update**: Uppdatera en bok:
  ```sql
  UPDATE Book SET title = 'Moby Dick - New Edition' WHERE book_id = 1;
  ```

- **Delete**: Ta bort en bok:
  ```sql
  DELETE FROM Book WHERE book_id = 1;
  ```

## 3b. Vyer (Views)

- **View 1**: Låneböcker och låntagare:
  ```sql
  CREATE VIEW LoanInfo AS
  SELECT Loan.loan_id, Borrower.name, Book.title, Loan.loan_date, Loan.due_date
  FROM Loan
  JOIN Borrower ON Loan.borrower_id = Borrower.borrower_id
  JOIN Book ON Loan.book_id = Book.book_id;
  ```

- **View 2**: Försenade lån:
  ```sql
  CREATE VIEW OverdueLoans AS
  SELECT Loan.loan_id, Borrower.name, Book.title, Loan.due_date
  FROM Loan
  JOIN Borrower ON Loan.borrower_id = Borrower.borrower_id
  JOIN Book ON Loan.book_id = Book.book_id
  WHERE Loan.due_date < DATE('now') AND Loan.return_date IS NULL;
  ```

# 4. Avancerade frågor

## 4a. JOIN mellan tre tabeller:
```sql
SELECT Borrower.name, Book.title, Loan.loan_date, Loan.due_date
FROM Loan
JOIN Borrower ON Loan.borrower_id = Borrower.borrower_id
JOIN Book ON Loan.book_id = Book.book_id;
```

## 4b. Subquery:
```sql
SELECT title FROM Book WHERE book_id = (SELECT book_id FROM Loan WHERE loan_id = 1);
```

## 4c. GROUP BY och HAVING:
```sql
SELECT Borrower.name, COUNT(Loan.loan_id) AS total_loans
FROM Loan
JOIN Borrower ON Loan.borrower_id = Borrower.borrower_id
GROUP BY Borrower.name HAVING total_loans > 5;
```

# 5. Säkerhet

## 5a. Enkel inloggningsfunktion
För att implementera en inloggningsfunktion kan man skapa en tabell för användare med användarnamn och lösenord (hashed). Säkerhetsaspekter inkluderar lösenordshashning och skydd mot SQL-injektion.

## 5b. Prepared statements
Prepared statements skyddar mot SQL-injektion genom att separera SQL-kod från inmatade data. Exempel:
```sql
INSERT INTO Borrower (name, phone, email) VALUES (?, ?, ?);
```

## 5c. Attackvektor: SQL-injektion
SQL-injektion är en attack där en angripare manipulerar SQL-frågor. Man skyddar sig genom att använda prepared statements och aldrig inkludera användarinmatad data direkt i SQL-frågor.

# 6. Analys och reflektion

## 6a. Potentiella förbättringar
En förbättring kan vara att lägga till fler constraints, till exempel begränsa antal böcker en låntagare får låna samtidigt.

## 6b. Reflektion över designval
Designen är gjord för att säkerställa att systemet är skalbart och lätt att underhålla. Databasnormalisering används för att undvika datainkonsistens och redundans.

# 7. Dokumentation

## 7a. Rapport
Databasdesignen följer normaliseringsprinciper upp till tredje normalformen. Primärnycklar och främmande nycklar används för att säkerställa referensintegritet. CRUD-operationer och vyer är implementerade för att hantera och kombinera data effektivt.

## 7b. Instruktioner
Skapa databasen med de SQL-skript som tillhandahålls och använd INSERT-kommandon för att lägga till initial data.

## 7c. SQL-skript
Skriptet för att skapa och populera databasen ingår i denna dokumentation.
