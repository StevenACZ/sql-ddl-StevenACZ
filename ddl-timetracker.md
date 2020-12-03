## Create a Database

Create a database using the following ERD for the MVP of Time Tracker

![](https://p-vvf5mjm.b1.n0.cdn.getcloudapp.com/items/6quxL9AE/Image%202020-12-01%20at%207.57.45%20PM.png?source=viewer&v=9d21601cf2984d2b5d66d5b4330b6c63)

Some clarifications:

- The `rate` of the user is their cost per hour (in US\$)
- The `total_budget` is the amount of money allocated to one user on one project between the start and end date of the project.
- The `closed` attribute of a Project define if a project is open (`false`) or closed (`true`).
- The Daily-Log register how many hour each member has invest on a project in one day.

1. Create a new database called `timetracker`

```SQL
CREATE DATABASE timetracker;
```

Expected result:

```bash
postgres=# \l

              Name               |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
---------------------------------+----------+----------+-------------+-------------+-----------------------
 ...
 timetracker                     | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 ...
(xx rows)
```

2. Create the table `user` following the ERD.
   - id should be auto incremented.
   - No field should be null.
   - `rate` should be grater than or equal to 0.

```SQL
CREATE TABLE users (
    id BIGSERIAL NOT NULL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    email VARCHAR(25) NOT NULL,
    role VARCHAR(50) NOT NULL,
    rate INT CHECK (rate >= 0) NOT NULL
);
```

Expected result:

```bash
timetracker=# \d user
                                   Table "public.user"
 Column |         Type          | Collation | Nullable |             Default
--------+-----------------------+-----------+----------+----------------------------------
 id     | integer               |           | not null | nextval('user_id_seq'::regclass)
 name   | character varying(50) |           | not null |
 email  | character varying(25) |           | not null |
 role   | character varying(50) |           | not null |
 rate   | integer               |           | not null |
Indexes:
    "user_pkey" PRIMARY KEY, btree (id)
Check constraints:
    "user_rate_check" CHECK (rate >= 0)
```

3. Create the table `project` following the ERD
   - id should be auto incremented.
   - No field should be null
   - By default the `closed` attribute should be `false`

```SQL
CREATE TABLE projects (
    id BIGSERIAL NOT NULL PRIMARY KEY,
    name VARCHAR(25) NOT NULL,
    category VARCHAR(25) NOT NULL,
    start_date DATE NOT NULL,
    end_date DATE NOT NULL,
    closed BOOLEAN DEFAULT false NOT NULL
);
```

Expected result:

```bash
timetracker=# \d project
                                     Table "public.project"
   Column   |         Type          | Collation | Nullable |               Default
------------+-----------------------+-----------+----------+-------------------------------------
 id         | integer               |           | not null | nextval('project_id_seq'::regclass)
 name       | character varying(25) |           | not null |
 category   | character varying(25) |           | not null |
 start_date | date                  |           | not null |
 end_date   | date                  |           | not null |
 closed     | boolean               |           | not null | false
Indexes:
    "project_pkey" PRIMARY KEY, btree (id)
```

## Create relationships

4. Create the `user-project` table.
   - id should be auto incremented.
   - `total_budget` should be grater than or equal to 0.

```SQL
CREATE TABLE users_projects (
    id BIGSERIAL NOT NULL PRIMARY KEY,
    user_id BIGINT REFERENCES users (id) NOT NULL,
    project_id BIGINT REFERENCES projects (id) NOT NULL,
    total_budget INT CHECK (total_budget >= 0) NOT NULL
);
```

Expected result:

```bash
timetracker=# \d user-project
                                Table "public.user-project"
    Column    |  Type   | Collation | Nullable |                  Default
--------------+---------+-----------+----------+--------------------------------------------
 id           | integer |           | not null | nextval('"user-project_id_seq"'::regclass)
 user_id      | integer |           | not null |
 project_id   | integer |           | not null |
 total_budget | integer |           | not null |
Indexes:
    "user-project_pkey" PRIMARY KEY, btree (id)
Check constraints:
    "user-project_total_budget_check" CHECK (total_budget >= 0)
Foreign-key constraints:
    "user-project_project_id_fkey" FOREIGN KEY (project_id) REFERENCES project(id)
    "user-project_user_id_fkey" FOREIGN KEY (user_id) REFERENCES "user"(id)
```

5. Create the `daily-log` table,
   - id should be auto incremented.
   - All fields are required.
   - `hours` should be grater than or equal to 0.

```SQL
CREATE TABLE dailys_logs (
    id BIGSERIAL NOT NULL PRIMARY KEY,
    user_project_id BIGINT REFERENCES users_projects (id) NOT NULL,
    date DATE NOT NULL,
    hours INT CHECK (hours >= 0) NOT NULL
);
```

Expected result:

```bash
timetracker=# \d daily-log
                                  Table "public.daily-log"
     Column      |  Type   | Collation | Nullable |                 Default
-----------------+---------+-----------+----------+-----------------------------------------
 id              | integer |           | not null | nextval('"daily-log_id_seq"'::regclass)
 user_project_id | integer |           | not null |
 date            | date    |           | not null |
 hours           | integer |           | not null |
Indexes:
    "daily-log_pkey" PRIMARY KEY, btree (id)
Check constraints:
    "daily-log_hours_check" CHECK (hours >= 0)
Foreign-key constraints:
    "daily-log_user_project_id_fkey" FOREIGN KEY (user_project_id) REFERENCES "user-project"(id)
```

## Alter a database

After some revision, some changes should be done to our database.

6. Changes on the `user` table
   - The `email` field should be unique.
   - The name and role fields length are not enough. Change it to a maximum of 50 characters.

```SQL
ALTER TABLE users ADD UNIQUE (email);
ALTER TABLE users ALTER name TYPE VARCHAR(50);
ALTER TABLE users ALTER role TYPE VARCHAR(50);
```

Expected result:

```bash
timetracker=# \d user
                                   Table "public.user"
 Column |         Type          | Collation | Nullable |             Default
--------+-----------------------+-----------+----------+----------------------------------
 id     | integer               |           | not null | nextval('user_id_seq'::regclass)
 name   | character varying(50) |           | not null |
 email  | character varying(25) |           | not null |
 role   | character varying(50) |           | not null |
 rate   | integer               |           | not null |
Indexes:
    "user_pkey" PRIMARY KEY, btree (id)
    "User_email_key" UNIQUE CONSTRAINT, btree (email)
Check constraints:
    "user_rate_check" CHECK (rate >= 0)
Referenced by:
    TABLE ""user-project"" CONSTRAINT "user-project_user_id_fkey" FOREIGN KEY (user_id) REFERENCES "user"(id)
```

7. Changes on the `project` table,
   - `end_date` should be greater than `start_date`

```SQL
ALTER TABLE projects ADD CHECK (end_date > start_date);
```

Expected result:

```bash
timetracker=# \d project
                                     Table "public.project"
   Column   |         Type          | Collation | Nullable |               Default
------------+-----------------------+-----------+----------+-------------------------------------
 id         | integer               |           | not null | nextval('project_id_seq'::regclass)
 name       | character varying(25) |           | not null |
 category   | character varying(25) |           | not null |
 start_date | date                  |           | not null |
 end_date   | date                  |           | not null |
 closed     | boolean               |           | not null | false
Indexes:
    "project_pkey" PRIMARY KEY, btree (id)
Check constraints:
    "End_after_start_check" CHECK (end_date > start_date)
Referenced by:
    TABLE ""user-project"" CONSTRAINT "user-project_project_id_fkey" FOREIGN KEY (project_id) REFERENCES project(id)
```

8. Changes on the `user-project` table to have unique combination of user_id and project_id

```SQL
ALTER TABLE users_projects ADD UNIQUE (user_id, project_id);
```

Expected result:

```bash
timetracker=# \d user-project
                                Table "public.user-project"
    Column    |  Type   | Collation | Nullable |                  Default
--------------+---------+-----------+----------+--------------------------------------------
 id           | integer |           | not null | nextval('"user-project_id_seq"'::regclass)
 user_id      | integer |           | not null |
 project_id   | integer |           | not null |
 total_budget | integer |           | not null |
Indexes:
    "user-project_pkey" PRIMARY KEY, btree (id)
    "User-Project_unique_FKs" UNIQUE CONSTRAINT, btree (user_id, project_id)
Check constraints:
    "user-project_total_budget_check" CHECK (total_budget >= 0)
Foreign-key constraints:
    "user-project_project_id_fkey" FOREIGN KEY (project_id) REFERENCES project(id)
    "user-project_user_id_fkey" FOREIGN KEY (user_id) REFERENCES "user"(id)
Referenced by:
    TABLE ""daily-log"" CONSTRAINT "daily-log_user_project_id_fkey" FOREIGN KEY (user_project_id) REFERENCES "user-project"(id)
```

9. Changes on the `daily-log` table to have unique combination of user_project_id and date

```SQL
ALTER TABLE dailys_logs ADD UNIQUE (user_project_id, date);
```

Expected result:

```bash
timetracker=# \d daily-log
                                  Table "public.daily-log"
     Column      |  Type   | Collation | Nullable |                 Default
-----------------+---------+-----------+----------+-----------------------------------------
 id              | integer |           | not null | nextval('"daily-log_id_seq"'::regclass)
 user_project_id | integer |           | not null |
 date            | date    |           | not null |
 hours           | integer |           | not null |
Indexes:
    "daily-log_pkey" PRIMARY KEY, btree (id)
    "user_project_id-date unique" UNIQUE CONSTRAINT, btree (user_project_id, date)
Check constraints:
    "daily-log_hours_check" CHECK (hours >= 0)
Foreign-key constraints:
    "daily-log_user_project_id_fkey" FOREIGN KEY (user_project_id) REFERENCES "user-project"(id)
```

## Insert values (to check the result just make a SELECT query 🙂)

While the app gets ready, the team has been tracking their hours manually. Use [this google document](https://docs.google.com/spreadsheets/d/1KrQXWFzv4Nddss2p3xVC9V_ME5vH3ghN0n7JNhLoVQ8/edit?usp=sharing) to populate your newly created tables.

10. Insert the `user` data.

```SQL
INSERT INTO users (name, email, role, rate)
VALUES ('Renato', 'renato@codeable.la', 'front-end developer senior', 30);

INSERT INTO users (name, email, role, rate) values ('Paty', 'paty@codeable.la', 'back-end developer senior', 32);

INSERT INTO users (name, email, role, rate) values ('Franco', 'franco@codeable.la', 'front-end developer junior', 15);

INSERT INTO users (name, email, role, rate) values ('Luis', 'luis@codeable.la', 'back-end developer junior', 16);
```

11. Insert the `project` data.

```SQL
INSERT INTO projects (name, category, start_date, end_date, closed)
VALUES ('Shiftme', 'Business', '2020-05-13', '2020-08-11', false);

INSERT INTO projects (name, category, start_date, end_date, closed) V
ALUES ('Line Balancing', 'Business', '2020-05-13', '2020-09-10', false);

INSERT INTO projects (name, category, start_date, end_date, closed) VALUES ('Overbooking', 'Business', '2020-05-13', '2020-10-10', false);

INSERT INTO projects (name, category, start_date, end_date, closed) VALUES ('Kampu', 'Sports', '2020-05-13', '2020-06-27', false);

INSERT INTO projects (name, category, start_date, end_date, closed) VALUES ('Codeable App', 'Education', '2020-05-13', '2020-11-09', false);
```

12. Insert the `user-Project` data.

```SQL
INSERT INTO users_projects (user_id, project_id, total_budget) VALUES (1, 1, 4320);

INSERT INTO users_projects (user_id, project_id, total_budget) VALUES (1, 2, 8640);

INSERT INTO users_projects (user_id, project_id, total_budget) VALUES (1, 3, 18000);

INSERT INTO users_projects (user_id, project_id, total_budget) VALUES (2, 4, 5760);

INSERT INTO users_projects (user_id, project_id, total_budget) VALUES (2, 5, 23040);

INSERT INTO users_projects (user_id, project_id, total_budget) VALUES (3, 1, 2700);

INSERT INTO users_projects (user_id, project_id, total_budget) VALUES (3, 5, 16200);

INSERT INTO users_projects (user_id, project_id, total_budget) VALUES (4, 4, 2880);
```

13. Insert the `daily-log` data.

```SQL
<INSERT YOUR SQL STATEMENTS HERE>
```
