
# 🚀 Advanced Database Design & SQL

## Table of Contents
- [Advanced Database Design Patterns](#advanced-database-design-patterns)
- [Performance Tuning and Optimization](#performance-tuning-and-optimization)
- [Advanced Indexing Strategies](#advanced-indexing-strategies)
- [Partitioning and Sharding](#partitioning-and-sharding)
- [Replication and High Availability](#replication-and-high-availability)
- [Advanced Security](#advanced-security)
- [Data Warehousing Concepts](#data-warehousing-concepts)
- [NoSQL Integration](#nosql-integration)
- [Database Monitoring](#database-monitoring)

---

## 🏗️ Advanced Database Design Patterns

### Temporal Database Design

```sql
-- Slowly Changing Dimensions (SCD Type 2)
CREATE TABLE customer_history (
    surrogate_key INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    customer_name VARCHAR(100),
    email VARCHAR(100),
    address TEXT,
    phone VARCHAR(20),
    effective_date DATE NOT NULL,
    expiry_date DATE,
    is_current BOOLEAN DEFAULT TRUE,
    version_number INT DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_customer_current (customer_id, is_current),
    INDEX idx_effective_date (effective_date)
);

-- Trigger to manage SCD Type 2 updates
DELIMITER //
CREATE TRIGGER manage_customer_history
BEFORE UPDATE ON customers
FOR EACH ROW
BEGIN
    -- Close current record
    UPDATE customer_history 
    SET expiry_date = CURDATE(), is_current = FALSE
    WHERE customer_id = NEW.customer_id AND is_current = TRUE;
    
    -- Insert new record
    INSERT INTO customer_history (
        customer_id, customer_name, email, address, phone,
        effective_date, version_number
    ) VALUES (
        NEW.customer_id, NEW.customer_name, NEW.email, 
        NEW.address, NEW.phone, CURDATE(),
        (SELECT COALESCE(MAX(version_number), 0) + 1 
         FROM customer_history ch WHERE ch.customer_id = NEW.customer_id)
    );
END //
DELIMITER ;
```

### Event Sourcing Pattern

```sql
-- Event store table
CREATE TABLE event_store (
    event_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    aggregate_id VARCHAR(36) NOT NULL,
    aggregate_type VARCHAR(50) NOT NULL,
    event_type VARCHAR(100) NOT NULL,
    event_data JSON NOT NULL,
    event_metadata JSON,
    version_number INT NOT NULL,
    timestamp TIMESTAMP(6) DEFAULT CURRENT_TIMESTAMP(6),
    correlation_id VARCHAR(36),
    causation_id VARCHAR(36),
    INDEX idx_aggregate (aggregate_id, version_number),
    INDEX idx_timestamp (timestamp),
    INDEX idx_event_type (event_type)
);

-- Snapshot table for performance
CREATE TABLE aggregate_snapshots (
    snapshot_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    aggregate_id VARCHAR(36) NOT NULL,
    aggregate_type VARCHAR(50) NOT NULL,
    snapshot_data JSON NOT NULL,
    version_number INT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE KEY uk_aggregate_version (aggregate_id, version_number)
);

-- Projection for current order state
CREATE TABLE order_projections (
    order_id VARCHAR(36) PRIMARY KEY,
    customer_id VARCHAR(36),
    status ENUM('pending', 'confirmed', 'shipped', 'delivered', 'cancelled'),
    total_amount DECIMAL(10,2),
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    version_number INT
);
```

---

## ⚡ Performance Tuning and Optimization

### Query Optimization Techniques

```sql
-- Optimized pagination with offset alternatives
-- Instead of: SELECT * FROM products ORDER BY id LIMIT 1000, 20;
-- Use cursor-based pagination:
SELECT * FROM products 
WHERE id > 1000 
ORDER BY id 
LIMIT 20;

-- Optimized EXISTS vs IN
-- When checking for existence, use EXISTS instead of IN
SELECT c.customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.customer_id = c.customer_id 
    AND o.order_date >= '2024-01-01'
);

-- Optimized aggregation with partial indexes
-- Create filtered index for active orders only
CREATE INDEX idx_active_orders_date 
ON orders (order_date, customer_id) 
WHERE status IN ('pending', 'processing', 'shipped');

-- Query to use the filtered index
SELECT 
    customer_id,
    COUNT(*) as active_orders,
    MAX(order_date) as latest_order
FROM orders 
WHERE status IN ('pending', 'processing', 'shipped')
AND order_date >= '2024-01-01'
GROUP BY customer_id;
```

### Query Plan Analysis

```sql
-- Enable query profiling
SET profiling = 1;

-- Execute query
SELECT 
    p.product_name,
    c.category_name,
    AVG(oi.unit_price) as avg_price,
    SUM(oi.quantity) as total_sold
FROM products p
JOIN categories c ON p.category_id = c.category_id
JOIN order_items oi ON p.product_id = oi.product_id
JOIN orders o ON oi.order_id = o.order_id
WHERE o.order_date >= '2024-01-01'
GROUP BY p.product_id, p.product_name, c.category_name
HAVING total_sold > 100
ORDER BY total_sold DESC;

-- Analyze execution plan
EXPLAIN FORMAT=JSON 
SELECT /* your query here */;

-- View profiling results
SHOW PROFILES;
SHOW PROFILE FOR QUERY 1;
```

---

## 📈 Advanced Indexing Strategies

### Composite and Covering Indexes

```sql
-- Composite index for multi-column queries
CREATE INDEX idx_orders_customer_date_status 
ON orders (customer_id, order_date, status);

-- Covering index (includes all needed columns)
CREATE INDEX idx_order_items_covering 
ON order_items (order_id, product_id, quantity, unit_price);

-- Partial index for specific conditions
CREATE INDEX idx_high_value_orders 
ON orders (customer_id, order_date) 
WHERE total_amount > 1000;

-- Function-based index
CREATE INDEX idx_customer_email_domain 
ON customers ((SUBSTRING_INDEX(email, '@', -1)));

-- Full-text index for search
CREATE FULLTEXT INDEX idx_product_search 
ON products (product_name, description);

-- Use full-text search
SELECT product_id, product_name, 
       MATCH(product_name, description) AGAINST ('laptop gaming' IN NATURAL LANGUAGE MODE) as relevance
FROM products 
WHERE MATCH(product_name, description) AGAINST ('laptop gaming' IN NATURAL LANGUAGE MODE)
ORDER BY relevance DESC;
```

---

## 🔄 Partitioning and Sharding

### Table Partitioning

```sql
-- Range partitioning by date
CREATE TABLE orders_partitioned (
    order_id BIGINT AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date DATE NOT NULL,
    total_amount DECIMAL(10,2),
    status VARCHAR(20),
    PRIMARY KEY (order_id, order_date)
) PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- Hash partitioning for even distribution
CREATE TABLE user_sessions (
    session_id VARCHAR(128),
    user_id INT NOT NULL,
    session_data JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (session_id, user_id)
) PARTITION BY HASH(user_id) 
PARTITIONS 8;

-- List partitioning by region
CREATE TABLE regional_sales (
    sale_id BIGINT AUTO_INCREMENT,
    region VARCHAR(20),
    amount DECIMAL(10,2),
    sale_date DATE,
    PRIMARY KEY (sale_id, region)
) PARTITION BY LIST COLUMNS(region) (
    PARTITION p_north VALUES IN ('US-NORTH', 'CA-NORTH'),
    PARTITION p_south VALUES IN ('US-SOUTH', 'MX'),
    PARTITION p_europe VALUES IN ('UK', 'DE', 'FR'),
    PARTITION p_asia VALUES IN ('JP', 'CN', 'IN')
);
```

### Sharding Strategy

```sql
-- Horizontal sharding setup
-- Shard 1: customers with ID 1-100000
-- Shard 2: customers with ID 100001-200000
-- etc.

-- Shard routing function
DELIMITER //
CREATE FUNCTION get_shard_id(customer_id INT) 
RETURNS INT
READS SQL DATA
DETERMINISTIC
BEGIN
    RETURN FLOOR((customer_id - 1) / 100000) + 1;
END //
DELIMITER ;

-- Application-level sharding example
-- This would be implemented in application code
-- 
-- def get_database_connection(customer_id):
--     shard_id = (customer_id - 1) // 100000 + 1
--     return connections[f'shard_{shard_id}']
```

---

## 🔒 Advanced Security

### Row-Level Security

```sql
-- Enable row-level security
CREATE TABLE sensitive_data (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT NOT NULL,
    department VARCHAR(50),
    salary DECIMAL(10,2),
    confidential_notes TEXT
);

-- Create security policy
DELIMITER //
CREATE FUNCTION can_access_row(row_user_id INT, row_department VARCHAR(50))
RETURNS BOOLEAN
READS SQL DATA
DETERMINISTIC
BEGIN
    DECLARE current_user_dept VARCHAR(50);
    DECLARE current_user_role VARCHAR(50);
    
    -- Get current user's department and role
    SELECT department, role INTO current_user_dept, current_user_role
    FROM user_permissions 
    WHERE username = USER();
    
    -- Allow access based on business rules
    RETURN (
        row_user_id = (SELECT user_id FROM user_permissions WHERE username = USER())
        OR current_user_role = 'ADMIN'
        OR (current_user_role = 'MANAGER' AND current_user_dept = row_department)
    );
END //
DELIMITER ;

-- Data masking for sensitive information
CREATE VIEW masked_employee_data AS
SELECT 
    id,
    user_id,
    department,
    CASE 
        WHEN can_access_row(user_id, department) THEN salary
        ELSE NULL
    END as salary,
    CASE 
        WHEN can_access_row(user_id, department) THEN confidential_notes
        ELSE '[REDACTED]'
    END as confidential_notes
FROM sensitive_data;
```

### Encryption and Hashing

```sql
-- Column-level encryption
CREATE TABLE secure_customers (
    customer_id INT AUTO_INCREMENT PRIMARY KEY,
    email VARCHAR(255),
    encrypted_ssn VARBINARY(255),
    password_hash VARCHAR(255),
    salt VARCHAR(32),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Store encrypted data
INSERT INTO secure_customers (email, encrypted_ssn, password_hash, salt)
VALUES (
    'john@example.com',
    AES_ENCRYPT('123-45-6789', 'encryption_key'),
    SHA2(CONCAT('user_password', 'random_salt'), 256),
    'random_salt'
);

-- Retrieve and decrypt
SELECT 
    customer_id,
    email,
    AES_DECRYPT(encrypted_ssn, 'encryption_key') as ssn
FROM secure_customers
WHERE customer_id = 1;
```

---

## 📊 Data Warehousing Concepts

### Star Schema Implementation

```sql
-- Fact table
CREATE TABLE fact_sales (
    sale_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    date_key INT NOT NULL,
    customer_key INT NOT NULL,
    product_key INT NOT NULL,
    store_key INT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10,2) NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    discount_amount DECIMAL(10,2) DEFAULT 0,
    cost_amount DECIMAL(10,2) NOT NULL,
    INDEX idx_date (date_key),
    INDEX idx_customer (customer_key),
    INDEX idx_product (product_key),
    INDEX idx_store (store_key)
);

-- Dimension tables
CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,
    full_date DATE NOT NULL,
    day_of_week VARCHAR(20),
    day_of_month INT,
    day_of_year INT,
    week_of_year INT,
    month_name VARCHAR(20),
    month_number INT,
    quarter INT,
    year INT,
    is_weekend BOOLEAN,
    is_holiday BOOLEAN
);

CREATE TABLE dim_customer (
    customer_key INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT NOT NULL,
    customer_name VARCHAR(100),
    customer_segment VARCHAR(50),
    geographic_region VARCHAR(50),
    effective_date DATE,
    expiry_date DATE,
    is_current BOOLEAN DEFAULT TRUE
);

-- ETL procedure for loading fact table
DELIMITER //
CREATE PROCEDURE load_daily_sales(IN load_date DATE)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    
    -- Clear existing data for the date
    DELETE FROM fact_sales 
    WHERE date_key = (SELECT date_key FROM dim_date WHERE full_date = load_date);
    
    -- Load new data
    INSERT INTO fact_sales (
        date_key, customer_key, product_key, store_key,
        quantity, unit_price, total_amount, discount_amount, cost_amount
    )
    SELECT 
        dd.date_key,
        dc.customer_key,
        dp.product_key,
        ds.store_key,
        oi.quantity,
        oi.unit_price,
        oi.quantity * oi.unit_price as total_amount,
        COALESCE(o.discount_amount, 0) as discount_amount,
        oi.quantity * p.cost_price as cost_amount
    FROM order_items oi
    JOIN orders o ON oi.order_id = o.order_id
    JOIN dim_date dd ON DATE(o.order_date) = dd.full_date
    JOIN dim_customer dc ON o.customer_id = dc.customer_id AND dc.is_current = TRUE
    JOIN dim_product dp ON oi.product_id = dp.product_id AND dp.is_current = TRUE
    JOIN dim_store ds ON o.store_id = ds.store_id
    JOIN products p ON oi.product_id = p.product_id
    WHERE DATE(o.order_date) = load_date;
    
END //
DELIMITER ;
```

This advanced SQL guide covers sophisticated database design patterns, performance optimization, advanced indexing, partitioning, security, and data warehousing concepts for enterprise-level applications.
