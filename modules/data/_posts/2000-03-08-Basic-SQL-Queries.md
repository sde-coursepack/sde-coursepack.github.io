---
Title: Basic Queries
---

# Basic SQL Queries

In this unit, we will cover running basic SQL queries via the sqlite shell. A later module will cover using SQL queries from Java via JDBC (Java DataBase Connection). We will be using the SQLite dialect for this unit. It is often completely compatible with PostgreSQL, MySQL, etc., but you may run into some specific differences.

* TOC
{:toc}

# SQL Queries

SQL Queries are commands that we run that Create, Read, Update, and Delete data (often abbreviated to CRUD). In this unit, we will look at basic implementations of the following specific queries. For this, understand that our database is going to be made up of typically multiple **tables**, each table typically storing one type of data.

* `CREATE` - create a new table  
* `INSERT` - add data to a table
* `SELECT` - retrieve data from a table  
* `UPDATE` - edit data that is already in a table
* `DELETE` - remove records from a table
* `DROP` - delete a table completely (removes all data AND deletes the structure)

We will also look at **transaction management**

* `BEGIN TRANSACTION` - sets the starting point of a transaction. Changes to the database after this statement are "temporary", and can either be `COMMIT`ed or `ROLLBACK`-ed
* `COMMIT` - make all changes since the last `BEGIN TRANSACTION` permanent (that is, actually be reflected in the database)
* `ROLLBACK` - undo all change since the last `BEGIN TRANACTION` (meaning add data added, changed, or deleted is undone)

Be aware that by default, **SQLite has auto-commits turned on!** This means any commits are permanent by default. This is only disabled when you begin a transaction.

## Case-sensitivity

As a brief note, SQL commands are, by default **case-insensitive**. This means that `SELECT`, `Select`, and `select` (and even `sElEcT` which can be used in sarcastic queries) are all the same command. Additionally, table and column names are not case-sensitive. The only case-sensitive thing in SQL are text literals (that is, when storing textual data, case is preserved)

As a general point of style, I will use ALLCAPS for SQL commands and keywords like SELECT and UPDATE. I will generally use CapitalizedCase for TableNames and ColumnNames.

Note that there are **many** styles you will see out there. For instance, I have seen:

* fully lower case command names, capitalized table names, camelCase for column names
* capitalized commands, snake_case for everything else
* and many others

The most important thing is to be **consistent**. Whether you use my style or a different style is up to you. But picking a style and sticking with it is going to make your queries easier to read and understand, just like code.

## Create Table

For this module, we will be creating a table to keep track of some of Prof. McBurney's favorite books. As such, we will create a table called **<ins>Books<ins>** - note that in general, when displaying table names in documentation, they are typically bolded and underlined.

I want a table like so:

| BookID | Title                            | Series                           | SeriesOrder | Author         | Published | Rating |
|--------|----------------------------------|----------------------------------|-------------|----------------|-----------|--------|
| 1      | Hitchhiker's Guide to the Galaxy | Hitchhiker's Guide to the Galaxy | 1           | Douglas Adams  | 1979      | 4.2    |
| 2      | Memories of Ice                  | Malazan Book of the Fallen       | 3           | Steven Erikson | 2001      | 5.0    |
| 3      | Unsouled                         | Cradle                           | 1           | Will Wight     | 2016      | 3.7    |
| 4      | The Shadow Rising                | The Wheel of Time                | 4           | Robert Jordan  | 1992      | 4.8    |
| 5      | Happier as Werewolves            |                                  |             | Andrew Heaton  | 2016      | 3.9    |

Here, for the fields, I have:

* BookID - my `PRIMARY KEY`, a unique ID for each book that automatically increments as each book is added.
* Title - the Title of the Book
* Series - if the book is part of a series, it's name as Text goes here. Not all books are part of a series (for instance, "Happier as Werewolves" by Andrew Heaton, so this field is nullable.
* SeriesOrder - if the book is part of a series, the order of this (as an Integer) goes here. 1-indexed, so the first book in a series is 1.
* Author - the Text name of the author or author(s) goes here.
* Published - the year published as an Integer
* Rating - my personal rating of the book on a 1-5 as a Real (5 being the best). If no rating is given, default to 1.0.

