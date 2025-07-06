# SQL_task8

--Using CREATE PROCEDURE and CREATE FUNCTION
--Using parameters and conditional logic



DROP TABLE IF EXISTS Orders;
DROP TABLE IF EXISTS Customers;

CREATE TABLE Customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    city VARCHAR(50)
);

CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    amount DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES Customers(customer_id)
);

INSERT INTO Customers (customer_id, name, city) VALUES
(1, 'Alice', 'New York'),
(2, 'Bob', 'Chicago'),
(3, 'Charlie', 'Los Angeles'),
(4, 'David', 'Houston');

INSERT INTO Orders (order_id, customer_id, order_date, amount) VALUES
(101, 1, '2024-06-01', 250.00),
(102, 1, '2024-06-05', 180.00),
(103, 2, '2024-06-10', 340.00);

SELECT c.name AS customer_name, o.order_id, o.amount
FROM Customers c
INNER JOIN Orders o ON c.customer_id = o.customer_id;

SELECT c.name AS customer_name, o.order_id, o.amount
FROM Customers c
LEFT JOIN Orders o ON c.customer_id = o.customer_id;

SELECT c.name AS customer_name, o.order_id, o.amount
FROM Customers c
RIGHT JOIN Orders o ON c.customer_id = o.customer_id;

SELECT c.name AS customer_name, o.order_id, o.amount
FROM Customers c
FULL JOIN Orders o ON c.customer_id = o.customer_id;

SELECT name, (SELECT COUNT(*) FROM Customers) AS total_customers
FROM Customers;

SELECT c.name, (SELECT SUM(o.amount) FROM Orders o WHERE o.customer_id = c.customer_id) AS total_order_amount
FROM Customers c;

SELECT name
FROM Customers
WHERE customer_id IN (SELECT customer_id FROM Orders);

SELECT c.name
FROM Customers c
WHERE EXISTS (
    SELECT 1 FROM Orders o WHERE o.customer_id = c.customer_id
);

SELECT name
FROM Customers
WHERE (
    SELECT SUM(amount)
    FROM Orders
    WHERE Orders.customer_id = Customers.customer_id
) = 430;

CREATE VIEW CustomerOrderSummary AS
SELECT 
    c.customer_id,
    c.name AS customer_name,
    c.city,
    COALESCE(SUM(o.amount), 0) AS total_order_amount
FROM Customers c
LEFT JOIN Orders o ON c.customer_id = o.customer_id
GROUP BY c.customer_id, c.name, c.city;

CREATE VIEW PublicCustomerOrders AS
SELECT 
    c.name AS customer_name,
    o.order_id,
    o.order_date,
    o.amount
FROM Customers c
INNER JOIN Orders o ON c.customer_id = o.customer_id;

DELIMITER //

CREATE PROCEDURE GetCustomerOrders(IN p_customer_id INT)
BEGIN
    SELECT o.order_id, o.order_date, o.amount
    FROM Orders o
    WHERE o.customer_id = p_customer_id;
END //

CREATE FUNCTION GetCustomerTotalOrders(p_customer_id INT)
RETURNS DECIMAL(10, 2)
DETERMINISTIC
BEGIN
    DECLARE total DECIMAL(10, 2);
    SELECT COALESCE(SUM(amount), 0) INTO total
    FROM Orders
    WHERE customer_id = p_customer_id;
    RETURN total;
END //

CREATE PROCEDURE ConditionalOrderSummary(IN p_city VARCHAR(50))
BEGIN
    IF p_city = 'All' THEN
        SELECT c.name, COALESCE(SUM(o.amount), 0) AS total_order_amount
        FROM Customers c
        LEFT JOIN Orders o ON c.customer_id = o.customer_id
        GROUP BY c.name;
    ELSE
        SELECT c.name, COALESCE(SUM(o.amount), 0) AS total_order_amount
        FROM Customers c
        LEFT JOIN Orders o ON c.customer_id = o.customer_id
        WHERE c.city = p_city
        GROUP BY c.name;
    END IF;
END //

DELIMITER ;

CALL GetCustomerOrders(1);
SELECT GetCustomerTotalOrders(1);
CALL ConditionalOrderSummary('Chicago');
CALL ConditionalOrderSummary('All');
