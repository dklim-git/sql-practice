[TOC]

# Card Launch Success

Your team at JPMorgan Chase is soon launching a new credit card. You are asked to estimate how many cards you'll issue in the first month.

Before you can answer this question, you want to first get some perspective on how well new credit card launches typically do in their first month.

Write a query that outputs the name of the credit card, and how many cards were issued in its launch month. The launch month is the earliest record in the `monthly_cards_issued` table for a given card. Order the results starting from the biggest issued amount.

`monthly_cards_issued` table:

|  Column Name  |  Type   |
| :-----------: | :-----: |
|  issue_month  | integer |
|  issue_year   | integer |
|   card_name   | string  |
| issued_amount | integer |

`monthly_cards_issued` example input:

| issue_month | issue_year |       card_name        | issued_amount |
| :---------: | :--------: | :--------------------: | :-----------: |
|      1      |    2021    | Chase Sapphire Reserve |    170000     |
|      2      |    2021    | Chase Sapphire Reserve |    175000     |
|      3      |    2021    | Chase Sapphire Reserve |    180000     |
|      3      |    2021    |   Chase Freedom Flex   |     65000     |
|      4      |    2021    |   Chase Freedom Flex   |     70000     |

---

### Solution Steps

Prompt asks to estimate the amount of cards issued **during its launch month**:

```postgresql
select rank() over (partition by card_name order by issue_year, issue_month) as launch_rank, issue_month, issue_year, card_name, issued_amount
from monthly_cards_issued
group by issue_month, issue_year, card_name, issued_amount
```

> **`rank() over (partition by card_name order by issue_year, issue_month) as launch_rank`**

To pass the test case where a card has been issued over several years, `rank()` needs to be partitioned by the card's name and ordered by the issue year, issue month.  `order by` is in ascending order by default, so the minimum value is taken from the columns (launch year and month).   Order of the sort specification matters, `issue_year` needs to be the primary sort key or the final solution will result with an incorrect output.

Every cards' launch year and month will be ranked first, which simplifies the main query once the initial code block is used as a CTE (final solution):

```postgresql
with card_history as (
	select rank() over (partition by card_name order by issue_year, issue_month) as 	launch_rank, issue_month, issue_year, card_name, issued_amount
	from monthly_cards_issued
	group by issue_month, issue_year, card_name, issued_amount
)

select card_name, issued_amount
from card_history
where launch_rank = 1
order by issued_amount desc
```

The main query will return `card_name` and `issued_amount` given that it is ranked first (initial launch) and is ordered by `issued_amount` in descending order.

---

### Notes

In my initial attempt, I was able to return a table that sorted the issued year and month in ascending order:

```postgresql
select card_name, issued_amount, issue_month, issue_year
from monthly_cards_issued
group by card_name, issued_amount, issue_month, issue_year
order by card_name, issue_year, issue_month
```

Which outputted:

|       card_name        | issued_amount | issue_month | issue_year |
| :--------------------: | :-----------: | :---------: | :--------: |
|   Chase Freedom Flex   |     55000     |      1      |    2021    |
|   Chase Freedom Flex   |     60000     |      2      |    2021    |
|   Chase Freedom Flex   |     65000     |      3      |    2021    |
| Chase Sapphire Reserve |    150000     |     11      |    2020    |
| Chase Sapphire Reserve |    160000     |     12      |    2020    |
| Chase Sapphire Reserve |    170000     |      1      |    2021    |

I ran into difficultly trying to query the minimum year and month from here and forgotten `rank()` can be used to rank cards by the value of their issue month and year.

---

### Alternative Solution

```postgresql
select card_name, issued_amount
from (
  select rank() over (partition by card_name order by issue_year, issue_month) as ranking, card_name, issued_amount, issue_month, issue_year
  from monthly_cards_issued
  group by card_name,issued_amount, issue_month, issue_year
) as s1
where ranking = 1
order by issued_amount desc
```

Based on my initial attempt - instead of using a CTE, the query that contains `rank()` is nested inside of the from clause.  The main query filters `card_name` and `issued_amount` by `ranking`.

This is an accepted solution, but I prefer to use CTEs if possible for better readability and maintainability.

---

### Prompt Details

Prompt can be found on [DataLemur](https://datalemur.com/questions/card-launch-success), is medium rated, and found in JP Morgan Chase interviews.