It's important to visualize what data I have and what data I want to store before going forward. Now, I will actually criticize this table in a later unit, as it arguably violates what's called 3rd normal form. But let's not worry about making "good" database tables yet. Let's focus on just making our table.

### Example

The table above can be made with the following `CREATE` query in the sqlite shell.

```sql
CREATE TABLE Books (
    BookID INTEGER PRIMARY KEY,
    Title TEXT NOT NULL,
    Series TEXT,
    SeriesOrder INTEGER,
    Author TEXT NOT NULL,
    Published INTEGER NOT NULL,
    Rating REAL DEFAULT 1.0 CHECK(Rating >= 1.0 AND RATING <= 5.0)
) STRICT;
```

Note that **all** SQL statements end with a semi-colon. Like Java, linebreaks are used for style and readability, and are *not* considered syntax.

Now, when I run this command, nothing obviously immediately happens, but if I then run the `.tables` command, I can see:

```shell
sqlite> .tables
Books
```

Our table is created! Additionally, if later I forget the **schema** of my table (that is, the structure and constraints of the table), I can use the `.schema [table-name]` command

```shell
sqlite> .schema Books
CREATE TABLE Books (
    BookID INTEGER PRIMARY KEY,
    Title TEXT NOT NULL,
    Series TEXT,
    SeriesOrder INTEGER,
    Author TEXT NOT NULL,
    Published INTEGER NOT NULL,
    Rating REAL DEFAULT 1.0 CHECK(Rating >= 1.0 AND RATING <= 5.0)
) STRICT;
```


Be aware that creating a table results in an empty table with no data. We will use `INSERT` queries to add data later on.

### Syntax

The syntax of the command is as follows. Note that square brackets denote **[optional]** things, whereas everything `lower_case` is a placeholder. Additionally, anything after -- is a comment:

```sql
CREATE TABLE [IF NOT EXISTS] table_name
(
    column_name1 data_type1 [column_constraints1],
    column_name2 data_type2 [column_constraints2],
    -- as many columns as needed
    [table_constraints]
) [STRICT];
```

Each column is listed in a comma-separated list in parentheses along with the datatype and optionally, and *constraints*. Additionally, if needed, you can add table_constraints (that is, constraints that affect 1 or more columns) after listing all data. Now...**technically** datatypes aren't required on the columns by SQLite, but you should always include them. They *are required* when using a `STRICT` table, and you will be graded on the datatypes being defined correctly in class. (Also, no enforcing strict typing can lead to a lot of defensive programming headaches with Java).

### IF NOT EXISTS

In the above syntax, you can see that we have an optional phrase `IF NOT EXISTS` that we may or may not include. In SQL, Tables are required to have **unique**. For instance, if I try to run my `CREATE` query a second time after I have already created the table, I get the following error:

```shell
sqlite> CREATE TABLE Books (
   ...>     BookID INTEGER PRIMARY KEY,
   ...>     Title TEXT NOT NULL,
   ...>     Series TEXT,
   ...>     SeriesOrder INTEGER,
   ...>     Author TEXT NOT NULL,
   ...>     Published INTEGER NOT NULL,
   ...>     Rating REAL DEFAULT 1.0
   ...> ) STRICT;
Parse error: table Books already exists
  CREATE TABLE Books (     BookID INTEGER PRIMARY KEY,     Title TEXT NOT NULL,
               ^--- error here
```

Often, when running starting a program that uses SQLite, we want to check if the necessary tables are set up. As such, at start-up, we can run:

```sql
CREATE TABLE IF NOT EXISTS Books (
    BookID INTEGER PRIMARY KEY,
    Title TEXT NOT NULL,
    Series TEXT,
    SeriesOrder INTEGER,
    Author TEXT NOT NULL,
    Published INTEGER NOT NULL,
    Rating REAL DEFAULT 1.0 CHECK(Rating >= 1.0 AND RATING <= 5.0)
) STRICT;
```

This is probably the easiest way to ensure necessary tables are created. If the tables are already created, this query doesn't do anything (no error, no change to the existing table or data). If the table doesn't exist (because this is the first time starting our application), then the table is created.

### Column Constraints

Constraints put rules on what data is allowed to be added to the table. When inserting data, if a record you attempt to insert violates any constraint, that insert will be rejected with an error message. We define our constraints **at the time of table creation**.

