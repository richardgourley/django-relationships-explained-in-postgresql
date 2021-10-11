## One To Many Relationship with ON DELETE SET NULL
## Postgres Tables vs. Django Models 

### Situation
In this example, a property company wants to store details of its' properties and the agents who work for the company in their website database.
The company prefer to assign ONE and only one agent to look after and rent out each property.
The agent can be assigned to many properties.

Here we have a ONE to MANY relationship, also referred to as a MANY to ONE relationship.

- If an agent leaves the company and is deleted from the database, the company don't want to delete any of their properties. 

They prefer to assign a new or different agent to the properties the previous agent was looking after.

### POSTGRES - How would it work in Postgres
- We have two tables, agent and property
- Think of agent as the 'parent' table, which 'owns' properties.
- Think of the property table as the 'child' table, taken care of by agents.

- Unlike a one to one relationship, the agent_id for property shouldn't be unique.  The agent with agent_id of '2' could appear many times in the property table for different properties.
- The agent_id can't be set to NOT NULL (nullable by default) because the Foreign Key is set to ON DELETE SET NULL.
- The agent_id will be set to NULL for a property if an agent is deleted. (Rather than deleting the property).

### The Tables

```
CREATE TABLE agent(
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(50) NOT NULL
);

CREATE TABLE property(
    id SERIAL PRIMARY KEY,
    agent_id INT,
    bedrooms SMALLINT NOT NULL,
    bathrooms SMALLINT NOT NULL,
    address VARCHAR(250) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    FOREIGN KEY(agent_id)
        REFERENCES agent(id)
        ON DELETE SET NULL
);

```

### Add data
Let's add some data

```
INSERT INTO agent(name, email)
VALUES                                        
('Stacy Jackson', 'sjackson@inventedpropertycompany.com');

INSERT INTO property(
agent_id, bedrooms, bathrooms, address, price)
VALUES                                                    
(1,2,2,'10 Main Road','290000');

INSERT INTO property(
agent_id, bedrooms, bathrooms, address, price)
VALUES                                                    
(1,2,1,'27 Hill Road','230000');
```

### Test delete an agent
Let's see what happens if we delete an agent.  You can perform this inside a transaction (BEGIN, followed by query/action followed by COMMIT if you are happy with the query or ROLLBACK if you want to undo the action.)

```
BEGIN;

DELETE FROM agent
WHERE name = 'Stacy Jackson';
```

```
COMMIT;  --- Or ROLLBACK to undo this action.

```

Let's check the tables...

```
SELECT name, email 
FROM agent;
```

```
 id | name | email 
----+------+-------
(0 rows)
```

- Let's see if our properties have been deleted, and if not, what has happened to the 'agent_id' field

```
SELECT id, agent_id, bedrooms, bathrooms, address, price
FROM property;
```

```
 id | agent_id | bedrooms | bathrooms |   address    |   price   
----+----------+----------+-----------+--------------+-----------
  1 |          |        2 |         2 | 10 Main Road | 290000.00
  2 |          |        2 |         1 | 27 Hill Road | 230000.00
(2 rows)
```

### DJANGO - How it works in Django

- Compare the Django model below with the Postgres tables above.

```
class Agent(models.Model):
    name = models.CharField(max_length=100, blank=False, null=False)
    email = models.EmailField(null=False, blank=False)
    
class Property(models.Model):
    agent = models.ForeignKey(Agent, on_delete=models.SET_NULL)
    bedrooms = models.SmallIntegerField(null=False, blank=False)
    bathrooms = models.SmallIntegerField(null=False, blank=False)
    address = models.CharField(max_length=250, null=False, blank=False)
    price = models.DecimalField(max_digits=10, decimal_places=2, null=False, blank=False)
```

- A one to many relationship is created with a 'ForeignKey' field passing in the related object.
- If set to 'on_delete=models.SET_NULL', this object will not be deleted if a 'parent' object is deleted, but the parent object will be set to null.
- Django will take care of the auto increment of property ids.
- Django checks fields with 'blank=False' are not empty strings in the Django admin or on the client side.

### Accessing Django objects

- Django makes it easy to access an object, or objects on either side of the relationship...

```
AGENT

properties = agent.property_set.all()  # appending '_set.all()' to 'agent.property' retrieves all properties being looked after by the agent.

count = agent.property_set.count() 
```

```
PROPERTY

agent = property.agent # will return the agent assigned to this property.
```

### SUMMARY:  
- One agent can look after many properties (One To Many) but in this use case, the property can only have one agent, not many.
- If the agent leaves the company and is deleted from the database, the properties are not deleted from the database, but the 'agent_id' field (Postgres) or the 'Agent' model (Django) are set to NULL.
- Another agent can be assigned to a property after an agent is deleted from the database.