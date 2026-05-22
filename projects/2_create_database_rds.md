# Step 1: Create a Database Schema

When you first connect, you aren't inside any specific database workspace. Let's create one called exam_practice.

Run the following commands directly inside your MariaDB [(none)]> or mysql> prompt:

1. Create a new workspace database
CREATE DATABASE exam_practice;

```
MySQL [(none)]> 
MySQL [(none)]> CREATE DATABASE exam_practice;
Query OK, 1 row affected (0.024 sec)
```

2. Tell the system to switch into this database context
USE exam_practice;

```
MySQL [(none)]> USE exam_practice;
Database changed
```

Your prompt should change from [(none)]> to [exam_practice]>, confirming you are in the right spot.

# Step 2: Create a Table Structure

Before adding entries, you must define the layout (the columns and data types) of your table. Let's make a simple users directory.

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

What this means:
id: A unique number for each row that increments automatically (1, 2, 3...).
VARCHAR: Text strings up to a set maximum length.
PRIMARY KEY & UNIQUE: Constraints that ensure data integrity (no duplicate IDs or emails).

```
MySQL [exam_practice]> CREATE TABLE users (
    ->     id INT AUTO_INCREMENT PRIMARY KEY,
    ->     first_name VARCHAR(50) NOT NULL,
    ->     last_name VARCHAR(50) NOT NULL,
    ->     email VARCHAR(100) UNIQUE,
    ->     created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    -> );
Query OK, 0 rows affected (0.114 sec)
```

# Step 3: Insert Entries into the Table

Now, let's write data rows into your table. Paste these lines one by one or all together:

**Insert a single entry**
INSERT INTO users (first_name, last_name, email) 
VALUES ('Karthik', 'Dev', 'karthik@example.com');

```
MySQL [exam_practice]> INSERT INTO users (first_name, last_name, email)
    -> VALUES ('Karthik', 'Dev', 'karthik@example.com');
Query OK, 1 row affected (0.037 sec)
```


**Insert multiple entries at once**
INSERT INTO users (first_name, last_name, email) 
VALUES 
('Alice', 'Smith', 'alice@example.com'),
('Bob', 'Jones', 'bob@example.com');

If it works, you will see a message saying: Query OK, 3 rows affected.

```
MySQL [exam_practice]> INSERT INTO users (first_name, last_name, email) 
    -> VALUES 
    -> ('Alice', 'Smith', 'alice@example.com'),
    -> ('Bob', 'Jones', 'bob@example.com');
Query OK, 2 rows affected (0.003 sec)
Records: 2  Duplicates: 0  Warnings: 0
```

# Step 4: Retrieve and Verify Your Data
To verify that your RDS storage layer is permanently holding onto these entries, query the table

SELECT * FROM users;

The Output
Your terminal will render a clean text table directly from your cloud database:

```
MySQL [exam_practice]> SELECT * FROM users;
+----+------------+-----------+---------------------+---------------------+
| id | first_name | last_name | email               | created_at          |
+----+------------+-----------+---------------------+---------------------+
|  1 | Karthik    | Dev       | karthik@example.com | 2026-05-22 12:21:21 |
|  2 | Alice      | Smith     | alice@example.com   | 2026-05-22 12:21:57 |
|  3 | Bob        | Jones     | bob@example.com     | 2026-05-22 12:21:57 |
+----+------------+-----------+---------------------+---------------------+
3 rows in set (0.002 sec)
```
+----+------------+-----------+---------------------+---------------------+
3 rows in set (0.00 sec)

