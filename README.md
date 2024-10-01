# MakeItEasy - Database Setup for MySQL Server

## Project Overview
This project involves deploying databases for the **MakeItEasy** platform, designed to manage customer service, order processing, and seller activities. The databases are hosted on **Google Cloud MySQL** and serve as the backbone for three primary services:

- **Customer Service**
- **Order Service**
- **Seller Service**

Each service interacts with its respective database. This document outlines the schemas and setup required for each service.

## Prerequisites
Before setting up the databases, ensure you have:

- Access to **Google Cloud MySQL** with an active instance.
- A **MySQL client** or compatible database management tool to execute SQL scripts.
- Correct **user permissions** for database creation and modification in your Google Cloud MySQL instance.

## Database Schemas




### 1. Customer Service Database
The **Customer Service** database manages customer-related data, including registration details and support ticket tracking.

#### Schema
```sql
-- Switch to the Customer_Service database
USE Customer_Service;

-- Customer Table
CREATE TABLE Customer (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    address TEXT NOT NULL,
    phone VARCHAR(15),
    password_hash VARCHAR(255) NOT NULL
);

-- SupportTicket Table
CREATE TABLE SupportTicket (
    ticket_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT,
    issue TEXT NOT NULL,
    status ENUM('Open', 'In Progress', 'Closed') DEFAULT 'Open',
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (customer_id) REFERENCES Customer(customer_id) ON DELETE CASCADE
);
```



### 2. Order Service Database
The **Order Service** database is responsible for order management, payment processing, and cart functionality.

#### Schema

```sql
-- Switch to the Order_Service database
USE Order_Service;

-- Order Table
CREATE TABLE `Order` (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT,
    total_amount DECIMAL(10, 2) NOT NULL,
    status ENUM('Pending', 'Shipped', 'Delivered', 'Cancelled') DEFAULT 'Pending',
    created_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    tracking_number VARCHAR(255),
    FOREIGN KEY (customer_id) REFERENCES Customer_Service.Customer(customer_id) ON DELETE CASCADE
);

-- OrderItem Table
CREATE TABLE OrderItem (
    item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    product_id INT,
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES `Order`(order_id) ON DELETE CASCADE
);

-- Payment Table
CREATE TABLE Payment (
    payment_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    payment_method ENUM('Credit Card', 'PayPal') NOT NULL,
    amount DECIMAL(10, 2) NOT NULL,
    status ENUM('Pending', 'Completed', 'Refunded') DEFAULT 'Pending',
    transaction_id VARCHAR(255),
    FOREIGN KEY (order_id) REFERENCES `Order`(order_id) ON DELETE CASCADE
);

-- Cart Table
CREATE TABLE Cart (
    cart_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT,
    total_price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (customer_id) REFERENCES Customer_Service.Customer(customer_id) ON DELETE CASCADE
);

-- CartItem Table
CREATE TABLE CartItem (
    cart_item_id INT PRIMARY KEY AUTO_INCREMENT,
    cart_id INT,
    product_id INT,
    quantity INT NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (cart_id) REFERENCES Cart(cart_id) ON DELETE CASCADE
);
```


### 3. Seller Service Database
The **Seller Service** database allows sellers to manage products, track orders, and maintain order history.

#### Schema

```sql
-- Switch to the Seller_Service database
USE Seller_Service;

-- Seller Table
CREATE TABLE Seller (
    seller_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL
);

-- Product Table
CREATE TABLE Product (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    seller_id INT,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10, 2) NOT NULL,
    stock INT NOT NULL,
    description TEXT,
    category VARCHAR(255),
    FOREIGN KEY (seller_id) REFERENCES Seller(seller_id) ON DELETE CASCADE
);

-- SellerOrderHistory Table
CREATE TABLE SellerOrderHistory (
    seller_order_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    seller_id INT,
    product_id INT,
    quantity INT,
    FOREIGN KEY (order_id) REFERENCES Order_Service.`Order`(order_id) ON DELETE CASCADE,
    FOREIGN KEY (seller_id) REFERENCES Seller(seller_id) ON DELETE CASCADE,
    FOREIGN KEY (product_id) REFERENCES Product(product_id) ON DELETE CASCADE
);
```



## Steps to Set Up the Databases

### 1. Create the Schemas (Databases)

If the schemas (**Customer_Service**, **Order_Service**, **Seller_Service**) do not exist, create them:

```sql
CREATE DATABASE Customer_Service;
CREATE DATABASE Order_Service;
CREATE DATABASE Seller_Service;
```

### 2. Execute the SQL Scripts

- Connect to your Google Cloud MySQL instance using a MySQL client.
- Run the SQL scripts provided above for each service, ensuring you're using the correct schema with the `USE` statement before executing each script.

### 3. Verify the Setup

After executing the scripts, verify that the tables have been created:

```sql
USE Customer_Service;
SHOW TABLES;

USE Order_Service;
SHOW TABLES;

USE Seller_Service;
SHOW TABLES;
```

### Ensure that all foreign key constraints are properly established.

## Notes on Cross-Schema References

- **Cross-Schema References:** Some tables reference tables in other schemas (databases). For example, the `Order` table in the **Order_Service** schema references the `Customer` table in the **Customer_Service** schema.
- **User Permissions:** Ensure that your MySQL user has the necessary permissions to create foreign keys that reference tables across different schemas.
- **Foreign Key Constraints:** The `FOREIGN KEY` constraints ensure referential integrity between tables. For example, an order must be associated with an existing customer.

## Conclusion

This document outlines the database setup for the **MakeItEasy** platform in a Google Cloud MySQL environment. By following the steps provided, you'll establish the foundational databases necessary for the **Customer Service**, **Order Service**, and **Seller Service** components of the project.

Ensure that:

- The correct schema is used for each service.
- Foreign key relationships are properly established for data integrity across services.
- User permissions are correctly configured to allow cross-schema operations.


