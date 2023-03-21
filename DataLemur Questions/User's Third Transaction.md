[TOC]

# User's Third Transaction

Assume you are given the table below on Uber transactions made by users. Write a query to obtain the third transaction of every user. Output the user id, spend and transaction date.

`transactions` table:

|   Column Name    |   Type    |
| :--------------: | :-------: |
|     user_id      |  integer  |
|      spend       |  decimal  |
| transaction_date | timestamp |

`transactions` example input:

| user_id | spend  |  transaction_date   |
| :-----: | :----: | :-----------------: |
|   111   | 100.50 | 01/08/2022 12:00:00 |
|   111   | 55.00  | 01/10/2022 12:00:00 |
|   121   | 36.00  | 01/18/2022 12:00:00 |
|   145   | 24.99  | 01/26/2022 12:00:00 |
|   111   | 89.60  | 02/05/2022 12:00:00 |

---

### Solution Steps

Find the third transaction of each user using `row_number()`:

```postgresql
select row_number() over (partition by user_id order by transaction_date), user_id, spend, transcation_date
from transactions
order by user_id, transaction_date
```

`row_number()` works similar to `rank()` - partitioning by `user_id`, each user is ordered by `transaction_date` and assigned a number.  This can be used as a CTE and filtered:

```postgresql
with third as (
	select row_number() over (partition by user_id order by transaction_date) as order_num, 	user_id, spend, transaction_date
  from transactions
  order by user_id, transaction_id
)

select user_id, spend, transaction_date
from third
where order_num = 3
```

The main query only returns instances where `order_num` is 3 (third transaction).

---

### Notes

My initial attempt used `rank()` instead of `row_number`:

```postgresql
with third_transaction as (
SELECT rank() over (partition by user_id order by transaction_date asc) as ranking, 
  user_id, spend, transaction_date FROM transactions
order by user_id, transaction_date
)

select user_id, spend, transaction_date
from third_transaction
where ranking = 3
```

I never used `row_number()` until now, this solution is accepted because it is impossible for a user to have orders with the same rank as time doesn't stop after their initial order.

---

### Prompt Details

Prompt can be found on [DataLemur](https://datalemur.com/questions/sql-third-transaction), is medium rated, and found in Uber interviews.

