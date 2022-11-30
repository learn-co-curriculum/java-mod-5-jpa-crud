# CRUD with JPA

## Learning Goals

- Define database creation behavior with the `hibernate.hbm2ddl.auto` property.
- Read an entity from the database.
- Update an entity in the database.
- Delete an entity from the database.

## Introduction

We have used JPA to persist an entity to the database.  In this lesson we will
learn how to read, update, and delete an entity from the database. 
We will also see how  to set database creation 
behavior using the `hibernate.hbm2ddl.auto` property.

## Code Along

We will continue with the project from the previous lesson.

Add 3 new classes to the `org.example` package (make sure they are not created in org.example.model):

1. JpaReadStudent
2. JpaUpdateStudent
3. JpaDeleteStudent


Your project structure should look like this:

![Final JPA project structure](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa/jpa_final_project_structure.png)

## Persistence Context Behavior

The file `persistence.xml` currently looks like this:

```xml
<persistence xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://java.sun.com/xml/ns/persistence
                         http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd"
             version="2.0" xmlns="http://java.sun.com/xml/ns/persistence">

    <persistence-unit name="example" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.ejb.HibernatePersistence</provider>
        <properties>
            <!-- connect to database -->
            <property name="javax.persistence.jdbc.driver" value="org.postgresql.Driver" /> <!-- DB Driver -->
            <property name="javax.persistence.jdbc.url" value="jdbc:postgresql://localhost:5432/student_db" /> <!--DB URL-->
            <property name="javax.persistence.jdbc.user" value="postgres" /> <!-- DB User -->
            <property name="javax.persistence.jdbc.password" value="postgres" /> <!-- DB Password -->
            <!-- configure behavior -->
            <property name="hibernate.hbm2ddl.auto" value="create" /> <!-- create / create-drop / update -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.PostgreSQL94Dialect"/>
            <property name="hibernate.show_sql" value="true" /> <!-- Show SQL in console -->
            <property name="hibernate.format_sql" value="true" /> <!-- Show SQL formatted -->
        </properties>
    </persistence-unit>
</persistence>
```

The `hibernate.hbm2ddl.auto` property defines how database creation works when
the program starts. Some common values are:

- `create-drop`: This drops all databases and creates new ones from scratch. It
  drops the database schema when the entity manager is closed using the
  `entityManager.close()` method or when the `try-with-resources` statement completes.
- `create`: It is similar to `create-drop` but it does not drop the database
  tables when the entity manager is closed.  
- `validate`: Checks if the entity definitions match an existing table schema.
- `update`: Does not drop databases. Only updates the table schema.
- `none`: Does not make any changes to the database.

In the previous lessons, we used the `create` value to create and insert data into the database.

In this lesson, we will read, update, and delete data from the database.
We need to change the `hibernate.hbm2ddl.auto` property to `update`
to prevent the database from being recreated when we run JPA code.

1. Update `persistence.xml` to set the `hibernate.hbm2ddl.auto` value to `update`:
2. Be sure to save the file before proceeding.

```java
<property name="hibernate.hbm2ddl.auto" value="update" />
```

## Read Data

Now we will use JPA to fetch a student object from the database.

Add the following code to the `JpaReadStudent` class:

```java
package org.example;

import org.example.model.Student;
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

public class JpaReadStudent {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // get student data using primary key id=1
        Student student1 = entityManager.find(Student.class, 1);
        System.out.println(student1);

        // close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

The steps to fetch a `Student` object from the database
based on the primary key value are as follows:

1. Use `EntityManagerFactory` to create a single instance of `EntityManager`.
2. The `find` method returns a `Student` object based on the method parameters:
    - `Student.class` indicates the entity class, which provides information about the database table and primary key.
    - `1` indicates the primary key value.

The `find` method returns null if the entity is not in the database.

### Run `JpaReadStudent.main`

1. Run the `JpaReadStudent.main` method. This will query the database for the student entity with `id=1`.
2. In IntelliJ, check out the “Run” tab to see the exact query that Hibernate used
   to find the entity.

```text
Hibernate: 
    select
        student0_.id as id1_0_0_,
        student0_.dob as dob2_0_0_,
        student0_.name as name3_0_0_,
        student0_.studentGroup as studentg4_0_0_ 
    from
        STUDENT_DATA student0_ 
    where
        student0_.id=?