* **PRIMARY KEY** - Consider the way we declared our `BookID` column in the `CREATE` query: `BookID INTEGER PRIMARY KEY,`. This makes the `BookID` column in our `Books` Table (or, `Books.BookID`) the PRIMARY KEY of the table. In general, you want to list your Primary Key column first (this is a convention, not a syntactic requirement), and that primary key should generally be an Integer (this is good practice when starting out - I would avoid being "clever" with multi-column primary keys). Only **one** column can have the `PRIMARY KEY` constraint. **PRIMARY KEY** as a constraint means both **Unique** and **Not Null**. 
* **NOT NULL** - By default, any record can have any attribute be null (that is, blank - no data). However, we can require that an attribute **must** be present. For instance, we are syntactically requiring that `Title`, `Author`, and `Published` must not be missing. Note that, like Java and other programming languages, an empty string ("") **is not** the same thing as null in SQL.
* **UNIQUE** - We don't have any `UNIQUE` constraints in the table directly, however, a UNIQUE Constraint means that **no two records** can have the same value in this column. 
* **DEFAULT** - If we leave this column null when inserting data, that null value is replaced by the default value. For instance, if we insert a book but do not specify it's `Rating`, then our constraint `Rating REAL DEFAULT 1.0 CHECK(Rating >= 1.0 AND RATING <= 5.0)` ensures the value is set to 1.0. The `DEFAULT` is only used when we insert null. If we insert a value, the `DEFAULT` is ignored.
* **CHECK** - A check constraint includes a **boolean expression**. When inserted, if this expression returns FALSE, the inserted record is rejected and an error message given. If the expression, is TRUE, then that means the data conforms to the standard. Looking at `Rating REAL DEFAULT 1.0 CHECK(Rating >= 1.0 AND RATING <= 5.0)`, the `CHECK` constraint ensures that the Rating is between 1 and 5, inclusive, since that is my rating scale.

### Table Constraints

We could also use the same constraints above, but define table constraints over 1 or more columns. For instance, let's say that in our table, no `Author` is allowed to use the same `Title`. By this, we mean we cannot have two books that are both `Author`-ed by "Steven Erikson" AND `Title`-d "Memories of Ice" (we can, however, have other authors use that title, and Steven Erikson can write other books). In this case, we would add a table-constraint **after** our columns:

```sql
CREATE TABLE IF NOT EXISTS Books (
  BookID INTEGER PRIMARY KEY,
  Title TEXT NOT NULL,
  Series TEXT,
  SeriesOrder INTEGER,
  Author TEXT NOT NULL,
  Published INTEGER NOT NULL,
  Rating REAL DEFAULT 1.0 CHECK(Rating >= 1.0 AND RATING <= 5.0),
  UNIQUE(Title, Author)
) STRICT;
```

### STRICT

As mentioned in our last module, by default, SQLite does not enforce types for our tables. However, I am generally a strong proponent of forcing types (there is almost never a downside, and there are significant problems you can avoid this way). The `STRICT` keyword after the close parentheses, but before the semi-colon, tells our database "actually enforce data-types, please". 

### Changing Tables after creation

I will often get a question about adding a column, changing a table or column name, adding constraints etc. to a table that already exists. And this is something you will absolutely have to do at some point. These are done with the `ALTER TABLE` command. 

However, Alter Table **really** doesn't work well with SQLite, and can be a pain to work with. One of the reasons is that the way SQLite stores your table's SCHEMA is that it **literally** stores the copy of your `CREATE` table query, and any `ALTER TABLE` commands have to both modify the table itself **any** modify that `CREATE` query.

Other issues are more structural. For instance, let's say after adding 20 books, I decide to also add the `PublicationCity` column to my `Books` Table, and I say it's needs to be `NOT NULL`. Well... I can't. At least not in one step. Because the issue is that when I add a new column to a table, all existing records have `NULL` for that column, since they weren't inserted with that data. I have two choices:

1) Add that column with both `NOT NULL` and `DEFAULT "some city"`, so that the default value is automatically added to existing data.  
2) Add the column *without* `NOT NULL`, *then* add the publication city to all existing data, *then* modify the column to add the `NOT NULL` constraint. 

**Except** - you can't actually do (2) in SQLite. Because SQLite *doesn't* have an `ALTER COLUMN` command.

