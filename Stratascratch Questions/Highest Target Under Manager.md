[TOC]

# Highest Target Under Manager

Find the highest target achieved by the employee or employees who works under the manager id 13. Output the first name of the employee and target achieved. The solution should show the highest target achieved under manager_id=13 and which employee(s) achieved it.

`salesforce_employees` table:

![image-20230310004632966](/Users/darren/Library/Application Support/typora-user-images/image-20230310004632966.png)

---

### Solution Steps

First, find all employees where `manager_id` = 13 and rank each employee by their `target` value:

```mysql
select dense_rank() over (order by target desc) as ranking, first_name, target
from salesforce_employees
where manager_id = 13
```

Then, use as a subquery and filter where `ranking` = 1:

```mysql
select first_name, target
from (
	select dense_rank() over (order by target desc) as ranking, first_name, target
	from salesforce_employees
	where manager_id = 13
) as s1
where ranking = 1
```

---

### Alternative Solution

```mysql
with top_emp as (
	select dense_rank() over (order by target desc) as ranking, first_name, target
  from salesforce_employees
  where manager_id = 13
)

select first_name, target
from top_emp
where ranking = 1
```

Instead of a subquery, a CTE can be used to achieve the same result.

---

### Notes

Both `rank()` and `dense_rank()` will work with the queries, but in the event where you need to filter more employees - its best to use `dense_rank()` as the ranking will be continuous.

---

### Prompt Details

Prompt can be found on [Stratascratch](https://platform.stratascratch.com/coding/9905-highest-target-under-manager?code_type=3), is medium rated, and found in Salesforce interviews.

Stratascratch does not offer an easier way to copy/paste table information, visit link to see full prompt details.