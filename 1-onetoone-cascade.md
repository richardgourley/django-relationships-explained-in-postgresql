## One To One Relationship with ON DELETE CASCADE
## Postgres Tables vs. Django Models 

### Situation
An application or web application often has users.  Let's imagine we are creating a user model in Django or a user table in Postgres and a profile model or postgres table called profile.

- We want to be able to create a user who can then create a profile.
- We only want to have one profile per user (and one user associated with a profile).
- For this we will create a one to one relationship.
- What happens when a user is deleted?  In this scenario, we want to delete the profile from the database associated with a user when the user is deleted.
- For this we will use ON DELETE CASCADE

### POSTGRES - How would it work in Postgres
- We have two tables - site_user and profile.

- Think of the site_user table as the 'parent' table that 'owns' a profile.
- Think of the profile table as the 'child' table.  A profile is 'owned' by a user.

- The site_user_id is set as a FOREIGN KEY referencing or linking to the id column of the site_user table.
- The site_user_id is also set as UNIQUE which means only one id from the site_user table can be linked to a profile.
- The FOREIGN KEY of site_user_id in the profile table is also set to ON DELETE CASCADE. 
- ON DELETE CASCADE means that when the 'parent' row (site_user) is deleted, then this profile row will also be deleted.

### The Tables

```
CREATE TABLE site_user(
    id SERIAL PRIMARY KEY,
    username VARCHAR(100) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password VARCHAR(100) NOT NULL
);

CREATE TABLE profile(
    site_user_id INT PRIMARY KEY UNIQUE,
    description TEXT NOT NULL,
    FOREIGN KEY(site_user_id)
    REFERENCES site_user(id)
    ON DELETE CASCADE
);
```

### Add data
Let's add some data

```
INSERT INTO site_user(
    username,
    email,
    password
)
VALUES
('bobwatkins','bobwatkins@anemailaccount.com','1234');

INSERT INTO profile(
    site_user_id,
    description
)
VALUES
(1, 'Hi my name is Bob.');
```

### Test the Unique Constraint
Let's test the unique constraint by trying to create another profile for user of id '1'
- We can see that an error is returned because only ONE profile is allowed for each user.

```
INSERT INTO profile(
    site_user_id,
    description
)
VALUES
(1, 'Hi, I go by the name of Bob.');

ERROR:  duplicate key value violates unique constraint "profile_pkey"
DETAIL:  Key (site_user_id)=(1) already exists.
```

### Test Delete a Site User
Let's see what happens to the profile table if we delete a user...

```
DELETE FROM site_user 
WHERE username = 'bobwatkins';
```

Because we have ON DELETE CASCADE set to the site_user_id in the profile table, when a user is deleted, the profile is also deleted.

```
SELECT username, email, password 
FROM site_user;

 id | username | email | password 
----+----------+-------+----------
(0 rows)


SELECT site_user_id, description 
FROM profile;

 site_user_id | description 
--------------+-------------
(0 rows)
```

### Delete the data and restart the increment counter
- If you want to add more site_users and profiles to test the tables, the increment counter on site_user will continue with an id of '2'.
- If you want to reset the PRIMARY KEY increment counter back to 1 (and delete all data in the site_user table) you can use TRUNCATE TABLE...CASCADE

```
TRUNCATE TABLE site_user
RESTART IDENTITY
CASCADE;

```

### Drop a table referenced by another table
- If you try and drop the parent table 'site_user' you would have the error shown below...

```
DROP site_user;

ERROR:  cannot drop table site_user because other objects depend on it
DETAIL:  constraint profile_site_user_id_fkey on table profile depends on table site_user
HINT:  Use DROP ... CASCADE to drop the dependent objects too.
```

Here, we can see when we use DROP... CASCADE, the reference is removed from the profile table... (but we still have to DROP the profile table)

```
DROP TABLE site_user
CASCADE;

NOTICE:  drop cascades to constraint profile_site_user_id_fkey on table profile

--- Describe the setup of profile - the Foreign Key has been removed...
\d profile

Table "public.profile"
    Column    |  Type   | Collation | Nullable | Default 
--------------+---------+-----------+----------+---------
 site_user_id | integer |           | not null | 
 description  | text    |           | not null | 
Indexes:
    "profile_pkey" PRIMARY KEY, btree (site_user_id)

DROP profile;

```

### DJANGO - How it works in Django

- Compare the Django model below with the Postgres table for profile above.

```
class Profile(models.Model):
    # unique=True is automatically created in the 'profile' database table by Django when you use a OneToOneField.
    
    user = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
    )
    description = models.TextField(
        null = False, 
        blank=False
    )
```

- The site_user_id is set as a FOREIGN KEY referencing or linking to the id column of the site_user table.
- The site_user_id is also set as UNIQUE which means only one id from the site_user table can be linked to a profile.
- The FOREIGN KEY of site_user_id in the profile table is also set to ON DELETE CASCADE. 
- ON DELETE CASCADE means that when the 'parent' row (site_user) is deleted, then this profile row will also be deleted.











NOTE: In this example, instead of 'site_user' (only used as an example to avoid the postgres 'user' keyword) we use the built in django user model.

NOTE: 'settings.AUTH_USER_MODEL' refers to the built in Django 'user' model.

NOTE:  Using 'models.OneToOneField' effectively does the same as above - adding a foreign key that is set to unique. This means there can be only 1 user associated with the profile and vice versa.

NOTE:  The 'on_delete=models.CASCADE' part means that the same things will happen as above.  If the user is deleted, the profile is irrelevant and will be deleted too!

NOTE: Django will automatically create the SERIAL (AUTO INCREMENTED) PRIMARY KEY integer for your model.

- Here is how you would represent the 'profile' table with a Django model...



### Accessing Django objects

This is from the Django docs:
"A one-to-one relationship. Conceptually, this is similar to a ForeignKey with unique=True, but the “reverse” side of the relation will directly return a single object."

- Basically, you could create your Django model with models.ForeignKey(Object, unique=True) but the OneToOneField in Django makes querying really easy.

- Accessing the model on the other side of the relationship is super easy...

```
user = profile.user
profile = user.profile # you may have to handle ObjectDoesNotExist error, if profile hasn't been created yet
```

- Exception handling of ObjectDoesNotExist

```
try:
    profile = user.profile
except ObjectDoesNotExist:
    print("No profile has been created.")

- You could even use hasattr to avoid exception writing..

hasattr(user, 'profile')
```

- A few more query examples...

```
--- Find profile belonging to username starting with a specific string
Profile.objects.get(user__username__startswith="bobwatkins")

- Finds profile whose users' username contains 'Bob'
Profile.objects.get(user__username__contains="Bob")
```

SUMMARY:  
- The Django 'OneToOneField' effectively means this model belongs to the parent model and only ONE of this model can belong to the parent.
- 'on_delete=models.CASCADE' means that IF the parent model ('user') is deleted, the 'profile' object will also be deleted.