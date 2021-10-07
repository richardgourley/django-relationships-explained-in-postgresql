# Django Relationships Explained in PostgreSQL

This repo is for anyone who isn't 100% sure how or when to:
1. Use the 'ForeignKey', 'OneToOne' or the 'ManyToMany' fields in their Django models
2. Query the other side of model relationships - do you use 'a.object', 'a.object_set.all()' or 'a.object.all()'
3. Use 'on_delete=models.SET_NULL' or 'on_delete=models.CASCADE'

- One thing that took me a while to completely understand in Django was how to consistently and correctly setup objects with different types of relations, and know how to query the objects.

- These examples try to demonstrate the equivalent of what is happening behind the scenes in Postgres database tables when you create Django models. 
(Apart from a few Postgres specific commands, most of these example tables will work with MySQL too.)

## EXAMPLE SCENARIOS

### Each example tries to answer these questions...
How are the tables and relationships created in Postgres tables?
What happens if parent table rows are deleted?
How are the objects and their relationships created with Django models?
What happens if a referencing object is deleted?
How do I correctly query and retrieve the object or objects on the other side of the relationship?  Do I use '.object', '.object_set.all()', '.object.all()'?

## List of Examples

1. OneToOne relationship with ON DELETE CASCADE.
- An example of a user and user profile as used in many applications.

2. OneToOne relationship with ON DELETE SET NULL
- An example of a customer and a unique special offer code.

3. OneToMany relationship with ON DELETE CASCADE
- An example of a company which receives numerous visitors, who are obliged to store their electronic devices away when visiting the company.

4. OneToMany relationship with ON DELETE SET NULL
- An example of a property and agent relationship for a property rental company.

5. Many To Many relationship example.
- An example of a university with subjects taught by multiple teachers.


