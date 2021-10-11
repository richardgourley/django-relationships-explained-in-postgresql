## One To Many Relationship with ON DELETE CASCADE
## Postgres Tables vs. Django Models 

### Situation
In this example, we have a company that keeps a database of trainers that they hire to train their staff. 

Each trainer in the database has written many courses that are also saved in the database.  The courses are written by each trainer and are unique to them.

When the company no longer wishes to continue working with the trainer, they want to be able to delete the trainers contact details and all of the courses that the trainer offers, to keep the database of available courses up to date.

### POSTGRES - How would it work in Postgres

- We have two tables, trainer and course.
- Think of 'trainer' as the 'parent' table, the owner and creator of the unique courses.
- Think of the courses as the 'child' table - multiple courses are written and owned by one trainer.



- The courses table can have multiple entries with the same trainer_id.  However a course can't be owned by more than one trainer - hence the ONE to MANY relationship (also called MANY to ONE).

- As we have ON DELETE CASCADE set, when a trainer is no longer used by the company, we also want his or her courses to be deleted from the database.

### The Tables

```
CREATE TABLE trainer(
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    email VARCHAR(100) NOT NULL
);

CREATE TABLE course(
    trainer_id INT NOT NULL,
    course_name VARCHAR(100) NOT NULL,
    duration_in_hours CHAR(2) NOT NULL,
    FOREIGN KEY(trainer_id)
    REFERENCES trainer(id)
    ON DELETE CASCADE
);

```

### Add data
Let's add some data

```
INSERT INTO trainer(first_name, last_name, email)
VALUES
('Bob','Symons','bob@bobthetrainer.com'),
('Tracy','Gibson','tracy@tracythetrainer.com'),
('Sally','Jenkins','sally@sallythetrainer.com');

INSERT INTO course(trainer_id, course_name, duration_in_hours)
VALUES
(1,'Javasript for Beginners','12'),
(1,'Advanced Python', '15'),
(1,'Django in 4 hours','4'),
(2,'Up and Running with Java','10'),
(2,'Java Spring - the Basics',15),
(3,'Laravel for Beginners','25'),
(3,'Learn Basic SQL','10'),
(3,'Learn AWS RDS and Deploy Databases',25);
```

--- Let's find trainers who have a course with the word 'Java' in the title, with an INNER JOIN query to get trainer and course names.
--- Note the LIKE clause is used to find courses where 'Java' is the final word and also 'Java ' is followed by a space to ensure 'Javascript' courses are not returned.

```
SELECT first_name, last_name, email, course_name, duration_in_hou
FROM trainer
INNER JOIN course ON trainer.id = course.trainer_id
WHERE course.course_name LIKE '%Java' OR course.course_name LIKE '%Java %';


first_name | last_name  |           email           |       course_name |duration_in_hours 
------------+-----------+---------------------------+--------------------------+-------------------
 Tracy      | Gibson    | tracy@tracythetrainer.com | Up and Running with Java | 10
 Tracy      | Gibson    | tracy@tracythetrainer.com | Java Spring - the Basics | 15
(2 rows)

```

### Test delete a visitor
Let's see what happens if the trainer 'Tracy Gibson' retires and we delete her from our database...

```
BEGIN;

DELETE FROM trainer
WHERE first_name = 'Tracy'
AND last_name = 'Gibson';
```

```
COMMIT;  --- Or ROLLBACK to undo this action.
```

- Let's perform another query to see if 'Stacy Jackon' has been deleted from the trainer table and to see if all of her courses have been deleted from the course table...

```
SELECT 
    first_name ||' '|| last_name full_name,
    email,
    course_name,
    duration_in_hours
FROM 
    trainer
INNER JOIN course ON trainer.id = course.trainer_id;

```

- Here you should see that 'Tracy Gibson' and her courses are no longer available.

```
   full_name   |           email           |            course_name    | duration_in_hours 
---------------+---------------------------+------------------------------------+---------
 Bob Symons    | bob@bobthetrainer.com     | Javasript for Beginners            | 12
 Bob Symons    | bob@bobthetrainer.com     | Advanced Python                    | 15
 Bob Symons    | bob@bobthetrainer.com     | Django in 4 hours                  | 4 
 Sally Jenkins | sally@sallythetrainer.com | Laravel for Beginners              | 25
 Sally Jenkins | sally@sallythetrainer.com | Learn Basic SQL                    | 10
 Sally Jenkins | sally@sallythetrainer.com | Learn AWS RDS and Deploy Databases | 25
(6 rows)

```


### DJANGO - How it works in Django

- Compare the Django model below with the Postgres tables above.

```
from django.db import models

class Trainer(models.Model):
    first_name = models.CharField(max_length=100, null=False, blank=False)
    last_name = models.CharField(max_length=100, null=False, blank=False)
    email = models.EmailField(null=False, blank=False)    

class Course(models.Model):
    trainer = models.ForeignKey(Trainer, on_delete=models.CASCADE)
    course_name = models.CharField(max_length=100, null=False, blank=False)
    duration_in_hours = models.CharField(max_length=2, null=False, blank=False)

```

- A one to many relationship is created with a 'ForeignKey' field passing in the related object.
- The 'Trainer' class object is passed in to the Foreign Key for One To Many relationships
- The 'Trainer' object is not Unique (unlike OneToOne relationship)
- The 'EmailField' will be validated in django-admin by Django
- As 'on_delete=models.CASCADE' is set on the 'ForeignKey' field of the 'Course' class, it means that when the instance of 'Teacher' related to this course is deleted, THIS course instance will also be deleted.

### Accessing Django objects

- Django makes it easy to access an object, or objects on either side of the relationship...

```
COURSES 

courses = trainer.course_set.all()

count = trainer.course_set.count()

trainer = course.trainer
```

```
TRAINER

trainer = course.trainer
```

--- Query courses with 'Python' in the course name and then loopover the courses and print out the name of the course trainer...

python_courses = Course.objects.filter(course_name__contains="Python")

for course in python_courses:
    print(course.trainer.first_name, course.trainer.last_name, course.trainer.email, course.course_name, course.duration_in_hours)

### SUMMARY:
- MANY courses can belong to ONE trainer, hence the ONE to MANY relationship.
- ONE to MANY are setup using the 'ForeignKey' field in Django.
- In another scenario, it could be that the course could be assigned to a different trainer, if a trainer retires.  However, in this specific user case, ON DELETE CASCADE works because the trainer takes their courses with them when they are deleted from the database.

- One other thing, you can also use the RESTRICT option instead of ON DELETE CASCADE, or SET NULL.  That will RESTRICT any deletions of a 'parent' table happening at all. It depends on the use case in each situation.