# CRUD with JPA

## Learning Goals

- Modify database creation behavior in persistence context.
- Read from the database.
- Update database records.
- Delete database records.

## Introduction

We have only been creating and inserting data up to now. In this lesson we will
learn how to fetch, update, and delete records from the database. But before
that, we need to change the database creation behavior so that it doesn’t drop
and recreate the database every time the program is run.

## Change Persistence Context Behavior

This what our `persistence.xml` file currently looks like:

```xml
<persistence xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
                      http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
             version="2.0" xmlns="http://java.sun.com/xml/ns/persistence">

    <persistence-unit name="example" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.ejb.HibernatePersistence</provider>
        <properties>
            <!-- connect to database -->
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:tcp://localhost/~/test" />
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver" />
            <property name="javax.persistence.jdbc.user" value="sa" />
            <property name="javax.persistence.jdbc.password" value="" />
            <!-- configure behavior -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="hibernate.hbm2ddl.auto" value="create-drop" />
        </properties>
    </persistence-unit>
</persistence>
```

The `hibernate.hbm2ddl.auto` property defines how database creation works when
the program starts. Some of the common values are:

- `create-drop`: This drops all databases and creates new ones from scratch. It
  drops the database schema when the entity manager is closed using the
  `entityManager.close()` method.
- `create`: It is similar to `create-drop` but it does not drop the database
  tables when the entity manager is closed.
- `validate`: Checks if the entity definitions match an existing table schema.
- `update`: Does not drop databases. Only updates the table schema.
- `none`: Does not make any changes to the database.

We will be using the `create` value to create and insert data into the database
and then use the `update` value for reading, updating, and deleting data.

Change the `hibernate.hbm2ddl.auto` value in the `persistence.xml` file to
`create`:

```java
<property name="hibernate.hbm2ddl.auto" value="create" />
```

## Create and Insert Data

We will rename our `JpaMain.java` file to `JpaCreate.java`. Make sure to use the
“refactor” feature to rename the directory to prevent any errors. We will also
explicitly close the `EntityManager` and `EntityManagerFactory` instances. If
you run the following code, the H2 database in your machine should have the two
student records.

```java
// JpaCreate.java

package org.example;

import org.example.enums.StudentGroup;
import org.example.models.Student;

import javax.persistence.Persistence;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityManager;
import javax.persistence.EntityTransaction;
import java.util.Date;

public class JpaMain {
    public static void main(String[] args) {
        // create student instances
        Student student1 = new Student();
        student1.setName("Jack");
        student1.setDob(new Date());
        student1.setStudentGroup(StudentGroup.LOTUS);

        Student student2 = new Student();
        student2.setName("Leslie");
        student2.setDob(new Date());
        student2.setStudentGroup(StudentGroup.ROSE);

        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // access transaction object
        EntityTransaction transaction = entityManager.getTransaction();

        // create and use transactions
        transaction.begin();
        entityManager.persist(student1);
        entityManager.persist(student2);
        transaction.commit();

        // close entity manager
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

| ID  | DOB        | NAME   | STUDENTGROUP |
| --- | ---------- | ------ | ------------ |
| 1   | 2022-06-12 | Jack   | LOTUS        |
| 2   | 2022-06-12 | Leslie | ROSE         |

## Read Data

We can read from the database regardless of what mode the `hbm2ddl` property
value is. We will be using the `update` value for the `hbm2ddl` property because
often times you will have to read from an existing database. Modify your
`persistence.xml` file so that the following property has the `udpate` value:

```xml
<property name="hibernate.hbm2ddl.auto" value="update" />
```

From this point on, we will perform the read, update, and delete operations in
their own files. Create `JpaRead`, `JpaUpdate`, and `JpaDelete` files. Your
directory should look like this:

```xml
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   │   └── org
    │   │       └── example
    │   │           ├── JpaCreate.java
    │   │           ├── JpaDelete.java
    │   │           ├── JpaRead.java
    │   │           ├── JpaUpdate.java
    │   │           ├── enums
    │   │           │   └── StudentGroup.java
    │   │           └── models
    │   │               └── Student.java
    │   └── resources
    │       └── META-INF
    │           └── persistence.xml
    └── test
        └── java
```

We will also add a `toString()` method to the `Student` class so it’s easier to
read values logged in the console.

```java
@Entity
@Table(name = "STUDENT_DATA")
public class Student {
    @Id
    @GeneratedValue
    private int id;

