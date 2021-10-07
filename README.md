# Django Relationships Explained in PostgreSQL

A repo of use case examples of one to one, one to many and many to many relationships explained using both Postgres tables and Django models in each example.

The examples also aim to explain when to use on delete cascade or on delete set null.

The examples try to clarify what is happening in Postgres tables under the hood, when Django models are created with simple clear example scenarios.

EXAMPLE
- Each example answers these questions...
How are the tables and relationships created in Postgres tables?
How are the objects and their relationships created with Django models?
What happens if a referencing object or row is deleted?

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

