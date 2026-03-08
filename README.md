# 🎞️ Product Demand & Inventory Analysis for a Multi-Store Rental Business

![movie1](https://github.com/user-attachments/assets/2fb575c5-2957-41b5-a2b9-337ee91fc36d)

**Project Overview:**
This project analyzes rental operations and inventory demand for a multi-store DVD rental company using SQL.

Using the Sakila Sample Database, created by MySQL, the analysis evaluates customer behavior, product demand, inventory utilization, and store-level revenue performance.

The goal is to demonstrate how SQL can be used to generate business insights that support inventory planning, demand analysis, and revenue monitoring.

### 🧩 Business Problem

A multi-store rental business needs visibility into operational performance, including:

• Which movie genres generate the most revenue
• Which customers drive the most rentals
• How often customers rent movies
• How efficiently inventory is utilized
• Whether rental durations impact inventory availability
• How revenue is distributed across stores

Without structured analysis, these operational insights remain hidden within multiple relational tables.

### 🎯Project Objective

Use SQL to analyze rental transactions and uncover insights about:

• Revenue contribution by movie genre
• Customer lifetime value
• Rental frequency and customer engagement
• Late return patterns affecting inventory availability
• Inventory utilization across films
• Store-level revenue performance

## 📊 Dataset

The project uses the Sakila Sample Database, a relational dataset designed to simulate a DVD rental business.

Dataset Source: [MySQL Sakila Sample Database](https://github.com/jOOQ/sakila) 

Approximate dataset size:

• 50,000+ rental and payment records
• Handling 16+ relational tables representing customers, inventory, films, and stores

<img width="799" height="521" alt="Sakila - ERD" src="https://github.com/user-attachments/assets/4ffc3c2f-1090-4a1a-88f1-1b94ca97054d" />

# 🗂️ Data Model

### Key Tables Used

| Table | Business Meaning |
|------|------------------|
| Film | Product catalog |
| Inventory | Warehose |
| store | Distribution |
| category | Movie genres |
| Rental | Customer transactions |
| payment | Revenue transactions |
| customer | Customer profiles |

---

# 📈 SQL Business Analysis

## 1️⃣ Revenue by Genre

### Business Question
Which movie genres generate the highest revenue?

### Metric Used
`SUM(payment.amount) AS total_revenue`

### SQL Query

```sql
SELECT 
    c.name AS genre,
    SUM(p.amount) AS total_revenue
FROM payment p
JOIN rental r 
    ON p.rental_id = r.rental_id
JOIN inventory i 
    ON r.inventory_id = i.inventory_id
JOIN film f 
    ON i.film_id = f.film_id
JOIN film_category fc 
    ON f.film_id = fc.film_id
JOIN category c 
    ON fc.category_id = c.category_id
GROUP BY c.name
ORDER BY total_revenue DESC;
```

### Insight

Certain genres contribute a larger share of total revenue, indicating stronger customer demand patterns.

---

## 2️⃣ Customer Lifetime Value (CLV)

### Business Question
Which customers generate the highest lifetime revenue?

### Metric Used
`SUM(payment.amount) AS customer_lifetime_value`

### SQL Query

```sql
SELECT 
    c.customer_id,
    c.first_name,
    c.last_name,
    SUM(p.amount) AS customer_lifetime_value
FROM customer c
JOIN payment p
    ON c.customer_id = p.customer_id
GROUP BY c.customer_id, c.first_name, c.last_name
ORDER BY customer_lifetime_value DESC;
```

### Insight

A small group of customers contributes a significant portion of the overall revenue, highlighting the importance of customer retention strategies.

---

## 3️⃣ Rental Frequency Analysis

### Business Question
How frequently do customers rent movies?

### Metric Used
`COUNT(rental_id) AS total_rentals`

### SQL Query

```sql
SELECT 
    customer_id,
    COUNT(rental_id) AS total_rentals
FROM rental
GROUP BY customer_id
ORDER BY total_rentals DESC;
```

### Customer Segments

- Frequent renters  
- Occasional renters  
- One-time renters  

### Insight

Frequent renters represent the most engaged customers and contribute significantly to total rental activity.

---

## 4️⃣ Late Return Analysis

### Business Question
How long do customers keep rented movies before returning them?

### Metric Used
`DATEDIFF(return_date, rental_date) AS rental_duration`

### SQL Query

```sql
SELECT 
    rental_id,
    customer_id,
    DATEDIFF(return_date, rental_date) AS rental_duration
FROM rental;
```

### Insight

Longer rental durations reduce inventory availability and slow down product circulation.

---

## 5️⃣ Inventory Utilization

### Business Question
Which movies are rented the most?

### Metric Used
`COUNT(r.rental_id) AS rental_count`

### SQL Query

```sql
SELECT 
    f.title,
    COUNT(r.rental_id) AS rental_count
FROM film f
JOIN inventory i
    ON f.film_id = i.film_id
JOIN rental r
    ON i.inventory_id = r.inventory_id
GROUP BY f.title
ORDER BY rental_count DESC;
```

### Insight

A small number of films account for a large share of rentals, indicating concentrated product demand.

---

## 6️⃣ Revenue by Store

### Business Question
How does revenue differ between store locations?

### Metric Used
`SUM(payment.amount) AS total_revenue`

### SQL Query

```sql
SELECT 
    s.store_id,
    SUM(p.amount) AS total_revenue
FROM payment p
JOIN staff st
    ON p.staff_id = st.staff_id
JOIN store s
    ON st.store_id = s.store_id
GROUP BY s.store_id
ORDER BY total_revenue DESC;
```

### Insight

Revenue varies across store locations, suggesting differences in local customer demand.

---

# 📊 Dashboard Visualization

A **Power BI dashboard** was built to visualize key operational metrics including:

- Revenue by genre  
- Customer lifetime value distribution  
- Rental frequency trends  
- Inventory utilization  
- Store revenue comparison  

Dashboard images are available in the **Visuals** folder.

---

# 📁 Repository Structure

```
Movie-Rental-Inventory-Analytics-SQL
│
├── Data
│   └── rental_records.csv
│
├── SQL_Queries
│   ├── 01_revenue_by_genre.sql
│   ├── 02_customer_lifetime_value.sql
│   ├── 03_rental_frequency.sql
│   ├── 04_late_return_analysis.sql
│   ├── 05_inventory_utilization.sql
│   └── 06_revenue_by_store.sql
│
├── Visuals
│   ├── demand_by_genre_chart.png
│   └── rental_dashboard.png
│
├── ER_Diagram
│   └── Sakila_ERD.png
│
├── README.md
└── .gitignore
```

---

# 🛠️ Tools & Technologies

- **SQL (MySQL)**
- **Power BI**
- Relational Data Modeling
- Data Analysis

---

# 📚 Skills Demonstrated

- Advanced SQL joins across multiple tables  
- Business metric calculations using aggregations  
- Customer analytics and segmentation  
- Inventory utilization analysis  
- Revenue performance analysis  
- Data storytelling using analytical insights  

---

# 👤 About Me

**A. Sai Arvind**  
Data Analyst | SQL | Power BI | Business Analytics  

📧 Email: saiarvind5081@gmail.com  
🔗 LinkedIn: https://www.linkedin.com/in/saiarvindofficial/  
💻 GitHub: https://github.com/Sai-Arvind  

⭐ If you found this project useful, consider giving it a **star**.
What this fixes
