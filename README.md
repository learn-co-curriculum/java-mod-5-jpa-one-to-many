# One-to-Many Relationship

## Learning Goals

- Create one-to-many/many-to-one associations between entities with JPA convention.
- Modify a property of a join column.

## Introduction

We will update the model to add an entity to represent a project that
a student works on and submits for scoring. There’s a one-to-many
relationship between the student and projects.
Each student can have multiple projects and each project belongs to a specific
student.

![One to many relationship](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-1to1/student_project_erd.png)

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
2. Edit the `JpaCreate` class to create three new projects  and save them in the
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

## Modeling the Relationship

In a one-to-many relationship, we want to add the foreign key to the entity that
is on the “many” side (`PROJECTS` in our case). If we were to add a foreign key
on the `STUDENT_DATA` table, there would be multiple `project_id` columns and
updates would require schema modification.

![One to many relationship](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-jpa-1to1/student_project_erd.png)

We have to add annotations in both the `Student` and `Project` classes. First
let’s modify the `Project` class so that the table has the associated student
id. We are using the `@ManyToOne` annotation since multiple projects can belong
to a single student.

Initially we name the field `student`, which will cause
the column in the `PROJECT` table to be named `student_id`.  Eventually we will
set the property on the join column to name it `submitted_by`, thus matching the
entity relationship model.

Edit `Project` to add the `student` field,
along with `getStudent`, `setStudent`, and `toString` methods.
We are using the `@ManyToOne` annotation here as many different projects may be
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
                ", student=" + student +
                '}';
    }
}
```

Let’s add the association in the `Student` class now.
Edit `Student` to add the `projects` field, along with getter and setter methods:

```java
package org.example.models;

import org.example.enums.StudentGroup;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;

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

    @OneToOne(fetch = FetchType.LAZY)
    private IdCard card;

    @OneToMany(mappedBy = "student")
    private List<Project> projects = new ArrayList<>();

    public void addProject(Project project) {
        projects.add(project);
    }

    public List<Project> getProjects() {
        return projects;
    }

    // other getters, setters, and toString
}
```

We are using the `@OneToMany` annotation here as one student can have many
projects and using an `ArrayList` to track all the projects that belong to a
specific `Student` instance. 

The `mappedBy` property is telling JPA that the
`projects` field in the `Student` class and the `student` field in the `Project`
class are the same. The `Project` class owns the relationship since each project
has a `student_id` column, thus the relationship will be stored in the `PROJECT`
table in the database.

We will now define instance associations in the `JpaCreate` class by adding the
following (place after the card associations):

```java
// create student -< projects associations
project1.setStudent(student1);
project2.setStudent(student1);
project3.setStudent(student2);

student1.addProject(project1);
student1.addProject(project2);
student2.addProject(project3);
```

Now if we run the `JpaCreate.main` method, it will add the data
with the associations to the database.

| ID   | SCORE  | SUBMISSIONDATE  | TITLE                        | STUDENT_ID   |
|------|--------|-----------------|------------------------------|--------------|
| 7    | 85     | 2022-07-20      | Ant Hill Diorama             | 1            |
| 8    | 90     | 2022-07-25      | Saturn V Poster Presentation | 1            |
| 9    | 87     | 2022-07-30      | Mars Rover Presentation      | 2            |

Each project row can only store one integer value in the `STUDENT_ID` column,
thus a project is associated with exactly one student.
Notice however multiple project rows may
store the same student id, thus a student may be associated with multiple projects.

### Modifying Join Column

The `STUDENT_ID` column in the `PROJECT` table is called a join column since it
is referring to a relationship in another table. We can use the `@JoinColumn`
annotation to modify the column similar to the `@Column` annotation. For
example, we can modify the column name by adding the following annotation and
property to the `Student` class:

```java
@ManyToOne
@JoinColumn(name = "SUBMITTED_BY")
private Student student;
```

If we run the `JpaCreate.main` method again, the `PROJECT` table will look like this:

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
        STUDENT_DATA student0_ 
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
        STUDENT_DATA student1_ 
            on idcard0_.id=student1_.card_id 
    where
        idcard0_.id=?
IdCard{id=4, isActive=true}
Hibernate: 
    select
        projects0_.SUBMITTED_BY as submitte5_1_0_,
        projects0_.id as id1_1_0_,
        projects0_.id as id1_1_1_,
        projects0_.score as score2_1_1_,
        projects0_.SUBMITTED_BY as submitte5_1_1_,
        projects0_.submissionDate as submissi3_1_1_,
        projects0_.title as title4_1_1_ 
    from
        Project projects0_ 
    where
        projects0_.SUBMITTED_BY=?
[Project{id=7, title='Ant Hill Diorama', submissionDate=2022-07-20, score=85, student=Student{id=1, name='Jack', dob=2000-01-01, studentGroup=ROSE}}, Project{id=8, title='Saturn V Poster Presentation', submissionDate=2022-07-25, score=90, student=Student{id=1, name='Jack', dob=2000-01-01, studentGroup=ROSE}}]
```

## Code Check

Since we have modified and worked on different files, here are the final
versions of `Student`, `Project`, `JpaCreate`, and `JpaRead` for reference.

### Student.java

```java
package org.example.model;

import javax.persistence.*;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "STUDENT_DATA")
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
    @JoinColumn(name = "SUBMITTED_BY")
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
                ", student=" + student +
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

        student1.addProject(project1);
        student1.addProject(project2);
        student2.addProject(project3);

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

A project is associated with exactly one student, thus the `Project` class stores
the relationship as:

```java
public class Project {
    ...
    
    @ManyToOne
    @JoinColumn(name = "SUBMITTED_BY")
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