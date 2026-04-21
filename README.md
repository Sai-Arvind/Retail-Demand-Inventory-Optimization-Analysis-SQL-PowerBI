# Retail Store Movie Business Analysis

- **Industry:** Rental & Subscription Retail (Netflix DVD Era Simulation)

- Demand, Inventory & Customer Intelligence via Product & Store insights
- Demand is widely distributed across categories, with the top category contributing only ~7% and even the top 5 categories accounting for just ~35% of total demand, indicating no strong category dominance.




<img width="1280" height="720" alt="image" src="https://github.com/user-attachments/assets/c8759eac-a1d2-4239-afee-1dc7f47237b2" />

---

### ⚡ Executive Summary

This project simulates a **2005-era DVD Netflix rental business**, where:

📦 Inventory availability
👥 Customer behavior
⏳ Return patterns

directly influence **revenue, utilization, and operational efficiency**.


---

### 🚩 Business Problem

The business faces key operational challenges:

- **Inventory Mismatch**
Why do stores with higher inventory generate lower revenue?
- **Revenue Leakage**
How much demand is lost due to late returns and stock unavailability?
- **Customer Concentration**
Who are the high-value users driving consistent revenue?
- **Content Strategy**
Is revenue dependent on specific genres?

---

### 🎯 Objective

Build a scalable analytics framework to:

- Track Business Health KPIs
- Identify High-Value Customers
- Optimize Inventory Allocation
- Improve Operational Efficiency
- Detect Revenue Leakage Drivers


---

### 📊 Primary KPI Framework ⭐

| Category         | KPIs                                   | Insight                                 |
|----------------|----------------------------------------|------------------------------------------|
| Business Health | Total Revenue, Total Rentals, AOV, Active customers  | Overall financial scale.   |
| Demand           | Rentals by Category, Rating           |    What do customers want?               |
|Inventory	        |   Demand per Inventory, Revenue per Inventory	  |  Are we stocking efficiently?  |
|Customer	         |   CLV, Segments, Spend	                         | Who drives revenue?            |
|Store	            |   Revenue, Rentals, Efficiency                 | 	Where is execution failing?    |
|Operations	       |    Rental Duration, Late Returns	             |  Where do we lose efficiency?    |


---


### 🗄️ Dataset

