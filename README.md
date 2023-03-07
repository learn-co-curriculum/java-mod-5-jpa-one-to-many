# One-to-Many Relationship

## Learning Goals

- Create one-to-many/many-to-one associations between entities with JPA convention.
- Pick the entity on the "many" side as the relationship owner.
- Modify a property of a join column.
- Perform eager and lazy fetching of related data from the database.

## Code Along

We will continue with the `Student` and `IdCard` entities from the previous lessons.

## Introduction

Let's update the model to add a `Project` entity to represent a project that
a student works on and submits for scoring.

- Each project has an id, title, submission date, and score. A project is submitted by one student.

We will implement the following one-to-many relationship between `Student` and `Project`
(which can also be modeled as many-to-one between `Project` and `Student`):

![One to many relationship](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-1to1/student_project_erd.png)

- A `Student` may be associated with many `Project` entities, while a `Project` is associated with one `Student`.
- The `Project` entity will be chosen as the relationship owner since it is on the "many" side of the relationship.
  This means a foreign key reference to the associated `Student` entity will be stored in the project database table.
  - `Project` will use the `@ManyToOne` annotation to implement the relationship.
  - `Student` will use the `@OneToMany` annotation with the `mappedBy` attribute to implement the relationship.

## Create Project Entity and Instances

Create the `Project` class in the `model` package and add the following code:

```java
package org.example.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import java.time.LocalDate;

@Entity
public class Project {
    @Id
    @GeneratedValue
    private int id;

    private String title;

    private LocalDate submissionDate;

    private int score;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public LocalDate getSubmissionDate() {
        return submissionDate;
    }

    public void setSubmissionDate(LocalDate submissionDate) {
        this.submissionDate = submissionDate;
    }

    public int getScore() {
        return score;
    }

    public void setScore(int score) {
        this.score = score;
    }
    
}
```

The directory structure should look like this:

![project class added to model](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-1to1/project_structure_projectclass.png)


1. Edit `persistence.xml` to set `hibernate.hbm2ddl.auto` to `create`.
2. Edit the `JpaCreate` class to create three new projects and save them in the
database as shown below.

