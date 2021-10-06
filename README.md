# Django Relationships Explained in PostgreSQL
A repo of examples of one to one, one to many and many to many relationships for Django, explained with under the hood examples in PostgreSQL.  Makes it easier to understand model relationships and when to use on delete cascade or on delete set null.

The guides here show you how a Django model would be setup as table equivalant using PostgreSQL.

The guides focus on helping users understand as clearly as possbile:

1. When to use a one to one relationship.
2. When to use a one to many relationship.
3. When a many to many relationship could be used.
4. How to setup the above relationships as PostgreSQL tables to test things out first before creating Django models.
5. What the difference is between 'ON DELETE CASCADE' and ON DELETE SET NULL and why it is so important in database design.

6. The guides also look at various Django model fields and attributes such as setting blank, null, unique attributes in Django.
7. Other Postgres concepts such as transactions using ROLLBACK is discussed.