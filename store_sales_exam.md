# Practical Exam: Grocery Store Sales

## Background

FoodYum is a grocery store chain based in the United States. It sells produce, meat, dairy, baked goods, snacks, and other household food staples.

As food costs rise, FoodYum wants to ensure it stocks products across all categories and at a range of prices, so it can serve a broad range of customers.

## Data

The data is available in the `products` table. The dataset contains product records for the last full year of the loyalty program.

| Column | Criteria |
|---|---|
| `product_id` | Nominal. Unique identifier for each product. Missing values are not possible. |
| `product_type` | Nominal. One of five categories: Produce, Meat, Dairy, Bakery, or Snacks. Missing values should be replaced with `Unknown`. |
| `brand` | Nominal. One of seven possible brands. Missing values should be replaced with `Unknown`. |
| `weight` | Continuous. Product weight in grams, rounded to two decimal places. Missing values should be replaced with the overall median weight. |
| `price` | Continuous. Product price in US dollars, rounded to two decimal places. Missing values should be replaced with the overall median price. |
| `average_units_sold` | Discrete. Average number of units sold monthly. Missing values should be replaced with `0`. |
| `year_added` | Nominal. Year product was first added to FoodYum stock. Missing values should be replaced with `2022`. |
| `stock_location` | Nominal. Origin warehouse location: A, B, C, or D. Missing values should be replaced with `Unknown`. |

---

## Task 1

In 2022, a product-system bug caused missing `year_added` values for some products added during that year. Because the year a product was added could affect price, identify how many records have a missing year.

The result must contain one column, `missing_year`, and one row.

```sql
SELECT COUNT(*) AS missing_year
FROM products
WHERE year_added IS NULL;
```


## Task 2

Clean the product data according to the defined criteria without updating the original `products` table.

```sql
SELECT
    product_id,
    COALESCE(product_type, 'Unknown') AS product_type,
    COALESCE(NULLIF(REPLACE(brand, '-', ''), ''), 'Unknown') AS brand,
    COALESCE(ROUND(CAST(REGEXP_REPLACE(weight, '[^\d.]', '', 'g') AS DECIMAL(10, 2)), 2), ROUND((SELECT PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY CAST(REGEXP_REPLACE(weight, '[^\d.]', '', 'g') AS DECIMAL(10, 2))) FROM products), 2)) AS weight,

COALESCE(
    TO_CHAR(CAST(price AS DECIMAL(10, 2)), '9999999999.99'),
    TO_CHAR(CAST((SELECT PERCENTILE_DISC(0.5) WITHIN GROUP (ORDER BY price) FROM products) AS DECIMAL(10, 2)), '9999999999.99')
) AS price,

    COALESCE(average_units_sold, 0) AS average_units_sold,
    COALESCE(year_added, 2022) AS year_added,
    COALESCE(UPPER(stock_location), 'Unknown') AS stock_location
FROM products;
```
-----

```sql
WITH cleaned_products AS (
    SELECT
        product_id,
        COALESCE(NULLIF(TRIM(product_type), ''), 'Unknown') AS product_type,
        CASE
            WHEN brand IS NULL OR TRIM(brand) IN ('', '-') THEN 'Unknown'
            ELSE TRIM(brand)
        END AS brand,
        NULLIF(
            TRIM(REPLACE(weight::TEXT, 'grams', '')),
            ''
        )::NUMERIC AS weight_clean,
        price::NUMERIC AS price_clean,
        COALESCE(average_units_sold, 0) AS average_units_sold,
        COALESCE(year_added, 2022) AS year_added,
        COALESCE(NULLIF(TRIM(stock_location), ''), 'Unknown') AS stock_location
    FROM products
),
median_values AS (
    SELECT
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY weight_clean) AS median_weight,
        PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price_clean) AS median_price
    FROM cleaned_products
)
SELECT
    product_id,
    product_type,
    brand,
    ROUND(COALESCE(weight_clean, median_weight)::NUMERIC, 2) AS weight,
    ROUND(COALESCE(price_clean, median_price)::NUMERIC, 2) AS price,
    average_units_sold,
    year_added,
    stock_location
FROM cleaned_products
CROSS JOIN median_values;
```

---

## Task 3

Find the minimum and maximum product price for each product type. Return `product_type`, `min_price`, and `max_price`.

```sql
SELECT
    product_type,
    MIN(price) AS min_price,
    MAX(price) AS max_price
FROM products
GROUP BY product_type;
```

---

## Task 4

Return `product_id`, `price`, and `average_units_sold` for Meat and Dairy products with an average monthly quantity sold greater than 10.

```sql
SELECT
    product_id,
    price,
    average_units_sold
FROM products
WHERE product_type IN ('Meat', 'Dairy') AND average_units_sold > 10;

```