```java
package org.example;

import org.example.model.IdCard;
import org.example.model.Project;
import org.example.model.Student;
import org.example.model.StudentGroup;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import java.time.LocalDate;

public class JpaCreate {
    public static void main(String[] args) {
        // create a new student instance
        Student student1 = new Student();
        student1.setName("Jack");
        student1.setDob(LocalDate.of(2000,1,1));
        student1.setStudentGroup(StudentGroup.ROSE);

        // create a new student instance
        Student student2 = new Student();
        student2.setName("Lee");
        student2.setDob(LocalDate.of(1999,1,1));
        student2.setStudentGroup(StudentGroup.DAISY);

        // create a new student instance
        Student student3 = new Student();
        student3.setName("Amal");
        student3.setDob(LocalDate.of(1980,1,1));
        student3.setStudentGroup(StudentGroup.LOTUS);


        // create id cards
        IdCard card1 = new IdCard();
        card1.setActive(true);
        IdCard card2 = new IdCard();
        IdCard card3 = new IdCard();

        // create projects
        Project project1 = new Project();
        project1.setTitle("Ant Hill Diorama");
        project1.setSubmissionDate(LocalDate.of(2022, 7, 20));
        project1.setScore(85);

        Project project2 = new Project();
        project2.setTitle("Saturn V Poster Presentation");
        project2.setSubmissionDate(LocalDate.of(2022, 7, 25));
        project2.setScore(90);

        Project project3 = new Project();
        project3.setTitle("Mars Rover Presentation");
        project3.setSubmissionDate(LocalDate.of(2022, 7, 30));
        project3.setScore(87);


        // create student-card associations
        student1.setCard(card1);
        student2.setCard(card2);
        student3.setCard(card3);

        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // access transaction object
        EntityTransaction transaction = entityManager.getTransaction();

        // create and use transactions
        transaction.begin();

        //persist the students
        entityManager.persist(student1);
        entityManager.persist(student2);
        entityManager.persist(student3);

        //persist the id cards
        entityManager.persist(card1);
        entityManager.persist(card2);
        entityManager.persist(card3);

        //persist the projects
        entityManager.persist(project1);
        entityManager.persist(project2);
        entityManager.persist(project3);

        transaction.commit();

        //close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

Use **pgAdmin** to query the new `PROJECT` table:

`SELECT * FROM PROJECT;`


| ID  | SCORE | SUBMISSIONDATE | TITLE                         |
|-----|-------|----------------|-------------------------------|
| 7   | 85    | 2022-07-20     | Ant Hill Diorama              |
| 8   | 90    | 2022-07-25     | Saturn V Poster Presentation  |
| 9   | 87    | 2022-07-30     | Mars Rover Presentation       |

## Modeling the One-To-Many Relationship

A one-to-many relationship between `Student` and `Project` (one student
creates many projects) can also be viewed as a many-to-one relationship
between `Project` and `Student` (many projects are created by one student).

![One to many relationship project owning side](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa/one_to_many_project_owning_side.png)

Here are the steps we have to follow to implement the one-to-many/many-to-one relationship with JPA:

- The entity on the "many" side is the owner of the relationship.
- The table corresponding to the owning side will store the foreign key reference to the non-owning side entity.
- The owning side Java class adds a new field with the annotation `@ManyToOne`.
    - The field stores a single reference to the non-owning side entity.
    - The `@JoinColumn` annotation is optional and can be used the change the foreign key column name in the database.  
- The non-owning side Java class adds a new field with the annotation `@OneToMany`.
    - The field stores a collection such as a set or list of references to entities on the owning side.
    - The `mappedBy` property references the `@ManyToOne` field that was added to the owning side class.

Let's go through the steps to implement the relationship between `Project` and `Student`.


- The `Project` entity on the "many" side is the owner of the relationship.
- The project database table will have a foreign key column to reference the student.
- The `Project` class implements the owning side of the relationship
  using a `@ManyToOne` annotation to reference the one associated `Student` object using a
  new field named `student`.

  ```java
  @ManyToOne
  private Student student;
  ```
  
- The `Student` class implements the referencing or non-owning side
  of the relationship using the `@OneToMany` annotation with a list of associated `Project` objects.
  The `@OneToMany` annotation includes the `mappedBy` attribute mapped
  by the `student` field in the `Project` class.   

  ```java
  @OneToMany(mappedBy = "student")
  private List<Project> projects = new ArrayList<>();
  ```

Edit `Project` to add the `student` field,
along with `getStudent`, `setStudent`, and `toString` methods.
We are using the `@ManyToOne` annotation here as many projects may be
associated with the same student, while a specific project is associated
with only one student. 

```java
package org.example.model;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.ManyToOne;
import java.time.LocalDate;

@Entity
public class Project {
    @Id
    @GeneratedValue
    private int id;

    private String title;

    private LocalDate submissionDate;

    private int score;

    @ManyToOne
    private Student student;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public LocalDate getSubmissionDate() {
        return submissionDate;
    }

    public void setSubmissionDate(LocalDate submissionDate) {
        this.submissionDate = submissionDate;
    }

    public int getScore() {
        return score;
    }

    public void setScore(int score) {
        this.score = score;
    }

    public Student getStudent() {
        return student;
    }

    public void setStudent(Student student) {
        this.student = student;
    }

    @Override
    public String toString() {
        return "Project{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", submissionDate=" + submissionDate +
                ", score=" + score +
                '}';
    }
}
```

Let’s add the association in the `Student` class now.
Edit `Student` to add the `projects` field, along with `addProject` and `getProjects` methods:

```java
package org.example.model;

