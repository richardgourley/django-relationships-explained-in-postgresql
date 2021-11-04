# Django Relationships Explained in PostgreSQL

**Here are some examples for anyone who isn't 100% sure how or when to:**
1. Use the 'ForeignKey', 'OneToOne' or the 'ManyToMany' fields in their Django models
2. Query the other side of model relationships - do you use 'a.object', 'a.object_set.all()' or 'a.object.all()'?
3. Use 'on_delete=models.SET_NULL' or 'on_delete=models.CASCADE'

- These examples try to demonstrate the equivalent of what is happening behind the scenes in Postgres database tables when you create Django models. 

- Hopefully the examples will help new developers understand how to **confidently** setup objects with different types of relations, without guessing or needing to completely change a model later on. The examples also give examples of querying objects on the **"other side"** of the relationship.

(Apart from a few Postgres specific commands such as /dt or /d table, these examples are relevant to MySQL InnoDB tables.)

## EXAMPLE SCENARIOS

## List of Examples

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


