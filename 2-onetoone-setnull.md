## One To One Relationship with ON DELETE SET NULL
## Postgres Tables vs. Django Models 

### Situation
- In this example, a company has customers.  The customers are assigned a special offer code that is used by other tables and parts of the business.
- In this case, the company wants the special offer code to be completely unique.  They also want only one special offer code per customer.
- Also, when a customer is deleted, they don't want to delete the special offer code from the database.  They want to keep all previously used unique special offer codes in the database because they never want to re-issue a special offer code to any new customers.

### POSTGRES - How would it work in Postgres
- The customer table is the 'parent' table.
- The special_offer_code table is the 'child' table - 'owned' by a customer.
- In the 'special_offer_code' table, the customer_id field is set to UNIQUE to ensure a ONE to ONE relationship.
- The code field in the 'special_offer_code' table is also set to UNIQUE to ensure it is never repeated.
- The customer_id field is also nullable, and the foreign key is set to ON DELETE SET NULL - so if a customer is deleted, the row remains in the special_offer_code table with the customer_id set to NULL.

### The Tables

- NOTE: The CONSTRAINT name is optional. Postgres will auto create a CONSTRAINT name for you if you don't specify one.

```
CREATE TABLE customer(
    id SERIAL PRIMARY KEY,
    customer_name VARCHAR(100) NOT NULL UNIQUE,
    customer_address VARCHAR(250) NOT NULL
);

CREATE TABLE special_offer_code(
    customer_id INT UNIQUE, 
    code CHAR(10) NOT NULL UNIQUE,
    CONSTRAINT customer_id_fk
        FOREIGN KEY(customer_id)
        REFERENCES customer(id)
        ON DELETE SET NULL
);
```

Let's look at the special_offer_code table...

```
\d special_offer_code;
```

```
Table "public.special_offer_code"
   Column    |     Type      | Collation | Nullable | Default 
-------------+---------------+-----------+----------+---------
 customer_id | integer       |           |          | 
 code        | character(10) |           | not null | 
Indexes:
    "special_offer_code_code_key" UNIQUE CONSTRAINT, btree (code)
    "special_offer_code_customer_id_key" UNIQUE CONSTRAINT, btree (customer_id)
Foreign-key constraints:
    "customer_id_fk" FOREIGN KEY (customer_id) REFERENCES customer(id) ON DELETE SET NULL

```

### Add data
Let's add some data

```
INSERT INTO customer(customer_name, customer_address)
VALUES
('Corby Spa', '10 Corby Road'),
('Jones Butchers','26 Corby Road');

INSERT INTO special_offer_code(customer_id, code)
VALUES
(1,'CORBSP5769'),
(2,'JONEBU7856');
```

### Test the UNIQUE constraint 
- Let's try and add another code for customer with id '1'

```
INSERT INTO special_offer_code(customer_id, code)
VALUES
(1,'CORBSP3792');

ERROR:  duplicate key value violates unique constraint "special_offer_code_customer_id_key"
DETAIL:  Key (customer_id)=(1) already exists.
```

- Let's create a new customer and try to assign an existing special offer code...

```
INSERT INTO customer(customer_name, customer_address)
VALUES 
('Corby Spanish Classes', '15 Corby Road')
RETURNING id;
```

```
id 
----
  3
```

```
INSERT INTO special_offer_code(customer_id, code)
VALUES
(3, 'CORBSP5769');

ERROR:  duplicate key value violates unique constraint "special_offer_code_code_key"
DETAIL:  Key (code)=(CORBSP5769) already exists.
```

### Test Delete a Site User

- Let's see what happens in the special_offer_code table if we delete a user...
- Let's do this inside a transaction starting with the BEGIN command, followed by COMMIT if the changes are correct, or ROLLBACK to undo the changes...

```
BEGIN;

DELETE FROM customer
WHERE id = 1;
```

```
COMMIT; ---Here you can enter ROLLBACK instead if you make an error and need to undo changes!
```

- Check the changes...

```
SELECT id, customer_name, customer_address
FROM customer;
```

```
 id |     customer_name     | customer_address 
----+-----------------------+------------------
  2 | Jones Butchers        | 26 Corby Road
  3 | Corby Spanish Classes | 15 Corby Road
```

```
SELECT customer_id, code 
FROM special_offer_code;
```

```
 customer_id |    code    
-------------+------------
           2 | JONEBU7856
             | CORBSP5769
```

- We can see above that when we delete a customer, the customer_id is set to NULL in the special_offer_code table.

### DJANGO - How it works in Django

- Compare the Django models below with the Postgres tables above.

```
from django.db import models

class Customer(models.Model):
    customer_name = models.CharField(
        max_length=100,
        null=False,
        unique=True,
        blank=False
    )
    customer_address = models.TextField(
        max_length=250,
        null=False,
        blank=False
    )

class SpecialOfferCode(models.Model):
    # unique=True is automatically set when Django creates a OneToOneField in the database

    customer = models.OneToOneField(
        Customer,
        null=True,
        on_delete=models.SET_NULL
    )
    code = models.CharField(
        max_length=10,
        null=False,
        blank=False,
        unique=True
    )
```

- Django automatically creates an incremented primary key for the 'Customer' model.
- The 'OneToOne' field takes care of making sure the object 'Customer' is unique in the table.
- null=True is set in the 'OneToOne' field allowing the 'Customer' to be NULL if the customer is deleted.

### Accessing Django objects

- Accessing the opposite side of a relationship is easy when using a 'OneToOne' field.

```
customer = code.customer

code = customer.code # may raise Code.DoesNotExist error
```

- Exception handling using the ObjectDoesNotExist exception...

```
try:
    code = customer.code
except ObjectDoesNotExist:
    print("No special offer code has been created.")
```

- You could use hasattr() instead to handle a customer who hasn't been assigned a special offer code.

```
hasattr(customer, 'code')
```

### SUMMARY:  

- In this example of a ONE to ONE relationship with ON DELETE SET NULL, the customer can have only ONE special offer code but when the customer is deleted, the unique special offer code remains in the database.

- In situations where a 'parent' table row is deleted but you want to keep the associated row from the child table in the database, use ON DELETE SET NULL.
