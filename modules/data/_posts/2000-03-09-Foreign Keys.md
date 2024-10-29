---
Title: Foreign Keys
---

# Foreign Keys

In this module, we will look at foreign keys in SQLite, how to enable foreign key enforcement in SQLite.

* TOC
{:toc}

## Courses Database

For this unit, we will be looking at a simple database for Course enrollment, which you can find here:

** [courses.db](https://github.com/sde-coursepack/JDBC-with-SQLite/blob/main/courses.db) **

You can download this file by clicking the download icon in the top right. This is a sqlite3 compatiable database file, like `books.db` from the last module. You can open this with the sqlite3 command:

```shell
> sqlite3 courses.db
```

This database has three tables:

### Students

This table represents our set of students. The query that created the table is:

```sqlite
CREATE TABLE IF NOT EXISTS Students (
    StudentId INTEGER PRIMARY KEY,
    FirstName TEXT,
    LastName TEXT NOT NULL,
    ComputingID TEXT UNIQUE NOT NULL
) STRICT;
```

Each student necessarily has a unique integer ID (primary key), a last name, and a unique computingID. They optionally have a first name (a student may have a mononym, or single name. That name would be stored in the last name field).

The starting data of the table is:

<ins>**Students**</ins>

| StudentID | FirstName | LastName | ComputingID |
|-----------|-----------|----------|-------------|
| 1         | Jonathan  | Doe      | abc2def     |
| 2         | Jane      | Smith    | ghi3jkl     |


### Courses

This table represents a list of Courses offered in a given semester. It's create table query:

```sqlite
CREATE TABLE IF NOT EXISTS Courses (
    Crn INTEGER PRIMARY KEY,
    Subject TEXT NOT NULL,
    CourseNumber INTEGER NOT NULL,
    Section INTEGER NOT NULL,
    MeetingTime TEXT,
    UNIQUE (Subject, Section, CourseNumber)
) STRICT;
```

Here, each course has a **Course Registration Number** (CRN) that acts as the course's unique integer ID and primary key. Each course must have a Subject, CourseNumber, and Section. For instance, CS 3140-001 has Subject "CS", CourseNumber "3140", and Section 1 (001 is mathematically equal to 1). From there, a course can optional have a MeetingTime (some courses may not have regular meetings, and would leave this null, or the meeting time is as yet undecided).

Additionally, each combination of Subject, CourseNumber, and Section must be unique. For instance, if we already have CS 3140-001, then we cannot add another record that is *also* CS 3140-001. However, we can have:

* CS 3140-001  
* CS 3140-002  
* APMA 3140-001  

None of these three courses would violate our uniqueness constraint, since we are saying that **all three in combination** must be unique.

For starters, we only have 1 course right now.

<ins>**Courses**</ins>

| CRN   | Subject | CourseNumber | Section | MeetingTime |
|-------|---------|--------------|---------|-------------|
| 12345 | CS      | 3140         | 1       | 14:00-15:15 |

### Enrollments

This table lists the pairs of StudentIDs and CourseIDs to track student enrollment. This is handled in a table separate from Students or Courses to ensure data normalization. In another module, we will discuss data normalization and normal forms. For now, just know this pattern is called a "Bridge Table", and is used to handle "many-to-many" connections. "Many-to-Many" means two things:

1) many students can enroll in 1 course  
2) 1 student can enroll in many courses  

The create table query:

```sqlite
CREATE TABLE Enrollments(
    EnrollmentID INTEGER PRIMARY KEY,
    StudentID INTEGER NOT NULL,
    CRN INTEGER NOT NULL,
    UNIQUE (StudentID, CRN),
    FOREIGN KEY (StudentID) REFERENCES Students (StudentID) ON DELETE CASCADE,
    FOREIGN KEY (CRN) REFERENCES Courses (CRN) ON DELETE CASCADE
) STRICT;
```

For now, let's skip over the foreign key and just focus on what data the table stores. Each record of the table has a unique integer key (EnrollmentID). It also has a StudentID and a CRN. The StudentID identifies a single Student (i.e. it matches exactly one StudentID in our Students Table) and the CRN represents a Course in our Courses Table (i.e., it matches exactly one CRN in our Courses Table)

