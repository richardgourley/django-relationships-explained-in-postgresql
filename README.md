# Django Relationships Explained in PostgreSQL

## INTRO
**Examples to help developers understand how to:**
1. Use the 'ForeignKey', 'OneToOne' or the 'ManyToMany' fields in their Django models
2. Query the other side of model relationships - do you use 'a.object', 'a.object_set.all()' or 'a.object.all()'?
3. Use 'on_delete=models.SET_NULL' or 'on_delete=models.CASCADE'

- These examples demonstrate the equivalent of what is happening behind the scenes in Postgresql database tables when you create Django models. 

- The aim is to help developers understand how to **confidently** setup objects with different types of relations, without having to completely change a model later on. 

- Examples of querying objects on the **"other side"** of the relationship are included.

(Apart from a few Postgres specific commands such as /dt or /d table, these examples are relevant to MySQL InnoDB tables, or any other relational databases utilizing foreign key constraints.)

## FEATURED EXAMPLES

1. **OneToOne relationship with ON DELETE CASCADE.**
- An example of a user and user profile as used in many applications.

2. **OneToOne relationship with ON DELETE SET NULL**
- An example of a customer table and a unique special offer code table. 

3. **OneToMany relationship with ON DELETE CASCADE**
- An example of a company which has a database of trainers and consultants, who each are the owners and creators of multiple training events that can be used to train team members.

4. **OneToMany relationship with ON DELETE SET NULL**
- An example of a property and agent relationship for a property rental company.

5. **Many To Many relationship example.**
- An example of a university with multiple subjects taught by multiple teachers.

## TOOLS

- Python Django
- PostgreSQL database

## GETTING STARTED

New to Django?
- Get started with Django...
https://docs.djangoproject.com

New to PostgreSQL?
- Get started with installing PostgreSQL on Linux, Windows or Mac
https://www.postgresqltutorial.com/

## SKILLS COVERED

DJANGO
- Django models.Model class
- Django User model

- models.OneToOneField
- models.ForeignKey
- models.CharField
- models.TextField
- models.EmailField
- models.SmallIntegerField
- models.DecimalField
- models.ManyToManyField
- max_length
- on_delete=models.CASCADE
- null=False, blank=False

- Exception Handling - ObjectDoesNotExist

- Obj.objects.get()
- Obj.object.filter()
- object.relatedobjects.all()
- object.relatedobject_set.all()
- object.relatedobject_set.count()
- hasattr()