    private String name;

    @Temporal(TemporalType.DATE)
    private Date dob;

    @Enumerated(EnumType.STRING)
    private StudentGroup studentGroup;

		// getters and setters

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", dob=" + dob +
                ", studentGroup=" + studentGroup +
                '}';
    }
}
```

Open the `JpaRead` class and add the following code:

```java
package org.example;

import org.example.models.Student;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

public class JpaRead {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // get records
        Student student1 = entityManager.find(Student.class, 1);
        System.out.println(student1);

        // close entity manager
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

Output of running the `main` method:

```java
Student{id=1, name='Jack', dob=2022-06-12, studentGroup=LOTUS}
Student{id=2, name='Leslie', dob=2022-06-12, studentGroup=ROSE}
```

We need an `EntityManager` for connecting to the database as before. The `find`
method on the `EntityManager` instance can be used to query the data from the
database. The first argument to `find` is the class reference of the entity we
are fetching and the second argument is the unique identifier.

## Update Data

We want to change the first student’s `StudentGroup` value from `LOTUS` to
`DAISY` in the database. Here are the steps we have to follow to update and
persist the change:

1. Create an entity manager.
2. Read the data from the database and create a local `Student` instance.
3. Modify the `studentGroup` field of the instance to `DAISY`.
4. Get the transaction from the entity manager.
5. Persist the updated instance within a new transaction.

Add the following code to the `JpaUpdate` class:

```java
package org.example;

import org.example.enums.StudentGroup;
import org.example.models.Student;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class JpaUpdate {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // get record
        Student student1 = entityManager.find(Student.class, 1);

        // modify student instance
        student1.setStudentGroup(StudentGroup.DAISY);

        // access transaction object
        EntityTransaction transaction = entityManager.getTransaction();

        // create and use transaction to save updated value
        transaction.begin();
        entityManager.persist(student1);
        transaction.commit();

        // close entity manager
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

Notice that the process of creating an entity manager, getting the data from the
database, and writing transactions is the exact same as what we used in the
`JpaCreate` and `JpaRead` classes. We only had to update the `studentGroup`
property of the `student1` instance before persisting it to update the value in
the database.

| ID  | DOB        | NAME   | STUDENTGROUP |
| --- | ---------- | ------ | ------------ |
| 1   | 2022-06-12 | Jack   | DAISY        |
| 2   | 2022-06-12 | Leslie | ROSE         |

## Delete Data

We can delete a record by using the `remove` method on the entity manager. The
deletion steps are similar to the update steps:

1. Create an entity manager.
2. Read the data from the database and create a local `Student` instance.
3. Get the transaction from the entity manager.
4. In the transaction, call the `remove` method on the entity manager and pass
   it the `Student` instance you want to remove from the database.

Here is what the `JpaDelete` class should look like:

```java
package org.example;

import org.example.models.Student;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class JpaDelete {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // get record
        Student student1 = entityManager.find(Student.class, 1);

        // access transaction object
        EntityTransaction transaction = entityManager.getTransaction();

        // create and use transaction to save updated value
        transaction.begin();
        entityManager.remove(student1);
        transaction.commit();

        // close entity manager
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

And this is what the `STUDENT_DATA` table will look like after running the
`main` method in the `JpaDelete` class.

| ID  | DOB        | NAME   | STUDENTGROUP |
| --- | ---------- | ------ | ------------ |
| 1   | 2022-06-12 | Jack   | DAISY        |
| 2   | 2022-06-12 | Leslie | ROSE         |

## Revert Database Changes

We have made a few changes using the update and delete operation in our
database. We will run the `JpaCreate` class to create the database from scratch
and insert the data:

1. Change the `hibernate.hbm2ddl.auto` value to `create`.
2. Run the `main` method in the `JpaCreate` class.

The `STUDENT_DATA` table should look like this:

| ID  | DOB        | NAME   | STUDENTGROUP |
| --- | ---------- | ------ | ------------ |
| 1   | 2022-06-12 | Jack   | LOTUS        |
| 2   | 2022-06-12 | Leslie | ROSE         |

## Conclusion

We have learned how to create, insert, read, update, and delete data from the
database. These are essential operations that you are likely to perform in
pretty much any app that requires data persistence.
