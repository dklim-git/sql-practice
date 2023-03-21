[TOC]

# Signup Activation Rate

New TikTok users sign up with their emails. They confirm their signup by replying to the text confirmation to activate their accounts.  Users may receive multiple text messages for account confirmation until they have confirmed their new account.

Write a query to find the activation rate of the users. Round the percentage to 2 decimal places.

`emails` table:

| Column Name |   Type   |
| :---------: | :------: |
|  email_id   | integer  |
|   user_id   | integer  |
| signup_date | datetime |

`texts` table:

|  Column Name  |  Type   |
| :-----------: | :-----: |
|    text_id    | integer |
|   email_id    | integer |
| signup_action | varchar |

---

### Solution Steps

Find `count(*)` where `signup_action` is confirmed:

```postgresql
select count(*) as num_confirmed
from texts
where signup_action = 'Confirmed'
```

Then, find the total number of rows in `texts`:

```postgresql
select count(*) as total_rows
from texts
```

Both queries are used as CTEs (final solution):

```postgresql
with confirmed as (
	select count(*) as num_confirmed
	from texts
	where signup_action = 'Confirmed'
),
total as (
	select count(*) as total_rows
	from texts
)

select round((num_confirmed*1.0/total_rows), 2)
from confirmed, total
```

The main query divides `num_confirmed` by `total_rows`, which gives the activation rate of users.  Both `num_confirmed` and `total_rows` are integers, dividing integers where the numerator < denominator will return an activation rate of 0.  `num_confirmed` is multiplied by `1.0` to cast the integer value to a floating-point number and the division operation will return a decimal value.

---

### Alternative Solution

```postgresql
select round(count(text_id)*1.0/(select count(email_id) from emails), 2)
from texts
where signup_action = 'Confirmed'
```

Found an alternative solution by Ame Wa (discussion section) where no CTEs are used.  `count(text_id)` represents the number of instances where `signup_action` is confirmed and the subquery in the select statement calculates all `email_id` in the emails table.

---

### Notes

The alternative solution is more compact than the solution I used (CTEs), but both are accepted answers.  I would prefer to use the solution with CTEs as the readability is easier, especially if the query needs to be maintained for any reason.

---

### Prompt Details

Prompt can be found on [DataLemur](https://datalemur.com/questions/signup-confirmation-rate), is medium rated, and found in TikTok interviews.

---

### Example Inputs

`emails` example input:

| email_id | user_id |     signup_date     |
| :------: | :-----: | :-----------------: |
|   125    |  7771   | 06/14/2022 00:00:00 |
|   236    |  6950   | 07/01/2022 00:00:00 |
|   433    |  1052   | 07/09/2022 00:00:00 |

`texts` example input:

| text_id | email_id | signup_action |
| :-----: | :------: | :-----------: |
|  6878   |   125    |   Confirmed   |
|  6920   |   236    | Not Confirmed |
|  6994   |   236    |   Confirmed   |

