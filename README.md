# Customer Behavior Analysis Dashboard

End-to-end customer behavior analysis: data cleaning in **Jupyter Notebook (pandas)**, SQL analysis in **PostgreSQL**, and an interactive **Power BI** dashboard on 3,900 customers.

## Description

This project analyzes the shopping behavior of 3,900 customers of a retail business to understand what they buy, how much they spend, and which customer groups drive the most revenue.

The workflow covers the full data analysis cycle: cleaning raw data in a Jupyter Notebook with Python (pandas), loading it into PostgreSQL, answering business questions with SQL, and presenting the results in an interactive Power BI dashboard.

## Dashboard Preview

![Customer Behavior Dashboard] <img width="1371" height="732" alt="Screenshot 2026-11-23 162631" src="https://github.com/user-attachments/assets/3a62c30a-d92d-44b2-85ef-9f9c33924286" />



## Business Questions Answered

- What is the average purchase amount and average review rating?
- Which product categories generate the most revenue and sales?
- Which age groups spend the most?
- What share of customers have a subscription?
- Which customers are New, Returning or Loyal?
- Which items have the highest discount rate?
- Which shipping type has the higher average purchase amount?
- Which discount customers spend more than the average?

## Tools and Technologies

| Tool | Use |
|---|---|
| Jupyter Notebook | Interactive environment for data cleaning and exploration |
| Python (pandas) | Column cleaning, redundant-column check, age grouping with `qcut` |
| SQLAlchemy, psycopg2 | Loading the cleaned DataFrame into PostgreSQL |
| PostgreSQL | SQL analysis (CTE, CASE WHEN, GROUP BY, subqueries) |
| Power BI | Dashboard with KPI cards, charts and slicers |

## Data Cleaning Samples (Jupyter Notebook)

The raw dataset was cleaned in Jupyter Notebook using pandas:

- Standardized column names to lowercase snake_case:
  ```python
  df.columns = df.columns.str.lower().str.replace(' ', '_')
  ```
- Created an `age_group` column using quartile-based binning:
  ```python
  labels = ['Young Adult', 'Adult', 'Middle-aged', 'Senior']
  df['age_group'] = pd.qcut(df['age'], q=4, labels=labels)
  ```
- Checked whether `discount_applied` and `promo_code_used` were identical, then dropped the redundant column:
  ```python
  (df['discount_applied'] == df['promo_code_used']).all()
  df = df.drop('promo_code_used', axis=1)
  ```
- Loaded the cleaned data into PostgreSQL:
  ```python
  df.to_sql('customer', engine, if_exists='replace', index=False)
  ```

## SQL Analysis (PostgreSQL)

Sample queries

**Average shipping-type purchase amount**
```sql
SELECT shipping_type,
       ROUND(AVG(purchase_amount), 2) AS avg_purchase
FROM customer
WHERE shipping_type IN ('Standard', 'Express')
GROUP BY shipping_type;
```

**Top 5 items by discount rate**
```sql
SELECT item_purchased,
       ROUND(SUM(CASE WHEN discount_applied = 'Yes' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS discount_rate
FROM customer
GROUP BY item_purchased
ORDER BY discount_rate DESC
LIMIT 5;
```

**Customer segmentation (CTE)**
```sql
WITH customer_type AS (
    SELECT customer_id,
           previous_purchases,
           CASE
               WHEN previous_purchases = 1 THEN 'New'
               WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
               ELSE 'Loyal'
           END AS customer_segment
    FROM customer
)
SELECT customer_segment, COUNT(*) AS number_of_customers
FROM customer_type
GROUP BY customer_segment;
```

**Discount customers spending above average (subquery)**
```sql
SELECT customer_id, purchase_amount
FROM customer
WHERE discount_applied = 'Yes'
  AND purchase_amount >= (SELECT AVG(purchase_amount) FROM customer);
```

## Key Insights

- Average purchase amount is **₹59.76** and average review rating is **3.75**.
- **73%** of customers do not have a subscription, which shows room to grow the subscription base.
- **Clothing** is the top category in both revenue and number of sales.
- **Young Adults** generate the highest revenue among age groups.

## Dashboard Features

- KPI cards: Average Purchase Amount, Average Review Rating, Number of Customers
- Charts: Subscription Status (donut), Revenue and Sales by Category, Revenue and Sales by Age Group
- Slicers: Subscription Status, Gender, Category, Shipping Type

## Project Structure

```
customer-behavior-analysis_dashboard/
├── data/
│   └── customer_shopping_behavior.csv
├── notebook/
│   └── customer_behavior_analysis.ipynb
├── sql/
│   └── product_db_sql_queries.sql
├── dashboard/
│   ├── customer_behavior_dashboard.pbix
│   └── dashboard.png
└── README.md
```

## How to Run

1. Install the libraries: `pip install pandas sqlalchemy psycopg2-binary`
2. Open `notebook/customer_behavior_analysis.ipynb` and run all cells to clean the data.
3. Create a PostgreSQL database and set your credentials as environment variables (do not hardcode the password):
   ```python
   import os
   password = os.environ['DB_PASSWORD']
   ```
4. Load the cleaned data into PostgreSQL using `df.to_sql()`.
5. Run the queries from `sql/product_db_sql_queries.sql`.
6. Open `dashboard/customer_behavior_dashboard.pbix` in Power BI Desktop.

## Author

[Avinash Chaudhary]