import javax.persistence.*;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Student {
    @Id
    @GeneratedValue
    private int id;

    private String name;

    private LocalDate dob;

    @Enumerated(EnumType.STRING)
    private StudentGroup studentGroup;

    @OneToOne(fetch = FetchType.LAZY)
    private IdCard card;

    @OneToMany(mappedBy = "student")
    private List<Project> projects = new ArrayList<>();

    public void addProject(Project project) {
        projects.add(project);
        project.setStudent(this);  //set relation on owning side
    }

    public List<Project> getProjects() {
        return projects;
    }

    //other getters, setters, toString
    ...
}
```

We are using the `@OneToMany` annotation here as one student can have many
projects. We use an `ArrayList` to track all the projects that belong to a
specific `Student` instance. 

The `mappedBy` property is telling JPA that the
`projects` field in the `Student` class and the `student` field in the `Project`
class are the same relationship. 

NOTE: Since `Project` is the owner of the relationship, the `addProject()` method
calls `project.setStudent(this)` to ensure the owning side properly established the
relationship.  We can also omit the `addProject()` method from `Student`, forcing the
relationship to always be established through the owning entity `Project`.

We will now define instance associations in the `JpaCreate` class by adding the
following (place after the card associations):

```java
// create student -< projects associations
project1.setStudent(student1);
project2.setStudent(student1);
project3.setStudent(student2);
```

Now if we run the `JpaCreate.main` method, it will add the data
with the associations to the database.    

Query the `PROJECT` table in the database.
The foreign key column in the database table is assigned a default name of  `STUDENT_ID`,
which is composed using the field name  `student` defined by the `@ManyToOne` annotation
in the `Project` class, concatenated with the primary key `id` of the `Student` class.


| ID   | SCORE  | SUBMISSIONDATE  | TITLE                        | STUDENT_ID   |
|------|--------|-----------------|------------------------------|--------------|
| 7    | 85     | 2022-07-20      | Ant Hill Diorama             | 1            |
| 8    | 90     | 2022-07-25      | Saturn V Poster Presentation | 1            |
| 9    | 87     | 2022-07-30      | Mars Rover Presentation      | 2            |

Each project row can only store one integer value in the `STUDENT_ID` column,
thus a project is associated with exactly one student.
Notice however multiple project rows may
store the same student id, allowing a student to be associated with multiple projects.

## Join Column

We can use the `@JoinColumn` annotation to modify the foreign key column
from the default name `student_id` to `submitted_by`.  

Edit the `Project` class to add the `@JoinColumn` annotation as shown:

```java
@ManyToOne
@JoinColumn(name = "submitted_by")
private Student student;
```

If we run the `JpaCreate.main` method again, the foreign key column in the `PROJECT` table
is now named `SUBMITTED_BY`:

| ID   | SCORE  | SUBMISSIONDATE  | TITLE                        | SUBMITTED_BY |
|------|--------|-----------------|------------------------------|--------------|
| 7    | 85     | 2022-07-20      | Ant Hill Diorama             | 1            |
| 8    | 90     | 2022-07-25      | Saturn V Poster Presentation | 1            |
| 9    | 87     | 2022-07-30      | Mars Rover Presentation      | 2            |


## Fetching Data

For one-to-many relationships, the fetch type is `LAZY` by default since getting
all records at the time of initial query can be an expensive operation. 

1. Set `hibernate.hbm2ddl.auto` to `none` in the `persistence.xml`
2. Edit the `JpaRead` class to get and print the student's projects as shown:

```java
package org.example;

import org.example.model.IdCard;
import org.example.model.Project;
import org.example.model.Student;
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;
import java.util.List;

public class JpaRead {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // get student data using primary key id=1
        Student student1 = entityManager.find(Student.class, 1);
        System.out.println(student1);
        
        // get the student's id card
        IdCard card1 = student1.getCard();
        System.out.println(card1);
        
        // get the student's projects
        List<Project> projects = student1.getProjects();
        System.out.println(projects);

