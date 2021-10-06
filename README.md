# Django Relationships Explained in PostgreSQL

Examples of one to one, one to many and many to many relationships in Django models, explained with under the hood table examples created in PostgreSQL.  

- Each example has an example company or customer need that is then solved using a relationship. 
- The aim is to make it easier to understand when and how to use model relationships and when to use on delete cascade or on delete set null.

- All of the examples show you how a Django model would be setup with table equivalents using PostgreSQL.

The examples focus on helping users understand these concepts clearly:

1. When to use a one to one relationship.
2. When to use a one to many relationship.
3. When a many to many relationship could be used.
4. How to setup the above relationships as PostgreSQL tables and Django models.
5. What the difference is between 'ON DELETE CASCADE' and ON DELETE SET NULL and when you might use each.

6. The examples also look at various Django model fields and attributes such as blank, null, unique, cascade and set null.
7. Other Postgres concepts such as truncating a table and transactions using ROLLBACK are discussed.
8. Basic queries are shown in how to access the other side of the model in Django.