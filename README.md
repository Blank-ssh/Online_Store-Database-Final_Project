# **Online Store database**

This project is an implementation of an online store database designed to manage products, customers, orders, shipping information, and payments. 

## **My database's structure**

The database consists of seven main tables :

1. **Customers** : Stores customer information.
2. **Categories** : Groups products by type (electronics, books, sport and others).
3. **Products** : Stores items available for purchase.
4. **ShippingAddresses** : Stores addresses for shipping orders.
5. **Orders** : Stores customer orders.
6. **OrderItems** : Links products with orders.
7. **Payments** : Stores payments made for orders.

### **Entity-Relationship Diagram (ERD)**

In this database there is :
- **Categories** and **Products** have a **one-to-many (1:N)** relationship.
- **Customers** and **Orders** have a **one-to-many (1:N)** relationship.
- **Orders** and **OrderItems** have a **one-to-many (1:N)** relationship.
- **Products** and **OrderItems** have a **one-to-many (1:N)** relationship.
- **Orders** and **Payments** have a **one-to-many (1:N)** relationship.
- **Customers** and **ShippingAddresses** have a **one-to-many (1:N)** relationship.

---

## **Schema**

### **1. Customers Table**
Stores customer data.

```sql
CREATE TABLE Customers (
    id SERIAL PRIMARY KEY,
    firstName VARCHAR(50) NOT NULL,
    lastName VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    phoneNumber VARCHAR(20),
    address VARCHAR(255),
    country VARCHAR(50)
);
```

### **2. Categories Table**
Stores product categories.

```sql
CREATE TABLE Categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) NOT NULL,
    description TEXT
);
```

### **3. Products Table**
Stores product details and links to categories.

```sql
CREATE TABLE Products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stockQuantity INT NOT NULL,
    categoryId INT REFERENCES Categories(id)
);
```

### **4. ShippingAddresses Table**
Stores shipping addresses for customers.

```sql
CREATE TABLE ShippingAddresses (
    id SERIAL PRIMARY KEY,
    addressLine1 VARCHAR(255) NOT NULL,
    addressLine2 VARCHAR(255),
    city VARCHAR(50),
    postalCode VARCHAR(20),
    country VARCHAR(50),
    customerId INT REFERENCES Customers(id)
);
```

### **5. Orders Table**
Stores customer orders.

```sql
CREATE TABLE Orders (
    id SERIAL PRIMARY KEY,
    orderDate DATE NOT NULL,
    shippingDate DATE,
    status VARCHAR(20) NOT NULL,
    totalAmount DECIMAL(10, 2),
    customerId INT REFERENCES Customers(id),
    shippingAddressId INT REFERENCES ShippingAddresses(id)
);
```

### **6. OrderItems Table**
Links products to orders with quantities and unit prices.

```sql
CREATE TABLE OrderItems (
    id SERIAL PRIMARY KEY,
    quantity INT NOT NULL,
    unitPrice DECIMAL(10, 2) NOT NULL,
    productId INT REFERENCES Products(id),
    orderId INT REFERENCES Orders(id)
);
```

### **7. Payments Table**
Stores payment details for orders.

```sql
CREATE TABLE Payments (
    id SERIAL PRIMARY KEY,
    paymentDate DATE NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    paymentMethod VARCHAR(50) NOT NULL,
    orderId INT REFERENCES Orders(id)
);
```

---

## **Relationships**

- **Categories (1) ↔ Products (N)** : Each category can have multiple products, but each product belongs to only one category.
- **Customers (1) ↔ Orders (N)** : Each customer can place many orders.
- **Orders (1) ↔ OrderItems (N)** : Each order can contain multiple products through the **OrderItems** table.
- **Products (1) ↔ OrderItems (N)** : Each product can be in many orders.
- **Orders (1) ↔ Payments (N)** : Each order can have multiple payments.
- **Customers (1) ↔ ShippingAddresses (N)** : Each customer can have multiple shipping addresses.

