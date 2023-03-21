[TOC]

# Odd and Even Measurements

Assume you are given the table containing measurement values obtained from a Google sensor over several days. Measurements are taken several  times within a given day.

Write a query to obtain the sum of the odd-numbered and even-numbered measurements on a particular day, in two different columns.

Definition:

- 1st, 3rd, and 5th measurements taken **within a day** are considered odd-numbered measurements and the 2nd, 4th, and 6th measurements are even-numbered measurements.

`measurements` table:

|    Column Name    |   Type   |
| :---------------: | :------: |
|  measurement_id   | integer  |
| measurement_value | decimal  |
| measurement_time  | datetime |

`measurements` example input:

| measurement_id | measurement_value |  measurement_time   |
| :------------: | :---------------: | :-----------------: |
|     131233     |      1109.51      | 07/10/2022 09:00:00 |
|     135211     |      1662.74      | 07/10/2022 11:00:00 |
|     523542     |      1246.24      | 07/10/2022 13:15:00 |
|     143562     |      1124.50      | 07/11/2022 15:00:00 |
|     346462     |      1234.14      | 07/11/2022 16:45:00 |

---

### Solution Steps

`row_number` can be used to number each instance of `measurement_time`:

```postgresql
select
	row_number() over (partition by date(measurement_time) order by measurement_time)
		as ordering,
	measurement_value,
	measurement_time
from measurements
```

This will partition rows by the day of `measurement_time`, see table below for example output:

| ordering | measurement_value |  measurement_time   |
| :------: | :---------------: | :-----------------: |
|    1     |      1109.51      | 07/10/2022 09:00:00 |
|    2     |      1662.74      | 07/10/2022 11:00:00 |
|    3     |      1246.24      | 07/10/2022 14:30:00 |
|    1     |      1124.50      | 07/11/2022 13:15:00 |
|    2     |      1234.14      | 07/11/2022 15:00:00 |
|    3     |      1252.62      | 07/11/2022 16:45:00 |
|    4     |      1246.56      | 07/11/2022 18:00:00 |

The initial query can be used as a CTE and `sum (case when)` to calculate even and odd numbered measurements (final solution):

```postgresql
with numbered as (
  select 
    row_number() over (partition by date(measurement_time) order by measurement_time)
      as ordering,
    measurement_value,
    measurement_time
  from measurements
)

select
  date(measurement_time),
  sum(case when mod(ordering, 2) = 1 then measurement_value else 0 end) as odd_sum,
  sum(case when mod(ordering, 2) = 0 then measurement_value else 0 end) as even_sum
from numbered
group by date(measurement_time)
order by date(measurement_time)
```

`mod(ordering, 2)` returns `measurement_value` that meets the condition and `date(measurement_time)` is used to only return the day (not time) in the output.

---

### Notes

My initial attempt used `dense_rank()` instead of `row_number()` and was accepted, but the latter function is better for the prompt as it asks to number the measurements of a given day.  Also, my initial query (inside CTE) had an unnecessary `group by`, it isn't needed because there are no aggregation functions in the select statement.

---

### Prompt Details

Prompt can be found on [DataLemur](https://datalemur.com/questions/odd-even-measurements), is medium rated, and found in Google interviews.



