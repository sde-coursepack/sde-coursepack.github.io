---
Title: Hibernate and ORMs
---

* TOC
{:toc}

# Hibernate and ORMs

So far, we've been writing raw SQL queries to handle database operations. Now, SQL can be very powerful, but in most cases all we're trying to do is save Java objects to the database and retrieve them as objects when needed. In these cases, adding in long-winded SQL makes everything less understandable and opens up more opportunities for defects. Wouldn't it be nice to have something that could handle these standard operations for us?

Enter **object-relational mappings**, or ORMs. ORMs take care of all the mapping between an object and its corresponding database record, allowing us to use databases without ever leaving the object-oriented paradigm.

## Hibernate

[**Hibernate**](https://hibernate.org/) is an open-source ORM for Java. It handles the mapping between objects and database records, and it allows us to query and retrieve objects from the database.

### Setting Up Hibernate

We can import Hibernate using Gradle:

```text
dependencies {
    // we still need JDBC!
    implementation group: 'org.xerial', name: 'sqlite-jdbc', version: '3.36.0.3'
    // Hibernate
    implementation group: 'org.hibernate', name: 'hibernate-core', version: '5.6.10.Final'
    // Java Persistence API (a standard that Hibernate is built off of)
    implementation group: 'javax.persistence', name: 'javax.persistence-api', version: '2.2'
    // add SQLite support to Hibernate
    implementation group: 'com.github.gwenn', name: 'sqlite-dialect', version: '0.1.2'
}
```

There are a couple files that we need to add to our application in order for Hibernate to work. The first one, `HibernateUtil.java`, can basically be copy-pasted into your code:

```java
package edu.virginia.cs; // replace with your package name

import org.hibernate.HibernateException;
import org.hibernate.SessionFactory;
import org.hibernate.boot.Metadata;
import org.hibernate.boot.MetadataSources;
import org.hibernate.boot.registry.StandardServiceRegistry;
import org.hibernate.boot.registry.StandardServiceRegistryBuilder;

public class HibernateUtil {
    private static SessionFactory sessionFactory = buildSessionFactory();

    private static SessionFactory buildSessionFactory()
    {
        try
        {
            if (sessionFactory == null)
            {
                StandardServiceRegistry standardRegistry = new StandardServiceRegistryBuilder()
                        .configure("hibernate.cfg.xml").build();

                Metadata metaData = new MetadataSources(standardRegistry)
                        .getMetadataBuilder()
                        .build();
                sessionFactory = metaData.getSessionFactoryBuilder().build();
            }
            return sessionFactory;
        } catch (HibernateException ex) {
            System.err.println("Initial sessionFactory creation failed." + ex);
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }

    public static void shutdown() {
        getSessionFactory().close();
    }
}
```

The other one is `hibernate.cfg.xml`, which goes under `src/main/resources` and tells Hibernate everything it needs to know for the mapping, such as where the database is located and which classes to map to the database:

```xml
<hibernate-configuration>
    <session-factory>
        <property name="show_sql">false</property>
        <property name="format_sql">false</property>
        <property name="dialect">org.sqlite.hibernate.dialect.SQLiteDialect</property>
        <property name="connection.driver_class">org.sqlite.JDBC</property>
        <!-- replace my_db.sqlite3 with the name of your database! -->
        <property name="connection.url">jdbc:sqlite:my_db.sqlite3</property>
        <property name="connection.username"></property>
        <property name="connection.password"></property>

        <property name="hibernate.hbm2ddl.auto" >update</property>

        <!-- list each class that should be mapped -->
        <mapping class="edu.virginia.cs.State" />
        <mapping class="edu.virginia.cs.Client" />
        <mapping class="edu.virginia.cs.Account" />
        <mapping class="edu.virginia.cs.Transaction" />
    </session-factory>
</hibernate-configuration>
```

### Using Hibernate

Hibernate revolves around **sessions**. Think of them like connections in JDBC.

We can get a session object using the `HibernateUtil` class from above:

```java
session = HibernateUtil.getSessionFactory().openSession();
```

Once we have a session, we need to start a **transaction**. Inside a transaction, we can do whatever we want. At the end, we just commit the whole transaction; if something fails, the whole thing rolls back to how it was before we started, and we have no mess.

```java
// start the transaction
session.beginTransaction();
// get an object from the database by ID
object = session.get(MyObject.class, id);
// change something about the object (this doesn't affect the database!)
object.setName("Changed Object");
// save the object back to the database (doesn't actually take effect until the transaction is committed!)
session.persist(object);
// now we commit the transaction and actually save the changes to the database
session.getTransaction().commit();
```

### Entity Classes

**Entity classes** are classes that map to tables and records in the database via Hibernate. We add some extra content to them so that Hibernate knows exactly how to handle their various properties. Take a look at this `State` class, for example:

```java
package edu.virginia.cs;

import javax.persistence.*;

// this is an entity class
@Entity
// name the database table
@Table(name = "STATES")
public class State {
    // this is equivalent to PRIMARY KEY AUTOINCREMENT in SQL
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    @Column (name = "ID")
    private int id;

    // specify the database column name, make values unique, don't allow null, cap length to 32
    @Column(name="STATE_NAME", unique = true, nullable = false, length = 32)
    private String name;

    // has to be an integer, can't be null
    @Column(name = "POPULATION", nullable = false)
    private int population;

    // has to be a string -- or VARCHAR -- and not null, though this doesn't have an explicit max length
    @Column(name = "CAPITOL_CITY", nullable = false)
    private String capitolCity;

    // foreign key! this is a many-to-one (many States to one Country) relation, referencing the Country's ID
    @ManyToOne
    @JoinColumn(name = "COUNTRY_ID", referencedColumnName = "ID")
    private Country country;

    // and in the other direction! Every City object has a state field of type State
    @OneToMany(mappedBy = "state")
    private List<City> cities;

    public State() {
        // a zero-argument constructor is required for Hibernate to work correctly
    }

    // past this point, it's just a normal class

    public State(String name, int population, String capitolCity) {
        this.name = name;
        this.population = population;
        this.capitolCity = capitolCity;
    }

    // getters and setters (omitted for brevity)
    // any field represented by a table column needs a getter and a setter
}
```

Note that if you're working with related objects (e.g., a `Country` and its `State`s), you'll need to call `session.persist()` for each object! Don't just call `persist` on the `Country` without all the `State`s or you'll have a bad time.

## Hibernate Query Language

Hibernate comes with its own language! **Hibernate Query Language** (HQL) is similar to SQL, but it interfaces with Hibernate instead of directly going to the database, so we can perform more advanced queries like in SQL without losing any of the benefits of Hibernate, such as automatically returning objects.

There are a few differences between HQL and SQL:

* HQL uses class names, not table names.
* HQL uses field names, not column names.
* `SELECT` is optional when the query is typed.
* We don't have to say `SELECT *`.

For instance, the HQL query `from State where name = 'Virginia'` is equivalent to the SQL query `SELECT * FROM STATES WHERE STATE_NAME = 'Virginia'` (but it's a lot easier to type!)

Once we've written our HQL query, we pass it to our session, which gives us a `Query` object. We can retrieve our results from this object.

```java
String hql = "from State where name = 'Virginia'";
Query<State> stateQuery = session.createQuery(hql);
State virginia = stateQuery.getSingleResult();
```

We can also use named parameters, which prevent issues like having a query break over a string such as `Fry's Spring` (note the apostrophe!):

```java
// :name creates a named parameter called "name"
String hql = "FROM Neighborhood where name = :name";
Query<Neighborhood> query = session.createQuery(hql, Neighborhood.class);
query.setParameter("name", "Fry's Spring");
Neighborhood result = query.getSingleResult();
```
