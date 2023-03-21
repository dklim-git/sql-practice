[TOC]

# Sending vs. Opening Snaps

Assume you are given the tables below containing information on Snapchat users, their ages, and their time spent sending and opening snaps.  Write a query to obtain a breakdown of the time spent sending vs. opening snaps (as a percentage of **total time spent** on these activities) for each age group.

Output the age bucket and percentage of sending and opening snaps. Round the percentage to 2 decimal places.

Percentages are calculated as followed:
$$
\begin{align*}
\text{Total Time Sent} &= \frac{\text{Time Sending}}{\text{Time Sending + Time Opening}} \\[.5cm]
\text{Total Time Opened} &= \frac{\text{Time Opening}}{\text{Time Sending + Time Opening}} \\
\end{align*}
$$
`activities` table:

|  Column Name  |              Type               |
| :-----------: | :-----------------------------: |
|  activity_id  |             integer             |
|    user_id    |             integer             |
| activity_type | string ('send', 'open', 'chat') |
|  time_spent   |              float              |
| activity_date |            datetime             |

`age_breakdown` table:

| Column Name |                Type                |
| :---------: | :--------------------------------: |
|   user_id   |              integer               |
| age_bucket  | string ('21-25', '26-30', '31-25') |

---

### Solution Steps

Calculate the total `time_spent` using `case when`:

```postgresql
select 
	b.age_bucket, 
	sum(case when a.activity_type = 'send' then a.time_spent else 0 end) as sent,
	sum(case when a.activity_type = 'open' then a.time_spent else 0 end) as opened,
	sum(time_spent) as total_time
from activities a
join age_breakdown b on a.user_id = b.user_id
where activity_type in ('open', 'send')
group by b.age_bucket
```

`activity_type` is filtered by `in ('open', 'send')` since the column contains three distinct string values (open, send, and chat) - without the filter, it would count rows where `activity_type = 'chat'` when using the sum function to calculate the total `time_spent`.

Query above would be used as a CTE:

```postgresql
with snap_stats as (
  select 
    b.age_bucket, 
    sum(case when a.activity_type = 'send' then a.time_spent else 0 end) as sent,
    sum(case when a.activity_type = 'open' then a.time_spent else 0 end) as opened,
    sum(time_spent) as total_time
  from activities a
  join age_breakdown b on a.user_id = b.user_id
  where activity_type in ('open', 'send')
  group by b.age_bucket
)

select age_bucket, round(sent/total_time*100, 2) as sent_per, round(opened/total_time*100, 2)
from snap_stats
```

Aliases `sent` and `opened` are used to calculate the decimal percentage of total time spent sending/opening snaps.  Both aliases are multiplied by 100 since the prompt asks to return the percentage.

---

### Notes

My initial attempts were incorrect because I didn't read the `activities` table carefully, I assumed there were only two distinct string values in `activity_type` (open, send, and **chat**).  Also, I had forgotten that `case when` can be used in a sum function to calculate all instances of `time_spent`.

---

### Prompt Details

Prompt can be found on [DataLemur](https://datalemur.com/questions/time-spent-snaps), is medium rated, and found in Snapchat interviews.

---

### Example Inputs

`activities` example input:

| activity_id | user_id | activity_type | time_spent |    activity_date    |
| :---------: | :-----: | :-----------: | :--------: | :-----------------: |
|    7274     |   123   |     open      |    4.50    | 06/22/2022 12:00:00 |
|    2425     |   123   |     send      |    3.50    | 06/22/2022 12:00:00 |
|    1413     |   456   |     send      |    5.67    | 06/23/2022 12:00:00 |
|    1414     |   789   |     chat      |   11.00    | 06/25/2022 12:00:00 |
|    2536     |   456   |     open      |    3.00    | 06/25/2022 12:00:00 |

`age_breakdown` example input:

| user_id | age_bucket |
| :-----: | :--------: |
|   123   |   31-35    |
|   456   |   26-30    |
|   789   |   21-25    |

