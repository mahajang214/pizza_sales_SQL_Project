# Pizza Sales – SQL Analysis (Solution)

Pizza Hut is a multinational chain of pizza restaurants and delivery outlets known for its extensive menu. This document analyzes the pizza sales data set on different parameters using SQL (PostgreSQL syntax).

## Data Set

The data set contains 4 tables.

### 1. `orders`
| Column | Description |
|---|---|
| `order_id` | Total orders placed at Pizza Hut (bill no) |
| `date` | Date on which the pizza is ordered |
| `time` | Timestamp when the order is placed |

### 2. `order_details`
| Column | Description |
|---|---|
| `order_details_id` | ID for each pizza that is ordered |
| `order_id` | Order (bill) number |
| `pizza_id` | Pizza ID with size that is ordered |
| `quantity` | Count of that particular pizza ordered |

### 3. `pizzas`
| Column | Description |
|---|---|
| `pizza_id` | Pizza ID with size, for all available pizzas |
| `pizza_type_id` | Pizza type ID for all available pizzas |
| `size` | Size of the pizza |
| `price` | Price of the pizza depending on its size |

### 4. `pizza_types`
| Column | Description |
|---|---|
| `pizza_type_id` | Pizza type ID for all available pizzas |
| `name` | Detailed name of the pizza |
| `category` | Category of the pizza |

---

# Basic

## 1. Retrieve the total number of orders placed

```sql
SELECT COUNT(order_id) FROM orders;
```

**Output**

| count |
|---|
| 21350 |

---

## 2. Calculate the total revenue generated from pizza sales

```sql
SELECT SUM(price) FROM pizzas;
```

**Output**

| sum |
|---|
| 1578.30 |

> ⚠️ **Note:** This query sums the *price list* of the `pizzas` table (one row per pizza/size), not actual sales. Actual revenue must join `order_details` and multiply by quantity (see the corrected query below). The result of that calculation is **817,860.05**, which is also the value returned in Advanced Q2.

**Corrected query (revenue from actual sales)**

```sql
SELECT ROUND(SUM(od.quantity * p.price)::numeric, 2) AS total_revenue
FROM order_details AS od
JOIN pizzas AS p
  ON p.pizza_id = od.pizza_id;
```

Expected result: `817860.05` (matches the total in Advanced Q2).

---

## 3. Identify the highest-priced pizza

```sql
SELECT MAX(price) FROM pizzas;
```

**Output**

| max |
|---|
| 35.95 |

---

## 4. Identify the most common pizza size ordered

```sql
SELECT
    P.SIZE
FROM
    PIZZAS P
WHERE
    P.PIZZA_ID = (
        SELECT
            PIZZA_ID
        FROM
            (
                SELECT
                    PIZZA_ID,
                    COUNT(QUANTITY) AS QTY
                FROM
                    ORDER_DETAILS
                GROUP BY
                    PIZZA_ID
                ORDER BY
                    QTY DESC
                LIMIT
                    1
            ) AS T
    );
```

**Output**

| size |
|---|
| S |

> ⚠️ **Note:** This finds the size of the single most-ordered `pizza_id`, which is not necessarily the most common size overall. A size-level aggregation is more accurate (not executed here, so no output is claimed):

```sql
SELECT p.size, SUM(od.quantity) AS total_quantity
FROM order_details AS od
JOIN pizzas AS p
  ON p.pizza_id = od.pizza_id
GROUP BY p.size
ORDER BY total_quantity DESC
LIMIT 1;
```

---

## 5. List the top 5 most ordered pizza types along with their quantities

```sql
SELECT pizza_id, COUNT(quantity) AS qty
FROM order_details
GROUP BY pizza_id
ORDER BY qty DESC
LIMIT 5;
```

**Output**

| Type | Quantity |
|---|---|
| big_meat_s | 1811 |
| thai_ckn_l | 1365 |
| five_cheese_l | 1359 |
| four_cheese_l | 1273 |
| classic_dlx_m | 1159 |

> **Note:** `COUNT(quantity)` counts order lines; `SUM(quantity)` gives the total pizzas ordered. Grouping is by `pizza_id` (type + size), which is why the results are shown with size suffixes.

---

# Intermediate

