# Pizza Sales SQL Project

This project is a SQL-based analysis of pizza sales data. It uses real-world-style sales data to answer business questions such as total revenue, most popular pizza types, category-wise performance, and order trends.

## Project Objective

The goal of this project is to explore and analyze pizza sales using SQL and derive meaningful insights from the dataset. The analysis helps understand:

- Total revenue generated
- Best-selling pizzas
- Most popular pizza categories
- Sales trends by day and time
- Revenue contribution by pizza size and type

## Dataset

This repository contains the following CSV files:

- `orders.csv` – order-level details
- `order_details.csv` – pizza items included in each order
- `pizzas.csv` – pizza names, sizes, and prices
- `pizza_types.csv` – pizza categories and pizza type names

## Table Structure

### 1. orders
| Column | Description |
|--------|-------------|
| order_id | Unique order identifier |
| date | Order date |
| time | Order time |

### 2. order_details
| Column | Description |
|--------|-------------|
| order_id | Order ID |
| pizza_id | Pizza identifier |
| quantity | Quantity ordered |

### 3. pizzas
| Column | Description |
|--------|-------------|
| pizza_id | Unique pizza identifier |
| pizza_type_id | Pizza category/type reference |
| size | Pizza size (S, M, L, XL, XXL) |
| price | Price of the pizza |

### 4. pizza_types
| Column | Description |
|--------|-------------|
| pizza_type_id | Unique pizza type ID |
| name | Name of the pizza |
| category | Category such as Chicken, Classic, Supreme, Veggie |

## Database Setup

Use PostgreSQL and create the following tables:

```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    date DATE,
    time TIME
);

CREATE TABLE order_details (
    order_id INT,
    pizza_id VARCHAR(50),
    quantity INT
);

CREATE TABLE pizzas (
    pizza_id VARCHAR(50) PRIMARY KEY,
    pizza_type_id VARCHAR(50),
    size VARCHAR(10),
    price DECIMAL(10,2)
);

CREATE TABLE pizza_types (
    pizza_type_id VARCHAR(50) PRIMARY KEY,
    name VARCHAR(200),
    category VARCHAR(50)
);