        // close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

The Hibernate and print statement output should be similar to this: 

```text
Hibernate: 
    select
        student0_.id as id1_2_0_,
        student0_.card_id as card_id5_2_0_,
        student0_.dob as dob2_2_0_,
        student0_.name as name3_2_0_,
        student0_.studentGroup as studentg4_2_0_ 
    from
        Student student0_ 
    where
        student0_.id=?
Student{id=1, name='Jack', dob=2000-01-01, studentGroup=ROSE}
Hibernate: 
    select
        idcard0_.id as id1_0_0_,
        idcard0_.isActive as isactive2_0_0_,
        student1_.id as id1_2_1_,
        student1_.card_id as card_id5_2_1_,
        student1_.dob as dob2_2_1_,
        student1_.name as name3_2_1_,
        student1_.studentGroup as studentg4_2_1_ 
    from
        ID_CARD idcard0_ 
    left outer join
        Student student1_ 
            on idcard0_.id=student1_.card_id 
    where
        idcard0_.id=?
IdCard{id=4, isActive=true}
Hibernate: 
    select
        projects0_.submitted_by as submitte5_1_0_,
        projects0_.id as id1_1_0_,
        projects0_.id as id1_1_1_,
        projects0_.score as score2_1_1_,
        projects0_.submitted_by as submitte5_1_1_,
        projects0_.submissionDate as submissi3_1_1_,
        projects0_.title as title4_1_1_ 
    from
        Project projects0_ 
    where
        projects0_.submitted_by=?
[Project{id=7, title='Ant Hill Diorama', submissionDate=2022-07-20, score=85}, Project{id=8, title='Saturn V Poster Presentation', submissionDate=2022-07-25, score=90}]
```

## Code Check

Since we modified and worked on different files, here are the final
versions of `Student`, `Project`, `JpaCreate`, and `JpaRead` for reference.

### Student.java

```java
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Student {
    @Id
    @GeneratedValue
    private int id;

    private String name;

    private LocalDate dob;

    @Enumerated(EnumType.STRING)
    private StudentGroup studentGroup;

    @OneToOne(fetch = FetchType.LAZY)
    private IdCard card;

    @OneToMany(mappedBy = "student")
    private List<Project> projects = new ArrayList<>();

    public void addProject(Project project) {
        projects.add(project);
        project.setStudent(this);
    }

    public List<Project> getProjects() {
        return projects;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public LocalDate getDob() {
        return dob;
    }

    public void setDob(LocalDate dob) {
        this.dob = dob;
    }

    public StudentGroup getStudentGroup() {
        return studentGroup;
    }

    public void setStudentGroup(StudentGroup studentGroup) {
        this.studentGroup = studentGroup;
    }

    public IdCard getCard() {
        return card;
    }

    public void setCard(IdCard card) {
        this.card = card;
    }

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

### Project.java

```java
package org.example.model;

import javax.persistence.*;
import java.time.LocalDate;

@Entity
public class Project {
    @Id
    @GeneratedValue
    private int id;

    private String title;

    private LocalDate submissionDate;

    private int score;

    @ManyToOne
    @JoinColumn(name = "submitted_by")
    private Student student;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public LocalDate getSubmissionDate() {
        return submissionDate;
    }

    public void setSubmissionDate(LocalDate submissionDate) {
        this.submissionDate = submissionDate;
    }

    public int getScore() {
        return score;
    }

    public void setScore(int score) {
        this.score = score;
    }

    public Student getStudent() {
        return student;
    }

    public void setStudent(Student student) {
        this.student = student;
    }

