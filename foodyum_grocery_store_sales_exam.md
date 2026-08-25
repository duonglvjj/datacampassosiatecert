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

### Result

| missing_year |
|---:|
| 170 |

---

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

### Sample output

| product_id | product_type | brand | weight | price | average_units_sold | year_added | stock_location |
|---:|---|---|---:|---:|---:|---:|---|
| 1 | Bakery | TopBrand | 602.61 | 11.00 | 15 | 2022 | C |
| 2 | Produce | SilverLake | 478.26 | 8.08 | 22 | 2022 | C |
| 3 | Produce | TastyTreat | 532.38 | 6.16 | 21 | 2018 | B |
| 4 | Bakery | StandardYums | 453.43 | 7.26 | 21 | 2021 | D |
| 5 | Produce | GoldTree | 588.63 | 7.88 | 21 | 2020 | A |
| ... | ... | ... | ... | ... | ... | ... | ... |
| 1696 | Meat | TastyTreat | 503.99 | 14.08 | 25 | 2017 | A |
| 1697 | Meat | GoldTree | 526.89 | 16.13 | 25 | 2016 | D |
| 1698 | Bakery | YumMie | 583.85 | 7.05 | 16 | 2021 | A |
| 1699 | Produce | TopBrand | 441.64 | 8.10 | 19 | 2019 | A |
| 1700 | Meat | TopBrand | 518.60 | 15.89 | 24 | 2021 | A |

**Output:** 1,700 rows × 8 columns.

---

## Task 3

Find the minimum and maximum product price for each product type. Return `product_type`, `min_price`, and `max_price`.

```sql
SELECT product_type,
   MIN(price) AS min_price,
   MAX(price) AS max_price
FROM products 
GROUP BY product_type;
```

### Result

| product_type | min_price | max_price |
|---|---:|---:|
| Snacks | 5.20 | 10.72 |
| Produce | 3.46 | 8.78 |
| Dairy | 8.33 | 13.97 |
| Bakery | 6.26 | 11.88 |
| Meat | 11.48 | 16.98 |

---

## Task 4

Return `product_id`, `price`, and `average_units_sold` for Meat and Dairy products with an average monthly quantity sold greater than 10.

```sql
SELECT product_id, price, average_units_sold
FROM products 
WHERE product_type IN ('Meat', 'Dairy')
  AND average_units_sold > 10;
```

### Sample output

| product_id | price | average_units_sold |
|---:|---:|---:|
| 6 | 16.20 | 24 |
| 18 | 15.77 | 28 |
| 29 | 11.57 | 30 |
| 310 | 13.94 | 27 |
| 411 | 9.26 | 26 |
| ... | ... | ... |
| 1694 | 16.00 | 25 |
| 1695 | 12.88 | 20 |
| 1696 | 14.08 | 25 |
| 1697 | 16.13 | 25 |
| 1700 | 15.89 | 24 |

**Output:** 698 rows × 3 columns.
