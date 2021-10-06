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