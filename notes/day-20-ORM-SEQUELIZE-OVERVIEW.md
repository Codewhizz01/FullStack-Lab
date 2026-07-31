# Day 20 — Storing Data (Sequelize + MySQL)

## What this class was about

Till now I had created a database and a `users` table manually in MySQL Workbench, and wrote INSERT and SELECT queries by hand. In this class, I did the exact same operations — but from JavaScript, using Sequelize, one operation at a time. For every step, I looked at three things: the JavaScript code I write, the actual SQL query Sequelize generates from it, and the result. By the end, I stored a real user and their resumes in MySQL directly from code, read the data back, and updated it — understanding exactly what each line does to the database underneath.

## The Setup: 4 files, each with one clear job

Before writing any CRUD logic, I needed four files in place. Three describe the tables (models), and one sets up the connection to MySQL.

### 1. config/database.js — the single connection to MySQL

This file creates one connection object that every other file in the project shares. Instead of every model opening its own separate connection to MySQL, they all talk to the database through this same object — which keeps things consistent and easy to manage from one place.

    const { Sequelize } = require('sequelize');

    const sequelize = new Sequelize(
      'resume_db',       // the database name, created earlier in Workbench
      'root',             // MySQL username
      'your_password',    // MySQL password, set during installation
      {
        host: 'localhost',    // MySQL runs on my own machine
        dialect: 'mysql',     // tells Sequelize to speak MySQL specifically
        logging: false,       // set to console.log if I want to see every SQL query sent
      }
    );

    module.exports = sequelize;

I think of this like a single landline into a house — everyone in the family (every model) uses this same line to call the database, instead of each person needing their own separate connection.

### 2. models/user.js — the User model

This defines what a "user" looks like as a database table, plus some extra logic around passwords.

    const { DataTypes } = require('sequelize');
    const bcrypt = require('bcryptjs');
    const sequelize = require('../config/database');

    const User = sequelize.define('User', {
      name: { type: DataTypes.STRING, allowNull: false },
      email: {
        type: DataTypes.STRING,
        allowNull: false,
        unique: true,
        validate: { isEmail: true },
      },
      password: { type: DataTypes.STRING, allowNull: false },
    });

    // hash the password before every save
    User.beforeCreate(async (user) => {
      const salt = await bcrypt.genSalt(10);
      user.password = await bcrypt.hash(user.password, salt);
    });

    User.prototype.checkPassword = function (plainText) {
      return bcrypt.compare(plainText, this.password);
    };

    // This model's relationships live in this file.
    // It receives every model as an argument, so it never has to
    // require Resume itself — that's exactly what avoids a circular require.
    User.associate = (models) => {
      User.hasMany(models.Resume, { foreignKey: 'userId', onDelete: 'CASCADE' });
    };

    module.exports = User;

Two things stood out to me here that I hadn't done before:

- **User.prototype.checkPassword** adds a method directly onto every user object. So instead of calling bcrypt.compare() manually every time I need to check a login, I can just write `user.checkPassword('typed_password')` and get back true or false. Much cleaner to reuse.
- **User.associate** is a new pattern — instead of setting up relationships in one central file, each model defines its own relationships, in its own file. It receives all the models as a parameter, so user.js never has to directly require('./resume') — and that's exactly what prevents a circular require (two files each trying to require the other, which would loop forever).

### 3. models/resume.js — the Resume model

    const { DataTypes } = require('sequelize');
    const sequelize = require('../config/database');

    const Resume = sequelize.define('Resume', {
      title: { type: DataTypes.STRING, allowNull: false },
      summary: { type: DataTypes.TEXT },
    });

    // This model's relationship lives in this file.
    Resume.associate = (models) => {
      Resume.belongsTo(models.User, { foreignKey: 'userId' });
    };

    module.exports = Resume;

hasMany and belongsTo are two sides of one relationship — User.hasMany(Resume) means "one user can own many resumes," and Resume.belongsTo(User) means "every resume points back to exactly one user." Sequelize needs both sides written down for the relationship, and the automatic joins that come with it, to actually work.

### 4. models/index.js — loads everything and connects it all

    const sequelize = require('../config/database');
    const User = require('./user');
    const Resume = require('./resume');

    const models = { User, Resume };

    // Now that all models are loaded, wire up each one's relationships.
    Object.values(models).forEach((model) => {
      if (model.associate) model.associate(models);
    });

    module.exports = { sequelize, ...models };

This file's job is simple: load both models, then loop through them and call .associate() on each one. The order matters — both models need to fully exist before .associate() runs, since User.associate needs models.Resume to already be there.

I'd compare this to introducing two coworkers to each other only after both have actually walked into the room — you can't introduce someone who hasn't arrived yet.

## Turning the models into real tables

    const { sequelize, User, Resume } = require('./models');
    await sequelize.sync();  // no force, so existing data is kept

