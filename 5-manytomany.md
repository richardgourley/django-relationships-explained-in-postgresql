## Many To Many Relationship
## Postgres Tables vs. Django Models 

### Situation
In this example, we have a university where there is a teachers table and a subjects table.
Imagine if this university taught programming and software development.  It's feasible that a teacher could posess enough knowledge to teach a front end language, a back end language, some database skills and some frameworks.

- We have a situation where a teacher can teach MANY subjects.
- The subjects can be taught by MANY teachers.
- So, we have a MANY to MANY relationship.

### POSTGRES - How would it work in Postgres
- Here we have a 'teacher' table and a 'subject' table.
- We also have a 'bridge' table named 'teacher_student', which links teachers to subjects and vice versa.
- Unlike ONE TO ONE and ONE TO MANY relationships, MANY to MANY relationships are usually created with the assistance of a 'bridge' table that has FOREIGN KEYS set on it.
- The bridge table links ids from both tables together.

### The Tables

```
CREATE TABLE teacher(
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE subject(
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

```

- Here is our 'bridge' table that links teachers to subjects and subjects to teachers
- NOTE: The CONSTRAINT name is completely optional - only shown as an example - Postgres will automatically create a name for the Foreign Key constraint if you don't.

```
CREATE TABLE teacher_subject(
    teacher_id INT NOT NULL,
    subject_id INT NOT NULL,
    PRIMARY KEY(teacher_id, subject_id),
    CONSTRAINT teacher_id_fk FOREIGN KEY(teacher_id) REFERENCES teacher(id),
    CONSTRAINT subject_id_fk FOREIGN KEY(subject_id) REFERENCES subject(id)
);
```

- Take a look at the settings of our new table...

```
\d teacher_subject

 Table "public.teacher_subject"
   Column   |  Type   | Collation | Nullable | Default 
------------+---------+-----------+----------+---------
 teacher_id | integer |           | not null | 
 subject_id | integer |           | not null | 
Indexes:
    "teacher_subject_pkey" PRIMARY KEY, btree (teacher_id, subject_id)
Foreign-key constraints:
    "subject_id_fk" FOREIGN KEY (subject_id) REFERENCES subject(id)
    "teacher_id_fk" FOREIGN KEY (teacher_id) REFERENCES teacher(id)
```

### Add data
Let's add some data

```
INSERT INTO teacher(name)
VALUES
('Jane Smith'),
('Paul Jones'),
('Peter Wilson');

INSERT INTO subject(name)
VALUES
('Javascript'),
('Python');
```

- Let's assign teachers to subjects they can teach...

```
INSERT INTO teacher_subject(teacher_id, subject_id)
VALUES
(1,1),
(1,2),
(2,1),
(2,2),
(3,2);
```

- Let's run a join Query to find out which teachers we have available to teach Python...

```
SELECT teacher.name Teacher, subject.name Subject
FROM teacher_subject
INNER JOIN teacher ON teacher.id = teacher_subject.teacher_id
INNER JOIN subject ON subject.id = teacher_subject.subject_id
WHERE subject.name = 'Python'
ORDER BY subject.name;

   teacher    | subject 
--------------+---------
 Jane Smith   | Python
 Paul Jones   | Python
 Peter Wilson | Python
(3 rows)
```

- Here we have another join query to find out what subjects Jane Smith can teach...

```
SELECT subject.name Subject
FROM teacher_subject
INNER JOIN teacher ON teacher.id = teacher_subject.teacher_id
INNER JOIN subject ON subject.id = teacher_subject.subject_id
WHERE teacher.name = 'Jane Smith'
ORDER BY subject.name;

  subject   
------------
 Javascript
 Python
(2 rows)
``` 

### Test Delete a Subject

- What happens to our bridge table if we delete a subject?  Let's delete Python from subjects (inside a transaction, incase we need to undo the action)

```
BEGIN;

DELETE FROM subject
WHERE name = 'Python'
;
```

- You should get this error:
```
ERROR:  update or delete on table "subject" violates foreign key constraint "subject_id_fk" on table "teacher_subject"
DETAIL:  Key (id)=(2) is still referenced from table "teacher_subject".
relationships=# 

--- Enter ROLLBACK to end the transaction
ROLLBACK;
```

### Adding ON DELETE CASCADE to a 'bridge' table

- We received the error above because we need to assign ON DELETE CASCADE to the foreign keys in our 'teacher_student' table in order for teacher -> student relationships to be removed when a teacher or student is deleted...

- Adding constraints to an existing table requires removing the constraint and then re-adding a new one.  It's good to know how to do this but it's better to remember to add any ON DELETE constraints to tables when they are first created!

```
BEGIN;

ALTER TABLE teacher_subject
DROP CONSTRAINT teacher_id_fk;

ALTER TABLE teacher_subject
DROP CONSTRAINT subject_id_fk;

ALTER TABLE teacher_subject
ADD CONSTRAINT teacher_id_fk
    FOREIGN KEY(teacher_id)
    REFERENCES teacher(id)
    ON DELETE CASCADE;

ALTER TABLE teacher_subject
ADD CONSTRAINT subject_id_fk
    FOREIGN KEY(subject_id)
    REFERENCES subject(id)
    ON DELETE CASCADE;

--- check table
\d teacher_subject

---Commit changes if no problems, or rollback (undo)
COMMIT; / ROLLBACK;
```

- Now we can delete 'Python' and see the effect the deletion had on our teacher_student table...

```
DELETE FROM subject
WHERE name = 'Python';

SELECT teacher_id, subject_id 
FROM teacher_subject;

 teacher_id | subject_id 
------------+------------
          1 |          1
          2 |          1
```

- It looks like all of the Python relationships have gone from the table.
- Let's check with a join query, creating an easier to read table...

```
SELECT teacher.name Teacher, subject.name Subject
FROM teacher_subject
INNER JOIN teacher ON teacher.id = teacher_subject.teacher_id
INNER JOIN subject ON subject.id = teacher_subject.subject_id
ORDER BY subject.name;

 teacher   |  subject   
------------+------------
 Jane Smith | Javascript
 Paul Jones | Javascript
(2 rows)

```

### DJANGO - How it works in Django

- Compare the Django model below with the Postgres tables above.

```
class Subject(models.Model):
    name = CharField(max_length=100, null=False, blank=False)

class Teacher(models.Model):
    name = models.CharField(max_length=100, null=False, blank=False)
    subjects = models.ManyToManyField(Subject)
```

- Django specifically has a ManyToMany field.
- As you can see, the 'ManyToMany' field is only used on one model, not both.
- Django will handle the creation of any 'bridge' tables when you run python manage.py makemigrations and migrate
- Note that a 'ManyToMany' field usually has a plural name, eg. subjects

### Accessing Django objects

- Here are some common queries from ManyToMany relationships

- Access the object on the 'other' side of the relationship
```
--- Get a teachers subjects
subject = teacher.subjects.all()

--- Get a subjects teachers (different query as we used ManyToMany field on the 'Subject' model)
teachers = subject.teacher_set.all()
```

- More specific search examples
```
Teacher.objects.filter(subjects__name="Python")
Teacher.objects.filter(subject__name="Javascript")

Subject.objects.filter(teacher__name__startswith="Jane")
Subject.objects.filter(teacher__id=2)
```

### SUMMARY:  
In this scenario, teachers can teach many different subjects.  Subjects can be taught by many different teachers.

- A many to many relationship is usually defined by a bridge table.
- You still need to specify Foreign Keys and decide to add ON DELETE CASCADE or ON DELETE SET NULL to your bridge table constraints.
- Django takes care of database creation when we use a 'ManyToMany' field.