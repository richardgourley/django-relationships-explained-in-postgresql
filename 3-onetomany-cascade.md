## One To Many Relationship with ON DELETE CASCADE
## Postgres Tables vs. Django Models 

### Situation
- A prestigious but top secret company has hundreds of visitors coming to their offices everyday.  Due to the secrecy around the company, when visitors arrive, they must place all electronic devices such as phones and laptops in a storage room.
- The company uses a database table to record general visitor information, and a separate table to record all of the visitors' laptops, phones and smartwatches to ensure they are returned to the visitor.
- The company hates bad publicity so they make sure all electronic devices are carefully returned to the visitor when they leave, with no mix ups.
- Due to the huge number of visitors they recieve, the company wants to quickly delete a visitor when the visitor signs out and after returning all electronic devices, delete the electronic devices from the database at the same time the visitor is deleted from the database.

In this case, we have MANY electronic devices that are returned to ONE and only one visitor.

### POSTGRES - How would it work in Postgres

- We have two tables, visitor and electronic_devices
- Think of visitor as the 'parent' table, the owner of electronic devices.
- Think of the electronic devices as the 'child' table, multiple devices 'owned' by a visitor.

- The electronic_devices table can have multiple entries with the same visitor_id.  However an electronic device can't be owned by more than one visitor - hence the ONE to MANY relationship.

- As we have ON DELETE CASCADE set, when a visitor leaves and is deleted, all of that visitors' devices will be deleted too.

### The Tables

```
CREATE TABLE visitor(
	id SERIAL PRIMARY KEY,
	first_name VARCHAR(100) NOT NULL,
	last_name VARCHAR(100) NOT NULL,
	time_arrived TIMESTAMP NOT NULL
);

CREATE TABLE electronic_devices(
	visitor_id INT NOT NULL,
	device_name VARCHAR(100) NOT NULL,
	color VARCHAR(100) NOT NULL,
	description TEXT NOT NULL,
	FOREIGN KEY(visitor_id)
	REFERENCES visitor(id)
	ON DELETE CASCADE
);
```

### Add data
Let's add some data

```
INSERT INTO visitor(first_name, last_name, time_arrived)
VALUES
('Stacy', 'Jackson', NOW()),
('Steve', 'Jones', NOW()),
('Kelly', 'Crawford', NOW());

INSERT INTO electronic_devices(visitor_id, device_name, color)
VALUES
(1,'Samsung Phone', 'Silver', 'Brand new.'),
(1,'Smartwach', 'Light Blue', 'Slightly scratched screen.'),
(2,'HP laptop', 'Black', 'Django logo on the back.'),
(3,'Apple laptop', 'Silver', 'Scratches on the lid.');
```

- This is how the tables will look with an inner join to get the name and devices of each visitor...
```
SELECT first_name, last_name, device_name, color, description
FROM visitor
INNER JOIN electronic_devices 
ON visitor.id = electronic_devices.visitor_id;

 first_name | last_name |  device_name  |   color    |        description         
------------+-----------+---------------+------------+----------------------------
 Stacy      | Jackson   | Samsung Phone | Silver     | Brand new.
 Stacy      | Jackson   | Smartwach     | Light Blue | Slightly scratched screen.
 Steve      | Jones     | HP laptop     | Black      | Django logo on the back.
 Kelly      | Crawford  | Apple laptop  | Silver     | Scratches on the lid.
(4 rows)

```

### Test delete a visitor
Let's see what happens if we delete the visitor 'Stacy Jackson' after she leaves and we return her electronic devices. (Query performed inside a transaction, see examples 1 and 2)

```
BEGIN;

DELETE FROM visitor
WHERE first_name = 'Stacy'
AND last_name = 'Jackson';

COMMIT;  --- Or ROLLBACK to undo this action.

```

- Let's see if the cascade works and if our electronic_devices table has updated...

```
SELECT visitor_id, device_name, color, description
FROM electronic_devices;

 visitor_id | device_name  | color  |       description        
------------+--------------+--------+--------------------------
          2 | HP laptop    | Black  | Django logo on the back.
          3 | Apple laptop | Silver | Scratches on the lid.
```

- Looks good but let's check run the inner join query again just to check 'Stacy Jackson' is no longer in the database...

```
 first_name | last_name | device_name  | color  |       description        
------------+-----------+--------------+--------+--------------------------
 Steve      | Jones     | HP laptop    | Black  | Django logo on the back.
 Kelly      | Crawford  | Apple laptop | Silver | Scratches on the lid.

```

### DJANGO - How it works in Django

- Compare the Django model below with the Postgres tables above.

```
from django.utils import timezone

class Visitor(models.Model):
	first_name = models.CharField(max_length=100, null=False, blank=False),
	last_name = models.CharField(max_length=100, null=False, blank=False),
	time_arrived = models.DateTimeField(default=timezone.now)

class ElectronicDevice(models.Model):
	visitor = models.ForeignKey(Visitor, on_delete=models.CASCADE)
	device_name = models.CharField(max_length=100, null=False, blank=False)
	color = models.CharField(max_length=100, null=False, blank=False)
	description = models.TextField()

```

- A one to many relationship is created with a 'ForeignKey' field passing in the related object.
- 'on_delete=models.CASCADE' in the visitor field of ElectronicDevice will result in an instance of ElectronicDevice being deleted when the instance of Visitor is deleted.
- Django will take care of the auto increment of visitor ids.
- Django checks fields with 'blank=False' are not an empty string in the Django admin or on the client side.

### Accessing Django objects

- Django makes it easy to access an object, or objects on either side of the relationship...

```
ELECTRONIC DEVICES 

electronic_devices = visitor.electronic_device_set.all() 

count = visitor.electronic_device_set.count() 
```

```
VISITOR

visitor = electronic_device.visitor # will return the visitor who owns this electronic device.
```

### SUMMARY:
When a visitor arrives, all electronic devices are recorded.  The visitor can have MANY devices.  The electronic device can only have ONE owner, the visitor.
When the visitor leaves, they are deleted from the database and all of their electronic devices are also deleted.  