## 6. Join the necessary tables to find the total quantity of each pizza category ordered

```sql
SELECT
    pt.category AS Category,
    COUNT(od.quantity) AS Total_Quantity
FROM order_details AS od
INNER JOIN pizzas AS p
    ON p.pizza_id = od.pizza_id
INNER JOIN pizza_types AS pt
    ON pt.pizza_type_id = p.pizza_type_id
GROUP BY Category
ORDER BY Total_Quantity DESC;
```

**Output**

| Category | Total_Quantity |
|---|---|
| Classic | 14579 |
| Supreme | 11777 |
| Veggie | 11449 |
| Chicken | 10815 |

---

## 7. Determine the distribution of orders by hour of the day

```sql
SELECT DATE_PART('hours', time) AS hours, COUNT(order_id)
FROM orders
GROUP BY hours
ORDER BY hours ASC;
```

**Output**

| Hour | Orders |
|---|---|
| 9 | 1 |
| 10 | 8 |
| 11 | 1231 |
| 12 | 2520 |
| 13 | 2455 |
| 14 | 1472 |
| 15 | 1468 |
| 16 | 1920 |
| 17 | 2336 |
| 18 | 2399 |
| 19 | 2009 |
| 20 | 1642 |
| 21 | 1198 |
| 22 | 663 |
| 23 | 28 |

**Insight:** Peak ordering is during lunch (12–13 h) and dinner (17–19 h).

---

## 8. Join relevant tables to find the category-wise distribution of pizzas

```sql
SELECT pt.name, pt.category
FROM pizza_types pt
GROUP BY pt.name, pt.category;
```

**Output**

| name | category |
|---|---|
| The Napolitana Pizza | Classic |
| The Hawaiian Pizza | Classic |
| The Green Garden Pizza | Veggie |
| The Thai Chicken Pizza | Chicken |
| The Calabrese Pizza | Supreme |
| The Four Cheese Pizza | Veggie |
| The Mexicana Pizza | Veggie |
| The Spinach Pesto Pizza | Veggie |
| The Pepperoni Pizza | Classic |
| The Soppressata Pizza | Supreme |
| The Big Meat Pizza | Classic |
| The Brie Carre Pizza | Supreme |
| The Prosciutto and Arugula Pizza | Supreme |
| The Sicilian Pizza | Supreme |
| The Mediterranean Pizza | Veggie |
| The Spinach Supreme Pizza | Supreme |
| The Italian Supreme Pizza | Supreme |
| The Chicken Pesto Pizza | Chicken |
| The Pepper Salami Pizza | Supreme |
| The Spinach and Feta Pizza | Veggie |
| The Barbecue Chicken Pizza | Chicken |
| The Five Cheese Pizza | Veggie |
| The Vegetables + Vegetables Pizza | Veggie |
| The Classic Deluxe Pizza | Classic |
| The Pepperoni, Mushroom, and Peppers Pizza | Classic |
| The Italian Vegetables Pizza | Veggie |
| The Chicken Alfredo Pizza | Chicken |
| The California Chicken Pizza | Chicken |
| The Italian Capocollo Pizza | Classic |
| The Greek Pizza | Classic |
| The Spicy Italian Pizza | Supreme |
| The Southwest Chicken Pizza | Chicken |

> **Note:** To get a count of pizza types per category instead, use:
> `SELECT category, COUNT(*) FROM pizza_types GROUP BY category;`

---

## 9. Group the orders by date and calculate the average number of pizzas ordered per day

```sql
SELECT DATE_PART('day', date) AS day, ROUND(AVG(order_id), 2)
FROM orders
GROUP BY day
ORDER BY day ASC;
```

**Output**

| day | round |
|---|---|
| 1 | 9755.62 |
| 2 | 9672.30 |
| 3 | 9904.87 |
| 4 | 10312.79 |
| 5 | 9686.82 |
| 6 | 10079.72 |
| 7 | 10352.64 |
| 8 | 9894.48 |
| 9 | 10601.36 |
| 10 | 10537.12 |
| 11 | 10500.22 |
| 12 | 10181.19 |
| 13 | 10388.52 |
| 14 | 10797.68 |
| 15 | 10978.28 |
| 16 | 10971.73 |
| 17 | 10693.99 |
| 18 | 11044.03 |
| 19 | 10700.93 |
| 20 | 10638.34 |
| 21 | 11256.51 |
| 22 | 11232.28 |
| 23 | 11158.25 |
| 24 | 10618.68 |
| 25 | 9894.03 |
| 26 | 11272.23 |
| 27 | 11405.50 |
| 28 | 11345.70 |
| 29 | 11751.13 |
| 30 | 11823.49 |
| 31 | 12375.51 |

