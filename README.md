# One-to-Many Relationship

## Learning Goals

- Create one-to-many/many-to-one associations between models with JPA
  convention.
- Modify join column.

## Introduction

We will be modeling the following relationship in this lesson:

![One to many relationship](https://curriculum-content.s3.amazonaws.com/java-spring-1/one-to-many-table.png)

There’s a one-to-many relationship between the student and projects. Each
student can have multiple projects and each project belongs to a specific
student.

## Create Project Entity and Instances

Create the `Project` class in the `models` package and add the following code:

```java
package org.example.models;

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

		@Override
    public String toString() {
        return "Project{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", submissionDate=" + submissionDate +
                ", student=" + student +
                ", score=" + score +
                '}';
    }
}
```

Now, open the `JpaCreate` class, create two new projects, and save them in the
database (`hibernate.hbm2ddl.auto` should be `create`):

```java
package org.example;

import org.example.enums.StudentGroup;
import org.example.models.IdCard;
import org.example.models.Project;
import org.example.models.Student;

import javax.persistence.Persistence;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityManager;
import javax.persistence.EntityTransaction;
import java.time.LocalDate;
import java.util.Date;

public class JpaCreate {
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

        // create id cards
        IdCard card1 = new IdCard();
        card1.setActive(true);

        IdCard card2 = new IdCard();
        card2.setActive(false);

        // create projects
        Project project1 = new Project();
        project1.setTitle("Ant Hill Diorama");
        project1.setSubmissionDate(LocalDate.of(2022, 7, 20));
        project1.setScore(85);

        Project project2 = new Project();
        project2.setTitle("Saturn V Poster Presentation");
        project2.setSubmissionDate(LocalDate.of(2022, 7, 25));
        project2.setScore(90);

        // create student-card associations
        student1.setCard(card1);
        student2.setCard(card2);

        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // access transaction object
        EntityTransaction transaction = entityManager.getTransaction();

        // create and use transactions
        transaction.begin();
        entityManager.persist(student1);
        entityManager.persist(student2);

        entityManager.persist(card1);
        entityManager.persist(card2);

        entityManager.persist(project1);
        entityManager.persist(project2);
        transaction.commit();

        // close entity manager
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

Your directory structure should look like this:

```java
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
    │   │               ├── IdCard.java
    │   │               ├── Project.java
    │   │               └── Student.java
    │   └── resources
    │       └── META-INF
    │           └── persistence.xml
    └── test
        └── java
```

## Modeling the Relationship

In a one-to-many relationship, we want to add the foreign key to the entity that
is on the “many” side (`PROJECTS` in our case). If we were to add a foreign key
on the `STUDENT_DATA` table, there would be multiple `project_id` columns and
updates would require schema modification.

![One to many relationship](https://curriculum-content.s3.amazonaws.com/java-spring-1/one-to-many-table.png)

We have to add annotations in both the `Student` and `Project` classes. First
let’s modify the `Project` class so that the table has the associated student
id. We are using the `@ManyToOne` annotation since multiple projects can belong
to a single student.

```java
package org.example.models;

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
    private Student student;

    // getters, setters, and toString
}
```

Let’s add the association in the `Student` class now:

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

    public void addProjct(Project project) {
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
specific `Student` instance. The `mappedBy` property is telling JPA that the
`projects` field in the `Student` class and the `student` field in the `Project`
class are the same. The `Project` class owns the relationship since each project
has a `student_id` column.

We will now define instance associations in the `JpaCreate` class by adding the
following:

```java
// create student -< projects associations
project1.setStudent(student1);
project2.setStudent(student1);

student1.addProjct(project1);
student1.addProjct(project2);
```

Now if we run the `main` method in the `JpaCreate` class, it will add the data
with the associations to the database.

| ID  | SCORE | SUBMISSIONDATE | TITLE                        | STUDENT_ID |
| --- | ----- | -------------- | ---------------------------- | ---------- |
| 5   | 85    | 2022-07-20     | Ant Hill Diorama             | 1          |
| 6   | 90    | 2022-07-25     | Saturn V Poster Presentation | 1          |

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

If we run the `JpaCreate` class again, the `PROJECT` table will look like this:

| ID  | SCORE | SUBMISSIONDATE | TITLE                        | SUBMITTED_BY |
| --- | ----- | -------------- | ---------------------------- | ------------ |
| 5   | 85    | 2022-07-20     | Ant Hill Diorama             | 1            |
| 6   | 90    | 2022-07-25     | Saturn V Poster Presentation | 1            |

## Fetching Data

For one-to-many relationships, the fetch type is `LAZY` by default since getting
all records at the time of initial query can be an expensive operation. You can
try out the data fetching by using the following `JpaRead` class (remember to
set `hibernate.hbm2ddl.auto` to `none` in the `persistence.xml`):

```java
package org.example;

import org.example.models.Project;
import org.example.models.Student;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;
import java.util.List;

public class JpaRead {
    public static void main(String[] args) {
        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // get records
        Student student1 = entityManager.find(Student.class, 1);
        List<Project> projects = student1.getProjects();
        System.out.println(projects);

        // close entity manager
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

## Code Check

Since we have modified and worked on different files, here are the final
versions of `Student`, `Project`, and `JpaCreate` for reference.

### Student.java

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

    public void addProjct(Project project) {
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

    public Date getDob() {
        return dob;
    }

    public void setDob(Date dob) {
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
package org.example.models;

import javax.persistence.*;
import java.time.LocalDate;

@Entity
public class Project {
    @Id
    @GeneratedValue
    private int id;

    private String title;

    private LocalDate submissionDate;

    @ManyToOne
    private Student student;

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
                ", student=" + student +
                ", score=" + score +
                '}';
    }
}
```

### JpaCreate.java

```java
package org.example;

import org.example.enums.StudentGroup;
import org.example.models.IdCard;
import org.example.models.Project;
import org.example.models.Student;

import javax.persistence.Persistence;
import javax.persistence.EntityManagerFactory;
import javax.persistence.EntityManager;
import javax.persistence.EntityTransaction;
import java.time.LocalDate;
import java.util.Date;

public class JpaCreate {
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

        // create id cards
        IdCard card1 = new IdCard();
        card1.setActive(true);

        IdCard card2 = new IdCard();
        card2.setActive(false);

        // create projects
        Project project1 = new Project();
        project1.setTitle("Ant Hill Diorama");
        project1.setSubmissionDate(LocalDate.of(2022, 7, 20));
        project1.setScore(85);

        Project project2 = new Project();
        project2.setTitle("Saturn V Poster Presentation");
        project2.setSubmissionDate(LocalDate.of(2022, 7, 25));
        project2.setScore(90);

        // create student-card associations
        student1.setCard(card1);
        student2.setCard(card2);

        // create student -< projects associations
        project1.setStudent(student1);
        project2.setStudent(student1);

        student1.addProjct(project1);
        student1.addProjct(project2);

        // create EntityManager
        EntityManagerFactory entityManagerFactory = Persistence.createEntityManagerFactory("example");
        EntityManager entityManager = entityManagerFactory.createEntityManager();

        // access transaction object
        EntityTransaction transaction = entityManager.getTransaction();

        // create and use transactions
        transaction.begin();
        entityManager.persist(student1);
        entityManager.persist(student2);

        entityManager.persist(card1);
        entityManager.persist(card2);

        entityManager.persist(project1);
        entityManager.persist(project2);
        transaction.commit();

        // close entity manager
        entityManager.close();
        entityManagerFactory.close();
    }
}
```

## Conclusion

We have learned how to create one-to-many associations and how to keep the
relationships in sync in the program by creating a list of related entities.