Each record indicates a single student enrolled in a single course. So, for example, CS 3140-001 (which has CRN = 12345) had 500 students, then you would expect 500 records in this table to have 12345 in the CRN attribute, each with a unique StudentID. Similarly, if Jonathan Smith (StudentID = 1) is enrolled in 5 courses, then you would expect 5 records with StudentID 1, each with a unique CRN.

Our initial data is:

<ins>**Enrollments**</ins>

| EnrollmentID | StudentID | CRN   |
|--------------|-----------|-------|
| 1            | 1         | 12345 |
| 2            | 2         | 12345 |

So in our initial data, we have two students and one course. Both students are enrolled in that course. We will add more data as we go, but this is our starting point.

## `FOREIGN KEY`

In SQL, a `FOREIGN KEY` is a **reference** from a record in one table to a record in, typically, another table (it is possible to have a foreign key to another attribute in the same table, but the use cases for that are limited enough that we won't focus on it).

Specifically, in our above tables, we want to ensure that, for every record in **Enrollments**, that the StudentID references a *real* student, and the CRN represents a *real* course. That is, we want to disallow someone adding:

| EnrollmentID | StudentID | CRN   |
|--------------|-----------|-------|
| 3            | 99999     | 99999 |

...because right now there **is** no student 99999, and there is no Course with CRN (99999). That is, we want to enforce **data integrity**.


### Foreign key syntax:

Looking at our `CREATE TABLE` query for Enrollments, we see the following table constraints

```sqlite
    FOREIGN KEY (StudentID) REFERENCES Students (StudentID) ON DELETE CASCADE,
    FOREIGN KEY (CRN) REFERENCES Courses (CRN) ON DELETE CASCADE
```

Focusing on the first one, let's ignore the `ON DELETE CASCADE` bit for now:

```sqlite
FOREIGN KEY (StudentID) REFERENCES Students (StudentID)
```

The above means is that in the **Enrollments** Table (the table being created by this query), the field `Enrollments.StudentID` is referencing the Students Table, specifically `Students.StudentsID`. This means that **every** `Enrollments.StudentID` for every record in our Enrollments table **must** exist in the `Students` table. So this would reject us adding a record to Enrollment for StudentID 99999, since there is no student with that ID in our Students table.

While it may seem confusing to have two columns with the same column name, this is actually intended. Just like when writing code, we want our database to be easily read and understood. The two columns having the same name immediately implies a foreign key relationship, even if you don't dig into the table schema.


### Enabling Foreign Keys in SQLite

By default, foreign keys in SQLite **are not enforced**. Meaning if I open the database in sqlite3 shell and write an insert query to add the data (3, 99999, 99999) to enrollments, I can. But that's bad, because now I've introduced data that **shouldn't** exist! How can a student that doesn't exist enroll in a class that doesn't exist?

I can **enable** foreign key enforcement with:

```sqlite
PRAGMA foreign_keys=ON;
```

In SQLite, PRAGMA statements are used to modify how SQLite operates **during the current connection**. There are [**many** PRAGMA options](https://www.sqlite.org/pragma.html), but for this class, we will really only worry about the one above to enable foreign key enforcement.

Additionally, realize this is **per connection**. This means if you `.quit` out of the sqlite3 terminal, when you reconnect, you will need to do `PRAGMA foreign_keys=ON;` **again**. There is no "permanent" way to turn on foreign key enforcement for a database. 

### Foreign Key Enforcement

Let's see what happens when we try to add our invalid data **after** using `PRAGMA foreign_keys=ON;`

```sqlite
INSERT INTO Enrollments(StudentID, Crn)
    VALUES (99999, 99999);
```

The shell will say:

```shell
sqlite> PRAGMA foreign_keys=ON;
sqlite> INSERT INTO Enrollments(StudentID, Crn)
   ...>     VALUES (99999, 99999);
Runtime error: FOREIGN KEY constraint failed (19)
```

That is, we violated a foreign key constraint. Specifically, the StudentID 99999 doesn't exist in the Students table, and the CRN 99999 doesn't exist in the Courses Table. Because of this violation the data was never added. To ensure this, we do a select.

```shell
sqlite> SELECT * FROM Enrollments;
EnrollmentID  StudentID  CRN
------------  ---------  -----
1             1          12345
2             2          12345
```

Note that we would get the same foreign key constraint error for:

```sqlite
INSERT INTO Enrollments(StudentID, Crn)
    VALUES (99999, 12345);
```

... and ...

```sqlite
INSERT INTO Enrollments(StudentID, Crn)
    VALUES (1, 99999);
```

The point here is that if *either* foreign key constraint is violated, then the entire record is rejected.

### ON DELETE CASCADE

Now let's look at the last bit:

```sqlite
FOREIGN KEY (StudentID) REFERENCES Students (StudentID) ON DELETE CASCADE,
```

`ON DELETE CASCADE` means "If the referenced Student in the Students table is deleted, then *cascade* that deletion to all records with that StudentID in Enrollments".

For instance, let's say we add a student "Bob Quitsalot" with the following queries.

```sqlite
INSERT INTO Students (FirstName, LastName, ComputingID)
    VALUES ('Bob','Quitsalot','qui7ter');
```

So now our StudentsTable is:

```shell
sqlite> SELECT * FROM Students;                                 
StudentId  FirstName  LastName   ComputingID
---------  ---------  ---------  -----------
1          Jonathan   Doe        abc2def
2          Jane       Smith      ghi3jkl
3          Bob        Quitsalot  qui7ter
```

And Bob registers for CS 3140-001:

```sqlite
INSERT INTO Enrollments (StudentID, Crn)
   VALUES(3, 12345);
```

Which we can see is in our table:

```shell
sqlite> SELECT * FROM Enrollments;                              
EnrollmentID  StudentID  CRN
------------  ---------  -----
1             1          12345
2             2          12345
3             3          12345
```

However, unfortunately, it appears that Bob Quitsalot has decided to withdraw from our university. This means we need to delete him from our Students table.

When we do this, `ON DELETE CASCADE` will ensure our table remains in a valid state by **also** deleting all records in Enrollments that **reference** Bob Quitsalot (i.e. StudentID = 3) **automatically**. Just like a waterfall cascades water from the top of a cliff to the bottom, we are telling SQLite to automatically cascade the `DELETE` query from the **referenced** table (Students) to the **referencing** table (Enrollment).

So, we run our `DELETE` query with appropriate transaction operations, using `SELECT` to verify our results:

```shell
sqlite> BEGIN TRANSACTION;
sqlite> DELETE FROM Students
   ...>   WHERE StudentID = 3;
sqlite> SELECT * FROM Students;                  
StudentId  FirstName  LastName  ComputingID
---------  ---------  --------  -----------
1          Jonathan   Doe       abc2def
2          Jane       Smith     ghi3jkl
sqlite> SELECT * FROM Enrollments;                              
EnrollmentID  StudentID  CRN
------------  ---------  -----
1             1          12345
2             2          12345
sqlite> COMMIT;
```

And you'll notice that in addition to deleting one record from Students, we also deleted one record from Enrollments (specifically the record with StudentID = 3), despite only running a query on Students.

### Without ON CASCADE DELETE

If we **didn't** have `ON CASCADE DELETE` for on our foreign key, then when I tried to delete the student, I couldn't! I would get:

```shell
sqlite> DELETE FROM STUDENTS WHERE StudentID=3;
Runtime error: FOREIGN KEY constraint failed (19)
```

This is because if I deleted the Bob Quitsalot from my Students Table, then my Enrollments table would be in an invalid state. Specifically, my enrollments table would still include the record:

| EnrollmentID | StudentID | CRN   |
|--------------|-----------|-------|
| 3            | 3         | 12345 |

But since StudentID 3 wouldn't exist anymore, this means that Enrollments is now in violation of our Foreign Key constraint!

We would have to do a workaround by first deleting all the records associated with StudentID 3 in Enrollment **first**, and only **then** could we delete Students. `ON CASCADE DELETE` simply does this for us automatically by ensuring that when we delete Bob Quitsalot in Students, it deletes any records that **reference** that record via a foreign key in any other table.