> ⚠️ **Note:** The values above are the average of `order_id` (a bill number), which is not a count of pizzas. To get the average pizzas ordered per day, sum the quantity per date first, then average (not executed here, so no output is claimed):

```sql
SELECT ROUND(AVG(daily_qty), 0) AS avg_pizzas_per_day
FROM (
    SELECT o.date, SUM(od.quantity) AS daily_qty
    FROM orders AS o
    JOIN order_details AS od
      ON od.order_id = o.order_id
    GROUP BY o.date
) AS t;
```

---

## 10. Determine the top 3 most ordered pizza types based on revenue

```sql
SELECT
    pt.name,
    SUM(p.price * od.quantity) AS revenue
FROM order_details AS od
INNER JOIN pizzas AS p
    ON p.pizza_id = od.pizza_id
INNER JOIN pizza_types AS pt
    ON pt.pizza_type_id = p.pizza_type_id
GROUP BY pt.name
ORDER BY revenue DESC
LIMIT 3;
```

**Output**

| name | revenue |
|---|---|
| The Thai Chicken Pizza | 43434.25 |
| The Barbecue Chicken Pizza | 42768.00 |
| The California Chicken Pizza | 41409.50 |

---

# Advanced

## 11. Calculate the percentage contribution of each pizza type to total revenue

```sql
SELECT pt.pizza_type_id, pt.name,
(
    ROUND(
        SUM(od.quantity * p.price) * 100 /
        (
            SELECT SUM(p2.price * od2.quantity)
            FROM order_details AS od2
            JOIN pizzas AS p2
              ON p2.pizza_id = od2.pizza_id
        ), 2)
)::text || '%' AS revenue
FROM order_details AS od
JOIN orders AS o
  ON od.order_id = o.order_id
JOIN pizzas AS p
  ON p.pizza_id = od.pizza_id
JOIN pizza_types AS pt
  ON pt.pizza_type_id = p.pizza_type_id
GROUP BY pt.pizza_type_id, pt.name
ORDER BY revenue DESC;
```

**Output**

| pizza_type_id | name | revenue |
|---|---|---|
| thai_ckn | The Thai Chicken Pizza | 5.31% |
| bbq_ckn | The Barbecue Chicken Pizza | 5.23% |
| cali_ckn | The California Chicken Pizza | 5.06% |
| classic_dlx | The Classic Deluxe Pizza | 4.67% |
| spicy_ital | The Spicy Italian Pizza | 4.26% |
| southw_ckn | The Southwest Chicken Pizza | 4.24% |
| ital_supr | The Italian Supreme Pizza | 4.09% |
| hawaiian | The Hawaiian Pizza | 3.95% |
| four_cheese | The Four Cheese Pizza | 3.95% |
| sicilian | The Sicilian Pizza | 3.78% |
| pepperoni | The Pepperoni Pizza | 3.69% |
| the_greek | The Greek Pizza | 3.48% |
| mexicana | The Mexicana Pizza | 3.27% |
| five_cheese | The Five Cheese Pizza | 3.19% |
| peppr_salami | The Pepper Salami Pizza | 3.12% |
| ital_cpcllo | The Italian Capocollo Pizza | 3.07% |
| veggie_veg | The Vegetables + Vegetables Pizza | 2.98% |
| prsc_argla | The Prosciutto and Arugula Pizza | 2.96% |
| napolitana | The Napolitana Pizza | 2.95% |
| spinach_fet | The Spinach and Feta Pizza | 2.85% |
| big_meat | The Big Meat Pizza | 2.81% |
| pep_msh_pep | The Pepperoni, Mushroom, and Peppers Pizza | 2.30% |
| ckn_alfredo | The Chicken Alfredo Pizza | 2.07% |
| ckn_pesto | The Chicken Pesto Pizza | 2.04% |
| soppressata | The Soppressata Pizza | 2.01% |
| ital_veggie | The Italian Vegetables Pizza | 1.96% |
| calabrese | The Calabrese Pizza | 1.95% |
| spin_pesto | The Spinach Pesto Pizza | 1.91% |
| mediterraneo | The Mediterranean Pizza | 1.88% |
| spinach_supr | The Spinach Supreme Pizza | 1.87% |
| green_garden | The Green Garden Pizza | 1.71% |
| brie_carre | The Brie Carre Pizza | 1.42% |

