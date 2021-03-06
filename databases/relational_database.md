# **Relational Database**

Virtually all relational database systems use SQL (Structured Query Language) for querying and maintaining the database.

<!-- TOC -->

- [**Relational Database**](#relational-database)
    - [1. Intro to databases](#1-intro-to-databases)
        - [1.1. Advantages of relational databases](#11-advantages-of-relational-databases)
        - [1.2. Data types](#12-data-types)
        - [1.3. Database properties](#13-database-properties)
        - [1.4. Normalized vs denormalized table](#14-normalized-vs-denormalized-table)
        - [1.5. Tips](#15-tips)
    - [2. Create a database, table, and view](#2-create-a-database-table-and-view)
        - [2.1. Create a database](#21-create-a-database)
        - [2.2. Create a table](#22-create-a-table)
        - [2.3. View](#23-view)
    - [3. Alter, insert, update, and delete table contents](#3-alter-insert-update-and-delete-table-contents)
        - [3.1. Alter table columns](#31-alter-table-columns)
        - [3.2. Insert entries](#32-insert-entries)
        - [3.3. Update entries](#33-update-entries)
        - [3.4. Delete entries](#34-delete-entries)
    - [4. Select clauses](#4-select-clauses)
        - [4.1. Simple select](#41-simple-select)
        - [4.2. Join](#42-join)
        - [4.3. Subquery](#43-subquery)
    - [5. Write code with DB-API and command line](#5-write-code-with-db-api-and-command-line)
        - [5.1. Basic structure of DB-API](#51-basic-structure-of-db-api)
        - [5.2. Select and insert with SQLite](#52-select-and-insert-with-sqlite)
        - [5.3. Select with PostgreSQL](#53-select-with-postgresql)
        - [5.4. SQL injection attack and script injection attack](#54-sql-injection-attack-and-script-injection-attack)
        - [5.5. PostgreSQL command line](#55-postgresql-command-line)

<!-- /TOC -->

## 1. Intro to databases

A database is a collection of tables

### 1.1. Advantages of relational databases

- **Databases:**

    - Persistent storage
    - Safe concurrent access by multiple programs and users

- **Relational databases:**

    - Flexible query languages with aggregation and join operations
    - Constraining rules for protecting consistency of data

### 1.2. Data types

- text, char(n), varchar(n)
- integer, real, double precision, decimal
- date, time, timestamp

Always write 'single quotes' around text strings and date/time values

### 1.3. Database properties

* **Atomicity**: a transaction happens as a whole or not at all
* **Consistency**: a transaction must be valid according to defined rules
* **Isolation**: read and write to multiple tables at the same time
* **Durability**: completed transactions are recorded even in the case of a system failure (e.g., power outage or crash)

### 1.4. Normalized vs denormalized table

- **Rules for normalized tables:**

    - Every row has the same number of columns.
    - There is a unique key, and everything in a row says something about the key.
    - Facts that don't relate to the key belong in different tables.
    - Tables shouldn't imply relationships that don't exist.

### 1.5. Tips

- Table names are usually singular
- Column name with special characters: ``` table1.`col name @#$%7` ```

- Wildcard:
  - `'%'`: match >=1 letter, e.g., `'JO%'` -> `'JOLIAN'`, `'JOLI'`, `'JOSEPH'`
  - `'_'`: match just 1 letter, e.g., `'_AN'` -> `'DAN'`, `'MAN'`

## 2. Create a database, table, and view

### 2.1. Create a database

- #### Create and use a database

  ```sql
  DROP DATABASE IF EXISTS animals_db;
  CREATE DATABASE animals_db;
  USE animals_db;
  ```

### 2.2. Create a table

- #### Basic structure

  ```sql
  CREATE TABLE animals (
      name TEXT [constraints],
      species TEXT [constraints],
      birthdate DATE [constraints],
      [row constraints]);
  ```

- #### Delete a table

  ```sql
  DROP TABLE animals;
  ```

- #### Assign primary key

  Primary key must be unique.

  ```sql
  CREATE TABLE people (
      id  INTEGER(11) AUTO_INCREMENT NOT NULL,
      first_name VARCHAR(30) NOT NULL,
      has_pet BOOLEAN NOT NULL,
      pet_name VARCHAR(30),
      pet_age INTEGER(10),
      PRIMARY KEY (id));
  ```

  ```sql
  CREATE TABLE animals (
      id SERIAL PRIMARY KEY,
      name TEXT,
      birthdate DATE);
  ```

- #### Assign multiple columns as primary key

  ```sql
  CREATE TABLE animals (
      postal_code TEXT,
      country TEXT,
      name TEXT,
      PRIMARY KEY (postal_code, country));
  ```

- #### Declare relationships

  Reference provides referential integrity - columns that are supposed to refer to each other are guaranteed to do so.

  ```sql
  CREATE TABLE sale (
      sku TEXT reference products (sku),
      sale_date DATE,
      count INTEGER);
  ```

- #### Assign foreign key

  A foreign key is a column or a set of columns in one table, that uniquely identifies rows in another table.

  ```sql
  CREATE TABLE animals_location (
      id INTEGER(11) AUTO_INCREMENT NOT NULL,
      location VARCHAR(30) NOT NULL,
      animal_id INTEGER(10) NOT NULL,
      PRIMARY KEY (id),
      FOREIGN KEY (animal_id) REFERENCES animals_all(id));
  ```

### 2.3. View

A view is a select query stored in the database in a way that lets you use it like a table.

- #### Create a view

  ```sql
  CREATE view course_size AS
      SELECT course_id, count(*) AS num
      FROM enrollment
      GROUP BY course_id;
  ```
- #### Delete a view

  ```sql
  DROP view course_size;
  ```

## 3. Alter, insert, update, and delete table contents

### 3.1. Alter table columns

- #### Add a column

  ```sql
  ALTER TABLE programming_languages
  ADD COLUMN new_column VARCHAR(15) DEFAULT "test" AFTER mastered;
  ```

- #### Change datatype of a column

  ```sql
  alter TABLE actor
  MODIFY middle_name blob;
  ```

- #### Delete a column

  ```sql
  alter TABLE actor
  DROP column middle_name;
  ```

### 3.2. Insert entries

- #### Insert one entry

  ```sql
  INSERT INTO people(first_name, has_pet, pet_name, pet_age)
  VALUES ("Dan", true, "Rocky", 400);
  ```

- #### Insert multiple entries

  ```sql
  INSERT INTO people(first_name, has_pet, pet_name, pet_age)
  VALUES ("Dan", true, "Rocky", 400), ("Dan", true, "Rocky", 400), ("Dan", true, "Rocky", 400);
  ```

- #### If values have same order as columns and start from the first column

  ```sql
  INSERT INTO people
  VALUES ("Dan", true, "Rocky", 400);
  ```

### 3.3. Update entries

- #### Update entries

  ```sql
  UPDATE animals
  SET name = 'cheese'
  WHERE name LIKE '%awful%';
  ```

### 3.4. Delete entries

- #### Delete entries

  ```sql
  SET SQL_SAFE_UPDATES = 0;

  DELETE FROM animals
  WHERE name = 'cheese';
  
  SET SQL_SAFE_UPDATES = 1;
  ```

## 4. Select clauses

**SELECT** *columns* **FROM** *tables* **WHERE** *conditions* **;**

- Columns are separated by commas; use * to select all columns.
- The condition is Boolean (`and`, `or`, `not`). [DeMorgan's Law](https://en.wikipedia.org/wiki/De_Morgan%27s_laws) 
- Comparison operators (`=`, `!=`, `<`, `>=`, etc.)

```sql
SELECT * FROM animals_db.people;
```

is equivalent to

```sql
Use animals_db;
SELECT * FROM people;
```

### 4.1. Simple select

- #### Limit, offset

  ```sql
  SELECT food FROM diet
  WHERE species = 'llama' AND name = 'Max'
  LIMIT 10   -- 10 rows
  offset 20; -- skip 20 rows
  ```

- #### Where

  ```sql
  ... WHERE species IN ('llama', 'orangutan')
  ```

  ```sql
  ... WHERE latitude between 50 AND 55
  ```

  ```sql
  ... WHERE col_1 LIKE 'pie' -- select where field contains words
  ```

  ```sql
  ... WHERE NOT department = 'Sports' -- equivalent to WHERE department != 'Sports'
  ```

- #### Order by

  ```sql
  SELECT * FROM animals
  WHERE species = 'orangutan'
  ORDER BY birthdate DESC; -- ascending if not specified
  ```

- #### Group by with aggregations

  ```sql
  SELECT species, min(birthdate) FROM animals
  GROUP BY species;
  ```

  ```sql
  SELECT name, count(*) AS num FROM animals
  GROUP BY name
  ORDER BY num DESC
  LIMIT 5;
  ```

- #### Having

  - `WHERE` is a restriction on the source tables
  - `HAVING` is a restriction on the results, after aggregation <br>

  ```sql
  SELECT species, count(*) AS num FROM animals
  GROUP BY species
  HAVING num = 1;
  ```

### 4.2. Join

![sql join venn diagram](resources/UI25E.jpg)

- #### Inner join

  ```sql
  SELECT animals.name, animals.species, diet.food 
  FROM animals
  INNER JOIN diet
  ON animals.species = diet.species
  WHERE diet.food = 'fish';
  ```

  Equivalently, without `join`

  ```sql
  SELECT animals.name, animals.species, diet.food
  FROM animals, diet
  WHERE animals.species = diet.species AND diet.food = 'fish';
  ```

- #### Join on the same column name

  ```sql
  SELECT products.name, products.sku, count(sales.sku) AS num
  FROM products
  LEFT JOIN sales
  USING (sku) -- equivalent to `on products.sku = sales.sku`
  ```

- #### Self Join

  ```sql
  SELECT a.id, b.id, a.building, a.room
          FROM residences AS a, residences AS b
      WHERE a.building = b.building
      AND a.room = b.room
      AND a.id < b.id
      ORDER BY a.building, a.room;
  ```

### 4.3. Subquery

- #### PostgreSQL

  ```sql
  SELECT avg(bigscore) FROM
      (SELECT max(score) AS bigscore
      FROM mooseball
      GROUP BY team)
  AS maxes; -- a table alias is required in PostgreSQL
  ```

  - [Scalar Subqueries](https://www.postgresql.org/docs/9.4/static/sql-expressions.html#SQL-SYNTAX-SCALAR-SUBQUERIES)
  - [Subquery Expressions](https://www.postgresql.org/docs/9.4/static/functions-subquery.html)
  - [FROM Clause](https://www.postgresql.org/docs/9.4/static/sql-select.html#SQL-FROM)

- #### MySQL

  Subquery after `WHERE`

  ```sql
  SELECT *
  FROM film
  WHERE length IN (
      SELECT MAX(length), MIN(length)
      FROM film
  );
  ```
  
  Subquery after `WHERE`, single input

  ```sql
  SELECT *
  FROM film
  WHERE length = (
      SELECT MAX(length)
      FROM film
  );
  ```

## 5. Write code with DB-API and command line

Python DB-API is a standard for python libraries that let the code to connect to the datebases.

### 5.1. Basic structure of DB-API

.connect(...) <br> &nbsp; &nbsp; &nbsp; ↓ <br>
**connection**.commit() <br> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .rollback() <br> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; .cursor() <br> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; ↓ <br> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;**cursor**.execute(query) <br> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;.fetchone() <br> &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;.fetchall()

### 5.2. Select and insert with SQLite

- #### Select

    ```python
    import sqlite3
    conn = sqlite3.connect('Cookies')
    cursor = conn.cursor()
    cursor.execute(
        'select host_key from cookies limit 10')
    results = cursor.fetchall()
    print(results)
    conn.close()
    ```

- #### Insert

    ```python
    import sqlite3
    conn = sqlite3.connect('Cookies')
    cursor = conn.cursor()
    cursor.execute(
        "insert into names values ('Jennifer Smith')")
    conn.commit()
    conn.close()
    ```

### 5.3. Select with PostgreSQL

- #### Select

    ```python
    import psycopg2
    psycopg2.connect("dbname=bears")
    ```

### 5.4. SQL injection attack and script injection attack

- #### Solve SQL injection attack

  Use query parameters instead of string substitution to execute insert query!

  ```python
  cursor.execute(
      "insert into names values (%s)", (content, ))
  ```

- #### Solve script injection attack

  - `bleach.clean(...)` see [documentation](http://bleach.readthedocs.io/en/latest/)
  - `update` command

### 5.5. PostgreSQL command line

[PostgreSQL documentation](https://www.postgresql.org/docs/9.4/static/app-psql.html)

- #### Connect to a database named `forum`

  `psql forum` <br>
  if already within psql, `\c forum`

- #### Get info about the database

  - display columns of a table named `posts`
  
    `\d posts`

  - list all the tables in the database
  
    `\dt`

  - list tables plus additional information
  
    `\dt+`

  - switch between printing tables in plain text vs HTML

    `\H`

- #### Display the contents of a table named `posts` and refresh it every two seconds

  `select * from posts \watch`

