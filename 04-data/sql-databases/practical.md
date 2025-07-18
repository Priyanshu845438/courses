
# 🗄️ Intermediate Database Design & SQL

## Table of Contents
- [Advanced Joins and Subqueries](#advanced-joins-and-subqueries)
- [Window Functions](#window-functions)
- [Stored Procedures and Functions](#stored-procedures-and-functions)
- [Triggers](#triggers)
- [Views and Materialized Views](#views-and-materialized-views)
- [Database Optimization](#database-optimization)
- [Data Modeling Patterns](#data-modeling-patterns)
- [Backup and Recovery](#backup-and-recovery)
- [Security and Permissions](#security-and-permissions)

---

## 🔄 Advanced Joins and Subqueries

### Complex JOIN Operations

```sql
-- Self-join to find employees and their managers
SELECT 
    e.first_name AS employee_name,
    m.first_name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;

-- Multiple table joins with aggregation
SELECT 
    c.customer_name,
    COUNT(o.order_id) as total_orders,
    SUM(oi.quantity * p.price) as total_spent
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
LEFT JOIN order_items oi ON o.order_id = oi.order_id
LEFT JOIN products p ON oi.product_id = p.product_id
GROUP BY c.customer_id, c.customer_name
HAVING total_spent > 1000;
```

### Advanced Subqueries

```sql
-- Correlated subquery
SELECT product_name, price
FROM products p1
WHERE price > (
    SELECT AVG(price)
    FROM products p2
    WHERE p2.category_id = p1.category_id
);

-- EXISTS clause
SELECT customer_name
FROM customers c
WHERE EXISTS (
    SELECT 1 
    FROM orders o 
    WHERE o.customer_id = c.customer_id 
    AND o.order_date >= '2024-01-01'
);

-- Common Table Expressions (CTEs)
WITH sales_summary AS (
    SELECT 
        product_id,
        SUM(quantity) as total_sold,
        AVG(unit_price) as avg_price
    FROM order_items
    GROUP BY product_id
),
top_products AS (
    SELECT product_id, total_sold
    FROM sales_summary
    WHERE total_sold > 100
)
SELECT p.product_name, ss.total_sold, ss.avg_price
FROM products p
JOIN sales_summary ss ON p.product_id = ss.product_id
JOIN top_products tp ON p.product_id = tp.product_id;
```

---

## 📊 Window Functions

### Basic Window Functions

```sql
-- ROW_NUMBER, RANK, DENSE_RANK
SELECT 
    employee_name,
    department,
    salary,
    ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) as row_num,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) as rank_num,
    DENSE_RANK() OVER (PARTITION BY department ORDER BY salary DESC) as dense_rank
FROM employees;

-- Running totals and moving averages
SELECT 
    order_date,
    daily_sales,
    SUM(daily_sales) OVER (ORDER BY order_date) as running_total,
    AVG(daily_sales) OVER (
        ORDER BY order_date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) as moving_avg_7_days
FROM daily_sales_summary;
```

### Advanced Window Functions

```sql
-- LAG and LEAD for comparing with previous/next rows
SELECT 
    order_date,
    revenue,
    LAG(revenue, 1) OVER (ORDER BY order_date) as prev_revenue,
    LEAD(revenue, 1) OVER (ORDER BY order_date) as next_revenue,
    revenue - LAG(revenue, 1) OVER (ORDER BY order_date) as revenue_change
FROM monthly_revenue;

-- NTILE for creating buckets
SELECT 
    customer_id,
    total_spent,
    NTILE(4) OVER (ORDER BY total_spent DESC) as spending_quartile
FROM customer_spending;
```

---

## ⚙️ Stored Procedures and Functions

### Stored Procedures

```sql
DELIMITER //

CREATE PROCEDURE GetCustomerOrders(
    IN customer_id INT,
    IN start_date DATE,
    IN end_date DATE
)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE order_count INT DEFAULT 0;
    
    -- Get order count
    SELECT COUNT(*) INTO order_count
    FROM orders 
    WHERE customer_id = customer_id 
    AND order_date BETWEEN start_date AND end_date;
    
    -- Return results
    SELECT 
        o.order_id,
        o.order_date,
        o.total_amount,
        o.status
    FROM orders o
    WHERE o.customer_id = customer_id 
    AND o.order_date BETWEEN start_date AND end_date
    ORDER BY o.order_date DESC;
    
    -- Return summary
    SELECT order_count as total_orders;
END //

DELIMITER ;

-- Call the procedure
CALL GetCustomerOrders(123, '2024-01-01', '2024-12-31');
```

### User-Defined Functions

```sql
DELIMITER //

CREATE FUNCTION CalculateDiscount(
    customer_level VARCHAR(20),
    order_amount DECIMAL(10,2)
) RETURNS DECIMAL(5,2)
READS SQL DATA
DETERMINISTIC
BEGIN
    DECLARE discount_rate DECIMAL(5,2) DEFAULT 0.00;
    
    CASE customer_level
        WHEN 'GOLD' THEN SET discount_rate = 0.15;
        WHEN 'SILVER' THEN SET discount_rate = 0.10;
        WHEN 'BRONZE' THEN SET discount_rate = 0.05;
        ELSE SET discount_rate = 0.00;
    END CASE;
    
    IF order_amount > 1000 THEN
        SET discount_rate = discount_rate + 0.05;
    END IF;
    
    RETURN LEAST(discount_rate, 0.25); -- Max 25% discount
END //

DELIMITER ;

-- Use the function
SELECT 
    order_id,
    customer_level,
    order_amount,
    CalculateDiscount(customer_level, order_amount) as discount_rate,
    order_amount * CalculateDiscount(customer_level, order_amount) as discount_amount
FROM orders_with_customer_level;
```

---

## 🔔 Triggers

### Before and After Triggers

```sql
-- Audit trigger for tracking changes
CREATE TABLE audit_log (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(50),
    operation VARCHAR(10),
    old_values JSON,
    new_values JSON,
    changed_by VARCHAR(50),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

DELIMITER //

CREATE TRIGGER products_audit_trigger
AFTER UPDATE ON products
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (
        table_name, 
        operation, 
        old_values, 
        new_values, 
        changed_by
    ) VALUES (
        'products',
        'UPDATE',
        JSON_OBJECT('price', OLD.price, 'stock', OLD.stock_quantity),
        JSON_OBJECT('price', NEW.price, 'stock', NEW.stock_quantity),
        USER()
    );
END //

-- Trigger for automatic inventory updates
CREATE TRIGGER update_inventory_on_order
AFTER INSERT ON order_items
FOR EACH ROW
BEGIN
    UPDATE products 
    SET stock_quantity = stock_quantity - NEW.quantity
    WHERE product_id = NEW.product_id;
    
    -- Check if stock is low
    IF (SELECT stock_quantity FROM products WHERE product_id = NEW.product_id) < 10 THEN
        INSERT INTO low_stock_alerts (product_id, current_stock, alert_date)
        VALUES (NEW.product_id, 
                (SELECT stock_quantity FROM products WHERE product_id = NEW.product_id),
                NOW());
    END IF;
END //

DELIMITER ;
```

---

## 👁️ Views and Materialized Views

### Complex Views

```sql
-- Customer summary view
CREATE VIEW customer_summary AS
SELECT 
    c.customer_id,
    c.customer_name,
    c.email,
    COUNT(o.order_id) as total_orders,
    COALESCE(SUM(o.total_amount), 0) as total_spent,
    COALESCE(AVG(o.total_amount), 0) as avg_order_value,
    MAX(o.order_date) as last_order_date,
    CASE 
        WHEN COALESCE(SUM(o.total_amount), 0) > 5000 THEN 'GOLD'
        WHEN COALESCE(SUM(o.total_amount), 0) > 2000 THEN 'SILVER'
        WHEN COALESCE(SUM(o.total_amount), 0) > 500 THEN 'BRONZE'
        ELSE 'STANDARD'
    END as customer_tier
FROM customers c
LEFT JOIN orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.customer_name, c.email;

-- Product performance view
CREATE VIEW product_performance AS
SELECT 
    p.product_id,
    p.product_name,
    p.category_id,
    p.price,
    p.stock_quantity,
    COALESCE(SUM(oi.quantity), 0) as total_sold,
    COALESCE(SUM(oi.quantity * oi.unit_price), 0) as total_revenue,
    COUNT(DISTINCT o.customer_id) as unique_customers,
    AVG(oi.unit_price) as avg_selling_price
FROM products p
LEFT JOIN order_items oi ON p.product_id = oi.product_id
LEFT JOIN orders o ON oi.order_id = o.order_id
GROUP BY p.product_id, p.product_name, p.category_id, p.price, p.stock_quantity;
```

This intermediate SQL guide covers advanced joins, window functions, stored procedures, triggers, views, and optimization techniques essential for building robust database applications.
