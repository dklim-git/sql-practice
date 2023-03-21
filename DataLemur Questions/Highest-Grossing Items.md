[TOC]

# Highest-Grossing Items

Assume you are given the table containing information on Amazon customers and their spending on products in various categories.

Identify the top two highest-grossing products within each category in 2022. Output the category, product, and total spend.

`product_spend` table:

|   Column Name    |   Type    |
| :--------------: | :-------: |
|     category     |  string   |
|     product      |  string   |
|     user_id      |  integer  |
|      spend       |  decimal  |
| transaction_date | timestamp |

`product_spend` example input:

|  category   |     product      | user_id | spend  |  transaction_date   |
| :---------: | :--------------: | :-----: | :----: | :-----------------: |
|  appliance  |   refrigerator   |   165   | 246.00 | 12/26/2021 12:00:00 |
|  appliance  |   refrigerator   |   123   | 299.99 | 03/02/2022 12:00:00 |
|  appliance  | washing machine  |   123   | 219.80 | 03/02/2022 12:00:00 |
| electronics |      vacuum      |   178   | 152.00 | 04/05/2022 12:00:00 |
| electronics | wireless headset |   156   | 249.90 | 07/08/2022 12:00:00 |
| electronics |      vacuum      |   145   | 189.00 | 07/15/2022 12:00:00 |

---

### Solution Steps

Prompt specifies to identify products sold in **2022** and their total gross:

```postgresql
select category, product, sum(spend) as total_spend
from product_spend
where extract(year from transaction_date) = 2022
group by category, product
```

For all products sold in 2022, each product and its category is listed along with the total gross.  PostgreSQL 14 filters timestamp data types differently, see notes for more info.

To determine the highest gross product sold, `rank()` is used:

```postgresql
  select *, rank() over (partition by category order by total_spend desc) as ranking 
  from total_sales
```

The first two queries are used as CTEs to filter the **top two** grossing products in each category (final solution):

```postgresql
with total_sales as (
  select category, product, sum(spend) as total_spend
  from product_spend
  where extract(year from transaction_date) = 2022
  group by category, product
),

top_spend as (
  select *, rank() over (partition by category order by total_spend desc) as ranking 
  from total_sales
)

select category, product, total_spend
from top_spend
where ranking between 1 and 2
order by category, ranking
```

---

### Notes

I tried filtering by year and was met with an error.  This syntax should work in MySQL, but DataLemur uses PostgreSQL for their questions:

```mysql
select category, product, sum(spend) as total_spend
from product_spend
where year(transaction_date) = 2022
group by category, product
```

In my initial attempt, I had `rank()` and `sum(spend)` in the same CTE which resulted in errors.  I used an alias for `sum(spend)` and used it inside of `rank()`, this wouldn't work because aliases are calculated after the `select` projection.  I was able to shorten the syntax of the solution I found online, see alternative solution.

---

### Alternative Solution

```postgresql
with total_sales as (
select rank() over (partition by category order by sum(spend) desc) as ranking, category,
	product, sum(spend) as total_spend
from product_spend
where extract(year from transaction_date) = 2022
group by category, product
)

select category, product, total_spend
from total_sales
where ranking between 1 and 2
```

`total_sales` holds the query that ranks categories by the total grosses of each product sold in 2022 and the main query filters `ranking` between values 1-2.  Syntax is cleaner because less CTEs are used.

---

### Prompt Details

Prompt can be found on [DataLemur](https://datalemur.com/questions/sql-highest-grossing), is medium rated, and found in Amazon interviews.