sync() checks my models against the actual MySQL database and creates whatever tables are missing (users, resumes). Without force: true, it's safe to run again and again — existing data stays untouched. With force: true, it would drop and rebuild the tables from scratch every time, wiping all data — something I'd only ever use early on, never once there's real data I care about.

## Doing the actual CRUD operations, one step at a time

### Step 1 — Creating a user

    const user = await User.create({
      name: 'Divya',
      email: 'divyau@example.com',
      password: 'secret123',
    });

    console.log('Saved user #' + user.id);

What Sequelize actually sends to MySQL:

    INSERT INTO `Users`
      (`name`, `email`, `password`, `createdAt`, `updatedAt`)
    VALUES ('divya', 'divya@example.com', ..., NOW(), NOW());

Result:

    Saved user #1

The password stored isn't the plain text I typed — it's already the bcrypt hash, because beforeCreate runs before this insert happens. createdAt and updatedAt get filled in automatically. And user.id is the real auto-generated ID MySQL just assigned — I can use it immediately in the next step.

### Step 2 — Creating resumes linked to that user

    await Resume.create({
      title: 'Full Stack Intern',
      summary: 'Built REST APIs with Node, Express and MySQL.',
      userId: user.id,
    });

    await Resume.create({
      title: 'QA Intern',
      summary: 'Manual test cases and API tests with Postman.',
      userId: user.id,
    });

    console.log('Saved 2 resumes');

SQL sent:

    INSERT INTO `Resumes`
      (`title`, `summary`, `userId`, `createdAt`, `updatedAt`)
    VALUES ('Full Stack Intern', '...', 1, NOW(), NOW());

The important part here is `userId: user.id` — this single line is what actually creates the real relationship in the data, linking each resume back to user #1. Without this, the resume would exist, but with no owner attached to it.

### Step 3 — Reading: getting all resumes for one user

    const resumes = await Resume.findAll({ where: { userId: user.id } });

    console.log('This user has', resumes.length, 'resumes:');
    resumes.forEach(r => console.log(' -', r.title));

SQL sent:

    SELECT id, title, summary, userId, createdAt, updatedAt
    FROM `Resumes`
    WHERE `userId` = 1;

Result:

    This user has 2 resumes:
     - Full Stack Intern
     - QA Intern

findAll returns every matching row as an array. `where: { userId: ... }` is just the JavaScript version of the same WHERE clause I've already practiced writing directly in SQLZoo.

### Step 4 — Reading with a join: getting the resume and its owner together

    const first = await Resume.findByPk(resumes[0].id, { include: User });

    console.log('Resume "' + first.title + '" belongs to ' + first.User.name);

SQL sent:

    SELECT Resume.*, User.name, User.email
    FROM `Resumes` AS `Resume`
    LEFT OUTER JOIN `Users` AS `User`
      ON `Resume`.`userId` = `User`.`id`
    WHERE `Resume`.`id` = 1;

Result:

    Resume "Full Stack Intern" belongs to divya

The key part is `include: User` — that single option tells Sequelize to also fetch the related user in the same query, which turns one simple findByPk call into a real SQL LEFT OUTER JOIN behind the scenes. The result comes back nested as first.User, so I don't need a second separate query just to get the owner's name.

I think of this like asking a librarian for a book and the author's short bio at the same time, instead of getting the book first and then walking to a completely different shelf for the author info.

### Step 5 — Updating a row

    first.title = 'Senior Full Stack Intern';
    await first.save();

    console.log('Updated title:', first.title);

SQL sent:

    UPDATE `Resumes`
    SET `title` = 'Senior Full Stack Intern', `updatedAt` = NOW()
    WHERE `id` = 1;

Result:

    Updated title: Senior Full Stack Intern

.save() only updates whatever actually changed on the object — Sequelize keeps track of which fields are "dirty" and only touches those columns, while automatically refreshing updatedAt too. The `WHERE id = 1` makes sure only this exact row gets updated, not every resume in the table.

## Double-checking directly in MySQL

    SELECT * FROM resumes;

    +----+---------------------------+---------+--------+
    | id | title                     | summary | userId |
    +----+---------------------------+---------+--------+
    |  1 | Senior Full Stack Intern  | ...     |      1 |
    |  2 | QA Intern                 | ...     |      1 |
    +----+---------------------------+---------+--------+

This confirms everything I did from JavaScript actually landed in the real database — the same table I'd see if I opened Workbench directly.

## What I'm taking away from this

Every Sequelize method I used today — .create(), .findAll(), .findByPk(), .save() — is really just a JavaScript-friendly wrapper around the same SQL operations I already know from SQLZoo (INSERT, SELECT, SELECT...WHERE, UPDATE). Sequelize is just handling the query-building, escaping, and connection details underneath, so the same CRUD logic becomes clean method calls on plain JS objects instead of raw query strings I'd have to write by hand.