> **Note:** Because `revenue` is cast to text, `ORDER BY revenue` sorts alphabetically rather than numerically. It happens to work here because all values have the same number of integer digits, but it is safer to order by the numeric expression before casting.

---

## 12. Analyze the cumulative revenue generated over time

```sql
WITH cte AS (
    SELECT SUM(p2.price * od2.quantity)
    FROM order_details AS od2
    JOIN pizzas AS p2
      ON p2.pizza_id = od2.pizza_id
)
SELECT SUM(cte) FROM cte;
```

**Output**

| sum |
|---|
| 817860.05 |

> ⚠️ **Note:** This query returns the **total** revenue, i.e. the final value of the cumulative series. To see revenue accumulating over time, use a window function (not executed here, so no output is claimed):

```sql
SELECT order_date,
       ROUND(SUM(daily_revenue) OVER (ORDER BY order_date)::numeric, 2) AS cumulative_revenue
FROM (
    SELECT o.date AS order_date,
           SUM(od.quantity * p.price) AS daily_revenue
    FROM orders AS o
    JOIN order_details AS od ON od.order_id = o.order_id
    JOIN pizzas AS p         ON p.pizza_id = od.pizza_id
    GROUP BY o.date
) AS t
ORDER BY order_date;
```

The last row of this result should equal **817860.05**.

---

## 13. Determine the top 3 most ordered pizza types based on revenue for each pizza category

```sql
SELECT pt.pizza_type_id, pt.name,
(
    ROUND(
        SUM(od.quantity * p.price) * 100 /
        (
            SELECT SUM(p2.price * od2.quantity)
            FROM order_details AS od2
            JOIN pizzas AS p2
              ON p2.pizza_id = od2.pizza_id
        ), 2)
) AS revenue
FROM order_details AS od
JOIN orders AS o
  ON od.order_id = o.order_id
JOIN pizzas AS p
  ON p.pizza_id = od.pizza_id
JOIN pizza_types AS pt
  ON pt.pizza_type_id = p.pizza_type_id
GROUP BY pt.pizza_type_id, pt.name
ORDER BY revenue DESC
LIMIT 3;
```

**Output**

| name | revenue (% of total) |
|---|---|
| The Thai Chicken Pizza | 5.31 |
| The Barbecue Chicken Pizza | 5.23 |
| The California Chicken Pizza | 5.06 |

> ⚠️ **Note:** This query returns the overall top 3 (all from the Chicken category), not the top 3 within **each** category. A per-category version using `RANK()` (not executed here, so no output is claimed):

```sql
SELECT category, name, revenue
FROM (
    SELECT pt.category,
           pt.name,
           SUM(od.quantity * p.price) AS revenue,
           RANK() OVER (
               PARTITION BY pt.category
               ORDER BY SUM(od.quantity * p.price) DESC
           ) AS rnk
    FROM order_details AS od
    JOIN pizzas AS p       ON p.pizza_id = od.pizza_id
    JOIN pizza_types AS pt ON pt.pizza_type_id = p.pizza_type_id
    GROUP BY pt.category, pt.name
) AS ranked
WHERE rnk <= 3
ORDER BY category, revenue DESC;
```

---

# Key Takeaways

- **Total orders:** 21,350
- **Total revenue (from actual sales):** 817,860.05
- **Highest-priced pizza:** 35.95
- **Best-selling category:** Classic (14,579 order lines), followed by Supreme, Veggie and Chicken
- **Top revenue pizzas:** Thai Chicken (5.31%), Barbecue Chicken (5.23%), California Chicken (5.06%)
- **Peak ordering hours:** 12:00–13:00 (lunch) and 17:00–19:00 (dinner)