Source: [MySQL Sakila Sample Database](https://github.com/jOOQ/sakila) 

Scale:
- **32,000+** rental & payment records
- Across 16+ relational tables
- Entities: Customers, Films, Inventory, Stores, Rentals


---


### ⚙️ SQL Deep-Dive Analysis

### Current Business Health

``` sql

-- Revenue
select sum(amount) as revenue from payment;

-- Rentals
select count(*) as total_rentals from rental;

-- Active Customers
select count(distinct customer_id) as active_customers from rental;

-- AOV
select avg(amount) as avg_order_value from payment;

```
### Demand Analysis Layer

``` sql

-- Category Demand

SELECT 
    c.name AS category,
    COUNT(r.rental_id) AS demand,
    ROUND(
        100 * COUNT(r.rental_id) / SUM(COUNT(r.rental_id)) OVER (), 
        2
    ) AS demand_pct
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film_category fc ON i.film_id = fc.film_id
JOIN category c ON fc.category_id = c.category_id
GROUP BY c.name
ORDER BY demand DESC;


-- Rating Demand

SELECT 
    f.rating,
    COUNT(r.rental_id) AS demand,
    ROUND(
        100 * COUNT(r.rental_id) / SUM(COUNT(r.rental_id)) OVER (), 
        2
    ) AS demand_pct
FROM rental r
JOIN inventory i ON r.inventory_id = i.inventory_id
JOIN film f ON i.film_id = f.film_id
GROUP BY f.rating
ORDER BY demand DESC;

> ✅ Insight
Demand is widely distributed across categories
However, revenue is concentrated, with top genres driving disproportionate value


```


### Inventory Efficiency Layer

``` sql

-- 1. Demand per Inventory

SELECT 
    f.title,
    COUNT(r.rental_id) AS demand,
    COUNT(DISTINCT i.inventory_id) AS inventory,
    ROUND(
        COUNT(r.rental_id) / COUNT(DISTINCT i.inventory_id), 
        2
    ) AS demand_per_inventory
FROM film f
JOIN inventory i ON f.film_id = i.film_id
LEFT JOIN rental r ON i.inventory_id = r.inventory_id
GROUP BY f.title
-- ORDER BY demand_per_inventory DESC
ORDER BY demand_per_inventory ASC
LIMIT 10;


> ✅ Insight
Top films → ~5 rentals per copy
Bottom films → ~2 rentals per copy
Indicates understocking of high-demand content and overstocking of low-demand content

```



### Customer Intelligence Layer 

``` SQL


-- 1. Customer Activity

SELECT 
    customer_id,
    COUNT(rental_id) AS total_rentals
FROM rental
GROUP BY customer_id
ORDER BY total_rentals DESC;


-- 2. Customer Segmentation

SELECT 
    CASE 
        WHEN rental_count >= 30 THEN 'High Value'
        WHEN rental_count BETWEEN 15 AND 29 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS customer_segment,
    COUNT(*) AS customer_count
FROM (
    SELECT 
        customer_id,
        COUNT(rental_id) AS rental_count
    FROM rental
    GROUP BY customer_id
) t
GROUP BY customer_segment;



-- 3. Revenue Contribution per each customer 


SELECT 
    c.customer_id,
    SUM(p.amount) AS total_spent
FROM customer c
JOIN rental r ON c.customer_id = r.customer_id
JOIN payment p ON r.rental_id = p.rental_id
GROUP BY c.customer_id
ORDER BY total_spent DESC;


> ✅ Insight
- Strong power-user base
- High retention in mid-tier customers
- Stable top spenders → predictable revenue



```


### STORE LAYER


``` sql

-- 1. Store Revenue

SELECT 
    s.store_id,
    COUNT(r.rental_id) AS total_rentals,
    SUM(p.amount) AS revenue
FROM store s
JOIN staff st ON s.store_id = st.store_id
JOIN rental r ON st.staff_id = r.staff_id
JOIN payment p ON r.rental_id = p.rental_id
GROUP BY s.store_id;

-- Which store drives more rentals & revenue




-- 2. Store Efficiency (Inventory vs Demand)

SELECT 
    s.store_id,
    COUNT(r.rental_id) AS demand,
    COUNT(DISTINCT i.inventory_id) AS inventory,
    ROUND(
        COUNT(r.rental_id) / COUNT(DISTINCT i.inventory_id), 
        2
    ) AS demand_per_inventory
FROM store s
JOIN inventory i ON s.store_id = i.store_id
LEFT JOIN rental r ON i.inventory_id = r.inventory_id
GROUP BY s.store_id;

> ✅ Insight
- Store with more inventory ≠ higher revenue
- Clear inventory allocation inefficiency




```


### Time & Operations


``` sql 

-- 1. How does demand/revenue change over time?

-- Demand Trend
SELECT 
    DATE_FORMAT(r.rental_date, '%Y-%m') AS month,
    COUNT(*) AS demand
FROM rental r
GROUP BY month
ORDER BY month;


-- 2. rev Trend

SELECT 
    DATE_FORMAT(p.payment_date, '%Y-%m') AS month,
    SUM(p.amount) AS revenue
FROM payment p
GROUP BY month
ORDER BY month;


```



---


### ER Diagram

<img width="799" height="521" alt="Sakila - ERD" src="https://github.com/user-attachments/assets/4ffc3c2f-1090-4a1a-88f1-1b94ca97054d" />



---


### ✨ Power BI Implementation

``` Powerbi
-- DAX Measures

1. Average Rental Duration: AVG_Duration = AVERAGE(Rental[Duration])

2. Customer Tiering: 

Loyalty_Tier = SWITCH(TRUE(),
[Rental_Count] >= 40, "Elite VIP",
[Rental_Count] >= 20, "Preferred",
"Occasional")

Store Revenue Gap: Revenue_Gap = [Store 2 Revenue] - [Store 1 Revenue]


```

---



### 📈 Business Performance Snapshot
- 💰 Revenue: **$67,416**
- 👥 Customers: **599**
- 📦 Inventory: **4,581 units**
- 🎬 Films: **~1,000 titles**
- ⏱ Avg Rental Duration: **~5 days**

---


### 📊 Key Insights

## 👥 Customer
- Stable high-value users
- Strong mid-tier retention
## 🎬 Product
- Demand distributed
- Revenue concentrated in top genres
## ⚙️ Inventory
- High variance in utilization (2x–5x)
- Overstock + understock coexist
## 🏬 Store
- Inventory ≠ performance
- Poor allocation strategy
## ⏳ Operations
- Late returns reduce availability
- Impacts peak demand fulfillment

--- 

### 💡 Business Recommendations

**1. Inventory Rebalancing**

Shift stock from low-performing (~2 rentals/copy)
to high-performing (~5 rentals/copy) titles

→ Potential **2–2.5x utilization improvement**

**2. Late Return Reduction**

Introduce:

- Early return incentives
- Loyalty rewards

→ Improves inventory availability

**3. Genre Strategy**

- Expand high-revenue genres (Sports, Sci-Fi)
- Bundle with low-performing categories

→ Increase AOV


**4. Store Optimization**

Reallocate inventory toward:

- High-demand stores
- High-footfall regions




--- 


```
📁 Repository Structure
Movie-Rental-Inventory-Analytics-SQL
│
├── Data
|   ├── Introduction 
│   └── rental_records
│
│
├── SQL_Queries
│   ├── 01_Business Health
│   ├── 02_Demand Analysis 
│   ├── 03_Inventory Gap
│   ├── 04_Customer_Behaviour
│   └── 05_Store & Operations
|
├── ER_Diagram
│   └── DVD_Store_ERD.png
│
├── Visuals
|   ├── Metrics
│   └── rental_dashboard.png
│
├── Power BI Analysis
│   ├── Modeling
│   └── Dax Measures 
│
├── README.md
└── .gitignore

```
---

### ⚙️ Tech Stack

| Tools      | Techniques ⭐                                               |
|-----------|----------------------------------------------------------|
| Advance Excel  | Pivot Tables, Formulas, Power Query, Cleaning, ETL         |
| MySQL       | Joins, Aggregations, Window Function, Ctes |
| Power BI  | Data modeling, Star Schema, DAX measures, dashboards, charts & visualization |

---

![movie1](https://github.com/user-attachments/assets/2fb575c5-2957-41b5-a2b9-337ee91fc36d)


### 👤 About Me

**A Sai Arvind**  

📧 Email: saiarvind5081@gmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/saiarvindofficial/  
💻 GitHub: https://github.com/Sai-Arvind  

⭐ If you found this project useful, consider giving it a star.

