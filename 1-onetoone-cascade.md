## DJANGO vs. POSTGRES - 
## One To One Relationship with ON DELETE CASCADE

## POSTGRES - How it works in Postgres
- OneToOne relationship
- 'site_user' is the parent table
- 'site_user' is used in the example because 'user' is a reserved keyword
- 'profile' is the child table

### Situation
- The site_user table is considered the 'parent' table - a bit like an owner.  
- The profile table is considered the 'child' table - owned by the user.

- Let's create the tables

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

Let's test the unique constraint by trying to create another profile for user of id '1'

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

Let's see what happens to the profile table if we delete a user...

```
DELETE FROM site_user 
WHERE username = 'bobwatkins';
```

Because we have ON DELETE CASCADE set to the site_user_id in the profile table, when a user is deleted, the profile is also deleted.

```
SELECT * FROM site_user;

 id | username | email | password 
----+----------+-------+----------
(0 rows)


SELECT * FROM profile;

 site_user_id | description 
--------------+-------------
(0 rows)
```

If you want to delete all of the data to add different user and profile examples but also want to reset the PRIMARY KEY increment counter back to 1 you can use TRUNCATE TABLE...

```
TRUNCATE TABLE site_user
RESTART IDENTITY
CASCADE;

TRUNCATE TABLE profile;
```

Also, to show you how the foreign key relationship affects tables, if you try and drop the parent table 'site_user' you would have the error shown below because the table is referenced by the 'profile' table.

```
DROP site_user;

ERROR:  cannot drop table site_user because other objects depend on it
DETAIL:  constraint profile_site_user_id_fkey on table profile depends on table site_user
HINT:  Use DROP ... CASCADE to drop the dependent objects too.
```

Here, we can see another example of how the idea of CASCADE runs down from a parent table to a child table.

```
DROP TABLE site_user
CASCADE;

NOTICE:  drop cascades to constraint profile_site_user_id_fkey on table profile
DROP TABLE
```

## DJANGO - How it works in Django

NOTE: In this example, instead of 'site_user' (only used as an example to avoid the postgres 'user' keyword) we use the built in django user model.

NOTE: 'settings.AUTH_USER_MODEL' refers to the built in Django 'user' model.

NOTE:  Using 'models.OneToOneField' effectively does the same as above - adding a foreign key that is set to unique. This means there can be only 1 user associated with the profile and vice versa.

NOTE:  The 'on_delete=models.CASCADE' part means that the same things will happen as above.  If the user is deleted, the profile is irrelevant and will be deleted too!

NOTE: Django will automatically create the SERIAL (AUTO INCREMENTED) PRIMARY KEY integer for your model.

- Here is how you would represent the 'profile' table with a Django model...

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

### Accessing Django objects

This is from the Django docs:
"A one-to-one relationship. Conceptually, this is similar to a ForeignKey with unique=True, but the “reverse” side of the relation will directly return a single object."

- Basically, you could create your Django model with models.ForeignKey(Object, unique=True) but the OneToOneField in Django makes querying really easy.

- Accessing the model on other side of relationship is super easy...

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