Because of this, in SQLite, you can never add a new `UNIQUE` column to an existing table (nor add a `UNIQUE` constraint to an existing column.)

This is why, in general, the recommended approach whenever you need to modify columns in an existing table is:

1) Create a brand new table with a different name, but the same columns plus whatever changes you want.  
2) Copy existing data and insert new data into the table at the same time with an `INSERT` command  
3) Once you have copied the data, `DROP` the old table  
4) Then use `ALTER TABLE` to rename the new Table to the old name  

Sound exhausting and error-prone? It is. So in short, do your best to create the table right the first time in SQLite.

And **always** back-up your database before trying to do any kind of change like this, or [you could lose all your data](https://www.theguardian.com/technology/2019/mar/18/myspace-loses-all-content-uploaded-before-2016).

## INSERT

Yay! We have a table. A lonely, empty, useless table...wee...

So now, let's add some data!

I'm going to start by adding Hitchhiker's Guide to the Galaxy. here is the `INSERT` command to add that. For this, it's useful to have our schema handy, so before writing the `INSERT` query, I would use the `.schema` command.

```sql
sqlite> .schema Books
CREATE TABLE Books (
    BookID INTEGER PRIMARY KEY,
    Title TEXT NOT NULL,
    Series TEXT,
    SeriesOrder INTEGER,
    Author TEXT NOT NULL,
    Published INTEGER NOT NULL,
    Rating REAL DEFAULT 1.0 CHECK(Rating >= 1.0 AND RATING <= 5.0)
) STRICT;
```

Using the column names, I write the following to add one record to our Books

```sql
INSERT INTO Books(BookID, Title, Series, SeriesOrder, Author, Published, Rating)
  VALUES (1, 'Hitchhiker''s Guide to the Galaxy', 'Hitchhiker''s Guide to the Galaxy',
         1, 'Douglas Adams', 1979, 4.2);
```
Once again, any linebreaks are style and to make sure things fit on the page.

To check if our `INSERT`, we can use a `SELECT` statement. We will explain what this means in the `SELECT` Section below, but for now, we can use our sqlite shell and type `SELECT * FROM Books;`

```shell
sqlite> SELECT * FROM Books;
1|Hitchhiker's Guide to the Galaxy|Hitchhiker's Guide to the Galaxy|1|Douglas Adams|1979|4.2
```

And there is our data! It's important to note that this data is **persistent**. If we close our SQLite shell and reopen it, the data is still there. If we turn off our computer and turn it back on, the data is still there. This is what persistence means. The only way to remove this data now is to use a `DELETE` or `DROP` command, or delete the entire database.

### Text Literals note

A quick note that the *typical* way of doing text literals in SQL is using **single-quotes** (aka, apostrophe's). Of course, the problem I have is "Hitchhiker's Guide to the Galaxy" contains an apostrophe. And so `'Hitchhiker's Guide to the Galaxy'` would cause a problem, since the apostrophe in the word "Hitchhiker's" would be mistaken for the end of the String. So, to indicate a String containing an apostrophe, and that it's not the end of the Text literal, we simply use 2 apostrophes ('') inside our text literal, which means "this is an apostrophe, not the end of the String".

SQLite also supports using double-quote literals, or `"Hitcherhiker's Guide to the Galaxy"`

### Inserting multiple records

Now, let's add more than one piece of data at a time. I'm also going leave the `BookID` field off to illustrate what happens.

```sql
INSERT INTO Books(Title, Series, SeriesOrder, Author, Published, Rating)
    VALUES ('Memories of Ice', 'Malazan Book of the Fallen',3, 'Steven Erikson', 2001, 5.0),
           ('Unsouled', 'Cradle', '1', 'Will Wight', 2016, 3.7);
```

Here, I can add multiple rows in one query by separate them with a column. You'll notice I only say `VALUES` once. But I run my query, and then use `SELECT * FROM Books;`

```shell
sqlite> INSERT INTO Books(Title, Series, SeriesOrder, Author, Published, Rating)
   ...>     VALUES ('Memories of Ice', 'Malazan Book of the Fallen',3, 'Steven Erikson', 2001, 5.0),
   ...>            ('Unsouled', 'Cradle', '1', 'Will Wight', 2016, 3.7);
sqlite> select * from Books;
1|Hitchhiker's Guide to the Galaxy|Hitchhiker's Guide to the Galaxy|1|Douglas Adams|1979|4.2
2|Memories of Ice|Malazan Book of the Fallen|3|Steven Erikson|2001|5.0
3|Unsouled|Cradle|1|Will Wight|2016|3.7
```
and my data has been added!

### Auto Increment

But wait...where did those `BookID`s come from? Well, by default, when I have an `INTEGER PRIMARY KEY` field in SQLite,
this *auto-increments* by default (that is, each time I add a new record, the ID is automatically assigned to the next
available ID). 

Now, be aware that in most SQL dialect, you would actually need to explicitly state that the primary key is auto-increment at the time you create the table. For instance, in PostgreSQL, you would use

```sql
CREATE TABLE Books (
  BookID SERIAL PRIMARY KEY
  ...);
```

In MySQL, you would use

```sql
CREATE TABLE Books (
    BookID INT NOT NULL AUTO_INCREMENT,
    ...
    PRIMARY KEY (BookID)
);
```

This is one of the places I've seen the most differences. Be aware that there is also an `AUTOINCREMENT` keyword in SQLite, but it has a bit of a unique behavior - it ensures that no two records in **any** have the same ID regardless of Table. In general, don't worry too much about this, just be aware of it.


### Inserting without column names

You **can** insert without column names. For instance

```sql
INSERT INTO Books 
  VALUES(4, 'The Shadow Rising', 'The Wheel of Time', 4, 'Robert Jordan', 1992, 4.8);
```

This is called a **blind insert**. <ins>But you should **never** do this!</ins> The reasons for this are:  
1) You're assuming the column name order which can change. Columns can also be removed, renamed, etc.  
2) We should think of a record like an entry in a Map, where the key is the primary key, and the value is a mapping of column names to values (not unlike JSON) - `BookID=4 -> {"Title"="The Shadow Rising", "Series"="The Wheel of Time"...}`
3) The scheme of our table may change in a way that we *think* this insert is valid, but now we're inserting "The Wheel of Time" into `PublicationCity` because of changes.

