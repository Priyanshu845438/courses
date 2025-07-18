
# 🗄️ Database Design & SQL Fundamentals

## Table of Contents
- [Database Fundamentals](#database-fundamentals)
- [SQL Basics](#sql-basics)
- [Database Design Principles](#database-design-principles)
- [Relationships](#relationships)
- [Normalization](#normalization)
- [Indexes and Performance](#indexes-and-performance)
- [Transactions](#transactions)
- [Advanced SQL](#advanced-sql)

---

## 🎯 Database Fundamentals

### What is a Database?

A **database** is an organized collection of structured data stored electronically in a computer system.

### Database Types:
```
Relational (SQL)     Non-Relational (NoSQL)
├── MySQL           ├── MongoDB (Document)
├── PostgreSQL      ├── Redis (Key-Value)
├── SQLite          ├── Cassandra (Column)
└── Oracle          └── Neo4j (Graph)
```

### RDBMS Components:
- **Tables**: Store data in rows and columns
- **Schema**: Structure definition
- **Primary Key**: Unique identifier
- **Foreign Key**: Reference to another table
- **Index**: Performance optimization

---

## 📊 SQL Basics

### Data Types

```sql
-- Numeric Types
INT, BIGINT, DECIMAL(10,2), FLOAT

-- String Types
VARCHAR(255), TEXT, CHAR(10)

-- Date/Time Types
DATE, TIME, DATETIME, TIMESTAMP

-- Boolean
BOOLEAN

-- Example table creation
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    date_of_birth DATE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### Basic CRUD Operations

```sql
-- CREATE (Insert)
INSERT INTO users (username, email, password_hash, first_name, last_name)
VALUES ('johndoe', 'john@example.com', 'hashedpassword', 'John', 'Doe');

-- Multiple inserts
INSERT INTO users (username, email, password_hash, first_name, last_name)
VALUES 
    ('alice', 'alice@example.com', 'hash1', 'Alice', 'Smith'),
    ('bob', 'bob@example.com', 'hash2', 'Bob', 'Johnson'),
    ('carol', 'carol@example.com', 'hash3', 'Carol', 'Williams');

-- READ (Select)
SELECT * FROM users;
SELECT username, email FROM users;
SELECT * FROM users WHERE is_active = TRUE;
SELECT * FROM users WHERE age > 18 AND is_active = TRUE;
SELECT * FROM users WHERE email LIKE '%@gmail.com';
SELECT * FROM users ORDER BY created_at DESC;
SELECT * FROM users LIMIT 10 OFFSET 20;

-- UPDATE
UPDATE users 
SET first_name = 'Jonathan', updated_at = CURRENT_TIMESTAMP
WHERE username = 'johndoe';

-- UPDATE with conditions
UPDATE users 
SET is_active = FALSE
WHERE last_login < DATE_SUB(NOW(), INTERVAL 1 YEAR);

-- DELETE
DELETE FROM users WHERE username = 'johndoe';
DELETE FROM users WHERE is_active = FALSE;
```

### Filtering and Sorting

```sql
-- WHERE clauses
SELECT * FROM users WHERE age BETWEEN 18 AND 65;
SELECT * FROM users WHERE username IN ('alice', 'bob', 'carol');
SELECT * FROM users WHERE email IS NOT NULL;
SELECT * FROM users WHERE first_name LIKE 'J%';

-- ORDER BY
SELECT * FROM users ORDER BY last_name ASC, first_name ASC;
SELECT * FROM users ORDER BY created_at DESC;

-- GROUP BY and aggregation
SELECT COUNT(*) as total_users FROM users;
SELECT COUNT(*) as active_users FROM users WHERE is_active = TRUE;
SELECT 
    DATE(created_at) as signup_date,
    COUNT(*) as signups
FROM users 
GROUP BY DATE(created_at)
ORDER BY signup_date DESC;

-- HAVING (filter groups)
SELECT 
    DATE(created_at) as signup_date,
    COUNT(*) as signups
FROM users 
GROUP BY DATE(created_at)
HAVING COUNT(*) > 5
ORDER BY signup_date DESC;
```

---

## 🏗️ Database Design Principles

### Design Process

```
1. Requirements Analysis
   ↓
2. Conceptual Design (ER Diagram)
   ↓
3. Logical Design (Tables, Relationships)
   ↓
4. Physical Design (Indexes, Constraints)
```

### Entity-Relationship (ER) Diagram

```sql
-- Example: E-commerce Database Design

-- Users table
CREATE TABLE users (
    user_id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Categories table
CREATE TABLE categories (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    parent_category_id INT,
    FOREIGN KEY (parent_category_id) REFERENCES categories(category_id)
);

-- Products table
CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stock_quantity INT DEFAULT 0,
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (category_id) REFERENCES categories(category_id)
);

-- Orders table
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled') DEFAULT 'pending',
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    shipping_address TEXT,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- Order items (many-to-many relationship)
CREATE TABLE order_items (
    order_item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT NOT NULL,
    product_id INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

---

## 🔗 Relationships

### Types of Relationships

```sql
-- One-to-One (User Profile)
CREATE TABLE user_profiles (
    profile_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL UNIQUE,
    bio TEXT,
    avatar_url VARCHAR(255),
    date_of_birth DATE,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- One-to-Many (User has many Posts)
CREATE TABLE posts (
    post_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);

-- Many-to-Many (Posts and Tags)
CREATE TABLE tags (
    tag_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE post_tags (
    post_id INT,
    tag_id INT,
    PRIMARY KEY (post_id, tag_id),
    FOREIGN KEY (post_id) REFERENCES posts(post_id) ON DELETE CASCADE,
    FOREIGN KEY (tag_id) REFERENCES tags(tag_id) ON DELETE CASCADE
);
```

### JOIN Operations

```sql
-- INNER JOIN (matching records in both tables)
SELECT 
    u.username,
    p.title,
    p.created_at
FROM users u
INNER JOIN posts p ON u.user_id = p.user_id;

-- LEFT JOIN (all users, even without posts)
SELECT 
    u.username,
    COUNT(p.post_id) as post_count
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
GROUP BY u.user_id, u.username;

-- Multiple JOINs
SELECT 
    u.username,
    p.title,
    GROUP_CONCAT(t.name) as tags
FROM users u
INNER JOIN posts p ON u.user_id = p.user_id
LEFT JOIN post_tags pt ON p.post_id = pt.post_id
LEFT JOIN tags t ON pt.tag_id = t.tag_id
GROUP BY p.post_id;

-- Self JOIN (categories with parent categories)
SELECT 
    c1.name as category,
    c2.name as parent_category
FROM categories c1
LEFT JOIN categories c2 ON c1.parent_category_id = c2.category_id;
```

---

## 🎯 Normalization

### Normal Forms

#### 1NF (First Normal Form)
- Eliminate repeating groups
- Each cell contains atomic values

```sql
-- ❌ Not in 1NF (multiple values in one cell)
CREATE TABLE bad_design (
    student_id INT,
    name VARCHAR(100),
    courses VARCHAR(255) -- "Math, Science, English"
);

-- ✅ 1NF compliant
CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE student_courses (
    student_id INT,
    course VARCHAR(50),
    PRIMARY KEY (student_id, course),
    FOREIGN KEY (student_id) REFERENCES students(student_id)
);
```

#### 2NF (Second Normal Form)
- Must be in 1NF
- No partial dependencies

```sql
-- ❌ Not in 2NF (course_name depends only on course_id, not on the full key)
CREATE TABLE bad_enrollments (
    student_id INT,
    course_id INT,
    course_name VARCHAR(100), -- Partial dependency
    grade CHAR(1),
    PRIMARY KEY (student_id, course_id)
);

-- ✅ 2NF compliant
CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100)
);

CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    grade CHAR(1),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(student_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

#### 3NF (Third Normal Form)
- Must be in 2NF
- No transitive dependencies

```sql
-- ❌ Not in 3NF (department_head depends on department_id, not student_id)
CREATE TABLE bad_students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    department_name VARCHAR(100), -- Transitive dependency
    department_head VARCHAR(100)  -- Transitive dependency
);

-- ✅ 3NF compliant
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100),
    department_head VARCHAR(100)
);

CREATE TABLE students (
    student_id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(department_id)
);
```

---

## ⚡ Indexes and Performance

### Creating Indexes

```sql
-- Single column index
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_created_at ON posts(created_at);

-- Composite index
CREATE INDEX idx_posts_user_date ON posts(user_id, created_at);

-- Unique index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Partial index (MySQL doesn't support this, but PostgreSQL does)
-- CREATE INDEX idx_active_users ON users(email) WHERE is_active = TRUE;

-- Full-text index
CREATE FULLTEXT INDEX idx_posts_content ON posts(title, content);

-- Show indexes
SHOW INDEX FROM users;
```

### Query Optimization

```sql
-- Use EXPLAIN to analyze query performance
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';

EXPLAIN SELECT 
    u.username,
    COUNT(p.post_id) as post_count
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
WHERE u.is_active = TRUE
GROUP BY u.user_id;

-- Optimize with proper indexing
-- Bad: Full table scan
SELECT * FROM posts WHERE YEAR(created_at) = 2023;

-- Good: Use index
SELECT * FROM posts 
WHERE created_at >= '2023-01-01' 
AND created_at < '2024-01-01';

-- Use LIMIT for pagination
SELECT * FROM posts 
ORDER BY created_at DESC 
LIMIT 10 OFFSET 20;
```

---

## 🔄 Transactions

### ACID Properties
- **Atomicity**: All or nothing
- **Consistency**: Valid state transitions
- **Isolation**: Concurrent transactions don't interfere
- **Durability**: Committed changes persist

### Transaction Examples

```sql
-- Basic transaction
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- Check if both operations succeeded
-- If any error occurs, ROLLBACK
-- If everything is fine, COMMIT
COMMIT;

-- Transaction with error handling
DELIMITER //

CREATE PROCEDURE transfer_money(
    IN from_account INT,
    IN to_account INT,
    IN amount DECIMAL(10,2)
)
BEGIN
    DECLARE exit handler FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        RESIGNAL;
    END;

    START TRANSACTION;
    
    -- Check if sufficient balance
    IF (SELECT balance FROM accounts WHERE account_id = from_account) < amount THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Insufficient balance';
    END IF;
    
    -- Perform transfer
    UPDATE accounts SET balance = balance - amount WHERE account_id = from_account;
    UPDATE accounts SET balance = balance + amount WHERE account_id = to_account;
    
    COMMIT;
END //

DELIMITER ;
```

---

## 🚀 Advanced SQL

### Window Functions

```sql
-- Row number
SELECT 
    username,
    email,
    created_at,
    ROW_NUMBER() OVER (ORDER BY created_at) as row_num
FROM users;

-- Ranking
SELECT 
    u.username,
    COUNT(p.post_id) as post_count,
    RANK() OVER (ORDER BY COUNT(p.post_id) DESC) as ranking
FROM users u
LEFT JOIN posts p ON u.user_id = p.user_id
GROUP BY u.user_id;

-- Running total
SELECT 
    order_date,
    total_amount,
    SUM(total_amount) OVER (ORDER BY order_date) as running_total
FROM orders
ORDER BY order_date;

-- Lag/Lead
SELECT 
    order_date,
    total_amount,
    LAG(total_amount, 1) OVER (ORDER BY order_date) as previous_amount,
    LEAD(total_amount, 1) OVER (ORDER BY order_date) as next_amount
FROM orders;
```

### Common Table Expressions (CTEs)

```sql
-- Simple CTE
WITH active_users AS (
    SELECT user_id, username, email
    FROM users
    WHERE is_active = TRUE
)
SELECT 
    au.username,
    COUNT(p.post_id) as post_count
FROM active_users au
LEFT JOIN posts p ON au.user_id = p.user_id
GROUP BY au.user_id;

-- Recursive CTE (organizational hierarchy)
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: top-level managers
    SELECT 
        employee_id,
        name,
        manager_id,
        1 as level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: employees with managers
    SELECT 
        e.employee_id,
        e.name,
        e.manager_id,
        eh.level + 1
    FROM employees e
    INNER JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy ORDER BY level, name;
```

### Stored Procedures and Functions

```sql
-- Stored procedure
DELIMITER //

CREATE PROCEDURE get_user_stats(IN user_id INT)
BEGIN
    SELECT 
        u.username,
        u.email,
        COUNT(p.post_id) as post_count,
        COUNT(c.comment_id) as comment_count,
        u.created_at as member_since
    FROM users u
    LEFT JOIN posts p ON u.user_id = p.user_id
    LEFT JOIN comments c ON u.user_id = c.user_id
    WHERE u.user_id = user_id
    GROUP BY u.user_id;
END //

DELIMITER ;

-- Call procedure
CALL get_user_stats(1);

-- Function
DELIMITER //

CREATE FUNCTION calculate_age(birth_date DATE)
RETURNS INT
READS SQL DATA
DETERMINISTIC
BEGIN
    RETURN TIMESTAMPDIFF(YEAR, birth_date, CURDATE());
END //

DELIMITER ;

-- Use function
SELECT 
    username,
    date_of_birth,
    calculate_age(date_of_birth) as age
FROM users;
```

---

## 📚 Summary

### Key Concepts:
- **Database design** principles and normalization
- **SQL fundamentals** for data manipulation
- **Relationships** and JOIN operations
- **Indexes** for performance optimization
- **Transactions** for data integrity
- **Advanced SQL** features for complex queries

### Best Practices:
1. **Normalize** your database design
2. **Use appropriate data types** and constraints
3. **Create indexes** on frequently queried columns
4. **Use transactions** for data consistency
5. **Optimize queries** with EXPLAIN
6. **Follow naming conventions**
7. **Regular backups** and maintenance

### Next Steps:
1. Practice with real datasets
2. Learn database-specific features (PostgreSQL, MySQL)
3. Study query optimization techniques
4. Explore NoSQL databases
5. Learn about database administration and tuning
