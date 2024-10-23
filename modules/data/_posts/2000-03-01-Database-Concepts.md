---
Title: Databases and SQL
---

# Databases and SQL

In the next several modules, we will discuss database usage. Specifically, for this class, we will use SQLite as our database.

* TOC
{:toc}

# The goal of this unit

In the next several modules (including this one) we will cover basic database concepts, usage, and best practices.

Software Development Essentials is not a class on databases. Rather, we are introducing "just enough" database content to use databases in our software projects for persistence. This means you should come into this unit with the following expectations:

* While we are using SQLite, there are many other database paradigms. SQLite in particular is lightweight and "easy to start up", but there are some important differences between SQLite and MySQL, PostgreSQL, and other popular database management systems. There are also some features that SQLite doesn't support, such as "users", database permissions, etc.
* The primary goal of this unit is to learn the basics of SQL queries for storing, retrieving, editing, and deleting data. Things like database optimization, synchronization, etc. are beyond the scope of this class.

# What is a database

In short, a database is "a place to persist data". In **Data Persistence**, we talked about the need to have persistent
data that can stored by our programs for later access. A database is just that, a place to store data.

## Relational Database

In a relational database, database is stored in one or more tables, often with constraints on the contents of the tables to store data integrity. Data is explicitly formatted where all data in a single table matches the format, and tables are "related" to on other through "foreign keys" (more on this later).

Some examples of relational database systems are **SQLite**, PostgreSQL, MySQL, etc. All popular relational database systems use some dialect of **SQL** - Structured Query Language. 

For this and the next several modules, we will be using **SQLite**. In general, difference between different models can exist, but are minimal. For instance, all three database systems above (as well as every other I worked with) have virtually identical queries for creating, querying, updating, and deleting tables. Where differences exist they are minor.

This is to say that by learning the basics SQLite, you are basically also learning PostgreSQL, MySQL, etc. outside of typically very minor syntax usage and individual quirks of management systems.

## Non-relationship database

While not the focus on this module, you should be aware of the existence of non-relational databases, often called NoSQL databases (since they do not use structurally enforced tables). In non-relational database, data is often stored in documents, often in a key-value JSON-like format, with typically no explicitly enforced structure (however, this is more variety among NoSQL).

Some examples of non-relational database systems are MongoDB, Apache Cassandra, Redis, etc. Firebase, a popular "backend" tool, especially for mobile and web applications, typically uses Cloud Firestore, a NoSQL database hosted by Google Cloud. 

The rest of this modules, and the next several modules, focuses on **relational databases** (aka, SQL Databases)

# Tables

In a SQLite Database, a **table**</ins> is, as the name suggests, a specific that divides data into multiple **attributes**, and store multiple
units of data as **records**. For instance, consider the table below.

**<ins>Students</ins>**  

| StudentID &#128273; | LastName | FirstName | ComputingID | Major                 | ClassYear |
|:--------------------|:---------|:----------|:------------|:----------------------|:----------|
| 123456789           | Doe      | Jane      | abc2def     | Computer Science (BS) | 1         |
| 987654321           | Smith    | John      | ghi9jkl     | Systems Engineering   | 2         |
| 008675309           | Jenny    | Jenny     | igo7urn     | Telecommunications    | 4         |

The above table may keep a record of information about students (hence the table name **<ins>Students</ins>**) -- tables should generally be named as **plural nouns**. 

Each row in the table represents one student's **record**. The data about a single student is all in the same row, and since our table as 3 records, this implies we only have 3 students. If more students are enrolled, we can add their information to the table.

Each column in our table represents a specific **attribute** of a student. For instance, the students first name and last name are in the 2nd and 3rd columns respectively. Every attribute has a name, which is a **singular noun phrase** describing what the column represents. While not visible in the above table explicitly, each attribute also has a specific **datatype**

## DataTypes

For now, we will describe 4 datatypes for columns that are used by SQLite. Note that many database implementations have many more datatypes. In fact, SQLite can *mimic* datatypes not listed, like Boolean, DateTime, etc. But to get started, focus on these 4.

* **Text** - like a Java `String`, text is represented as a String of text. This is typically stored in UTF-8, the default encoding of a SQLite database. The size and composition of the Text determines how much large it is (dynamically-sized)  
* **Integer** - a whole number, like a Java `int`, stored as a value of up to 8 bytes (which is equivalent to a Java `long`) depending on its magnitude.  
* **Real** - a floating point number, like a Java `double`. It is stored in 8 bytes (matching a Java double)  
* **Blob** - a group of data, stored exactly as it was input. Blob stands for **Binary Large OBject**, typically used for storing file data, like images, videos, or any other "raw" data. The storage size is dependent on the size of the object being stored.  

In the above table, StudentID and ClassYear are **Integer**. The other attributes are all **Text**. I could have a column like **GPA** that would be a Real (although most likely rather than **storing** GPA, I would **derive** GPA by looking a student's past grades), and I could have a **Blob** for something like "IDPicture", which is an image file of the photo on the student's ID.

There is also a **Null** datatype that can, by default, be stored in any column associated with the above datatypes. For instance, if a student is, say, taking some classes with UVA, but is not a full time student or is not seeking graduation, maybe their ClassYear is Null. This is signified by leaving the row blank. Or if a student uses a mononym (single name, likely stored as "LastName" in our table), then the FirstName column might be blank. We'll discuss how to make a column **not nullable** in a bit.

## Primary Key

You'll notice a "key" symbol (&#128273;) next to StudentID. That's because this column is the **primary key**. In every table, we want one column to serve as a **primary key** which can be used to identify a single record (in this case a single student). Note that while you *can* have a *composite primary key* (a primary key over more than one column) this is generally considered bad practice.

All primary keys must be:
* Unique - each record must have a **different** primary key than every other record
* Not null - no record may have a blank (or Null) primary key

In general, our Primary Key should always be a **Integer**, often automatically generated. We will discuss this more when talking about create queries in the next module.

# Other concepts

We will cover more concepts as we need them, like foreign keys, constraints, etc. In this module, we simple what to clarify what a relational database is, and what tables, attributes, and records are. In the next module, we will dive in SQLite command line and start looking at the basics of our SQL query-writing language.