    @Override
    public String toString() {
        return "Project{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", submissionDate=" + submissionDate +
                ", score=" + score
                '}';
    }
}
```

### JpaCreate.java

```java
package org.example;

import org.example.model.IdCard;
import org.example.model.Project;
import org.example.model.Student;
import org.example.model.StudentGroup;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityTransaction;
import javax.persistence.Persistence;
import java.time.LocalDate;

public class JpaCreate {
    public static void main(String[] args) {
        // create a new student instance
        Student student1 = new Student();
        student1.setName("Jack");
        student1.setDob(LocalDate.of(2000,1,1));
        student1.setStudentGroup(StudentGroup.ROSE);

        // create a new student instance
        Student student2 = new Student();
        student2.setName("Lee");
        student2.setDob(LocalDate.of(1999,1,1));
        student2.setStudentGroup(StudentGroup.DAISY);

        // create a new student instance
        Student student3 = new Student();
        student3.setName("Amal");
        student3.setDob(LocalDate.of(1980,1,1));
        student3.setStudentGroup(StudentGroup.LOTUS);


        // create id cards
        IdCard card1 = new IdCard();
        card1.setActive(true);
        IdCard card2 = new IdCard();
        IdCard card3 = new IdCard();

        // create projects
        Project project1 = new Project();
        project1.setTitle("Ant Hill Diorama");
        project1.setSubmissionDate(LocalDate.of(2022, 7, 20));
        project1.setScore(85);

        Project project2 = new Project();
        project2.setTitle("Saturn V Poster Presentation");
        project2.setSubmissionDate(LocalDate.of(2022, 7, 25));
        project2.setScore(90);

        Project project3 = new Project();
        project3.setTitle("Mars Rover Presentation");
        project3.setSubmissionDate(LocalDate.of(2022, 7, 30));
        project3.setScore(87);


        // create student-card associations
        student1.setCard(card1);
        student2.setCard(card2);
        student3.setCard(card3);

        // create student -< projects associations
        project1.setStudent(student1);
        project2.setStudent(student1);
        project3.setStudent(student2);

        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // access transaction object
        EntityTransaction transaction = entityManager.getTransaction();

        // create and use transactions
        transaction.begin();

        //persist the students
        entityManager.persist(student1);
        entityManager.persist(student2);
        entityManager.persist(student3);

        //persist the id cards
        entityManager.persist(card1);
        entityManager.persist(card2);
        entityManager.persist(card3);

        //persist the projects
        entityManager.persist(project1);
        entityManager.persist(project2);
        entityManager.persist(project3);

        transaction.commit();

        //close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

### JpaRead.java

```java
package org.example;

import org.example.model.IdCard;
import org.example.model.Project;
import org.example.model.Student;
import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;
import java.util.List;

public class JpaRead {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // get student data using primary key id=1
        Student student1 = entityManager.find(Student.class, 1);
        System.out.println(student1);
        // get the student's id card
        IdCard card1 = student1.getCard();
        System.out.println(card1);
        // get the student's projects
        List<Project> projects = student1.getProjects();
        System.out.println(projects);

        // close entity manager and factory
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

## Conclusion

We have learned how to create one-to-many associations and how to keep the
relationships in sync in the program by creating a list of related entities.


To implement a one-to-many/many-to-one relationship:

- The entity on the "many" side of the relationship is the owner.
- The table corresponding to the owning side will store the foreign key reference.
- The owning side Java class adds a new field with the annotation `@ManyToOne`.
   - The field stores a single reference to non-owning side entity.
- The non-owning side Java class adds a new field with the annotation `@OneToMany`.
   - The field stores a collection of references to owning side entities.
   - The `mappedBy` property references the `@ManyToOne` field that was added to the owning side class.


![One to many relationship project owning side](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa/one_to_many_project_owning_side.png)

A project is associated with exactly one student, thus the `Project` class stores
the relationship as:

```java
public class Project {
    ...
    
    @ManyToOne
    @JoinColumn(name = "submitted_by")
    private Student student;
    
    ...
}
```


A student may be associated with many projects, thus the `Student` class stores
the relationship as:

```java
public class Student {
    ...

    @OneToMany(mappedBy = "student")
    private List<Project> projects = new ArrayList<>();

    ...
}
```

The `mappedBy` property allows the project list to be generated based on the presence
of the student's id in the project table `submitted_by` column.