```

The `JpaReadStudent.main` implicitly calls the `toString()` method to print the `Student` object state:

```text
Student{id=1, name='Jack', dob=2000-01-01, studentGroup=ROSE}
```


## Update Data

Now we will use JPA to update a student in the database.

We will change the first student’s `StudentGroup` value from `ROSE` to
`DAISY`. The steps to update and persist the change are as follows:

1. Create an entity manager.
2. Read the data from the database and create a local `Student` instance.
3. Modify the `studentGroup` field of the instance to `DAISY`.
4. Get the transaction from the entity manager.
5. Persist the updated instance within a new transaction.

Add the following code to the `JpaUpdateStudent` class:

```java
package org.example;

import org.example.model.StudentGroup;
import org.example.model.Student;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class JpaUpdateStudent {
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
`JpaCreateStudent` and `JpaReadStudent`  classes. We only had to update the `studentGroup`
property of the `student1` instance before persisting it to update the value in
the database.

### Run `JpaUpdateStudent.main`

1. Run the `JpaUpdateStudent.main` method to change the student group for student entity with `id=1`.
2. In IntelliJ, check out the “Run” tab to see the exact query that Hibernate used
   to find the entity.


```text
Hibernate: 
    select
        student0_.id as id1_0_0_,
        student0_.dob as dob2_0_0_,
        student0_.name as name3_0_0_,
        student0_.studentGroup as studentg4_0_0_ 
    from
        STUDENT_DATA student0_ 
    where
        student0_.id=?
Hibernate: 
    update
        STUDENT_DATA 
    set
        dob=?,
        name=?,
        studentGroup=? 
    where
        id=?

```

Run `JpaReadStudent.main` to query the student table to confirm the update:

```text
Student{id=1, name='Jack', dob=2000-01-01, studentGroup=DAISY}
```


## Delete Data

We can delete a record by using the `remove` method on the entity manager. The
deletion steps are similar to the update steps:

1. Create an entity manager.
2. Read the data from the database and create a local `Student` instance.
3. Get the transaction from the entity manager.
4. In the transaction, call the `remove` method on the entity manager and pass
   it the `Student` instance to remove from the database.

Here is what the `JpaDeleteStudent` class should look like:

```java
package org.example;

import org.example.model.Student;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;

public class JpaDeleteStudent {
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


### Run `JpaDeleteStudent.main`

1. Run the `JpaDeleteStudent.main` method to delete the student entity with `id=1`.
2. In IntelliJ, check out the “Run” tab to see the exact query that Hibernate used
   to find the entity.

```text
Hibernate: 
    select
        student0_.id as id1_0_0_,
        student0_.dob as dob2_0_0_,
        student0_.name as name3_0_0_,
        student0_.studentGroup as studentg4_0_0_ 
    from
        STUDENT_DATA student0_ 
    where
        student0_.id=?
Hibernate: 
    delete 
    from
        STUDENT_DATA 
    where
        id=?
```

Run `JpaReadStudent.main` query the student table to confirm the deletion.
The program prints null since the student no longer exists in the database:

```text
null
```

You can also query the table in **pgAdmin** to confirm the row was deleted: 

![JPA deleted student row](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa/deleted_student.png)

## Revert Database Changes

We have made a few changes using the update and delete operation in our
database. We will run the `JpaCreate` class to create the database from scratch
and insert the data:

1. Edit `persistence.xml` to change the `hibernate.hbm2ddl.auto` value to `create`.
2. Run `JpaCreateStudent.main` to recreate the database and populate the `STUDENT_DATA` table.

## Conclusion

We have learned how to create, insert, read, update, and delete data from the
database, all without writing SQL! These are essential operations that you are likely to perform in
pretty much any app that requires data persistence.
