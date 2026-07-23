# Day 19- SQL, NoSQL & How Data Is Stored



## 1. Database vs DBMS

**Database** = an organised collection of data. That's it — just data, kept in order so you can find it.

But data doesn't manage itself. Something has to store it, protect it, and answer questions about it. That "something" is the **DBMS**.

> **DBMS — Database Management System**
> The software that stores and manages a database for you. It saves data, reads it back, keeps it safe, and lets many people use it at once.
> MySQL is a DBMS. So are PostgreSQL, Oracle, and MongoDB.
> When you "install MySQL", you're installing a DBMS.

**Takeaway:** the *database* is the data. The *DBMS* is the program looking after it. 

---

## 2. Language vs Query

You talk to a DBMS in a **language** — just like you write JavaScript to talk to a browser, you write database commands in a database language. The most common one is **SQL**.

> **Query**
> A single question or command sent to the database.
> "Give me all users from Haldwani" → a query.
> "Add this new user" → a query.
> Every time your app reads or writes data, it sends a query. ("Query" comes from "to ask".)

**Takeaway:** a *language* is how you speak to the database. A *query* is one thing you say in that language.

---

## 3. SQL vs NoSQL — the two families

Two big families of databases, differing in **how they organise data**.

### SQL databases
Store data in **tables with fixed columns**, like a spreadsheet. Every row has the same shape — you decide the columns in advance.
→ MySQL is a SQL database. ("SQL" is also the name of the language.)

### NoSQL databases
Store data more freely, often as **flexible documents**, where different records can have different fields. No need to fix the shape in advance.
→ MongoDB is the best-known NoSQL database.

| | SQL (e.g. MySQL) | NoSQL (e.g. MongoDB) |
|---|---|---|
| Shape | tables, fixed columns | flexible documents |
| Every record | same columns | can differ |
| Plan the shape first? | yes | not required |
| Good for | data with a clear, steady structure | data that changes shape a lot |

**Neither is "better"** — they're tools for different jobs.



---

## 4. RDBMS — where "relational" comes in

> **RDBMS — Relational Database Management System**
> A DBMS for SQL databases, where tables can be **related** to each other — e.g. a `users` table and a `resumes` table, linked so each resume knows its user. The **"R"** is that link.

You've already met the link itself: the **foreign key** — the `userId` column that ties a resume to its user. That relationship *is* the "relational" in RDBMS.

So MySQL is:
- a **DBMS** → it manages data
- a **SQL database** → it uses tables + the SQL language
- an **RDBMS** → its tables relate to each other

All three words describe the same tool from different angles.

---

## 5. Structured vs Unstructured data

This is the idea underneath the SQL/NoSQL split.

### Structured data
Data with a **fixed, predictable shape**. A `users` table where every row has a `name`, `email`, and `password`, and nothing else. Fits neatly into rows and columns. → SQL databases are built for this.

### Unstructured data
Data with **no fixed shape**: a photo, a video, a long free-text note, a chat message. Doesn't fit tidy columns. → Often lives in NoSQL stores or as files.

| Structured | Unstructured |
|---|---|
| a user's name and email | a profile photo |
| a resume's title | a voice recording |
| an order's total price | a free-text support chat |

**Tie it together:** structured data has a predictable shape → fits a SQL table. Unstructured data doesn't → often lives in NoSQL or as a file.

> Our resume fields (`name`, `email`, `title`) are structured — that's exactly why MySQL fits the project.

---

## 6. CHAR vs VARCHAR vs the STRING type

When storing text in MySQL, you pick *how*. Three names came up — easy to confuse.

### `CHAR`
**Fixed length.** `CHAR(10)` always reserves room for 10 characters, even if you only store 3 — it pads the rest with spaces.
Best when every value is genuinely the same length: a country code `"IN"`, a fixed-length ID.

### `VARCHAR`
**Variable length.** `VARCHAR(255)` can hold up to 255 characters, but only uses the room it actually needs.
This is what you use for almost all normal text: names, emails, titles.

| | `CHAR(n)` | `VARCHAR(n)` |
|---|---|---|
| Length | always `n`, padded up to `n` | only what is used |
| Best for | fixed-size codes | everyday text of varying length |
| Example | country code, PIN length | name, email, title |

### Visual: storing "Deepesh" (7 letters) in a size-10 column

**CHAR(10) — always reserves 10 cells**
D I V Y A · · ·   → 8 used (3 blank spaces padded in)

**VARCHAR(10) — stores only what you use**
D I V Y A         → 5 used (nothing stored for the rest)

> **Same name, less space.** `CHAR` wastes 3 cells on padding every time. `VARCHAR` keeps only the 5 it needs. Store a million names, and `CHAR` is storing a million rows of padding you never needed.

### Where `STRING` fits (Sequelize)

`STRING` is **not** a MySQL word — it's a **Sequelize** word.

When you write `DataTypes.STRING` in a model, Sequelize turns it into `VARCHAR(255)` in MySQL.

DataTypes.STRING  (Sequelize / JS name)
        ↓
VARCHAR(255)      (real MySQL column)

Same thing, two layers.

> **Rule of thumb:** use `VARCHAR` for text that varies in length. Use `CHAR` only when every value is exactly the same size.

---

## 7. More MySQL types: ENUM, BOOLEAN, TINYINT

Text isn't the only kind of column. A few more types came up, each with a specific job.

### `ENUM`
A column that can only hold **one value from a fixed list** you decide in advance.

status: ENUM('active', 'pending', 'deleted')

Try to save anything outside that list and MySQL refuses. It keeps a column honest when there are only a few valid options.

### `BOOLEAN`
A **true/false** column. e.g. `isVerified`, `termsAccepted`. Answers a yes-or-no question about the row.

### `TINYINT`
A very small whole number. Here's the connection: **MySQL has no separate true/false type underneath** — `BOOLEAN` is actually stored as `TINYINT(1)`, a tiny number that's `1` for true and `0` for false.

> When you ask for `BOOLEAN`, MySQL gives you a `TINYINT(1)`. Two names, one storage.

| Type | Holds | Use for |
|---|---|---|
| `ENUM` | one value from a fixed list | `status`, `role`, `provider` |
| `BOOLEAN` | true or false | `isVerified`, `isActive` |
| `TINYINT` | small number (0–255) | the storage behind `BOOLEAN`, small counts |

**Why this matters for design:** picking the tightest type that fits your data is good design.
- A `status` is not free text → make it an `ENUM` and the database guards it for you.
- A yes/no is a `BOOLEAN`, not the string `"yes"`.

The type is your **first line of defence** for clean data.

---

## 8. "Why learn SQL commands if Sequelize writes SQL for us?"

Fair question. Here's exactly why it matters:

1. **You can debug.** When a Sequelize query behaves strangely, turn on logging, read the SQL it generated, and run it yourself in Workbench to see what's really happening. Without SQL, that error is a black box.
2. **You can check your data directly.** "Did that row actually save?" Open Workbench, run `SELECT`. No app needed — straight to the source of truth.
3. **You understand what the ORM is doing.** Sequelize is a convenience over SQL, not a replacement for understanding it. Once you know the SQL, the ORM stops being magic and becomes a tool you control.
4. **Not every job uses Sequelize.** SQL is the same everywhere — MySQL, PostgreSQL, Python, Java, data analyst roles. The ORM changes between projects; SQL is the skill that transfers to all of them.

> **Honest summary:** the ORM writes SQL so you don't have to, every day. But when something breaks, when you need to check the truth, or when you're on a project without an ORM — SQL is what saves you. Learn the tool *and* the layer beneath it. That's the difference between someone who can use a library and someone who understands their craft.

---