As such, **never use** blind insert in your own code. Always explicitly list the columns you intend to insert to.

### Inserting null values

Now we reach Andrew Heaton's "Happier as Werewolves" which isn't part of a series. As such, we want to leave `Series` and `SeriesOrder` null.

One way to do this is simply to ignore those columns when inserting.

```sql
INSERT INTO Books(Title, Author, Published, Rating)
  VALUES ('Happier as Werewolves', 'Andrew Heaton', 2016, 3.9);
```

Now that we added our last row, we can use `SELECT * FROM Books` again and get:

```shell
sqlite> select * from books;
1|Hitchhiker's Guide to the Galaxy|Hitchhiker's Guide to the Galaxy|1|Douglas Adams|1979|4.2
2|Memories of Ice|Malazan Book of the Fallen|3|Steven Erikson|2001|5.0
3|Unsouled|Cradle|1|Will Wight|2016|3.7
4|The Shadow Rising|The Wheel of Time|4|Robert Jordan|1992|4.8
5|Happier as Werewolves|||Andrew Heaton|2016|3.9
```

You'll notice that the data isn't great to read in this format, it's hard to line-up for instance. We can somewhat improve this with:

```shell
sqlite> .mode columns
sqlite> SELECT * FROM Books;
BookID  Title                             Series                            SeriesOrder  Author          Published  Rating
------  --------------------------------  --------------------------------  -----------  --------------  ---------  ------
1       Hitchhiker's Guide to the Galaxy  Hitchhiker's Guide to the Galaxy  1            Douglas Adams   1979       4.2

2       Memories of Ice                   Malazan Book of the Fallen        3            Steven Erikson  2001       5.0

3       Unsouled                          Cradle                            1            Will Wight      2016       3.7

4       The Shadow Rising                 The Wheel of Time                 4            Robert Jordan   1992       4.8

5       Happier as Werewolves                                                            Andrew Heaton   2016       3.9

```

But in general, I wouldn't worry too much about making the console pretty, as in practice we're not going to be working with the console directly very frequently.


## SELECT

### WHERE

## Transactions

## UPDATE

## DELETE

## DROP