---

## **Queries for the database**

### 1. **Retrieve all products in a specific category**

This query retrieves all products belonging to a specific category "electronics" for example.

```sql
SELECT p.name AS ProductName, p.price 
FROM Products p
INNER JOIN Categories c ON p.categoryId = c.id
WHERE c.name = 'Electronics';
```

### 2. **Retrieve all orders for a specific customer**

This query retrieves all orders placed by a customer with a specific email address.

```sql
SELECT o.id AS OrderID, o.orderDate, o.totalAmount, o.status
FROM Orders o
INNER JOIN Customers c ON o.customerId = c.id
WHERE c.email = 'customer@example.com';
```

### 3. **Retrieve all products in a specific order**

This query lists all products, their quantities, and unit prices for a given order.

```sql
SELECT p.name AS ProductName, oi.quantity, oi.unitPrice
FROM OrderItems oi
INNER JOIN Products p ON oi.productId = p.id
WHERE oi.orderId = 1;
```

### 4. **Calculate total amount spent by a customer**

This query calculates the total amount a specific customer has spent across all their orders.

```sql
SELECT SUM(o.totalAmount) AS TotalSpent
FROM Orders o
INNER JOIN Customers c ON o.customerId = c.id
WHERE c.email = 'customer@example.com';
```

### 5. **List all payments for a specific order**

This query retrieves all payment details for a given order.

```sql
SELECT p.paymentDate, p.amount, p.paymentMethod
FROM Payments p
WHERE p.orderId = 1;
```

### **6. Find the products with low stock**

This query retrieves all products with a stock quantity below a certain threshold, meaning with less than 10 items in stock within this example.

```sql
SELECT name, stockQuantity 
FROM Products
WHERE stockQuantity < 10;
```

### **7. Calculate the total sales revenue by product**

This query calculates the total revenue generated by each product (quantity sold * unit price).

```sql
SELECT p.name, SUM(oi.quantity * oi.unitPrice) AS totalRevenue
FROM Products p
INNER JOIN OrderItems oi ON p.id = oi.productId
GROUP BY p.name
ORDER BY totalRevenue DESC;
```

### **8. List customers who have never placed an order**

This query lists all customers who haven't placed any orders yet.

```sql
SELECT c.firstName, c.lastName, c.email
FROM Customers c
LEFT JOIN Orders o ON c.id = o.customerId
WHERE o.id IS NULL;
```

### **9. Retrieve the most popular product**

This query finds the product that has been ordered the most based on quantity sold.

```sql
SELECT p.name, SUM(oi.quantity) AS totalSold
FROM Products p
INNER JOIN OrderItems oi ON p.id = oi.productId
GROUP BY p.name
ORDER BY totalSold DESC
LIMIT 1;
```

### **11. List orders that haven’t been shipped yet**

This query lists all orders that are still pending shipment.

```sql
SELECT o.id AS OrderID, o.orderDate, o.totalAmount, c.firstName, c.lastName
FROM Orders o
INNER JOIN Customers c ON o.customerId = c.id
WHERE o.status = 'Pending';
```

### **12. Calculate the average order value**

This query calculates the average amount spent per order.

```sql
SELECT AVG(totalAmount) AS averageOrderValue
FROM Orders;
```

### **13. Retrieve the total number of products in each category**

This query retrieves the number of products available in each category.

```sql
SELECT c.name AS CategoryName, COUNT(p.id) AS totalProducts
FROM Categories c
LEFT JOIN Products p ON c.id = p.categoryId
GROUP BY c.name;
```

---

## **How to use this database**

1. Set up a PostgreSQL instance or LibreOffice Base.
2. Create the database schema by running the provided SQL commands.
3. Insert sample data to populate the tables.
4. Use the sample SQL queries for retrieving and managing data.

---

## **License**

This project is free to use (I mean it's not much if you don't add anything to it but do feel free !).
