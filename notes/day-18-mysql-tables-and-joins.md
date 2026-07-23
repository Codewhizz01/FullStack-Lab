# Day 18  — MySQL, Tables & Joins



## 1. Why a file is not enough

Our resume data has lived in `data.json`. It works in class, but fails in the real world for two reasons.

**Data gets lost.**
Two users save at the same moment. Both read the file, both write it back — and the second write erases the first user's change. No error, the data is just gone. A file cannot stop this. A database can.

**Search is slow.**
To find one resume in a file, you load all of them and loop through every one. With ten thousand records, that's slow every single time. A database finds it instantly, because it was built for exactly this.

> **MySQL**
> A real database server. A separate program whose only job is to store data safely and find it fast, even with many people using it at once. It's one of the most widely used databases in the world — what you learn here is what companies actually run.

---

## 2. Install MySQL and MySQL Workbench

Two things to install:

| Install | What it is |
|---|---|
| **MySQL Server** | the database engine that stores everything. Runs in the background. |
| **MySQL Workbench** | a window into the database. You type SQL here and see results. |

During install, MySQL asks you to set a **root password** — write it down. `root` is the main admin user, and you'll need that password to connect from Workbench and later from your Node app. When install finishes, open Workbench and connect to `localhost` with user `root` and your password. You now have a live database in front of you.



---

## 3. Create a database, by hand, in Workbench

A MySQL server can hold **many databases**, one per project. First we make ours. In Workbench, open a query tab and run:

```sql
CREATE DATABASE resume_db;
```

That's it — you now have an empty database called `resume_db`. To work inside it, tell MySQL to use it:

```sql
USE resume_db;
```

> **Server vs database vs table**
> The **server** is the whole MySQL program. It holds many **databases**, one per project (`resume_db`, `shop_db`, ...). Each database holds many **tables** (`users`, `resumes`, ...). Each table holds many **rows**.
> Think folders inside folders: server → database → table → row.

---

## 4. Create a table, by hand

Now a table to hold users. Run this in Workbench, inside `resume_db`:

```sql
CREATE TABLE users (
  id    INT AUTO_INCREMENT PRIMARY KEY,
  name  VARCHAR(255) NOT NULL,
  email VARCHAR(255) NOT NULL UNIQUE
);
```

Read each line — you're declaring the **shape** of your data:

| Line | Meaning |
|---|---|
| `id INT AUTO_INCREMENT` | a whole number MySQL fills in and increases for each row: 1, 2, 3... |
| `PRIMARY KEY` | this column uniquely identifies a row — one id, one exact row |
| `VARCHAR(255)` | text up to 255 characters, good for names and emails |
| `NOT NULL` | this column cannot be empty — every user must have a name |
| `UNIQUE` | no two rows can share this value — no duplicate emails |

Refresh the schema panel on the left of Workbench and you'll see `users` appear under `resume_db`, with its three columns. You built a real table.

---

## 5. Add rows with INSERT, read them with SELECT

An empty table isn't much. Add rows with `INSERT`:

```sql
INSERT INTO users (name, email) VALUES ('Divya', 'Divya@example.com');
INSERT INTO users (name, email) VALUES ('Deepesh', 'Deepesh@example.com');
INSERT INTO users (name, email) VALUES ('Bhumi','bhumi@example.com');
```

You didn't give an `id` — MySQL filled it in: 1, 2, 3. Now read the table with `SELECT`:

```sql
SELECT * FROM users;
```

```
+----+-----------+-------------------------+
| id | name      | email                   |
+----+-----------+-------------------------+
| 1  | Divya     | divya@example.com       |
| 2  | Deepesh   | deepesh@example.com     |
| 3  | Bhumi     | bhumi@example.com       |
+----+-----------+-------------------------+
```

The `*` means "all columns." To find one person, filter with `WHERE`:

```sql
SELECT * FROM users WHERE email = 'ayush@example.com';
```

> **`WHERE` is not optional on `UPDATE` and `DELETE`.**
> `DELETE FROM users;` with no `WHERE` deletes **every** user.
> `DELETE FROM users WHERE id = 2;` deletes exactly one.
> The `WHERE` is the difference between removing one row and wiping your whole table — always double-check it before running a `DELETE` or `UPDATE`.

---

## 6. A second table, and the foreign key

A resume **belongs to** a user. We make a second table and connect it to `users` with a **foreign key** — a column that holds the `id` of the owning user.

```sql
CREATE TABLE resumes (
  id      INT AUTO_INCREMENT PRIMARY KEY,
  title   VARCHAR(255) NOT NULL,
  summary TEXT,
  userId  INT,
  FOREIGN KEY (userId) REFERENCES users(id)
);
```

```sql
INSERT INTO resumes (title, summary, userId) VALUES ('Full Stack Intern', 'Node, Express, MySQL', 1);
INSERT INTO resumes (title, summary, userId) VALUES ('QA Intern', 'Manual + API testing', 2);
```

> **Foreign key**
> A column in one table that points to a row in another. `resumes.userId` holds a `users.id`. A resume with `userId = 1` belongs to Himanshu. The `FOREIGN KEY ... REFERENCES` line tells MySQL to enforce this link — it keeps every resume pointing at a real, existing user.

**Note:** `summary` is `TEXT`, not `VARCHAR`. `VARCHAR` caps at 255 characters; a real summary is longer. `TEXT` holds long text. Choosing the right type is part of designing a table.

---

## 7. Joins: reading two tables together

You have `users` in one table and `resumes` in another, linked by `userId`. A **join** reads them together in one query, matching each resume to its owner.

> **Join**
> A query that combines rows from two tables using a matching column. Here we match `resumes.userId` to `users.id`, so each resume is paired with the user it belongs to.

```sql
SELECT resumes.title, users.name
FROM resumes
JOIN users ON resumes.userId = users.id;
```

```
+--------------------+----------+
| title              | name     |
+--------------------+----------+
| Full Stack Intern  | divya    |
| QA Intern          | deepesh  |
+--------------------+----------+
```

One query, two tables, joined on the matching id. **This is the heart of a relational database:** data lives in separate tables, and joins bring it back together on demand. Next class, Sequelize will write these joins for us — but now you know what it's doing underneath.

---

## 8. Normalization: why two tables, not one

Why split `users` and `resumes` into two tables? Why not one big table with the user's name repeated on every resume row? That question is what **normalization** answers.

> **Normalization**
> Organising data so each fact is stored in exactly one place. A user's name lives once, in the `users` table. Resumes point to it by `id`. You do not copy the name onto every resume row.

Imagine the bad, un-normalized version — one table where every resume row also stores the user's name and email:

| title | name | email |
|---|---|---|
| Full Stack Intern | Divya| himanshu@example.com |
| QA Intern | Divya| himanshu@example.com |

Divya's name and email are copied on every resume he has. Two problems follow:

1. **Waste** — the same data stored again and again.
2. **Update anomalies** — if Himanshu changes his email, you'd have to update it in every single row. Miss one, and your data disagrees with itself.

The normalized version stores the email **once**, in `users`. Change it in one place, and every resume — through the join — sees the new value instantly. Nothing to hunt down, nothing to fall out of sync.

> **The rule in one line.** Store each fact once, and link to it by id. Joins put the data back together when you need it. That's why relational databases use many small connected tables instead of one giant one.

---

